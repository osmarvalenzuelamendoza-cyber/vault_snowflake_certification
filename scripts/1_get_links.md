Realiza web scraping al sidebar de esta página de documentación de Snowflake:

**URL:** https://docs.snowflake.com/en/user-guide/warehouses

**Sección padre a scrapear:** Virtual Warehouses

**Objetivo:** Extraer los títulos y URLs de todos los links pertenecientes
a la sección padre indicada, respetando su jerarquía visual completa
(Nivel 0 → Nivel 1 → Nivel 2 → Nivel 3 si existe).

---

**Estrategia de extracción (ejecutar en este orden):**

1. Intenta con Playwright (Chromium headless). Si el dominio está bloqueado
   por el egress proxy del sandbox, pasa al paso 2.
2. Usa `web_fetch` sobre la URL raíz para obtener los ítems de Nivel 1.
3. **Fetch recursivo obligatorio:** el sidebar de Snowflake solo expande la
   rama activa en cada página. Para descubrir los hijos de cada ítem de
   Nivel 1, haz `web_fetch` individual a cada una de sus URLs y extrae
   los subniveles del sidebar renderizado.
4. Si algún ítem de Nivel 2 muestra hijos, repite el fetch sobre esa URL.
5. Si `web_fetch` falla, usa `web_search` para descubrir URLs hijas en
   docs.snowflake.com y luego fetch sobre cada resultado.

**Validación antes de generar el output:**
Confirma que cada ítem de Nivel 1 fue inspeccionado individualmente.
Si alguno NO fue fetched, márcalo como "⚠ sin verificar" en el output.
No asumir que un nodo sin hijos visibles en la raíz realmente carece de hijos.

---

**Formato de salida — Markdown con jerarquía:**

Usa listas anidadas respetando padre → hijo → subhijo.
Las cabeceras de grupo sin URL (ej: "Views", "Considerations") se muestran
como texto plano sin enlace. Ejemplo de estructura esperada:

- [Databases, Tables, & Views](URL)
  - [Table Structures](URL)
    - [Micro-Partitions & Data Clustering](URL)
    - [Cluster Keys & Clustered Tables](URL)
  - [Temporary And Transient Tables](URL)
  - Views
    - [Views](URL)
    - [Secure Views](URL)

---

**Restricciones:**
- NO extraer links de otras secciones del sidebar
- NO dar por completo el árbol usando solo el fetch de la URL raíz
- Distinguir cabeceras de grupo (sin href) de páginas reales (con URL)
- NO asumir que el sidebar está completamente expuesto en un solo request