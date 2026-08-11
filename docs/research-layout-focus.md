# Research — Layout de la vista Focus

**Feature solicitada:** disposición espacial de la vista Focus (uno de los 3
segmentos del selector ya shippeado): "Por hacer" en fila superior a todo el
ancho, "Haciendo"/"Hecho" en fila inferior 50/50, drag & drop idéntico a
Tablero, responsive con apilado vertical. Solo presentación — no toca modelo
de datos ni las otras vistas.

---

## 1. Resumen del problema

La vista Focus existe hoy como shell vacío (`renderFocus()`, feature
anterior: `docs/spec-selector-vista.md`) — un placeholder sin layout. Esta
feature le da forma real. La buena noticia: **todo lo que pide el brief ya
existe en Tablero** (mismas columnas por `status`, mismo drag & drop, mismas
tarjetas) — Focus no es un componente nuevo, es **el mismo sistema de
columnas de Tablero con otro arreglo espacial** (grid en vez de fila
flex-equal-width) y un ajuste de opacidad en "Hecho".

---

## 2. Archivos relevantes

Todo en `web/index.html`.

**Lo que Focus puede reutilizar tal cual (sin cambios):**
- `COL_ORDER`, `COL_NAMES`, `PREV_STATUS`, `NEXT_STATUS` (línea ~1595-1598)
- `SVG_PREV`/`SVG_NEXT`/`SVG_DEL`/`SVG_EDIT` (línea ~1600-1601)
- `getTaskStatus(t)` (línea 1238)
- Clases CSS: `.board-col`, `.board-col.empty-col`, `.board-col.drag-over`,
  `.board-col-header`, `.board-col-count`, `.board-col-body`, `.board-card`
  y toda su familia (`.board-card-text`, `.board-card-footer`,
  `.board-card-actions`, `.board-card-move-btn`, `.board-card-del-btn`),
  `.board-empty`, `.board-empty-filter`
- **`initBoardDnd(container)`** (línea ~1655): ya recibe el contenedor como
  parámetro — es 100% reutilizable llamándolo con el contenedor de Focus en
  vez de `#board`. El brief pide "drag & drop idéntico a Tablero, mismo
  resaltado de destino" — esto se cumple gratis, es literalmente la misma
  función, no hay que reimplementar nada de drag & drop.

**Lo que genera las columnas hoy — `renderBoard()`** (línea ~1665): calcula
`cols` agrupando `visible` por status, itera `COL_ORDER`, arma el HTML de
cada `.board-col` (header + cards) y el mensaje de columna vacía. Esta
lógica es idéntica a lo que Focus necesita — **candidato a extraer a una
función compartida** `renderBoardColumnsHtml(visible)` usada por
`renderBoard()` y la nueva `renderFocus()`, para no duplicar el template de
tarjeta/columna (evita que Tablero y Focus diverjan visualmente por un
copy-paste desprolijo).

**Lo que NO se reutiliza (Focus no lo necesita):**
- `initMobileDots(board)` / `#board-dots` / `.board-dot`: son el mecanismo
  de swipe horizontal con dots de Tablero en mobile. El brief de Focus pide
  apilado **vertical** en mobile, no un carrusel — Focus no debe llamar
  `initMobileDots` ni mostrar `#board-dots` (que además es un único elemento
  de `id` fijo, ya compartido/oculto correctamente para Focus desde la
  feature anterior).
- `.board.dragging-active .board-col { flex-grow: 1 !important; }`: existe
  para igualar el ancho de las 3 columnas en el layout **flex** de Tablero
  durante el arrastre. Focus usa un grid de tamaños fijos (todo 100%,
  doing/done 50/50) — no hay "ancho dominado por contenido" que igualar,
  esta regla no aplica y no hace falta un equivalente.

---

## 3. Conflicto real encontrado — selector CSS mobile sin scope

`@media (max-width: 640px) { .board-col { width: calc(100vw - 3rem); ... } }`
(línea ~1040) usa el selector **bare** `.board-col`, no `.board .board-col`.
Si Focus reutiliza la clase `.board-col` (como se propone arriba) dentro de
su propio contenedor, esta regla existente se filtraría también a Focus en
mobile, forzando el ancho/scroll-snap del carrusel de Tablero encima del
grid vertical que pide el brief — layout roto en mobile.

