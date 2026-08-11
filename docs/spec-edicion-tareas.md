## Spec — Edición de tareas

**Feature:** Edición de tareas (texto y categoría), desde vista Lista y Board
**Stack afectado:** JS, CSS, HTML — todo dentro de `web/index.html`

**Basado en:** `docs/research-edicion-tareas.md`, `docs/stories-edicion-tareas.md`

---

### Cambios en modelo de datos

Ninguno. Se reutilizan los campos existentes de `tasks/{taskId}`:
- `text` — se sobreescribe con el nuevo valor (trimmed)
- `cat` — se sobreescribe con la nueva categoría elegida
- `done`, `status`, `createdAt` — **no se tocan** al editar

---

### Cambios en estado JS (módulo)

```js
let editingTaskId = null;   // id de la tarea en edición, o null. Espejo exacto de editingCatId.
```

Al hacer logout (`onAuthStateChanged`, rama `else`, línea ~1128-1139), agregar
`editingTaskId = null;` junto al reset de `editingCatId`.

---

### Cambios en lógica JS

**Función nueva — `updateTask(id, text, cat)`** (junto a las demás operaciones
de tarea, después de `deleteTask`):

```js
async function updateTask(id, text, cat) {
  const trimmed = text.trim();
  if (!trimmed) return { error: 'El texto no puede estar vacío.' };
  const user = auth.currentUser;
  if (!user) return;
  try {
    await updateDoc(doc(db, 'users', user.uid, 'tasks', id), {
      text: trimmed, cat
    });
    editingTaskId = null;
  } catch (err) {
    console.error('updateTask error:', err);
    return { error: 'Error al actualizar la tarea.' };
  }
}
```

Espejo exacto de `updateCategory` (línea 1275) — mismo contrato de retorno
(`{ error }` o `undefined`), mismo try/catch, mismo `console.error`.

**Función de render nuevo — `renderTaskForm(id, currentText, currentCat)`**
(junto a `renderCatForm`, o cerca de `renderList`/`renderBoard`):

Genera el markup del form inline. Reutiliza el patrón de `renderCatForm`:
input de texto (con el valor actual) + selector de categoría + botones
confirmar/cancelar (`.icon-btn`, igual que en categorías).

Selector de categoría dentro del form: **no reutilizar el dropdown
`#cat-select-btn`/`#cat-menu`** (es un singleton con id fijo y estado propio
`catMenuOpen`, pensado para una sola instancia en `.add-row`). En su lugar,
reutilizar la clase visual `.cat-menu-item` (dot + nombre, ya estilada) pero
en una lista siempre visible (no dropdown), scoped por `data-form-id` igual
que `.color-picker` en `renderCatForm`:

```js
function renderCatPicker(formId, currentCatId) {
  return categories.map(cat => `
    <button class="cat-menu-item${cat.id === currentCatId ? ' selected' : ''}"
            data-action="select-task-cat" data-cat="${cat.id}" data-form-id="${formId}">
      <span class="cat-menu-dot cat-p${cat.colorIndex}"></span>
      <span class="cat-menu-name">${escHtml(cat.name)}</span>
    </button>
  `).join('');
}
```

(Verificar en Frontend Builder si `.cat-menu-item` ya trae un dot propio via
`::before` o si hace falta agregar `.cat-menu-dot` — revisar `renderCatMenu()`
línea 1616 para el markup exacto que ya funciona y clonarlo.)

**Modificar `renderList()`** (línea 1406): en el `.map()` de tareas visibles,
si `t.id === editingTaskId`, renderizar `renderTaskForm(t.id, t.text, t.cat)`
en vez del `.task-item` normal (mismo patrón que `renderSettings()` hace con
`editingCatId` en la línea 1684).

**Modificar `renderBoard()`** (línea 1448, dentro del map de cards por
columna): mismo swap — si `t.id === editingTaskId`, renderizar el form en vez
de `.board-card`. El form ocupa el lugar de la tarjeta en su columna actual,
sin mover `status`.

**Agregar ícono de editar:**
- `.task-item` (línea 1421-1436): nuevo `<button class="icon-btn" data-action="edit-task" data-id="${t.id}">` junto al `.del-btn` existente (mismo ícono de lápiz que usa `edit-cat`, línea 1692-1697)
- `.board-card` (línea 1471+): mismo botón, junto a `.board-card-move-btn`/`.board-card-del-btn`

**Listener `#task-list`** (línea 1760) — agregar casos:
```js
if (action === 'edit-task')   { editingTaskId = id; render(); /* focus input */ }
if (action === 'cancel-task') { editingTaskId = null; render(); }
if (action === 'select-task-cat') { /* togglear .selected en el picker, igual que select-color */ }
if (action === 'confirm-task') { /* leer input + .cat-menu-item.selected del form, llamar updateTask, manejar error igual que confirm-cat */ }
```

