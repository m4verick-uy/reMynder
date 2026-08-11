## Spec — Selector de vista unificado (Tablero | Focus | Lista)

**Feature:** Segmented control de 3 vistas + shell de la vista Focus
**Stack afectado:** JS, CSS, HTML — todo en `web/index.html`

**Basado en:** `docs/research-selector-vista.md`, `docs/stories-selector-vista.md`

---

### Cambios en modelo de datos

Ninguno. No hay campos Firestore involucrados — es 100% estado de UI local.

---

### Cambios en estado JS

```js
let viewMode = localStorage.getItem('viewMode') || 'board';   // antes: || 'list'
```

Cambia solo el fallback. El resto del ciclo de vida de `viewMode` es igual
(persistido en `localStorage` bajo la misma key).

No se agrega ningún estado nuevo — no hace falta una variable para
"segmento seleccionado", el segmento activo se deriva de `viewMode` en cada
render, igual que hoy se deriva `isBoard`.

---

### Cambios en lógica JS

**Reemplazar `toggleViewMode()`** (línea 1729) por:

```js
function setViewMode(mode) {
  viewMode = mode;
  localStorage.setItem('viewMode', viewMode);
  updateViewSwitch();
  render();
}
```

**Reemplazar `updateViewBtn()`** (línea 1736) por:

```js
function updateViewSwitch() {
  document.querySelectorAll('.view-switch-btn').forEach(btn => {
    btn.classList.toggle('active', btn.dataset.view === viewMode);
  });
  const isList = viewMode === 'list';
  ['f-all', 'f-pending', 'f-done'].forEach(id => {
    document.getElementById(id).style.display = isList ? '' : 'none';
  });
  document.getElementById('status-sep').style.display = isList ? '' : 'none';
}
```

