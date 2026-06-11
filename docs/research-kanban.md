# Research — Kanban board (Por hacer / Haciendo / Hecho)

## Resumen del problema

El usuario quiere un tablero estilo JIRA con 3 columnas que convive
con la vista de lista actual (toggle). Las tareas se mueven con drag & drop.

## Archivos relevantes

- `web/index.html` — único archivo: toda la app (HTML + CSS + JS)

## Estado actual del modelo de datos

```
tasks/{taskId}:
  text:      string
  done:      boolean
  cat:       string (category ID)
  createdAt: number
```

No existe campo `status`. `done` es boolean.

## Decisiones del Ingeniero Jefe

1. Toggle lista / tablero (conviven, toggle en nav)
2. Drag & drop como interacción principal, botones ← → como fallback mobile
3. Mobile: scroll horizontal snap, una columna visible con dots indicator
4. Migración manual: `done:false` → "por hacer" por defecto (read-side), sin Firestore migration batch

## Estrategia de migración sin batch

**Read-side default (zero Firestore writes):**
```javascript
const status = task.status ?? (task.done ? 'done' : 'todo');
```

Al mover una tarea por primera vez, se escribe `status` en Firestore.
Las tareas existentes no se tocan hasta que el usuario las mueve.

## Patrones y convenciones a respetar

- Estado en variables de módulo: agregar `viewMode`
- `render()` despacha a `renderList()` o `renderBoard()` según `viewMode`
- Delegación de eventos: los cards del board también usan delegación
- `escHtml()` en todo innerHTML
- Sin librerías externas: usar HTML5 DnD API nativo
- CSS variables existentes para colores y espaciado
- `localStorage` para persistir `viewMode`

## Impacto en campos existentes

- `done` se mantiene en sincronía con `status`:
  - Al mover a "Hecho": `{ status: 'done', done: true }`
  - Al mover fuera de "Hecho": `{ status: 'todo'|'doing', done: false }`
  - Al hacer click en check (lista): `{ done: !done, status: done ? 'todo' : 'done' }`
- `addTask()`: escribir `status: 'todo'` en los nuevos documentos

## Riesgos identificados

1. **Touch drag mobile**: HTML5 DnD no funciona en touch → fallback con botones ← → en cada card del board
2. **Altura del board**: necesita `height: calc(100vh - offset)` para columnas con scroll interno
3. **dots indicator mobile**: requiere `scroll` event listener + cleanup
4. **Filter nav en board mode**: "Todas/Pendientes/Completadas" pierden sentido → ocultarlos en board mode
5. **clearDone()**: debe usar `status === 'done'` como criterio (ya coincide con `done: true`)
6. **toggleTask() en lista**: debe sincronizar `status` junto con `done`