**Esto es exactamente el mismo tipo de bug de scope que ya documenta
`CLAUDE.md`** (regla de jerarquía visual, sección CSS): un selector sin
calificar captura descendientes en contextos no previstos. La corrección es
acotar esa regla (y las de `.board-col:first-child`/`:last-child`) a
`.board .board-col`, sin cambiar nada del comportamiento actual de Tablero
(`.board-col` solo existe hoy dentro de `.board`, así que acotar el selector
no cambia ni un píxel de lo existente) — y deja a Focus libre de definir su
propia regla mobile sin colisión.

---

## 4. Layout propuesto (para el Spec Writer)

- Nuevo contenedor `#focus-view` (ya existe, hoy vacío) con una nueva clase
  `.focus-board` (no `.board`, para no heredar el `display:flex` de fila
  horizontal de Tablero)
- CSS Grid: `grid-template-areas: "todo todo" "doing done"` en desktop,
  `"todo" "doing" "done"` apilado en mobile (mismo breakpoint 640px que ya
  usa Tablero, por consistencia)
- `[data-col-status="done"] .board-card { opacity: 0.7 }`, scopeado bajo
  `.focus-board` para no afectar la columna "Hecho" de Tablero
- `main.board-mode` (clase ya existente, línea 441: `max-width:none;
  padding:0`) debe aplicarse también al entrar a Focus — mismo motivo que en
  Tablero: el contenedor hijo (`.focus-board`) maneja su propio padding y
  centrado a 1200px, `main` no debe competir con eso. No hace falta una
  clase nueva, se reutiliza `board-mode` tal cual (aunque el nombre ya no
  sea 100% literal, renombrarla es un refactor no pedido y fuera de alcance)
- Agregar `.focus-board` a la media query `@media (min-width: 1200px)` que
  ya centra `.add-row, nav, .board` — mismo criterio, mismo ancho máximo,
  para cumplir "contenedor centrado 1200px" en Focus

---

## 5. Riesgos o conflictos detectados

1. **Selector `.board-col` sin scope en mobile** — descripto arriba, hay que
   corregirlo como parte de esta feature (no es opcional, sin la corrección
   el layout mobile de Focus queda roto por una regla de Tablero).
2. **`boardDndController` es una sola variable global** compartida entre
   Tablero y Focus (igual que ya pasa con Lista/`editingTaskId` u otros
   estados globales de esta app). Como las tres vistas son mutuamente
   excluyentes (nunca hay dos visibles a la vez), reusar la misma variable
   para "quien sea que esté usando `initBoardDnd` ahora" es seguro siempre
   que cada `render*()` aborte el controller anterior al entrar — patrón ya
   existente, solo hay que mantenerlo consistente en `renderFocus()`.
3. **Edición de tareas dentro de Focus**: como Focus reutiliza el mismo
   template de `.board-card`, la edición inline (`editingTaskId`,
   `renderTaskForm`, feature de sesión anterior) funciona gratis si se
   reusa el mismo template — no hay que tocar esa lógica para que funcione
   en Focus también. No estaba pedido explícitamente en el brief, pero es
   una consecuencia natural de reusar el componente tal cual ("hereda todo
   el sistema actual sin excepción").

---

## 6. Recomendaciones para los siguientes agentes

- **Story Writer**: no hay preguntas abiertas de producto — el brief es
  prescriptivo y coincide con lo ya construido en Tablero. Documentar
  igualmente el criterio de "edición inline funciona en Focus por herencia
  del componente" para que quede explícito, no implícito.
- **Spec Writer**: especificar `renderBoardColumnsHtml(visible)` compartida,
  `renderFocus()` nueva (mismo patrón que `renderBoard()` minus dots), la
  corrección de scope del selector mobile, y el agregado de `.focus-board`
  a los dos media queries relevantes (1200px centrado, 640px — con su propio
  breakpoint de apilado, no el de Tablero).
- **Test Verifier**: confirmar explícitamente que la corrección de scope del
  selector mobile no cambió el comportamiento de Tablero (revisar que
  `.board-col` sigue viviendo solo dentro de `.board` en el markup final).
