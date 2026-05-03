# Micropartions

La unidad fundamental de almacenamiento en Snowflake. Toda tabla se divide automáticamente en microparticiones de **50–500 MB descomprimidos** (comprimidas a nivel columnar).
Son inmutables, generando un historial de versiones por cada operación DML, estas microparticiones "antiguas" se encuentra inactivas y tienen un tiempo de vida definido por el time travel y fail - safe.

### Optimización

**Partition pruning** — cada **micropartición** tiene metadatos precalculados por columna (mín, máx, distintos, nulos). Cuando aplicas un filtro, el optimizer descarta las particiones que no pueden contener los valores buscados, **sin consumir créditos de compute**.

**Column scanning** — al ser columnar, una query solo lee las columnas que necesita, no la fila completa.

### Data Recuperation

- **Time Travel**: Configurable de 1 d a 90 días (Enterprise). Usuario puede recuperar los datos.
- **Fail-safe**: Solo accesible por Snowflake, por un máimo de 7 días.
### Metadata Only Operations

Existen operaciones que solo afectan la metadata de la microparrtions como el drop una columna. 
El optimizer lee el metastore e  ignora las columnas. Los bytes seguiran existiendo según el tiempo de vida del micropartitions.

### Clustering

El rendimiento del pruning depende del orden de los datos en la micropartición. Hay tres formas de controlarlo:

- **Carga ordenada** — cada carga individual contiene data de 1 fecha o la carga completa se encuentra ordenada por order by.
- **Clustering Keys** — subconjunto de columnas que ordenan la tabla físicamente, generando costos background.

---
#### Clustering Depth

Métrica que indica el **solapamiento promedio** entre microparticiones para una columna dada.

- **Depth bajo (cercano a 1)** → # particiones escaneadas cercano # particiones resultantes
- **Depth alto** → # particiones escaneadas >>>> # particiones resultantes

> A mayor valor, peor rendimiento(prunning). Posible necesidad de re-clustering.

Puedes monitorear esto con vistas del esquema `SNOWFLAKE.ACCOUNT_USAGE.CLUSTERING_INFORMATION`.

#### Clustering Keys

**Definición:** Conjunto de columnas a nivel de tabla (máximo 4) para controlar cómo Snowflake organiza datos en **micro-particiones**, maximizando el **partition pruning** en queries.

> Primero valida el clustering deep y luego aplicalo, recomendable tablas **> 1 TB**

##### Ordenamiento de Columnas

Ordenar de **menor a mayor cardinalidad** → maximiza pruning desde el primer filtro.

- Columnas frecuentes en `WHERE` y `JOIN`
- Columnas `TIMESTAMP` → truncar a `DATE` para reducir cardinalidad
- Soporte `VARIANT` con transformación explícita:

```
ALTER TABLE mi_tabla CLUSTER BY (v:campo::STRING);
```

---
### Clustering

Consume compute serveless, + costos D.R a # operaciones DML
Operación de clustering genera new micro-partitions, aumenta # partitions inactivos ( + storage) (+ costo)

> Alternativa: (Re)Ingesta controlada, campos ordenados

---
### Administración

```
-- Suspender
ALTER TABLE nombre_tabla SUSPEND RECLUSTER;
-- Reanudar
ALTER TABLE nombre_tabla RESUME RECLUSTER;
```

> Cualquier `ALTER TABLE` con clustering suspendido lo **reanuda automáticamente**.
#### Diagnóstico

| Función                           | Uso                                         |
| --------------------------------- | ------------------------------------------- |
| `SYSTEM$CLUSTERING_INFORMATION`   | Estado actual: clustering deep, particiones |
| `SYSTEM$ESTIMATE_CLUSTERING_KEYS` | Sugiere columnas candidatas                 |
# Tables

### Comparativa

| Característica | Permanent   | Transient   | Temporary   | External |
| -------------- | ----------- | ----------- | ----------- | -------- |
| Time Travel    | 90 días*    | 1 día       | No          |          |
| Fail-Safe      | 7 días      | No          | No          |          |
| Alcance        | Persistente | Persistente | Solo sesión |          |

