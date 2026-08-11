# Research — Edición de tareas

**Feature solicitada:** el usuario puede editar una tarea existente (texto y/o
categoría), no solo agregar/completar/eliminar. Cierra la limitación técnica
#2 documentada en `CLAUDE.md` ("Sin edición de tareas").

---

## 1. Resumen del problema

Hoy el modelo de interacción con una tarea es de solo tres operaciones:
crear (`addTask`), alternar estado (`toggleTask`) y eliminar (`deleteTask`).
Si el usuario comete un error de tipeo o quiere recategorizar una tarea, la
única vía es borrarla y crearla de nuevo — perdiendo `createdAt` original y,
en la vista Board, la posición de `status` si ya la había movido.

No hay que diseñar el patrón de interacción desde cero: **ya existe un patrón
de edición inline en la app**, usado para categorías en Ajustes
(`renderCatForm`, línea 1649). Es el precedente más fuerte a seguir para
mantener consistencia visual y de código.

---

## 2. Archivos relevantes

Todo vive en `web/index.html` (único archivo del proyecto, sin excepción).

**Estado de módulo** (línea ~1094-1107):
```js
let tasks           = [];
let editingCatId    = null;   // ← patrón existente a replicar como editingTaskId
```

**Operaciones sobre tareas** (línea ~1201-1246):
- `addTask()` — línea 1202
- `toggleTask(id)` — línea 1218
- `moveTask(id, newStatus)` — línea 1229 (drag & drop en Board)
- `getTaskStatus(t)` — línea 1238
- `deleteTask(id)` — línea 1242

**Patrón de edición inline a replicar — categorías** (línea ~1649-1716):
- `renderCatForm(id, currentName, currentColorIndex)` — línea 1649: genera el
  formulario inline (input + color picker + botones confirmar/cancelar) que
  reemplaza la fila normal cuando `editingCatId === cat.id`
- `renderSettings()` — línea 1675: decide por cada categoría si renderiza la
  fila normal o `renderCatForm(...)`, según `editingCatId`
- `createCategory(name, colorIndex)` / `updateCategory(id, name, colorIndex)`
  — línea 1259 y 1275: validan `name.trim()`, devuelven `{ error: '...' }` en
  vez de tirar excepción, así el caller puede mostrar el error inline sin
  try/catch propio
- Listener delegado en `#categories-list` (línea 1781): switch por
  `data-action` — `edit-cat`, `cancel-cat`, `confirm-cat`, `delete-cat`,
  `select-color`. `confirm-cat` lee el input y el dot seleccionado desde el
  DOM del form, no desde el estado

**Render de tareas — donde hay que enganchar la edición**:
- `renderList()` — línea 1406: vista lista, genera `.task-item` por tarea
  (línea 1421-1436)
- `renderBoard()` — línea 1448: vista Kanban, genera `.board-card` por tarea
  (línea 1471+)
- Listener delegado en `#task-list` (línea 1760): hoy solo maneja `toggle` y
  `delete`
- Listener delegado en `#board` (línea 1543): maneja drag & drop y acciones
  de `.board-card` (`move-prev`, `move-next`, `delete`)

**Selector de categoría reutilizable para el form de edición**:
- `renderCatSelector()` (línea 1596) y `renderCatMenu()` (línea 1616) — el
  mismo selector con dropdown de color/nombre que se usa en `.add-row` para
  elegir categoría al crear una tarea. Se podría reutilizar su markup/CSS
  (`.cat-select-btn`, `.cat-menu`) dentro del form de edición de tarea para
  permitir cambiar de categoría, en vez de inventar un selector nuevo

**Sanitización**: `escHtml()` — línea 1718. Debe aplicarse a `t.text` en
cualquier render nuevo, igual que ya se hace en `renderList`/`renderBoard`.

---

## 3. Hallazgo no documentado en CLAUDE.md

El modelo de datos real de una tarea en Firestore **no coincide** con el
documentado en `CLAUDE.md` (sección "Modelo de datos"): además de
`text`/`done`/`cat`/`createdAt`, cada tarea tiene un campo `status`
(`'todo' | 'doing' | 'done'`, ver `addTask` línea 1209-1215 y `moveTask` línea
1229-1236) que sostiene la vista Board (Kanban). `getTaskStatus()` (línea
1238) hace fallback a partir de `done` para tareas viejas sin `status`. Esto
no es parte del scope de esta feature, pero el Spec Writer debe tenerlo en
cuenta: **editar una tarea no debe tocar `status`**, y el `CLAUDE.md` debería
actualizarse en algún momento para reflejar el campo real (fuera de este
research, es deuda de documentación preexistente).

