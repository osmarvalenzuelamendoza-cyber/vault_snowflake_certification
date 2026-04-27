# Microparticiones

## ¿Qué son?

La unidad fundamental de almacenamiento en Snowflake. Toda tabla se divide automáticamente en microparticiones de **50–500 MB descomprimidos** (mucho menos comprimidos, ya que la compresión opera **a nivel de columna** dentro de cada partición).

---

## Cómo optimizan las consultas

**Partition pruning** — cada micropartición tiene metadatos precalculados por columna (mín, máx, distintos, nulos). Cuando aplicas un filtro, el Cloud Services layer descarta las particiones que no pueden contener los valores buscados, **sin consumir créditos de compute**.

**Column scanning** — al ser columnar, una query solo lee las columnas que necesita, no la fila completa.

---

## Clustering — ordenamiento de los datos

El rendimiento del pruning depende de cuán ordenados estén los datos dentro de las microparticiones. Hay tres formas de controlar esto:

- **Carga ordenada** — si insertas datos ya ordenados por fecha, cada micropartición tendrá un rango de fechas claro y el pruning será eficiente.
- **Clustering Keys** — le indicas a Snowflake por qué columna(s) debe mantener ordenada la tabla. Snowflake reclustariza en background automáticamente. ⚠️ _Corrección: las Clustering Keys no son solo una buena práctica, son un objeto configurable explícito con costo de compute asociado. No es lo mismo que simplemente cargar ordenado._
- **ORDER BY en la carga** — puedes ordenar los datos antes de insertarlos como alternativa más económica a las Clustering Keys.

---

## Clustering Depth

Métrica que indica el **solapamiento promedio** entre microparticiones para una columna dada.

- **Depth bajo (cercano a 1)** → particiones bien ordenadas, pruning eficiente, pocas particiones escaneadas por query.
- **Depth alto** → muchas particiones se solapan en rangos de valores → para filtrar por una fecha, Snowflake debe escanear decenas o cientos de microparticiones en lugar de unas pocas.

> **Conclusión:** a mayor Clustering Depth → peor pruning → peor rendimiento.

Puedes monitorear esto con vistas del esquema `SNOWFLAKE.ACCOUNT_USAGE` como `CLUSTERING_INFORMATION`.

---

## Inmutabilidad y ciclo de vida

Las microparticiones son **inmutables**. Cualquier operación DML (`UPDATE`, `DELETE`) no modifica la partición existente — la marca como inactiva y crea una nueva.

```
MP original  →  pasa a INACTIVA  →  Time Travel  →  Fail-safe  →  eliminación física
MP nueva     →  queda ACTIVA
```

- **Time Travel**: retención configurable (1 día por defecto, hasta 90 días en Enterprise). Permite consultar versiones anteriores de los datos.
- **Fail-safe**: 7 días adicionales gestionados exclusivamente por Snowflake. No es accesible por el usuario.

⚠️ _Corrección: el "tiempo de vida" no lo define el usuario libremente — Time Travel tiene un máximo según la edición de Snowflake. Fail-safe siempre son 7 días fijos y solo Snowflake puede actuar sobre esa data._

---
## DROP COLUMN — operación de solo metadatos

Eliminar una columna es una **metadata-only operation**. La micropartición física no se toca — los bytes de esa columna siguen en storage. El optimizer simplemente ignora esa columna al leer.

⚠️ _Corrección menor: no es exactamente que el optimizer "detecta que la columna no está en el micropartition" — la columna sí sigue físicamente ahí. Lo que cambia es que los metadatos de la tabla indican que esa columna ya no es visible, y por eso no se devuelve en los resultados._

Los bytes se liberan físicamente solo cuando esa micropartición es reemplazada por DML futuro.

---
## Resumen visual rápido

| Concepto         | Clave para el examen                        |
| ---------------- | ------------------------------------------- |
| Tamaño           | 50–500 MB descomprimido                     |
| Compresión       | A nivel de columna                          |
| Pruning          | Basado en metadatos, sin costo de compute   |
| Clustering Depth | Bajo = bueno, Alto = malo                   |
| DML              | Crea nuevas particiones, original retenida  |
| DROP COLUMN      | Solo metadatos, datos físicos persisten     |
| Clustering Keys  | Objeto configurable, tiene costo de compute |

---

# Snowflake — Clustering Key

## ¿Qué es un Clustering Key?

Es un conjunto de una o más columnas (máximo 4) definidas sobre una tabla, cuyo propósito es controlar cómo Snowflake organiza los datos en sus micro-particiones. Al agrupar filas con valores similares en las mismas micro-particiones, se maximiza el **partition pruning** durante la ejecución de queries.

---

## Orden de columnas y cardinalidad

### Regla de cardinalidad

Las columnas deben ordenarse de **menor a mayor cardinalidad**: la primera columna debe tener la cardinalidad más baja, y las siguientes deben ir aumentando progresivamente. Esto permite que el pruning elimine la mayor cantidad de micro-particiones irrelevantes desde el primer filtro.

### Recomendaciones para elegir columnas

- Priorizar columnas que aparecen frecuentemente en cláusulas `WHERE` y condiciones de `JOIN`.
- Si una columna es de tipo `TIMESTAMP`, se recomienda truncarla a granularidad de fecha (`DATE`) para reducir la cardinalidad y mejorar el agrupamiento.