> *90 días solo en edición Enterprise o superior.  El Time Travel te va a brindar mayor protección de los datos ya que va a ser recuperable.

Las tablas temporary existen solo durante la sesión activa, se elimina al cerrarla. Útil para almacenamiento intermedio en pipelines puntuales.
#### ¿Permanent vs Transient?

- **Transient:** fuente fija y reutilizable. En caso de pérdida, se reingesta desde la fuente.
- **Permanent:** fuente no siempre disponible (CDC, Kafka, Kinesis). El Time Travel es el único respaldo.

> Un `CREATE TRANSIENT SCHEMA` o `CREATE TRANSIENT DATABASE` hace que todos sus objetos internos hereden el tipo Transient automáticamente.

---
### External Tables

Read-only, datos físicos en object storage (S3, GCS, Azure Blob). Snowflake solo guarda **metadata**.

**Costos:**
- Storage lo paga el cliente al cloud provider
- Compute se cobra al ejecutar queries
- Sin warehouse cache → queries más lentas y más créditos consumidos
- `AUTO_REFRESH` genera costo serverless adicional

**Estructura:**
- Columna nativa `VALUE` de tipo `VARIANT`
- Particionamiento manual sobre el path del archivo es crítico para performance

**Sincronización de metadata:**
- Manual: `ALTER EXTERNAL TABLE <name> REFRESH;`
- Automática: `AUTO_REFRESH = TRUE` + event notifications del cloud provider (S3 Events, GCS Pub/Sub, Azure Event Grid)

> ⚠️ Sin partition pruning, cualquier archivo nuevo provoca re-escaneo de toda la tabla.

**Patrón recomendado:**

```
External Table (AUTO_REFRESH=TRUE) → Materialized View → Query
```

La MV recalcula en background; las queries leen datos pre-calculados sin tocar el storage.

**Cuándo usarlas:**
- Data lake sin movimiento de datos (lectura directa)
- Lecturas Near Real-Time → External Table + MV
- Lecturas Real-Time → mejor usar **Snowpipe** o **Snowpipe Streaming**

---
### Hybrid Tables (Unistore)

Workloads **HTAP**: combinan capacidad transaccional (OLTP) y analítica (OLAP) en un mismo objeto.

**Arquitectura:** mantiene dos estructuras simultáneas
- **Row store** → lookups por PK
	- Indice secundario, copia fisica de la fila, genera más costos
- **Columnar Store (micro-particiones)**

**Características:**
- Primary Key obligatoria
- Row-level locking: Bloquea fila para asegurar DML
- ❌  Time Travel,  ❌ Fail-Safe

> Una operacion DML representa doble operacion (columnar y row store) , generando + costos
> Solo usar con fines transaccionales

**Diferencial vs otras nubes:** GCP requiere Spanner + BigQuery; AWS requiere Aurora + Redshift. Snowflake lo unifica en un solo objeto.

---
# Search Optimization Service (SOS)

**Problema:** el pruning estándar usa rangos min/max. En columnas de **alta cardinalidad** (`email`, `transaction_id`) los rangos se solapan → se escanean casi todas las micro-particiones.

 **Solución:** construye un **Search Access Path (SAP)**, índice de punteros `valor → ubicación en µ-partición` que va directamente a las micro-particiones con valores específicos. No duplica datos, solo guarda referencias.
#### Activación
Solo en predicados de **igualdad o pertenencia**:
- `WHERE col = 'valor'`
- `WHERE col IN ('a','b','c')`
- Substring search, geo lookups
- ❌ No optimiza rangos (`>`, `<`, `BETWEEN`)
#### Configuración y Costos
- Habilitar a nivel de **tabla completa** (Snowflake decide columnas) o **columna específica**
- **Storage:** índices SAP ocupan espacio adicional
- **Compute serverless:** mantenimiento automático ante DML
#### ¿Cuándo usarlo?
- Query Profile indica que la tabla tiene un escaneo alto de micro particiones y se realizan filtros de igualdad por una columna con alta cardinalidad.
- NO USAR en tablas con DML frecuente → costo de mantenimiento puede superar el beneficio
#### SOS vs Clustering Keys

