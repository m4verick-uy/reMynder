# Research — Selector de vista unificado (Tablero | Focus | Lista)

**Feature solicitada:** segmented control de 3 segmentos en el header que
reemplaza el toggle "tablero/lista" actual, agregando una tercera vista
nueva ("Focus") cuyo contenido/comportamiento se definirá en una feature
futura — en esta feature solo se construye el shell de la vista y el
selector que alterna entre las tres.

**Decisión confirmada por el Ingeniero Jefe (bloqueaba el research):**
"Focus" es una vista nueva a construir. Se construye el contenedor/vista
como shell, sin comportamiento — el comportamiento llega en una feature
aparte.

---

## 1. Resumen del problema

Hoy la app tiene dos vistas (`viewMode`: `'list'` | `'board'`) y un toggle
binario de un solo botón (`#view-toggle-btn` en `<nav>`, texto dinámico
"tablero"/"lista"). No hay ninguna vista "Focus" en el código — hay que
construirla desde cero como tercer modo de `viewMode`, aunque sin
comportamiento propio todavía (placeholder).

---

## 2. Archivos relevantes

Todo en `web/index.html`.

**Estado de vista:**
- `let viewMode = localStorage.getItem('viewMode') || 'list';` — línea 1152.
  El default cambia a `'board'` según el brief ("Tablero como default al
  cargar si no hay preferencia guardada")

**Toggle actual a reemplazar por completo:**
- Markup: `<button class="nav-btn" id="view-toggle-btn">tablero</button>` —
  dentro de `<nav>`, línea 1077, precedido por un `<div class="sep"></div>`
  (línea 1076) que separa los filtros de categoría del toggle — ese sep
  queda huérfano si se saca el botón sin sacarlo también
- Lógica: `toggleViewMode()` (línea 1729, toggle binario list↔board) y
  `updateViewBtn()` (línea 1736, actualiza texto/clase del botón y oculta
  `#f-all`/`#f-pending`/`#f-done`/`#status-sep` cuando `viewMode === 'board'`)
- Wiring: `document.getElementById('view-toggle-btn').addEventListener('click', toggleViewMode)` + `updateViewBtn()` al final del script (línea ~2012-2013)

**Dispatch de render por vista** — `render()`, línea 1447: hoy es un
if/else binario (`viewMode === 'board'` vs. todo lo demás = lista). Hay que
convertirlo en un switch de 3 ramas, agregando el manejo de `#board-dots`
(ya existe, solo visible en board) y un contenedor nuevo para Focus.

**Header — dónde va el selector nuevo:**
- `.header-top` (línea 1039, CSS línea 156): `display:flex; justify-content:space-between`, con `h1` a la izquierda y `.user-info` a la derecha
- `.user-info` (línea 1041, CSS línea 172): `display:flex; gap:8px`, contiene
  `#theme-btn`, `#settings-btn`, avatar, nombre, `#logout-btn` — acá es
  donde el brief pide insertar el segmented control ("junto a los iconos de
  tema y ajustes")

**Filtros de categoría — NO tocar, pero coexisten:**
- `<nav>` con `.nav-btn` para filtros de estado (Todas/Pendientes/Completadas)
  y categoría (`renderNavCats()`, línea 1786) — el brief es explícito: el
  nuevo selector "no debe confundirse ni compartir estilos" con estas
  píldoras. Son componentes visualmente distintos y deben seguir siéndolo

**Tokens de color — coinciden exactamente con los ya existentes:**
- Fondo dark del brief (`#1C1C1E`) = `var(--surface)` dark exacto
- Acento activo (`#40C8E0` dark / `#1E7F91` light) = `var(--accent)` exacto
- Texto sobre acento (`#00252B` dark / blanco light) = `var(--accent-contrast)` exacto
- Borde hairline = `var(--border)` (ya documentado como "hairline translúcido")
- El brief no da un hex para el fondo claro ("superficie neutra clara") —
  dado que `var(--surface)` dark coincide exacto con el hex pedido, lo
  consistente es usar `var(--surface)` en ambos temas: en light es blanco
  puro, igual que el header mismo (`header { background: var(--surface) }`)
  — el control queda definido solo por su borde hairline, no por contraste
  de fondo contra el header, mismo criterio en los dos temas

**Radio 9px:** no coincide con ningún token existente (`--radius`: 10px,
`--radius-sm`: 8px). Es un valor explícito del brief (radio de segmented
control iOS real), no un descuido — se usa literal, no se fuerza a un token
existente.

---

## 3. Riesgos o conflictos detectados

1. **Espacio en mobile**: `.header-top` no tiene ningún manejo especial en
   el único breakpoint mobile existente (`@media (max-width: 640px)`, línea
   994, que solo cubre `.board`/`.board-col`/`.board-dots`). `.user-info` ya
   carga avatar (28px) + nombre (hasta 120px, con ellipsis) + botón salir +
   2 íconos. Sumarle un segmented control de 3 palabras sin ajuste de layout
   puede desbordar en pantallas angostas (320–375px). No es parte del brief
   de producto, es un riesgo técnico — el Frontend Builder debe resolverlo
   con `flex-wrap` u ocultando/comprimiendo elementos de `.user-info` en
   mobile, sin que eso cambie el comportamiento pedido.
2. **`#status-sep` huérfano**: al sacar `#view-toggle-btn` de `<nav>`, el
   `<div class="sep"></div>` que lo precedía (línea 1076, distinto del
   `#status-sep` que separa filtros de estado de filtros de categoría) queda
   sin propósito y debe eliminarse junto con el botón.
3. **Filtros de estado (`f-all`/`f-pending`/`f-done`) y Focus**: hoy se
   ocultan solo en modo Board (`updateViewBtn`, línea 1744). No está
   definido si deben verse en modo Focus. Dado que Focus no tiene
   comportamiento aún, lo más seguro es ocultarlos también en Focus (mismo
   criterio que Board: los filtros de estado son propios de la vista Lista)
   — se documenta como decisión técnica default, no bloqueante, reversible
   cuando llegue la feature de comportamiento de Focus.
4. **Nombre de la tercera vista en el modelo (`viewMode`)**: se propone el
   valor `'focus'` (en minúscula, consistente con `'list'`/`'board'`
   existentes) para no romper el patrón de string literals ya usado en
   `localStorage`.

---

## 4. Patrones y convenciones a respetar

- Segmented control es un patrón nuevo en la app — no hay un componente
  previo idéntico para clonar 1:1 (a diferencia de la edición de tareas,
  que sí tenía el precedente de `renderCatForm`). Sí hay precedentes
  parciales a mirar: `.nav-btn.active` (estado activo con acento) y
  `.board-col-header` (uso de `--accent` como color de texto), pero el
  brief pide explícitamente que este componente **no comparta clase** con
  `.nav-btn` para evitar acoplamiento visual accidental (la regla de
  jerarquía visual de `CLAUDE.md` aplica en espíritu: contenedores
  distintos, sin selectores compartidos)
- `render()` sigue el patrón de recalcular todo desde el estado — agregar
  la tercera rama ahí es consistente con lo existente
- `localStorage.setItem('viewMode', viewMode)` ya es el mecanismo de
  persistencia entre sesiones — no hace falta nada nuevo

---

## 5. Recomendaciones para los siguientes agentes

- **Story Writer**: modelar la historia como reemplazo 1:1 del selector +
  historia separada para el shell de Focus (placeholder, sin comportamiento,
  dejar explícito que el comportamiento es una feature futura para no
  generar expectativa de scope creep)
- **Spec Writer**: especificar `setViewMode(mode)` (reemplaza
  `toggleViewMode`), `updateViewSwitch()` (reemplaza `updateViewBtn`),
  tercera rama en `render()` con un nuevo contenedor `#focus-view` y
  `renderFocus()` placeholder. Especificar el manejo de mobile para
  `.header-top`/`.user-info` como parte del spec técnico (riesgo #1)
- **Frontend Builder**: nuevo componente `.view-switch`/`.view-switch-btn`,
  sin heredar de `.nav-btn`. Resolver mobile con `flex-wrap` en
  `.header-top` como mínimo viable
- **Test Verifier**: confirmar que sacar `#view-toggle-btn` no deja
  referencias colgantes (el `addEventListener` sobre un id que ya no
  existe rompe en `null.addEventListener`, hay que sacar esa línea también)
