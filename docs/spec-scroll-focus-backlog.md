## Spec — Scroll interno en "Por hacer" (vista Focus)

**Feature:** Altura máxima + scroll interno en la zona "Por hacer" de Focus
**Stack afectado:** CSS únicamente — `web/index.html`

**Basado en:** `docs/research-scroll-focus-backlog.md`, `docs/stories-scroll-focus-backlog.md`

---

### Cambios en modelo de datos

Ninguno.

### Cambios en estado JS / lógica JS

Ninguno. Confirmado en el research que `max-height` + `overflow-y: auto`
cubre ambos casos del brief (contraer si ≤4, scroll si más) sin lógica
condicional, y que el drag & drop nativo no requiere cambios para funcionar
con contenido scrolleado.

---

### Cambios en CSS

Una sola regla nueva, junto a la definición de `.focus-board` existente:

```css
.focus-board [data-col-status="todo"] .board-col-body {
  max-height: 380px;
  overflow-y: auto;
}
```

380px = ~4 tarjetas de una línea (12px padding + 24px texto + 8px gap +
24px footer + ~1px borde = 81px/tarjeta × 4 + 30px de gaps entre tarjetas +
20px de padding propio de `.board-col-body` = 374px, + ~6px de margen).
Cálculo completo en `docs/research-scroll-focus-backlog.md`.

Scopeada a `.focus-board [data-col-status="todo"] .board-col-body` — mismo
patrón de scoping que ya usa `.focus-board [data-col-status="done"]
.board-card { opacity: 0.7 }` de la feature de layout de Focus. No alcanza
a `.board-col-body` de Tablero (`.board`, no `.focus-board`) ni a Lista
(no usa esta clase).

---

### Archivos a modificar

- `web/index.html` — único archivo

### Archivos a crear

Ninguno

---

### Riesgos técnicos

Ninguno nuevo — confirmado en el research que no hay conflicto con
`min-height: 80px` existente de `.board-col-body` (80px < 380px siempre) ni
con el drag & drop nativo.

---

### Criterios de done

- [ ] `.focus-board [data-col-status="todo"] .board-col-body` con `max-height: 380px; overflow-y: auto;`
- [ ] Con ≤4 tareas: sin scrollbar, zona contraída a contenido
- [ ] Con >4 tareas: scroll interno, header de la zona siempre visible
- [ ] "Haciendo"/"Hecho" siempre visibles independientemente del tamaño de "Por hacer"
- [ ] Drag & drop sin cambios de código, funciona igual con contenido scrolleado
- [ ] Tablero y Lista sin cambios visibles