| Característica     | Clustering Keys            | SOS                           |
| ------------------ | -------------------------- | ----------------------------- |
| Cardinalidad ideal | Baja/media                 | Alta                          |
| Tipo de filtro     | Rangos, igualdad           | Solo igualdad / IN            |
| Efecto físico      | Reordena micro-particiones | Crea índice de punteros (SAP) |
| Costo              | Compute re-clustering      | Storage + serverless          |

> **Son complementarios**, no excluyentes. Al re-clusterizar, el SAP queda desactualizado y se reconstruye automáticamente → queries caen al pruning estándar durante la reconstrucción. Con DML frecuente + Clustering activo: **doble costo serverless** → monitorear.

---
# Vistas
### Standard vs Secure vs Materialized

| Característica     | Standard        | Secure                      | Materialized         |
| ------------------ | --------------- | --------------------------- | -------------------- |
| DDL visible        | Todos los roles | Solo owner / `ACCOUNTADMIN` | Todos los roles      |
| Predicate pushdown | ✅ Completo      | ❌ Solo en frontera          | ✅ Completo           |
| Performance        | Alta            | Menor                       | Alta (pre-computada) |
| Costo extra        | No              | No                          | Storage + serverless |

#### Standard
DDL visible para cualquier rol con acceso. El optimizer inspecciona la definición y aplica optimizaciones hacia las tablas base → mejor performance.
#### Secure
Oculta el DDL y desactiva optimizaciones del optimizer, bloqueando el **predicate pushdown** hacia las tablas base. Esto previene **side-channel attacks** (inferir datos mediante tiempos de ejecución o errores).

**Cuándo usarla:**

- Secure Data Sharing entre cuentas (Snowflake lo exige)
- Row-level filtering por cliente: `WHERE cliente_id = CURRENT_ACCOUNT()`
- DDL contiene lógica o información sensible

> ❌ Performance significativamente menor. No usar si los datos no son sensibles.
#### Materialized
Pre-computa y almacena físicamente el resultado de una query. Snowflake la mantiene sincronizada automáticamente vía proceso **serverless incremental** — solo recalcula particiones afectadas por DML.

**Limitaciones:**
- Una sola tabla base (❌ no soporta joins)
- ❌ `HAVING`, `UNION`, `ORDER BY`, subqueries
- El optimizer puede ignorarla en caso la vista este desactualizado y consultar la tabla base directamente

**Costos:** storage del resultado + compute serverless de mantenimiento ( según eventos DML )

**Cuándo usarla:**
- Query repetitiva y costosa sobre la misma agregación o filtros repetitivos
- External Table + MV (patrón de near real-time)

**Cuándo NO:**
- Tabla base con DML muy frecuente → costo de refresh supera el beneficio
- Queries variadas sobre la misma tabla → mejor usar **query result cache**

> ⚠️ Si `AUTO_REFRESH` de la External Table está mal configurado, la MV servirá datos desactualizados sin error.

> 💡 Si necesitas joins o múltiples fuentes: usa **Dynamic Tables** — soportan transformaciones complejas y permiten definir `TARGET_LAG` (frescura objetivo).

---
### 5. Zero-Copy Cloning

#### Concepto

`CREATE TABLE/SCHEMA/DATABASE <clon> CLONE <fuente>` crea un nuevo objeto que **comparte las mismas micro-particiones físicas** que la fuente. Solo se duplica la **metadata**, no los bytes.

#### Características

- **Storage inicial del clon = 0.**
- A medida que se hacen cambios en el clon, se crean **nuevas micro-particiones propias** (copy-on-write). Solo esos deltas consumen storage.
- Cambios en la **fuente NO se replican al clon** y viceversa — son objetos independientes desde el momento del clonado.
- Soporta `AT` y `BEFORE` (Time Travel) para clonar desde un punto histórico:

