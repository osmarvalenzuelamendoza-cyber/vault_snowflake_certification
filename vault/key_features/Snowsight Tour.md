## ❄️ Snowsight — Interface Principal de Snowflake (Domain 1 — 1.2)

### ¿Qué es Snowsight?

Es la **UI web principal de Snowflake**, accesible en `app.snowflake.com`. Centraliza todas las capacidades de la plataforma: desarrollo, ingesta, transformación, governance, monitoring y administración de cuenta.

---

### Capacidades clave por área

**Work with Data:**

|Feature|Descripción|
|---|---|
|**Workspaces**|Editor unificado para SQL y Python, organización por folders, integración con Git|
|**Notebooks**|Entorno cell-based para Python, SQL y Markdown. Usado para EDA, ML y data science|
|**Streamlit**|Apps interactivas de datos en Python sin gestionar infraestructura|
|**Ingestion**|Carga via Snowpipe, connectors y file upload directo desde UI|
|**Dynamic Tables / Tasks**|Transformaciones continuas y programadas directamente desde Snowsight|

**Horizon Catalog:**

|Feature|Descripción|
|---|---|
|**Universal Search**|Descubrimiento de objetos a través de todo el data estate|
|**Snowflake Horizon Catalog**|Catálogo central de datos, tablas, funciones y vistas|
|**Data Sharing**|Compartir datos internamente (dentro de la org) o externamente (otras organizaciones) como Provider o Consumer|
|**Governance & Security**|Masking policies, row access policies, tags, gestión de usuarios/roles y **Trust Center**|

**Account Management:**

|Feature|Descripción|
|---|---|
|**Compute**|Gestión de Virtual Warehouses: auto-suspend, auto-resume, utilización|
|**Admin**|Cost & billing, integrations con sistemas externos, Partner Connect|

---

### Conceptos con mayor probabilidad en el examen

**Monitoring desde Snowsight** — el examen puede preguntar qué herramienta usar para monitorear actividad:

- **Query History** → historial y performance de queries
- **Copy History** → monitoreo de actividad de data loading
- **Task Graphs** → ejecución y dependencias de Tasks

**Roles en Data Sharing desde Snowsight:**

- **Provider** → publica data products y listings
- **Consumer** → accede a datasets y app packages sin necesidad de pipelines propios

**Trust Center** → evaluación del security posture de la cuenta, ya referenciado en Domain 2.

---

> Esta página tiene **bajo peso directo** en el examen. Su valor está en confirmar que Snowsight es la herramienta central para ejecutar prácticamente todas las funciones de Snowflake desde una sola UI, y en familiarizarte con la terminología de sus secciones para preguntas de escenario.