Nota: el criterio anterior era "ocultar en board" (`isBoard`); pasa a ser
"mostrar solo en list" (`isList`), lo que oculta los filtros de estado
también en Focus — decisión técnica default documentada en el research
(riesgo #3), reversible sin costo cuando llegue la feature de comportamiento
de Focus.

**Extender `render()`** (línea 1447) de if/else a 3 ramas:

```js
const mainEl = document.querySelector('#app main');
document.getElementById('board').style.display      = 'none';
document.getElementById('board-dots').style.display  = 'none';
document.getElementById('task-list').style.display   = 'none';
document.getElementById('focus-view').style.display  = 'none';

if (viewMode === 'board') {
  mainEl.classList.add('board-mode');
  document.getElementById('board').style.display = '';
  document.getElementById('board-dots').style.display = '';
  renderBoard();
} else if (viewMode === 'focus') {
  mainEl.classList.remove('board-mode');
  if (boardDndController) { boardDndController.abort(); boardDndController = null; }
  if (dotsController)     { dotsController.abort();     dotsController     = null; }
  document.getElementById('focus-view').style.display = '';
  renderFocus();
} else {
  mainEl.classList.remove('board-mode');
  if (boardDndController) { boardDndController.abort(); boardDndController = null; }
  if (dotsController)     { dotsController.abort();     dotsController     = null; }
  document.getElementById('task-list').style.display = '';
  renderList();
}
```

(El abort de `boardDndController`/`dotsController` se repite en las ramas
`focus` y `list` porque ambas necesitan garantizar que no queden listeners
del Board colgados — igual que hoy hace la rama `else`.)

**Función nueva — `renderFocus()`** (placeholder, junto a `renderList`/`renderBoard`):

```js
function renderFocus() {
  document.getElementById('focus-view').innerHTML =
    `<div class="empty">el modo focus está en construcción.</div>`;
}
```

Reutiliza la clase `.empty` ya existente (mismo estilo que los estados
vacíos de la lista) — sin CSS nuevo para el mensaje.

**Wiring** (reemplazar las líneas de `view-toggle-btn` al final del script):

```js
document.querySelectorAll('.view-switch-btn').forEach(btn => {
  btn.addEventListener('click', () => setViewMode(btn.dataset.view));
});
updateViewSwitch();
```

---

### Cambios en HTML

**Sacar de `<nav>`** (línea 1076-1077): el `<div class="sep"></div>` y el
`<button ... id="view-toggle-btn">` completos — ninguno de los dos queda.

**Agregar a `<main>`** (junto a `#task-list`/`#board`, línea 1080-1088):

```html
<div id="focus-view" style="display:none"></div>
```

**Agregar a `.user-info`** (línea 1041), antes de `#theme-btn`:

```html
<div class="view-switch" id="view-switch">
  <button class="view-switch-btn" data-view="board">Tablero</button>
  <button class="view-switch-btn" data-view="focus">Focus</button>
  <button class="view-switch-btn" data-view="list">Lista</button>
</div>
```

---

### Cambios en CSS

**Clases nuevas** (ninguna reutiliza `.nav-btn`, por límite explícito del brief):

```css
.view-switch {
  display: flex;
  background: var(--surface);
  border: 0.5px solid var(--border);
  border-radius: 9px;
  padding: 3px;
  gap: 2px;
  flex-shrink: 0;
}

.view-switch-btn {
  border: none;
  background: transparent;
  color: var(--muted);
  font-family: var(--font);
  font-size: 12px;
  font-weight: 400;
  padding: 5px 10px;
  border-radius: 6px;
  cursor: pointer;
  white-space: nowrap;
  transition: background 0.15s, color 0.15s;
}

.view-switch-btn.active {
  background: var(--accent);
  color: var(--accent-contrast);
  font-weight: 500;
}
```

Radio interno (6px) = radio del contenedor (9px) menos el padding (3px),
mismo criterio de anidado que usa iOS en sus segmented controls.

**Mitigación de mobile (riesgo #1 del research):** `.header-top` no tiene
hoy ningún comportamiento responsive. Agregar:

```css
.header-top {
  /* ...existente... */
  flex-wrap: wrap;
  row-gap: 8px;
}

.user-info {
  /* ...existente... */
  flex-wrap: wrap;
  justify-content: flex-end;
}
```

Esto evita overflow horizontal en pantallas angostas (el contenido de
`.user-info` pasa a una segunda línea en vez de desbordar), sin cambiar el
layout en desktop donde todo entra en una sola fila. Es una decisión técnica
del Frontend Builder, no de producto — no requiere confirmación adicional.

---

### Archivos a modificar

- `web/index.html` — único archivo

### Archivos a crear

Ninguno

---

### Riesgos técnicos

- **Verificación visual real de mobile no disponible en este entorno**
  (Google Auth no accesible sin navegador logueado) — mitigación de
  `flex-wrap` es la mejor decisión técnica disponible sin poder confirmar
  visualmente; queda pendiente de prueba manual por el Ingeniero Jefe en el
  preview de `develop`, igual que en features anteriores de esta sesión.
- **Elemento removido con listener adjunto**: si se elimina
  `#view-toggle-btn` del HTML pero queda el
  `document.getElementById('view-toggle-btn').addEventListener(...)` al
  final del script, `getElementById` devuelve `null` y
  `null.addEventListener` tira una excepción que rompe la carga de toda la
  app. Hay que sacar esa línea junto con el resto de la lógica de
  `toggleViewMode`/`updateViewBtn` en el mismo cambio — no se puede hacer en
  dos pasos.

---

### Criterios de done

- [ ] `viewMode` acepta `'board' | 'focus' | 'list'`, default `'board'` sin preferencia guardada
- [ ] `#view-toggle-btn` y toda su lógica asociada (`toggleViewMode`, `updateViewBtn`, listener, sep huérfano) eliminados por completo
- [ ] `.view-switch` visible en el header, junto a tema/ajustes, sin compartir clases con `.nav-btn`
- [ ] Los 3 segmentos cambian la vista correctamente y son mutuamente excluyentes
- [ ] Preferencia de vista persiste entre sesiones vía `localStorage`
- [ ] `#focus-view` existe, se muestra/oculta igual que `#task-list`/`#board`, contenido placeholder honesto sin lógica de negocio
- [ ] Filtros de estado (`f-all`/`f-pending`/`f-done`) y `#status-sep` visibles solo en modo Lista
- [ ] Sin regresión en Tablero ni Lista existentes
- [ ] Sin errores de consola por referencias a elementos removidos