sql

```sql
  CREATE TABLE clon CLONE fuente AT (OFFSET => -3600);
```

> ⚠️ **Corrección a tu nota:** escribiste `ADD y BEFORE`. La sintaxis correcta es **`AT` y `BEFORE`** (cláusulas del Time Travel).

#### Qué se clona y qué no

- ✅ Estructura, datos lógicos, constraints.
- ❌ **Privilegios/grants no se clonan** (excepto en algunos objetos hijo dentro de DBs/schemas clonados).
- ❌ Snowpipes, tasks, streams clonados pero suspendidos por defecto.

#### Zero-Copy vs Deep Copy

||Zero-Copy Clone|Deep Copy (`CTAS`)|
|---|---|---|
|Storage inicial|0|Duplicado completo|
|Metadata|Apunta a particiones de la fuente|Particiones nuevas|
|Tiempo|Instantáneo|Proporcional al tamaño|
|Aislamiento|Total tras la creación|Total|

#### Casos de uso

1. **Prod → Dev:** datasets de prueba sin costo de storage.
2. **Backup pre-deployment:** clonar antes de un pipeline destructivo.
3. **Recuperación post-DROP:** combinar Time Travel + clone para restaurar.

#### Equivalencias

- **BigQuery:** existe `CREATE TABLE CLONE` solo a **nivel de tabla**, no de dataset/proyecto. Snapshots = copias físicas.
- **AWS:** no existe nativo. Snapshots de Redshift y de S3 son copias físicas. Lo más cercano: **Apache Iceberg** con shadow clones, pero con administración manual.

> ✅ **Diferencial fuerte de Snowflake** vs el ecosistema AWS/GCP.

---

### 6. Clustering y Re-clustering

#### Concepto

Snowflake mantiene las micro-particiones **co-localizadas** según una **clustering key** definida explícitamente. El **Automatic Clustering** monitorea el **clustering depth** (medida de solapamiento de rangos entre micro-particiones) y dispara re-clustering serverless cuando supera un umbral.

#### Comportamiento

- **No** se re-clusteriza por cada `INSERT`. Solo cuando hay **degradación significativa** del clustering depth (muchos inserts desordenados acumulados).
- El re-clustering **crea nuevas micro-particiones que reemplazan a las antiguas**.

#### Costo oculto: interacción con Time Travel

- Las micro-particiones **antiguas no se borran** — quedan retenidas durante el período de **Time Travel** (1–90 días según edición).
- Si la tabla tiene `DATA_RETENTION_TIME_IN_DAYS = 90`, las particiones reemplazadas viven 90 días extra en storage.
- Combinado con DML frecuente → **costo de storage se dispara**.

#### Interacción peligrosa: Clustering + Search Optimization Service

- Cada re-clustering reemplaza micro-particiones.
- SOS debe **reconstruir su Search Access Path** sobre las nuevas particiones → más compute serverless.
- En tablas con re-clustering frecuente y SOS habilitado, el costo de mantenimiento puede ser muy alto.

#### Cuándo considerar Hybrid Tables como alternativa

Si el patrón es **DML frecuente + lookups puntuales**, una Hybrid Table elimina:

- El costo de re-clustering.
- El costo de mantenimiento del SOS (sus índices secundarios son nativos).
- El costo doble (clustering + SOS) en tablas calientes.

**Pero a cambio pierdes:**

- Time Travel.
- Fail-safe.
- Eficiencia analítica para tablas muy grandes (>cientos de TB).

#### Decisión rápida

|Patrón|Solución recomendada|
|---|---|
|Tabla grande, queries con rangos, baja/media cardinalidad|Clustering Key|
|Tabla grande, lookups por igualdad, alta cardinalidad|Search Optimization Service|
|Tabla grande, ambos patrones, DML moderado|Clustering + SOS (con cuidado del costo)|
|Tabla mediana, DML frecuente + lookups|Hybrid Table (si no necesitas Time Travel)|

