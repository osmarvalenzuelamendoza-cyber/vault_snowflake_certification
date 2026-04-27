## ❄️ Continuous Data Protection (CDP)

### ¿Qué es CDP?

CDP es el conjunto de features de Snowflake que protege los datos ante errores humanos, actos maliciosos y fallas de software, en **cada etapa del ciclo de vida del dato**.

---
### Componentes del CDP — todos evaluados en el examen

| Feature              | Concepto clave                                                                                                                                                                              |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Network Policies** | Control de acceso por IP (allow/deny lists)                                                                                                                                                 |
| **Autenticación**    | MFA y SSO habilitados por usuario                                                                                                                                                           |
| **Security Roles**   | Control de acceso a todos los objetos del sistema (RBAC, Control Acceso basado en roles)                                                                                                    |
| **Encriptación**     | AES-256 server-side en internal stages por defecto. Client-side con clave de **128 bits** por defecto, configurable a **256 bits**                                                          |
| **Time Travel**      | Consulta y restauración de datos históricos modificados o eliminados. Hasta **90 días** en **versión Enterprise**, versiones menores solo son 7 días. Genera costos adicionales de storage. |
| **Fail-safe**        | Recuperación ante desastres, **solo ejecutable por Snowflake**. Genera costos adicionales de storage.                                                                                       |

---
### Datos críticos para el examen

**Disponibilidad por edición:**
- La mayoría de features CDP están disponibles en **todas las ediciones**
- Algunas features requieren **Enterprise Edition o superior** (Column/Row-level Security, Extended Time Travel)