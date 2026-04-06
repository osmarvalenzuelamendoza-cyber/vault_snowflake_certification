## ❄️ Data Lifecycle — Snowflake (Domain 1 y Domain 4)

### Jerarquía de organización de datos

La jerarquía es database -> schema -> table
Snowflake **no impone límites** en el número de databases, schemas ni tables que se pueden crear.

---
### Operaciones del ciclo de vida — comandos clave

| Etapa           | Operaciones                                                                |
| --------------- | -------------------------------------------------------------------------- |
| **Organizar**   | `CREATE / ALTER DATABASE`, `CREATE / ALTER SCHEMA`, `CREATE / ALTER TABLE` |
| **Cargar**      | `INSERT`, `COPY INTO <table>`                                              |
| **Consultar**   | `SELECT`                                                                   |
| **Transformar** | `UPDATE`, `MERGE`, `DELETE`, `CREATE <object> … CLONE`                     |
| **Eliminar**    | `TRUNCATE`, `DROP`                                                         |
### Punto clave para el examen

**Cloning** (`CREATE <object> … CLONE`) opera a nivel de databases, schemas y tables. Es una operación DDL, no DML. El examen distingue entre eliminar filas (`DELETE`) y eliminar el objeto completo (`DROP` / `TRUNCATE`).

> `TRUNCATE` elimina todos los datos de la tabla pero **mantiene la estructura**. `DROP` elimina la tabla por completo. Esta distinción puede aparecer en preguntas de escenario.