## Spec — Layout de la vista Focus

**Feature:** Disposición espacial de Focus (backlog arriba full-width,
haciendo/hecho 50/50 abajo) + drag & drop idéntico a Tablero + apilado
responsive
**Stack afectado:** JS, CSS — todo en `web/index.html`

**Basado en:** `docs/research-layout-focus.md`, `docs/stories-layout-focus.md`

---

### Cambios en modelo de datos

Ninguno.

---

### Cambios en estado JS

Ninguno nuevo. Se reutiliza `boardDndController` (ya existente) para
trackear los listeners de drag & drop de Focus, con el mismo criterio de
abort-on-entry que ya usan `renderBoard()`/`render()`.

---

### Cambios en lógica JS

**1. Extraer función compartida `renderBoardColumnsHtml(visible)`** — el
cuerpo exacto que hoy arma las columnas dentro de `renderBoard()` (cálculo
de `cols`, iteración de `COL_ORDER`, HTML de cada `.board-col` con header +
cards), sin cambios de comportamiento, solo movido a una función propia:

```js
function renderBoardColumnsHtml(visible) {
  const cols = { todo: [], doing: [], done: [] };
  visible.forEach(t => cols[getTaskStatus(t)].push(t));

  return COL_ORDER.map(status => {
    const items   = cols[status];
    const isEmpty = items.length === 0;
    const cards = items.length
      ? items.map(t => t.id === editingTaskId ? renderTaskForm(t.id, t.text, t.cat) : `
          <div class="board-card" draggable="true" data-id="${t.id}" data-status="${status}">
            <span class="board-card-text">${escHtml(t.text)}</span>
            <div class="board-card-footer">
              ${getCatBadge(t.cat)}
              <div class="board-card-actions">
                <button class="board-card-move-btn" data-action="move-prev" data-id="${t.id}" data-status="${status}"
                        aria-label="Mover atrás" ${!PREV_STATUS[status] ? 'disabled' : ''}>${SVG_PREV}</button>
                <button class="board-card-move-btn" data-action="move-next" data-id="${t.id}" data-status="${status}"
                        aria-label="Mover adelante" ${!NEXT_STATUS[status] ? 'disabled' : ''}>${SVG_NEXT}</button>
                <button class="board-card-move-btn" data-action="edit-task" data-id="${t.id}" aria-label="Editar">${SVG_EDIT}</button>
                <button class="board-card-del-btn" data-action="delete" data-id="${t.id}" aria-label="Eliminar">${SVG_DEL}</button>
              </div>
            </div>
          </div>
        `).join('')
      : `<div class="board-empty">soltá acá</div>`;
    return `
      <div class="board-col${isEmpty ? ' empty-col' : ''}" data-col-status="${status}" style="flex-grow:${isEmpty ? 1 : 3}">
        <div class="board-col-header">
          <span>${COL_NAMES[status]}</span>
          <span class="board-col-count">${items.length}</span>
        </div>
        <div class="board-col-body">${cards}</div>
      </div>`;
  }).join('');
}
```

(El `style="flex-grow:..."` inline queda tal cual — es inocuo dentro del
grid de Focus, un item de grid ignora `flex-grow`. No vale la pena una rama
condicional solo para omitirlo.)

**2. `renderBoard()` pasa a usar la función compartida:**

```js
function renderBoard() {
  if (boardDndController) boardDndController.abort();
  if (dotsController)     dotsController.abort();

  const visible = tasks.filter(t => catFilter ? t.cat === catFilter : true);
  const board = document.getElementById('board');

  if (!visible.length) {
    const catName = catFilter ? categories.find(c => c.id === catFilter)?.name : null;
    board.innerHTML = `<div class="board-empty-filter">${catName ? `Sin tareas en ${escHtml(catName)}` : 'Sin tareas'}</div>`;
    document.getElementById('board-dots').style.display = 'none';
    return;
  }

  board.innerHTML = renderBoardColumnsHtml(visible);
  initBoardDnd(board);
  initMobileDots(board);
}
```

**3. `renderFocus()` reemplaza el placeholder actual, mismo patrón sin dots:**

```js
function renderFocus() {
  if (boardDndController) boardDndController.abort();

  const visible = tasks.filter(t => catFilter ? t.cat === catFilter : true);
  const focusEl = document.getElementById('focus-view');

  if (!visible.length) {
    const catName = catFilter ? categories.find(c => c.id === catFilter)?.name : null;
    focusEl.innerHTML = `<div class="board-empty-filter">${catName ? `Sin tareas en ${escHtml(catName)}` : 'Sin tareas'}</div>`;
    return;
  }

  focusEl.innerHTML = renderBoardColumnsHtml(visible);
  initBoardDnd(focusEl);
}
```

No se llama `initMobileDots` — Focus no usa el carrusel de dots, se apila
por CSS puro (ver sección CSS).

