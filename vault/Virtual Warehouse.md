# Virtual Warehouses en Snowflake

> _Notas para certificación SnowPro Core_
---
## 1. Tipos de Virtual Warehouse
### 1.1 Standard
Usa **resource constraints** de generación **Gen1** o **Gen2**.
- **Gen2** ofrece mayor rendimiento y soporta operaciones DML pesadas: `DELETE`, `MERGE`, `UPDATE` y **large scans**. No disponible en todos los tamaños.
### 1.2 Snowpark-Optimized
Diseñado específicamente para cargas de trabajo con **UDFs** (Python, Java, Scala) y procesamiento intensivo en memoria.
- Proporciona **16x más RAM por slot** que un warehouse estándar.
- **Crédito/hora más caro** que Standard.
- **Inicio más lento** — no conviene para cargas mixtas o queries SQL simples.
- Úsalo **solo si** tu workload incluye funciones Snowpark; de lo contrario no justifica el costo.
---
## 2. Tamaños y Escalado Vertical (Scale Up / Scale Down)

Los warehouses escalan desde `X-Small` hasta `6X-Large`. Cada incremento de tamaño **duplica** los nodos (y con ello CPU, RAM y créditos/hora).

| Tamaño   | Nodos | Créditos/hora |
| -------- | ----- | ------------- |
| X-Small  | 1     | 1             |
| Small    | 2     | 2             |
| Medium   | 4     | 4             |
| ...      | ...   | ...           |
| 6X-Large | 128   | 128           |
### Cambio de tamaño en caliente (sin downtime)
El resize **no requiere suspender** el warehouse. Sin embargo, durante la transición existe un **costo dual temporal**:
- Las queries en ejecución continúan con el tamaño anterior.
- Las queries nuevas (en cola) usan el tamaño nuevo.
> **Analogía**: Similar a los _slots_ de BigQuery o los tipos de nodos de Redshift, pero con granularidad de tamaño fijo.

### Impacto en caché al reducir tamaño
Al hacer **Scale Down**, la `warehouse cache` se reduce, lo que puede ralentizar queries que antes aprovechaban datos en memoria. Evalúa el trade-off costo/rendimiento antes de reducir.

---
## 3. Multi-Cluster Warehouses y Scaling Policies

Los **multi-cluster** resuelven problemas de **concurrencia** (no de velocidad de query individual). Se configuran con `MIN_CLUSTER_COUNT` y `MAX_CLUSTER_COUNT`.
### 3.1 Modos de escalado

| Modo           | Comportamiento                                                                                   |
| -------------- | ------------------------------------------------------------------------------------------------ |
| **Maximize**   | Levanta todos los clusters desde el inicio. Ideal para cargas predecibles con picos programados. |
| **Auto-scale** | Escala dinámicamente según demanda, respetando min/max.                                          |
### 3.2 Políticas de escalado (Auto-scale)

| Política     | Cuándo agrega cluster                            | Cuándo quita cluster                                |
| ------------ | ------------------------------------------------ | --------------------------------------------------- |
| **Standard** | Al detectar la primera query encolada            | Cuando el cluster queda sin queries                 |
| **Economy**  | Al estimar que habrá cola en los próximos ~6 min | Al estimar que no habrá cola en los próximos ~6 min |

> Prioriza **Standard** si necesitas responsividad; **Economy** si priorizas ahorro.
### 3.3 Billing en multi-cluster

```
Costo = Tamaño del warehouse × Número de clusters activos
```

Un resize afecta **todos los clusters simultáneamente**.

---
## 4. Query Acceleration Service (QAS)
Servicio habilitado a **nivel de warehouse** para acelerar queries con scans muy grandes o comportamiento _outlier_.
### Queries elegibles (Snowflake decide automáticamente)
- Muchas particiones escaneadas.
- `SELECT`, `INSERT`, `COPY INTO`, `CREATE TABLE AS SELECT`.
### Scale Factor
- Default: **8**
- Ajústalo según análisis de tus queries. En multi-cluster, se recomienda **aumentarlo** para que el servicio esté disponible para todos los clusters.
### Herramientas de diagnóstico
sql

```sql
-- Ver si una query específica es elegible
SELECT SYSTEM$ESTIMATE_QUERY_ACCELERATION('<query_id>');

-- Explorar queries elegibles y variación por scale factor
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_ACCELERATION_ELIGIBLE;
```

**Estados posibles de una query:**

|Estado|Significado|
|---|---|
|`accelerated`|Fue acelerada por QAS|
|`eligible`|Podría acelerarse, pero no fue aplicada|
|`not_eligible`|No cumple los requisitos|

---
## 5. Auto Suspend y Auto Resume

