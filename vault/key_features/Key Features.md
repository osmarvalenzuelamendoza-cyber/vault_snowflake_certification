### Domain 2 — Security & Data Protection

**Métodos de autenticación** — todos evaluados en el examen:

- MFA, Federated Authentication, SSO, Snowflake OAuth, External OAuth, Key-pair authentication
- Toda comunicación cliente-servidor protegida por **TLS**
- Despliegue dentro de **VPC** (AWS/GCP) o **VNet** (Azure)

**Data Protection con límites numéricos críticos:**

|Feature|Detalle|Edición mínima|
|---|---|---|
|**Time Travel**|1 día estándar, hasta **90 días**|Enterprise+|
|**Fail-safe**|**7 días fijos**, solo para DR, gestionado por Snowflake|Todas|
|**Column-level Security**|Masking policies en columnas|Enterprise+|
|**Row-level Security**|Row access policies en tablas/vistas|Enterprise+|
|**Object Tagging**|Tracking de datos sensibles y uso de recursos|Enterprise+|
|**PHI / HIPAA**|Datos de salud regulados|Business Critical+|
|**Differential Privacy**|Protección contra ataques de privacidad dirigidos|Enterprise+|

> Fail-safe opera **después** del período de Time Travel. No es configurable por el usuario ni se puede activar/desactivar. Es exclusivo de Snowflake Support.

---

### Domain 1 — Tipos de Tablas y Vistas

**Tablas** — comportamiento diferenciado en Time Travel y Fail-safe:

|Tipo|Persistencia|Time Travel|Fail-safe|
|---|---|---|---|
|**Permanent**|Indefinida|✔ hasta 90d|✔ 7 días|
|**Temporary**|Solo sesión activa|✔ hasta 1d|✗|
|**Transient**|Entre sesiones|✔ hasta 1d|✗|
|**Iceberg**|External (open format)|Limitado|✗|
|**External**|Cloud storage externo|✗|✗|
|**Dynamic**|Materialización automática por SQL|✔|✔|

**Vistas:**

- **Standard** → vista SQL estándar
- **Materialized** → resultados pre-computados, mantenimiento automático → requiere **Enterprise+**
- **Secure** → oculta la definición de la vista a usuarios no autorizados

---

### Domain 1 — Interfaces y Herramientas

- **Snowsight** → UI principal: gestión de cuenta, monitoring, queries
- **Snowflake CLI** → cliente open source para workloads de desarrollo
- **SnowSQL** → cliente Python para queries, bulk loading/unloading y DDL
- **VS Code Extension** → integración IDE evaluada en el examen
- Virtual Warehouses se pueden crear, redimensionar (**zero downtime**), suspender y eliminar desde GUI o CLI

---

### Domain 3 — Data Import & Export

**Formatos soportados para carga:**

- Estructurados: CSV, TSV
- Semi-estructurados: JSON, Avro, ORC, Parquet, XML
- Soporta carga desde archivos comprimidos y desde cloud storage

**Mecanismos de carga:**

|Mecanismo|Tipo|Fuente|
|---|---|---|
|**Bulk loading** (`COPY INTO`)|Por lotes|Internal/External stages|
|**Snowpipe**|Micro-batches continuos|Internal stages, S3, GCS, Azure|

---

### Domain 5 — Data Sharing & Replication

- Data Sharing entre cuentas Snowflake en modo **Provider → Consumer**
- **Data Clean Rooms** → compartir datos en entorno de privacidad preservada
- **Replication y Failover** entre cuentas de la misma organización, en distintas regiones y cloud platforms

---

### Domain 1 — Apps & Extensibility (evaluado en examen)

- **Snowpark** → ejecutar código Python, Java, Scala directamente en Snowflake sin mover datos
- **Streamlit in Snowflake** → apps web nativas para ML y data science
- **UDFs** → Java, JavaScript, Python, Scala, SQL
- **Stored Procedures** → mismos lenguajes + Snowflake Scripting
- **Native Apps Framework** → compartir datos y lógica de aplicación entre cuentas