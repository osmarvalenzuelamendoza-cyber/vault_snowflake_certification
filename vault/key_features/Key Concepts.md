## ❄️ Snowflake — Key Concepts & Architecture (Domain 1 — 31%)

### Snowflake como Self-Managed Service

Snowflake no requiere gestión de hardware ni software por parte del usuario. Mantenimiento, upgrades y tuning son responsabilidad de Snowflake. **No puede instalarse on-premises ni en nubes privadas**, solo corre sobre infraestructura cloud pública.

---

### Arquitectura — El tema más evaluado del examen

Snowflake implementa una arquitectura **híbrida** que combina:

- **Shared-disk** → repositorio central de datos accesible desde todos los nodos
- **Shared-nothing** → procesamiento mediante **MPP (Massively Parallel Processing)**, cada nodo procesa una porción del dataset localmente

Esta combinación entrega la simplicidad de gestión del shared-disk con el rendimiento y escalabilidad del shared-nothing.

**Las 3 capas de la arquitectura:**

|Capa|Función|
|---|---|
|**Database Storage**|Almacenamiento persistente en cloud, formato columnar comprimido, organizado en micro-partitions|
|**Compute**|Virtual Warehouses — clusters independientes de cómputo, sin compartir recursos entre sí|
|**Cloud Services**|Coordinación global: autenticación, metadata, query parsing/optimization, governance, infrastructure management|

> Cada Virtual Warehouse es un cluster **completamente independiente**. El rendimiento de uno no afecta a los demás. Este comportamiento es pregunta frecuente en el examen.

---

### Tipos de Tablas — Domain 1

|Tipo|Características clave|
|---|---|
|**Snowflake Tables**|Formato columnar comprimido, micro-partitions automáticas, para structured y semi-structured data. Ideal para data warehouses|
|**Apache Iceberg Tables**|Data y metadata en cloud storage **externo** (S3, GCS, Azure). Ideal para data lakes/lakehouses que no se quieren mover a Snowflake|
|**Hybrid Tables**|Optimizadas para **baja latencia y alto throughput**. Soportan row locking y constraints de integridad referencial. Para workloads **transaccionales + analíticos (Unistore)**|

---

### Cloud Services Layer — Qué gestiona

Componente crítico evaluado en el examen. Gestiona:

- Seguridad, autenticación y control de acceso
- **Snowflake Horizon Catalog**
- Metadata management (SNOWFLAKE database + Information Schema)
- Query parsing y optimización
- Infrastructure management con cloud providers
- Regulatory compliance

---

### Mecanismos de Ingesta — Domain 3

|Mecanismo|Descripción|
|---|---|
|**COPY INTO**|Carga desde archivos en un stage hacia una tabla|
|**Snowpipe**|Carga automática desde archivos en cuanto están disponibles en un stage|
|**Snowpipe Streaming**|Carga continua de datos a **nivel de fila**, baja latencia, via SDK o REST API directamente a tablas Snowflake o Iceberg|
|**Snowflake Connectors**|Conectan sistemas externos y hacen streaming hacia Snowflake|

> La diferencia entre **Snowpipe** (basado en archivos) y **Snowpipe Streaming** (row-level, baja latencia, sin archivos) es un punto frecuente de confusión en el examen.

### Mecanismos de Transformación — Domain 3/4

| Mecanismo           | Descripción                                                                   |
| ------------------- | ----------------------------------------------------------------------------- |
| **Dynamic Tables**  | Se refrescan automáticamente según target de freshness definido por query SQL |
| **Streams + Tasks** | Streams capturan cambios (CDC), Tasks ejecutan transformaciones programadas   |
| **Snowpark**        | Transformaciones complejas en Python, Java, Scala dentro de Snowflake         |
| **dbt**             | Framework open-source de transformación SQL integrado con Snowflake           |

---

### Snowgrid — Domain 1

**Snowgrid** es la capa tecnológica cross-region y cross-cloud de Snowflake. Permite:

- Conectar ecosistemas de datos entre distintas regiones y cloud providers
- Aplicar políticas de seguridad y governance consistentes entre clouds
- Habilitar **disaster recovery y business continuity** entre regiones via replication

---

### AI/ML — Domain 1

| Suite                | Función                                                                                                                                                                 |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Snowflake Cortex** | Features de AI con LLMs para datos no estructurados, preguntas en lenguaje natural, resúmenes, traducciones. Incluye Cortex AI Functions, Cortex Search, Cortex Analyst |
| **Snowflake ML**     | Construcción de modelos propios. ML Functions para predicciones automatizadas                                                                                           |

---

### Formas de conectarse a Snowflake — Domain 1

- **Snowsight** → UI web principal
- **Snowflake CLI** → cliente command-line
- **Drivers** → JDBC, ODBC para conectar aplicaciones externas
- **Native Connectors** → Apache Kafka, Apache Spark
- **APIs** → Python APIs, REST APIs para gestión programática