**Listener `#board`** (línea 1543) — agregar los mismos cuatro casos
(`edit-task`, `cancel-task`, `select-task-cat`, `confirm-task`), con cuidado
de no interferir con el listener de drag & drop existente (los botones ya
usan `e.stopPropagation()` en patrones similares como `move-prev`/`move-next`
— verificar y replicar si hace falta para que el drag no se dispare al tocar
el ícono de editar).

**Reusar `render()`** como disparador global después de cambiar
`editingTaskId` — ya recalcula tanto lista como board según `viewMode`, no
hace falta invalidar nada manualmente.

---

### Cambios en UI / CSS

Ningún token de color nuevo. Reutilizar:
- `--border-card`, `--radius-sm`, `--accent`, `--fill` — igual que
  `.cat-row-form`/`.cat-form-input` (línea 689-716)
- `.icon-btn` (línea 651) para los botones de editar/confirmar/cancelar —
  clase ya genérica, sin scope a Ajustes
- `.cat-menu-item` (línea 320) para las opciones de categoría del picker

**Clases nuevas a definir** (Frontend Builder, siguiendo el mismo criterio
visual que `.cat-row-form`):
- `.task-item-form` — contenedor del form inline en la lista (mismo
  padding/radius/border que `.task-item`, pero layout en columna para
  acomodar input + picker + acciones)
- `.board-card-form` — equivalente dentro de una columna del Board (más
  angosto, verificar que el picker de categorías no rompa el layout en
  mobile — columnas del Board ya son angostas, ver media query línea 949)
- `.task-form-input` — puede reusar exactamente el estilo de
  `.cat-form-input` (mismo `border-color: var(--border-card)`) sin
  duplicar reglas si se le da la misma clase, o extenderla

**Regla de jerarquía visual (CLAUDE.md):** si el form de tarea comparte
contenedor con algún botón de acento existente, usar selectores de hijo
directo (`>`) para los botones de acción del form — no depender de
selectores descendientes que puedan filtrarse desde `.add-row` u otro
contenedor de acento.

---

### Archivos a modificar

- `web/index.html` — único archivo, todos los cambios de arriba (CSS + HTML
  templates + JS)

### Archivos a crear

Ninguno.

---

### Riesgos técnicos

- **Snapshot concurrente durante edición** (riesgo #4 del research): como
  `onSnapshot` dispara `render()` en cada cambio remoto, si llega un update
  mientras `editingTaskId` está seteado, el form se re-renderiza desde
  `t.text`/`t.cat` del snapshot nuevo, perdiendo lo que el usuario tenía
  tipeado sin confirmar. **Mitigación propuesta:** aceptar el comportamiento
  tal cual (es preexistente e idéntico en `renderCatForm`, no se resuelve acá)
  — documentar como limitación conocida, no como bug de esta feature. Si el
  Test Verifier lo marca como crítico, hay que volver al Spec Writer para
  decidir una mitigación (ej. no re-renderizar el form activo ante snapshots
  remotos), pero eso es scope adicional no pedido por el Ingeniero Jefe.
- **Drag & drop vs. click en ícono de editar en Board**: `.board-card` es
  `draggable="true"`. Hay que confirmar que tocar el ícono de editar no
  dispare un `dragstart`. El patrón ya existe para `.board-card-del-btn`/
  `.board-card-move-btn` — replicar exactamente su manejo de eventos.
- **Duplicación de markup del picker de categorías**: hay ahora tres lugares
  que listan categorías con dot+nombre (`renderCatMenu`, `renderCatPicker`
  nuevo, `renderNavCats`). No se propone unificarlos en esta feature (fuera
  de scope, sería un refactor), pero queda como candidato futuro.

---

### Criterios de done

- [ ] `updateTask(id, text, cat)` implementado y no afecta `done`/`status`/`createdAt`
- [ ] Ícono de editar visible en `.task-item` y `.board-card`
- [ ] Form inline funcional en ambas vistas: precarga texto y categoría actuales
- [ ] Confirmar guarda y cierra el form; cancelar descarta y cierra
- [ ] Validación de texto vacío con mensaje inline (mismo patrón que categorías)
- [ ] Solo una tarea editable a la vez (`editingTaskId` único, igual que `editingCatId`)
- [ ] `editingTaskId` se resetea en logout
- [ ] `escHtml()` aplicado a todo texto de usuario en el form y su render
- [ ] Sin regresión en `addTask`, `toggleTask`, `deleteTask`, `moveTask`, drag & drop del Board
- [ ] Funciona en mobile (lista y Board, incluyendo columnas angostas)
