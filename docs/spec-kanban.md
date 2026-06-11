# Spec — Kanban board

## Feature: Kanban board (Por hacer / Haciendo / Hecho)
## Stack afectado: JS, CSS, HTML (solo web/index.html)

---

## Cambios en modelo de datos

- **tasks/{taskId}.status**: nuevo campo `"todo" | "doing" | "done"` — escrito al crear o mover
- **tasks/{taskId}.done**: se mantiene sincronizado con status (backward compat)
- **Lectura con default**: `task.status ?? (task.done ? 'done' : 'todo')` — sin batch migration

---

## Estado JS nuevo

```javascript
let viewMode = localStorage.getItem('viewMode') || 'list'; // 'list' | 'board'
let dragTaskId = null;
```

---

## Funciones JS nuevas o modificadas

- `getTaskStatus(t)` — devuelve status efectivo con default para tareas legacy
- `moveTask(id, newStatus)` — updateDoc: `{ status, done: status === 'done' }`
- `renderBoard()` — renderiza .board con 3 .board-col, cards con DnD
- `renderList()` — lo que hoy es `render()`, sin cambios funcionales
- `render()` — despacha a renderList() o renderBoard() según viewMode
- `toggleViewMode()` — alterna viewMode, persiste en localStorage, llama render()
- `addTask()` — agregar `status: 'todo'` al nuevo documento
- `toggleTask()` — sincronizar `status` junto con `done`

---

## HTML: cambios mínimos

- Botón `#view-toggle-btn` al final del `<nav>` existente
- `<div id="board" style="display:none">` dentro de `<main>`

---

## CSS nuevo

- `.board` — flex container, scroll horizontal en mobile (snap)
- `.board-col` — columna con header + body scrollable, `.drag-over` state
- `.board-card` — card draggable, `.dragging` state
- `.board-dots` — dots indicator, visible solo mobile
- `@media (max-width: 640px)` — snap, botones ← → visibles

---

## Criterios de done

- [ ] Toggle lista/tablero en nav, persiste en localStorage
- [ ] Board: 3 columnas, tareas legacydefaultean a "por hacer"/"hecho"
- [ ] Nuevas tareas se crean con `status: 'todo'`
- [ ] Drag & drop actualiza Firestore en desktop
- [ ] Mobile: scroll snap, dots, botones ← →
- [ ] Filtro de categoría aplica en board mode
- [ ] toggleTask en lista sincroniza status
- [ ] XSS: todo contenido pasa por escHtml()
- [ ] Cero regresiones en lista mode
