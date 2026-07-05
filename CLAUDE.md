# CLAUDE.md — El Plan de Úrsula

Contexto para cualquier sesión de Claude (u otra IA) que trabaje en este
repositorio. Léelo entero antes de tocar código: varias reglas de aquí
existen porque ya se rompieron una vez.

## Qué es esto

Una PWA de salud personal para **Úrsula**, hecha por su marido **Francisco**.
Un único archivo `index.html` — sin build, sin dependencias externas, sin
backend. Todo el estado vive en `localStorage` del móvil de Úrsula (ver
`SCHEMA.md` para la estructura exacta). Se despliega en GitHub Pages;
Francisco sube el `index.html` a mano cuando hay una versión nueva.

## Contexto clínico — NO MODIFICAR SIN INDICACIÓN MÉDICA

Esto no es una app de fitness genérica. Úrsula tiene:

- Gastroplastia endoscópica restrictiva (nov 2025) → estómago pequeño,
  necesita comer despacio y con cantidades reales, no genéricas.
- Resistencia a la insulina en mejora (HOMA 13→5) — sin diabetes, sin
  fármacos para la glucosa.
- Artrodesis lumbar L5-S1 → columna neutra siempre. Prohibido: flexión o
  rotación lumbar con carga, impacto, hiperextensión, abdominales clásicos,
  peso muerto.
- Fibromialgia → el descanso por brote NUNCA rompe la cadena de días. Nunca
  presentar el descanso como fallo.
- No sabe nadar → ejercicios de piscina solo de pie, donde hace pie.

**Los menús de `MENUS` en el código salen literalmente del documento del
Dr. Fernández Corbelle.** Si no aparece en ese documento, no se añade —
aunque parezca una combinación obviamente sana (pasó con gazpacho y
lentejas: sus ingredientes están permitidos sueltos, pero el plato como tal
no estaba nombrado, y se quitó). Hay un test de regresión para esto
(`test.js`, sección "cumplimiento estricto") — si lo tocas, es que estás
a punto de romper esta regla.

**Las kcal son siempre orientativas, nunca un límite.** El rango real
(1.350-1.430 kcal/día) lo dio su endocrino. No se debe:
- Inventar un rango distinto sin que Francisco confirme que viene del médico.
- Hacer que el menú se autoajuste para forzar un déficit calórico.
- Añadir lógica de restricción automática de ningún tipo.

Zumos naturales y refrescos light/zero: **excluidos** aunque el documento
del médico los mencione como permitidos — Francisco no ha confirmado si
sustituye a una norma previa más estricta del endocrino. No reintroducir
sin que lo confirme explícitamente.

## Estilo de trabajo esperado

- **Español siempre.**
- **Tono**: positivo, con su nombre, nunca punitivo. El buzón de ideas de la
  app puede llegar pegado como "informe" — hay que implementar lo que pida
  respetando las restricciones de arriba; si algo choca, se propone una
  alternativa seguoya en vez de implementarlo tal cual.
- **Antes de cualquier petición nueva de nutrición o ejercicio**: comprobar
  compatibilidad con el contexto clínico de arriba. Si hay conflicto, decirlo
  claramente y preguntar antes de construir, no decidir en silencio.
- **Nunca** meter datos personales identificables (nombre completo, CIP,
  NHC, resultados de analíticas reales) en el código — el repo es público.

## Arquitectura

- Un solo `<script>` en `index.html`. Sin imports, sin librerías.
- Estado global `S`, cargado de `localStorage` al arrancar, guardado con
  `save()` tras cada cambio.
- Cada campo nuevo de `S` se inicializa con `S.campo = S.campo || default`
  — así una actualización nunca borra datos de una versión anterior. Ver
  `SCHEMA.md` para el porqué y para cuándo hace falta algo más (migración).
- Pictogramas de ejercicio: SVG propios, dibujados a mano en el código
  (función `fig()`), no imágenes externas.
- Vídeos: enlaces a búsqueda de YouTube (`yt: "..."` en cada ejercicio),
  nunca IDs de vídeo concretos — no hay vídeos embebidos ni aprobados por
  nadie del equipo médico/fisio todavía. Si algún día se embeben vídeos
  reales, tienen que venir aprobados por Francisco o el fisio, no elegidos
  por la IA.

## Tests — obligatorio antes de entregar

`test.js` es una suite jsdom (`node test.js`). **Cualquier cambio requiere
pasar la suite al 100% antes de entregar el archivo.** Si el cambio afecta a
comida, hay que revisar en particular la sección de "cumplimiento estricto"
(no debe volver a aparecer queso, frutos secos, hummus, aguacate, chía,
gazpacho, lenteja, garbanzo, crema de, arroz, zumo, refresco).

Al escribir un test nuevo: muchas `const` de nivel superior (`MENUS`,
`PROTEIN_TAGS`, `KCAL_DATA`, variables `let` como `hungerPickLevel`) **no
son accesibles como `window.loQueSea`** en jsdom — son variables de módulo,
no globals. Para probarlas hay que interactuar con el DOM real (clicks en
botones, `dispatchEvent` en selects) en vez de asignarlas directamente desde
el test.

## Entrega

Cada versión: subir número en 3 sitios (comentario de cabecera del HTML,
`<title>`, `const APP_VERSION`) y añadir una línea al historial de
cabecera. El número de versión se ve dentro de la app en dos sitios: la
cabecera (arriba a la derecha) y la pestaña Yo.

Archivos finales van a `/mnt/user-data/outputs/index.html` — es el mismo
contenido que Francisco sube a mano al repo de GitHub Pages. No hay
conector de GitHub disponible en estas sesiones: nunca asumir que se puede
hacer commit directo.
