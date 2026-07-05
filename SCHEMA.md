# Estructura de datos — El Plan de Úrsula

Todo el estado de la app vive en **una sola clave de `localStorage`**:

```
localStorage["planUrsulaV2"]  →  JSON.stringify(S)
```

`S` es un único objeto JavaScript. No hay servidor, no hay base de datos: todo
lo que ves aquí es exactamente lo que hay guardado en el móvil de Úrsula.

También existe una clave auxiliar, **no editable por la app**, que solo sirve
para detectar actualizaciones y disparar la copia de seguridad automática:

```
localStorage["planUrsulaV2_lastVersion"]  →  "5.8"  (string con la última APP_VERSION vista)
```

---

## Convención de compatibilidad (léelo antes de cambiar nada)

Cada campo de `S` se inicializa así, en el arranque de la app:

```js
S.campo = S.campo || valorPorDefecto;
```

Esto es lo que hace que **actualizar la app nunca borre datos**: si un campo
nuevo no existe todavía (viene de una versión anterior), se crea vacío; si ya
existe, se respeta tal cual.

**Regla de oro**: añadir un campo nuevo es siempre seguro. Lo que NO es seguro
sin pensarlo es **cambiar la forma** de un campo que ya existe (por ejemplo,
pasar `S.steps` de array a objeto, o cambiar qué representan las claves de un
diccionario). Si алguna vez hace falta, se escribe un paso de migración dentro
de `function migrate()` (busca ese nombre en el código), protegido por
`S.schemaVersion`, para que los datos viejos se transformen solos al formato
nuevo la primera vez que se abra la app tras la actualización.

---

## Campos de `S`

### Identidad / control de versión
| Campo | Tipo | Descripción |
|---|---|---|
| `schemaVersion` | number | Versión de la FORMA de los datos (no de la app). Solo sube cuando `migrate()` transforma algo. |

### Ejercicio
| Campo | Tipo | Descripción |
|---|---|---|
| `days` | `{ "YYYY-MM-DD": "full"\|"mini"\|"rest" }` | Qué se hizo cada día: sesión completa, mínima (regla de los 5 min) o descanso por brote. |
| `acts` | `{ "YYYY-MM-DD": "gym"\|"pool"\|"home"\|"walk" }` | Qué actividad se eligió ese día. |
| `steps` | `{ "YYYY-MM-DD\|tipo": [bool, bool, ...] }` | Qué pasos de la sesión de ese día están marcados como hechos. **Indexado por posición** en el array que genera `getSteps()` — ver limitación más abajo. |
| `levels` | `{ "id_ejercicio": "A"\|"B"\|"C" }` | Nivel actual de cada ejercicio con id (ej. `g_prensa`). Si no aparece, se asume `"A"`. |
| `checkins` | `{ "YYYY-MM-DD\|tipo": {...} }` | Resumen del check-in de fin de sesión. |
| `checkinAnswered` | `{ "YYYY-MM-DD\|id_ejercicio": "ok"\|"pain" }` | Qué se respondió en el feedback inmediato tras cada tanda. Con esto se evita volver a preguntar el mismo día. |
| `painStreak` | `{ "id_ejercicio": number }` | Cuántas veces seguidas se ha respondido "bien" en un nivel no-A (para sugerir subir de nivel). |

### Comida
| Campo | Tipo | Descripción |
|---|---|---|
| `menu` | objeto | **Legacy** (de antes del menú semanal). Ya no se usa para generar nada nuevo; se deja por compatibilidad. |
| `menuDays` | `{ "YYYY-MM-DD": true }` | Días en los que se marcó "Hoy comí según el plan" (+5 puntos). |
| `weekMenu` | `{ "YYYY-MM-DD": { "Desayuno": "nombre del plato", ... } }` | El menú real de cada día, una entrada por franja (las franjas son las claves de `MENUS`). |
| `weekMenuQty` | `{ "YYYY-MM-DD": { "Desayuno": 150, ... } }` | Cantidad elegida (en gramos o unidades, según el alimento) para cada franja de cada día. Si no hay entrada, se usa la cantidad por defecto de ese alimento. |
| `weekPrefs` | `["pescado", "jamon", ...]` | Etiquetas de preferencia marcadas para el generador semanal (ver `PREF_TAGS`). |
| `mealDone` | `{ "YYYY-MM-DD": { "Desayuno": true } }` | Registro ligero de "ya comí esto" por franja (distinto de `menuDays`, que es el resumen del día completo). |
| `customFoods` | `{ "Desayuno": [{name, cat}, ...] }` | Alimentos que Úrsula ha añadido ella misma, por franja. `cat` referencia una entrada de `CUSTOM_CATS` (para saber la kcal orientativa y si cuenta como proteína/jamón/pescado/huevo a efectos del generador). |
| `shop` | `{ "Categoría\|artículo": true }` | Qué está marcado como ya comprado en la lista de la compra. |
| `hungerLog` | `[{ d, t, level, tags }]` | Registros del medidor de hambre. `d`=fecha, `t`=hora "HH:MM", `level`=1-5, `tags`=causas marcadas. |

