## Test Report — Scroll interno en "Por hacer" (vista Focus)

**Feature:** Altura máxima + scroll interno en la zona "Por hacer" de Focus
**Estado general:** ✅ Aprobado

---

### Criterios de aceptación

**Historia 1 — Backlog acotado con scroll interno**
- [x] `max-height: 380px` en `.board-col-body`, scopeado a
      `.focus-board [data-col-status="todo"]` — con ≤4 tareas de una línea
      (~324px de contenido + 30px de gaps + 20px de padding = 374px) el
      contenido queda por debajo del límite, `overflow-y: auto` no muestra
      scrollbar en ese caso (comportamiento nativo del navegador, sin
      necesidad de lógica condicional)
- [x] Con más de 4, el contenido excede 380px y aparece scroll — mismo
      mecanismo, automático
- [x] Header de la zona (`.board-col-header`) es hermano de
      `.board-col-body`, no descendiente — queda fuera de la región con
      `overflow-y`, permanece visible siempre
- [x] "Haciendo"/"Hecho" en la segunda fila del grid (`grid-template-areas:
      "doing done"`) — su altura se calcula independiente de la primera
      fila ("todo"), que ahora está acotada; no pueden ser empujadas fuera
      de vista
- [x] Cero cambios en `.board` (Tablero) o en cualquier regla de Lista — el
      selector nuevo solo puede matchear dentro de `.focus-board`

**Historia 2 — Drag & drop con scroll**
- [x] Scroll nativo del navegador (`overflow-y: auto`) — funciona con
      mouse/touch/teclado sin código adicional
- [x] `initBoardDnd` no se tocó — sigue usando `e.target.closest('.board-col')`,
      que resuelve por ancestría del DOM, no por posición visual/scroll.
      Confirmado que no hay ninguna dependencia de coordenadas de pantalla
      en los handlers de `dragover`/`drop`

---

### Regresiones detectadas

Ninguna. Diff de una sola regla CSS nueva (10 líneas incluyendo el
comentario), sin tocar JS. `node --check` sobre el módulo: sin errores (no
había nada que cambiara ahí, verificado igual por consistencia).

---

### Edge cases verificados

- Backlog vacío o con pocas tareas: **ok** — `min-height: 80px` existente
  de `.board-col-body` (80px) es menor que el nuevo `max-height: 380px`,
  sin conflicto; la zona de drop sigue siendo descubrible
- Texto largo que envuelve a 2+ líneas: **comportamiento esperado, no es
  bug** — menos de 4 tarjetas completas caben en 380px si el texto es
  largo; el brief mismo pide "~4" como aproximación, documentado en el
  research
- Verificación visual/interactiva del scroll y el drag en navegador real:
  **no disponible en este entorno** (sin sesión logueada) — mismo caveat de
  toda la sesión. El razonamiento de por qué el drag & drop no debería
  romperse está confirmado a nivel de código (ancestría DOM, no
  coordenadas), pero la confirmación visual queda pendiente

---

### Seguridad

No aplica — sin cambios de JS, sin `innerHTML` nuevo, sin tocar Firestore
ni Security Rules.

---

### Recomendaciones

Ninguna. Feature mínima y bien acotada — CSS puro, sin riesgo de
regresión fuera de la zona ya scopeada.