---

## 4. Patrones y convenciones a respetar

- Un solo archivo (`web/index.html`), sin build — cualquier cambio va ahí
- `async/await` + `try/catch` en toda operación Firestore que pueda fallar
  (ver `updateCategory` como plantilla exacta para `updateTask`)
- Estado en variables de módulo, no en el DOM
- `render()` recalcula toda la UI desde el estado — no hay diffing manual
- Delegación de eventos por contenedor (`#task-list`, `#board`,
  `#categories-list`), switch por `data-action` + `data-id`
- `escHtml()` obligatorio antes de cualquier `innerHTML` con texto de usuario
- CSS: variables existentes (`--border-card`, `--radius-sm`, `--accent`,
  etc.), nunca colores nuevos sin justificación
- Regla de jerarquía visual del `CLAUDE.md`: si el form de edición de tarea
  incluye un botón de acción primaria (confirmar) y uno secundario (cancelar)
  compartiendo contenedor con `.add-row > button` u otro botón de acento,
  usar selectores de hijo directo (`>`), no descendiente

---

## 5. Riesgos o conflictos detectados

1. **Dos vistas a sincronizar**: lista (`renderList`) y Board
   (`renderBoard`) muestran la misma tarea con markup distinto. Si la edición
   se activa desde ambas vistas, hay que decidir si el form inline se ve
   igual en las dos o si el Board abre el form en la vista lista. Esto es una
   decisión de producto — el Story Writer debe marcarla como pregunta abierta
   si el Ingeniero Jefe no lo especificó.
2. **`editingCatId` es global y único** (una sola categoría editable a la
   vez). El mismo patrón para tareas (`editingTaskId`) implica que abrir la
   edición de una tarea nueva debería cerrar la anterior — comportamiento ya
   validado en categorías, replicable sin riesgo.
3. **Recategorizar una tarea completada/en Board**: si se permite cambiar
   `cat` de una tarea, no hay riesgo técnico (no hay validación cruzada de
   categoría↔status), pero si se permite cambiar el texto de una tarea ya
   `done`, hay que decidir si eso tiene sentido de producto (probablemente
   sí, es solo texto).
4. **Concurrencia con Firestore listeners**: como `render()` se dispara en
   cada `onSnapshot`, si el usuario está escribiendo en el input de edición
   y llega un snapshot (ej. otro dispositivo cambió algo), `renderList()`
   destruye y recrea el DOM — el input de edición perdería el texto tipeado.
   Esto ya es un riesgo preexistente idéntico en `renderCatForm` (no es nuevo
   de esta feature), pero vale la pena que el Test Verifier lo chequee
   explícitamente como edge case.
5. **`--border-card` recién introducido** (sesión 2026-08-11, hotfixes
   visuales) ya cubre `.task-item`/`.board-card`/input de nueva tarea — el
   form de edición de tarea debería heredar el mismo token si usa `<input>`
   dentro de la tarjeta, no un valor nuevo.

---

## 6. Recomendaciones para los siguientes agentes

- **Story Writer**: modelar las historias directamente sobre el patrón
  edit-cat existente (mismo verbo de interacción: click en ícono de editar →
  form inline → confirmar/cancelar). Marcar como pregunta abierta si la
  edición debe estar disponible desde el Board o solo desde la lista.
- **Spec Writer**: especificar `updateTask(id, text, cat)` como espejo casi
  literal de `updateCategory(id, name, colorIndex)` (mismo manejo de error,
  mismo `trim()`, mismo return `{ error }`). Especificar `editingTaskId` como
  espejo de `editingCatId`. Reutilizar `.cat-select-btn`/`.cat-menu` para el
  selector de categoría dentro del form, en vez de un componente nuevo.
- **Backend Builder**: no tocar `status` al editar. Reusar `escHtml()`.
- **Frontend Builder**: reusar `--border-card`, `--radius-sm`, `--accent` tal
  cual se usan en `renderCatForm`. Revisar que el form quepa en mobile igual
  que el de categorías (ya validado ahí).
- **Test Verifier**: cubrir explícitamente el edge case de snapshot
  concurrente destruyendo el input de edición (riesgo #4).