| Parámetro      | Tipo               | Descripción                                                         |
| -------------- | ------------------ | ------------------------------------------------------------------- |
| `AUTO_SUSPEND` | Integer (segundos) | Tiempo de inactividad antes de suspender el warehouse               |
| `AUTO_RESUME`  | Boolean            | Si `TRUE`, reactiva el warehouse automáticamente al recibir queries |
### Impacto al suspender
- Se **pierde la `warehouse cache`** (datos en memoria local de los nodos).
- Al reanudar, las primeras queries serán más lentas mientras se reconstruye el caché desde el **Result Cache** o **storage remoto**.
### Regla práctica para `AUTO_SUSPEND`
Si tus queries se ejecutan cada ~2 minutos, configura `AUTO_SUSPEND` en **4-6 minutos**. Esto evita el costo de reinicio frecuente, ya que Snowflake cobra un **mínimo de 60 segundos** por cada inicio de warehouse.
Usa la tabla warehouse_load_history para conocer los tiempos y cuanto podrías acomodar.

---
## 6. Query Load y Toma de Decisiones

El **Query Load** es un ratio calculado como:

```
Query Load = Σ(tiempo de ejecución de queries activas) / ventana de tiempo (≈300s)
```
### Estados de queries considerados

| Estado                  | Descripción                        |
| ----------------------- | ---------------------------------- |
| `running`               | En ejecución activa                |
| `queued`                | Esperando slot disponible          |
| `queued (provisioning)` | Esperando recursos al hacer resume |
| `blocked`               | Bloqueada por locks                |
### Árbol de decisión
El objetivo es que el valor del factor query load sea lo más cercano a 1, eso significa que el tiempo que esta activo el warehouse es el mismo tiempo que se ejecutan las queries.

```
Alto tiempo en "running" + bajo tiempo en "queued"
  → SCALE UP (queries complejas, necesitan más recursos)

Alto tiempo en "queued"
  → SCALE OUT (concurrencia alta, agregar clusters)

Alto tiempo en "queued (provisioning)"
  → Aumentar AUTO_SUSPEND (el warehouse se suspende y reactiva con demasiada frecuencia)

Bajo uso sostenido
  → SCALE DOWN o SCALE IN
```

---
## 7. Tipos de Caché en Snowflake

|Tipo|Persistencia|Se pierde cuando...|
|---|---|---|
|**Result Cache**|24 horas|Los datos subyacentes cambian|
|**Metadata Cache**|Persistente (nivel de servicio)|Casi nunca|
|**Warehouse Cache**|Mientras el warehouse está activo|El warehouse se suspende|

---
## 8. Casos de Uso y Recomendaciones

### 8.1 Queries complejas e impredecibles
- Habilita `AUTO_SUSPEND` y `AUTO_RESUME`.
- Usa **QAS** para queries con scans extensos o tiempo de ejecución variable.
- Comienza con tamaño **Small/Medium/Large** y escala según evidencia.
### 8.2 Data Loading
- Aumentar el tamaño del warehouse **no siempre mejora** el throughput de carga.
- El rendimiento depende más de la **cantidad y tamaño de los archivos**.
- Para cargas normales: `Small` o `Medium` es suficiente. Para cientos de miles de archivos: considera `Large` o superior.
- **Usa warehouses dedicados** por tipo de workload (ej: uno para ETL, otro para analítica, otro para reporting).
### 8.3 Concurrencia alta con picos predecibles
- Multi-cluster en modo **Maximize** con política **Standard**.
### 8.4 Cargas variables con picos impredecibles
- Multi-cluster en modo **Auto-scale** con política **Economy** (o **Standard** si la latencia importa).

---
## 📌 Resumen Ejecutivo (5 puntos clave)
1. **Tipos**: Standard (Gen1/Gen2) para cargas SQL generales; Snowpark-Optimized solo para UDFs con alta demanda de memoria (16x RAM, mayor costo).
2. **Scale Up vs Scale Out**: Scale Up resuelve queries lentas (más recursos por query); Scale Out resuelve concurrencia (más queries en paralelo). Son estrategias complementarias, no intercambiables.
3. **Caché es crítico**: El `warehouse cache` desaparece al suspender. Calibra `AUTO_SUSPEND` para balancear ahorro de créditos vs. penalidad de "cold start".
4. **QAS acelera outliers**: No acelera todas las queries, solo las de scan extenso. Usa `SYSTEM$ESTIMATE_QUERY_ACCELERATION` para validar elegibilidad y ajustar el `scale_factor`.
5. **Billing multi-cluster**: `clusters activos × tamaño × tiempo`. Un resize aplica a todos los clusters activos simultáneamente.

---

## 📚 Qué estudiar a continuación
1. **Resource Monitors** — Control de presupuesto de créditos por warehouse o cuenta. Muy probable en el examen.
2. **Caching layers en detalle** — Cómo interactúan Result Cache, Metadata Cache y Warehouse Cache con micro-partitions.
3. **Query Profile** — Herramienta visual para diagnosticar bottlenecks (complementa el análisis de Query Load).
4. **Task & Serverless Compute** — Alternativa a warehouses para pipelines ligeros; importante para comparar costos.
5. **Micro-partitions y Clustering Keys** — Entienden por qué QAS ayuda en large scans y cómo reducir el dato escaneado desde origen.