### Cuerpo / salud
| Campo | Tipo | Descripción |
|---|---|---|
| `waist` | `[{ d, v }]` | Historial de cintura (cm). |
| `weight` | `[{ d, v }]` | Historial de peso (kg). |
| `bodyComp` | `[{ d, fat?, water?, muscle?, bmi? }]` | Composición corporal de la báscula. Todos los campos son opcionales — se guarda solo lo que se rellene. |
| `feels` | `[{ d, v }]` | Registro de "¿cómo te sientes? (0-10)". |
| `mealPhotos` | `[{ d, data (base64) }]` | Diario de fotos de comida. **Ojo**: crece el tamaño de `localStorage`; si algún día da problemas de espacio, es el primer sitio a mirar. |

### Progreso / gamificación
| Campo | Tipo | Descripción |
|---|---|---|
| `pts` | number | Puntos acumulados. |
| `badges` | `{ "id_badge": true }` | Insignias conseguidas. |
| `wishes` | `[{ texto, fecha }]` | Buzón de ideas. |
| `pick` | objeto interno | Estado de UI del selector de actividad (no crítico). |

### Preferencias de la app
| Campo | Tipo | Descripción |
|---|---|---|
| `bigText` | boolean | Modo de texto grande activado. |
| `reminderDismiss` | `{ "YYYY-MM-DD": true }` | Días en los que se cerró el aviso de "no has entrado hoy". |

---

## Datos de referencia (NO están en `S`, viven en el código)

Estos son constantes definidas en el `<script>`, no datos de usuario. Cambiar
sus valores es una decisión de contenido (y, en el caso de comida, una
decisión médica), no una migración de datos:

- **`EXOS`** — catálogo de ejercicios por tipo (`gym`/`pool`/`home`), con sus
  niveles A/B/C, pictograma, y texto de búsqueda de YouTube.
- **`MENUS`** — platos disponibles por franja. **Todo lo que hay aquí debe
  salir literalmente del documento del Dr. Fernández Corbelle** (ver el test
  de regresión "cumplimiento estricto" en `test.js`).
- **`KCAL_DATA`** — kcal por 100g (o por unidad) de cada plato de `MENUS`,
  más cantidad por defecto y paso del stepper.
- **`CUSTOM_CATS`** — categorías disponibles para que Úrsula añada alimentos
  propios, con su kcal orientativa por 100g/unidad.
- **`PROTEIN_TAGS`** / **`PREF_TAGS`** — qué plato de proteína corresponde a
  qué etiqueta de "apetece" (pescado, jamón, marisco, huevo, carne).
- **`SHOP`** — lista de la compra por categoría (independiente de `MENUS`).
- **`HUNGER_LEVELS`** / **`HUNGER_TAGS`** — opciones fijas del medidor de
  hambre.

---

## Limitación conocida (no corregida, documentada a propósito)

`S.steps` guarda qué pasos de la sesión de hoy están marcados **por
posición** en el array (`stepState[i]`), no por contenido. Si cambia el
número de pasos de un ejercicio (por ejemplo, al subir/bajar de nivel a
mitad de sesión), el código actual **borra los checks de todo el día** para
evitar que un check se desplace al paso equivocado (ver `cycleLevel()`).
Funciona, pero es fráfil: cualquier cambio futuro que reordene o añada
ejercicios a mitad de sesión debe tener en cuenta este mismo riesgo.

---

## Copia de seguridad

- **Manual**: botón "Descargar copia" en la pestaña Yo → genera un `.json`
  con el contenido completo de `S`.
- **Automática**: al detectar que `APP_VERSION` cambió desde la última vez
  que se abrió la app, se descarga sola un `.json` de respaldo antes de
  continuar (ver `autoBackupOnUpdate()`).
- **Importar**: en Yo, subir un `.json` de copia sustituye `S` por completo
  (pide confirmación antes).