**4. `render()` — aplicar `board-mode` también en la rama `focus`**, mismo
motivo que Tablero (el hijo maneja su propio padding/centrado, `main` no
debe competir). La rama `focus` ya llama `renderFocus()` desde la feature
anterior; el único cambio es agregar `mainEl.classList.add('board-mode')`
ahí en vez de `remove`:

```js
} else if (viewMode === 'focus') {
  mainEl.classList.add('board-mode');
  document.getElementById('focus-view').style.display = '';
  renderFocus();
}
```

(El abort de `boardDndController`/`dotsController` que hoy vive en esta
rama de `render()` se saca de acá — `renderFocus()` ya lo hace por su
cuenta, mismo criterio que `renderBoard()`. Evita el doble abort
redundante.)

---

### Cambios en CSS

**1. Corrección de scope — obligatoria antes de agregar el layout de Focus**
(sin esto, la regla mobile de Tablero se filtra a Focus, ver research):

```css
/* Antes: */
.board-col { flex-shrink: 0; width: calc(100vw - 3rem); ... }
.board-col:first-child { margin-left: 1.5rem; }
.board-col:last-child  { margin-right: 1.5rem; }

/* Después: */
.board .board-col { flex-shrink: 0; width: calc(100vw - 3rem); ... }
.board .board-col:first-child { margin-left: 1.5rem; }
.board .board-col:last-child  { margin-right: 1.5rem; }
```

Cero cambio de comportamiento en Tablero (`.board-col` solo vive dentro de
`.board` hoy). Deja a Focus libre de su propia regla mobile.

**2. Layout de Focus:**

```css
.focus-board {
  display: grid;
  grid-template-columns: 1fr 1fr;
  grid-template-areas:
    "todo todo"
    "doing done";
  gap: 16px;
  padding: 1.5rem 1.5rem 5rem;
  min-height: calc(100vh - 180px);
}

.focus-board [data-col-status="todo"]  { grid-area: todo; }
.focus-board [data-col-status="doing"] { grid-area: doing; }
.focus-board [data-col-status="done"]  { grid-area: done; }

/* Hecho con menos peso visual, sin ocultarse — solo en Focus, no en Tablero */
.focus-board [data-col-status="done"] .board-card { opacity: 0.7; }

@media (max-width: 640px) {
  .focus-board {
    grid-template-columns: 1fr;
    grid-template-areas:
      "todo"
      "doing"
      "done";
  }
}
```

**3. Contenedor centrado 1200px** — agregar `.focus-board` a la media query
existente:

```css
@media (min-width: 1200px) {
  .add-row,
  nav,
  .board,
  .focus-board {
    max-width: 1200px;
    margin-left: auto;
    margin-right: auto;
  }
}
```

**4. Markup** — `#focus-view` (ya existe) necesita la clase `.focus-board`
en el HTML estático:

```html
<div id="focus-view" class="focus-board" style="display:none"></div>
```

(La clase va fija en el HTML, no se agrega/saca por JS — solo el
`display` cambia entre `render()`, igual que hoy.)

---

### Archivos a modificar

- `web/index.html` — único archivo

### Archivos a crear

Ninguno

---

### Riesgos técnicos

- **Verificación visual en navegador real no disponible** (sin sesión
  logueada) — mismo caveat de toda la sesión. La corrección de scope del
  selector mobile es la mitigación técnica más segura disponible sin poder
  confirmar visualmente; queda pendiente de prueba manual.
- **Extracción de `renderBoardColumnsHtml`**: es un refactor puro del
  cuerpo de `renderBoard()` — hay que verificar con cuidado que el HTML
  resultante sea carácter por carácter idéntico al actual antes del cambio,
  para no introducir una regresión sutil en Tablero al mismo tiempo que se
  construye Focus.

---

### Criterios de done

- [ ] `renderBoardColumnsHtml(visible)` extraída y usada por `renderBoard()` y `renderFocus()`, sin cambio de comportamiento en Tablero
- [ ] Selector mobile de `.board-col` scopeado a `.board .board-col`, Tablero sin regresión
- [ ] `.focus-board`: grid `todo` full-width arriba, `doing`/`done` 50/50 abajo, apilado 1 columna en mobile
- [ ] `.focus-board [data-col-status="done"] .board-card { opacity: 0.7 }`, no afecta a Tablero
- [ ] `.focus-board` centrado a 1200px en desktop ancho
- [ ] Drag & drop en Focus vía `initBoardDnd(focusEl)`, sin reimplementación
- [ ] `initMobileDots` NO se llama para Focus, `#board-dots` no aparece en Focus
- [ ] `main.board-mode` aplicado también en modo Focus
- [ ] Edición inline de tareas funciona en Focus (heredada del componente, sin código nuevo)
- [ ] Sin cambios visibles en Tablero ni Lista
- [ ] Sin errores de consola, `node --check` sobre el módulo JS sin errores