### Soporte para campos VARIANT

Se puede aplicar clustering sobre atributos de columnas semiestructuradas (`VARIANT`), siempre que se aplique una expresión de transformación explícita. Por ejemplo:

sql

```sql
ALTER TABLE mi_tabla CLUSTER BY (v:campo::STRING);
```

---

## Comportamiento automático y costos

### Automatic Clustering

Al definir un Clustering Key, Snowflake activa el **Automatic Clustering** en background. Este proceso reorganiza los datos continuamente generando nuevas micro-particiones optimizadas. El ciclo de vida de las micro-particiones antiguas es:

```
Activa → Inactiva → Time Travel → Fail-Safe → Eliminada
```

### Modelo de costos

El reclustering automático opera bajo el modelo **serverless** de Snowflake: genera consumo de créditos en background, independiente del warehouse del usuario. Este costo debe monitorearse, especialmente en tablas con alta tasa de inserción o modificación.

---

## Administración del Clustering Key

El Automatic Clustering puede suspenderse y reanudarse con `ALTER TABLE`:

sql

```sql
-- Suspender
ALTER TABLE nombre_tabla SUSPEND RECLUSTER;

-- Reanudar
ALTER TABLE nombre_tabla RESUME RECLUSTER;
```

> Al ejecutar cualquier `ALTER TABLE` sobre una tabla con clustering suspendido, el reclustering se **reanuda automáticamente**.

---

## Alternativas y consideraciones de tamaño

### Alternativa manual: ingesta ordenada

Si no se desea habilitar el Automatic Clustering, una alternativa más sencilla es reingestar los datos en el orden deseado. Snowflake organiza los datos en micro-particiones en el orden en que llegan, por lo que una carga ordenada puede ser suficiente para obtener buen pruning sin incurrir en costos de reclustering.

### Tamaño mínimo recomendado

El Clustering Key solo es rentable en tablas con un volumen **mayor a 1 TB**. En tablas más pequeñas, Snowflake ya realiza pruning eficiente con sus micro-particiones naturales, por lo que agregar un Clustering Key no justifica el costo adicional.

---

## Funciones de diagnóstico

Snowflake provee funciones nativas para evaluar el estado y eficiencia del clustering:

|Función|Descripción|
|---|---|
|`SYSTEM$CLUSTERING_INFORMATION`|Muestra el estado actual del clustering (profundidad, particiones, overlap).|
|`SYSTEM$ESTIMATE_CLUSTERING_KEYS`|Sugiere columnas candidatas para el Clustering Key.|

Ambas son útiles para validar si el clustering definido está siendo efectivo antes y después de habilitarlo.

# Snowflake — Tipos de Tablas: Permanent, Transient y Temporary

## Comparativa general

|Característica|Permanent|Transient|Temporary|
|---|---|---|---|
|Time Travel máximo|90 días*|1 día|1 día|
|Fail-Safe|7 días|No tiene|No tiene|
|Alcance|Persistente|Persistente|Solo sesión activa|

> *El Time Travel de 90 días está disponible en las ediciones Enterprise y superiores. En ediciones inferiores el máximo es 1 día.

---
## Descripción de cada tipo

### Permanent

Tabla estándar de Snowflake. Persiste indefinidamente hasta que se elimine explícitamente. Ofrece el mayor nivel de protección de datos gracias a su Time Travel extendido y su período de Fail-Safe.

### Transient

Persiste entre sesiones igual que una tabla Permanent, pero sin Fail-Safe y con Time Travel limitado a 1 día. Es útil cuando se quiere reducir costos de almacenamiento y la fuente de datos es confiable y reutilizable.

### Temporary

Existe únicamente durante la sesión activa. Al cerrar la sesión, la tabla se elimina automáticamente. Útil para almacenamiento intermedio dentro de un proceso o pipeline puntual.

---
## ¿Cuándo usar cada una?

En la práctica, la decisión de diseño relevante es entre **Permanent y Transient**, ya que las tablas Temporary tienen un uso muy acotado por su naturaleza efímera.

### Usar Transient cuando...

La fuente de datos es **fija y reutilizable**, como un bucket en S3, Google Cloud Storage u otro almacenamiento objeto externo. En caso de pérdida de datos, es posible eliminar la tabla y reingestar desde la fuente sin riesgo. Esto evita el costo de almacenamiento del Fail-Safe.

sql

```sql
CREATE TRANSIENT TABLE nombre_tabla (...);
```

### Usar Permanent cuando...

La fuente de datos **no siempre está disponible**, como un sistema CDC (Change Data Capture) o una fuente de streaming (Kafka, Kinesis, etc.). Si los datos se pierden por un `DROP TABLE` accidental o una eliminación de registros, el Time Travel de hasta 90 días es el único respaldo disponible.

sql

```sql
CREATE TABLE nombre_tabla (...);  -- Permanent por defecto
```

---
## Esquemas Transient

Si se crea un esquema como `TRANSIENT`, **todas las tablas creadas dentro de ese esquema heredan el tipo Transient** de forma automática, sin necesidad de declararlo explícitamente en cada tabla.

sql

```sql
CREATE TRANSIENT SCHEMA nombre_schema;
```

> Esto aplica también a nivel de base de datos: un `CREATE TRANSIENT DATABASE` hace que todos sus esquemas y tablas sean Transient por defecto.