---

## 📋 Resumen Ejecutivo (5 puntos clave)

1. **External Tables son read-only y sin cache efectivo** — el patrón óptimo es combinarlas con Materialized Views (Near Real-Time) o sustituirlas por Snowpipe/Snowpipe Streaming (Real-Time).
2. **Secure Views protegen el DDL pero pagan un costo de performance** porque desactivan predicate pushdown hacia las tablas base. Son **obligatorias** para Secure Data Sharing.
3. **Zero-Copy Cloning** duplica solo metadata (storage inicial = 0). Es uno de los diferenciales más fuertes de Snowflake vs BigQuery (limitado) y AWS (no existe nativo).
4. **Clustering Keys vs Search Optimization Service son complementarios, no excluyentes** — clustering para rangos/baja cardinalidad, SOS para igualdad/alta cardinalidad. Cuidado con el costo combinado en tablas con DML frecuente.
5. **Hybrid Tables (Unistore) son la única respuesta nativa de Snowflake a HTAP**, pero sacrifican Time Travel y Fail-safe. Solo justifican su overhead de doble estructura cuando realmente necesitas OLTP + OLAP en el mismo objeto.

---

## 🔧 Correcciones aplicadas a tus notas originales

|Error|Tipo|Corrección|
|---|---|---|
|"GSC"|Tipográfico|**GCS** (Google Cloud Storage)|
|"BigQuery LAKES"|Producto|**BigLake**|
|"Sears Optimation Service"|Tipográfico|**Search Optimization Service**|
|"Road Level Looking"|Tipográfico/conceptual|**Row-Level Locking**|
|"operaciones ADD y BEFORE"|Conceptual|**AT y BEFORE** (Time Travel clauses)|
|"no existe s" (Hybrid Tables)|Frase incompleta|Probablemente **"no existe Time Travel ni Fail-safe"**|
|"OLTP + escrituras transaccionales analytics"|Conceptual|**OLTP + OLAP**|
|"material view si cachea el resultado"|Impreciso|MV **materializa** (almacena físicamente), no cachea — son cosas distintas|
|"Deep Copy en Snowflake"|Confusión|En Snowflake el equivalente sería un **`CTAS` (CREATE TABLE AS SELECT)**, no es un comando "Deep Copy" propio|

---

## 🎯 Recomendaciones de qué estudiar a continuación

Basado en los temas que ya cubriste, estos son los gaps más relevantes para el COF-C03:

#### Gaps directos del Domain 1.0 (31% — el de mayor peso)

- **Tipos de tabla restantes:** Permanent vs Temporary vs Transient vs Dynamic (esta última la mencionaste pero no la profundizaste).
- **Tipos de vista:** Standard, Materialized, Secure (cubriste 2 de 3 — falta confirmar Standard).
- **Apache Iceberg Tables:** muy preguntado en la versión COF-C03 actual.
- **Micro-particiones internamente:** estructura columnar, header, min/max, orden de inserción.

#### Gaps del Domain 4.0 (21%)

- **Caching layers:** Result Cache (24h), Metadata Cache, Warehouse Cache (local SSD). Saber qué invalida cada uno.
- **Query Acceleration Service** (distinto del Search Optimization).
- **Query Profile:** interpretar bytes spilled, exploding joins, pruning eficiente.

#### Gaps del Domain 2.0 (20%)

- **Tagging + Masking Policies** (column-level y row-level).
- **Data Classification** (apareció en una sample question del Study Guide).
- **Resource Monitors:** suspend, suspend_immediate, notify.

#### Sugerencia de orden

1. Cerrar **Storage** (Iceberg + Dynamic Tables) — extiende tus notas actuales.
2. **Caching** — base teórica corta, alto retorno en preguntas.
3. **Governance/Tagging/Masking** — Domain 2.0 completo.