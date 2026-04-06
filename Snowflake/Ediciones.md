## ❄️ Snowflake Editions — Resumen estratégico para el examen

La jerarquía es **acumulativa**, cada edición hereda todo lo anterior:
```
Standard → Enterprise → Business Critical → VPS
```
---
### 🔵 Standard
La edición base. Cubre todo el core de Snowflake: Virtual Warehouses, Streams, Tasks, Snowpipe, Snowpark, Data Sharing.
Límite clave: **Time Travel máximo 1 día**

---
### 🟡 Enterprise
Agrega las features orientadas a **escala y governance**:
- **Time Travel hasta 90 días** → límite numérico, pregunta segura
- **Multi-cluster Warehouses** → solución a problemas de concurrencia
- **Materialized Views** → optimización de queries complejos
- **Column-level y Row-level Security** → masking y access policies
- **Search Optimization y Query Acceleration** → performance en queries específicos
> 🧠 Escenario típico: _"alta concurrencia"_, _"data masking"_, _"Time Travel mayor a 1 día"_ → mínimo **Enterprise**
---
### 🔴 Business Critical
Entra cuando existen **obligaciones regulatorias o de compliance**:
- **HIPAA / PHI y PCI DSS** → datos médicos y financieros regulados
- **Tri-Secret Secure** → customer-managed encryption keys
- **AWS PrivateLink / Azure Private Link / GCP PSC** → conectividad privada
- **Failover y Failback** → disaster recovery entre cuentas Snowflake

> 🧠 Escenario típico: _"regulación"_, _"el cliente controla sus propias keys"_, _"disaster recovery"_, _"conectividad sin internet público"_ → **Business Critical**
---
### ⚫ VPS
Entorno Snowflake **completamente aislado a nivel de hardware**. No comparte recursos con otras cuentas.
Dos datos críticos:
1. Metadata store y compute **100% dedicados**
2. **Snowflake Marketplace NO disponible** ⚠️ — dato contraintuitivo frecuente en el examen
> 🧠 Escenario típico: _"aislamiento total"_, _"instituciones financieras con máximos requisitos de seguridad"_ → **VPS**
---
### 🎯 Árbol de decisión rápida

```
¿Necesita Marketplace?              → Descarta VPS
¿Regulación HIPAA/PCI/PrivateLink?  → Business Critical
¿Alta concurrencia o TT > 1 día?    → Enterprise
¿Todo lo demás?                     → Standard
```