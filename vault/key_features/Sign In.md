## ❄️ Sign in to Snowflake (Domain 1 — Interfaces)

### Requisitos para conectarse a Snowflake

Todo acceso a Snowflake requiere dos elementos:

- **Account Identifier** → identificador único de la cuenta
- **Método de autenticación** → definido por la organización

---

### Métodos de autenticación principales

|Método|Descripción|
|---|---|
|**SSO (Federated Auth)**|El usuario se autentica via un Identity Provider (IdP) externo como Okta o Microsoft Entra ID, no directamente con Snowflake|
|**Password + MFA**|Usuario/contraseña más un segundo factor: **passkeys**, **authenticator apps** o **Duo**|

---

### Formas de conectarse

|Método|Descripción|
|---|---|
|**Snowsight**|UI web principal, accesible en `app.snowflake.com`, vía internet público o private connectivity|
|**Snowflake CLI**|Cliente command-line|
|**JDBC / ODBC**|Para aplicaciones de terceros|
|**Connectors y Drivers**|Python, Node.js, Spark, entre otros|

---

> Esta página no agrega conceptos nuevos al examen más allá de lo ya cubierto. Su valor real está en confirmar la terminología de autenticación (SSO, MFA, IdP) que sí aparece en el **Domain 2**.