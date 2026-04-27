Eres un instructor experto preparando a un Data Engineer con experiencia 
en GCP, AWS, Kafka y Databricks para el examen SnowPro Core (COF-C03).

Te adjunto el PDF del temario oficial del examen. Úsalo como filtro 
permanente para determinar qué es relevante en cada paso.

---

## FASE 1 — Lectura de URLs

Tengo los siguientes links de documentación oficial de Snowflake 
sobre un tema en común. Por cada URL:

1. Accede a la URL completa
2. Lee TODO el contenido sin excepción:
   - Secciones colapsadas o expandibles
   - Tabs o pestañas dentro de la página
   - Ejemplos de código y sus comentarios
   - Notas, advertencias y tips en recuadros especiales
   - Tablas completas con todos sus valores
   - Secciones de consideraciones o limitaciones
3. Si hay links internos directamente relacionados al tema, 
   accede a ellos con máximo 1 nivel de profundidad adicional.
   No sigas links que lleven a temas distintos del tema principal.
4. Si una URL no carga o devuelve error, indícamelo explícitamente 
   antes de continuar. No la omitas en silencio.

**Links del tema:**
[PEGA AQUÍ TUS LINKS]

Al terminar la lectura, confirma que leíste cada URL listando:
- URL leída
- Títulos de sección encontrados
- Si hubo links internos que seguiste y cuáles

No avances a la Fase 2 hasta completar esto.

---

## FASE 2 — Construcción del resumen

1. Extrae temas, subtemas y términos técnicos de todo el contenido leído
2. Cruza con el temario del PDF — descarta lo que no es relevante para el examen
3. Agrupa conceptos que se repitan entre links y sintetiza en subtemas reales
4. Por cada subtema entrega:

   **Concepto:** explicación técnica suficiente para entenderlo sin 
   fuente adicional

   **Utilidad práctica:** cómo aplica este concepto en el trabajo 
   diario de un data engineer

   **Buena práctica actual:** qué recomienda la industria hoy 
   respecto a este componente o feature
   *(separado visualmente del contenido de examen)*

   **Analogía GCP/AWS:** solo si existe un equivalente real y 
   la comparación no genera confusión

   **Posible pregunta de examen:** qué tipo de pregunta o trampa 
   podría aparecer en el COF-C03 según el temario oficial

   **Nota sobre tablas:** si el contenido tiene tablas, resúmelas 
   en texto — no las reproduzcas completas

---

## FASE 3 — Auditoría por bloque

Después de escribir cada subtema, antes de continuar al siguiente, 
verifica en una línea:
- ✅ Concepto completo y preciso según el temario
- ⚠️ [indicar si algo quedó incompleto y por qué]

Al terminar todos los subtemas, haz una auditoría final del resumen completo:
1. ¿Hay algún concepto del temario que no quedó cubierto?
2. ¿Hay algo en los links leídos que omitiste pero podría caer en el examen?
3. ¿Algún concepto quedó explicado de forma imprecisa?

Si encuentras gaps, actualiza el resumen antes de generar el HTML.

---

## FASE 4 — Entrega en HTML

Genera un único archivo HTML autocontenido con estas características:

**Estructura:**
- Sidebar fijo izquierdo con navegación por subtemas (links ancla)
- Área de contenido principal con scroll independiente
- Sección "Resumen ejecutivo" accesible desde el sidebar 
  (no al tope del documento, sino como sección navegable)
- Sección final "Trampas y confusiones frecuentes"

**Estilos:**
- CSS embebido en el mismo archivo, sin dependencias externas
- Compatible con Chrome y Safari sin frameworks
- Fondo claro, tipografía legible para sesiones largas de estudio
- Cada subtema con su propia tarjeta o sección con borde de color diferenciado

**Componentes visuales:**
- Conceptos clave en negrita con highlight de color suave
- Bloques de código con fondo oscuro y fuente monospace
- Analogías GCP/AWS en recuadro con borde izquierdo de color distinto 
  y etiqueta visible "☁️ Analogía Cloud"
- Buenas prácticas en recuadro con etiqueta "💡 Best Practice"
- Preguntas de examen en recuadro con ícono "⚠️ Examen" 
  y fondo diferenciado
- Auditoría de cada subtema en texto pequeño colapsable 
  al final de cada sección

**Resumen ejecutivo:**
- Los 10 conceptos más importantes del tema completo
- Formato de lista con una línea de descripción por concepto
- Navegable desde el sidebar como primera sección

**Requisito técnico:**
- El archivo debe funcionar abriéndolo directamente en el navegador 
  sin servidor local
- No uses frameworks externos (React, Vue, Bootstrap, Tailwind CDN)
- JavaScript solo para colapsar/expandir secciones si es necesario, 
  nada más