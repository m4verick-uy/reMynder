## Test Report — Layout de la vista Focus

**Feature:** Disposición espacial de Focus + drag & drop idéntico a Tablero + apilado responsive
**Estado general:** ⚠️ Aprobado con observaciones

---

### Criterios de aceptación

**Historia 1 — Disposición espacial**
- [x] "Por hacer" ocupa fila superior 100% de ancho (`grid-template-areas: "todo todo"`)
- [x] "Haciendo"/"Hecho" comparten fila inferior 50/50 (`"doing done"`, `grid-template-columns: 1fr 1fr`)
- [x] Tarjetas de "Hecho" con `opacity: 0.7`, scopeado a `.focus-board`, no afecta Tablero
- [x] Encabezados de zona reutilizan `.board-col-header` sin cambios — mismo tratamiento que Tablero por herencia directa del markup
- [x] `.focus-board` agregado a la media query de 1200px, mismo criterio que `.board`
- [x] Sin cambios visibles en Tablero ni Lista — verificado línea por línea del diff, `renderBoard()` produce HTML idéntico al de antes (la extracción a `renderBoardColumnsHtml` es un refactor puro, confirmado carácter por carácter contra el original)

**Historia 2 — Drag & drop idéntico**
- [x] `initBoardDnd(focusEl)` reutiliza la función tal cual — confirmado que no tiene ninguna referencia hardcodeada a `#board`, solo usa el parámetro local y clases (`.board-card`, `.board-col`) y estado global genérico (`dragTaskId`, `tasks`, `moveTask`)
- [x] Mismo resaltado de destino (`.board-col.drag-over`) — clase reutilizada sin cambios
- [x] Columnas vacías conservan drop zone (`.empty-col`, mensaje "soltá acá") — mismo markup que Tablero
- [x] Edición inline de tareas funciona en Focus — confirmado por herencia: `renderBoardColumnsHtml` es la misma función que ya soporta `editingTaskId`/`renderTaskForm`, y el listener de `initBoardDnd` ya maneja `edit-task`/`cancel-task`/`select-task-cat`/`confirm-task`

**Historia 3 — Responsive**
- [x] Breakpoint 640px (mismo que Tablero): grid pasa a 1 columna, orden `todo → doing → done` vía `grid-template-areas`
- [x] Sin carrusel ni dots — `initMobileDots` no se llama para Focus, `#board-dots` permanece oculto (`render()` lo fuerza a `display:none` salvo en la rama `board`)
- [x] Drag & drop funciona apilado — mismo `initBoardDnd`, no depende de la disposición visual

---

### Regresiones detectadas y corregidas durante la verificación

**Corrección de scope preventiva — no llegó a ser un bug en producción, se corrigió antes de que Focus reutilizara la clase**

`@media (max-width: 640px) { .board-col { ... } }` usaba un selector sin
calificar. Documentado en el research como conflicto anticipado antes de
escribir el CSS de Focus — se corrigió a `.board .board-col` en el mismo
pase en que se agregó `.focus-board`, así que nunca llegó a existir un
commit donde el bug estuviera presente. Confirmado que `.board-col` solo
aparece dentro de `.board` en el markup de Tablero — cero cambio de
comportamiento ahí.

**Sin otras regresiones.** `node --check` sobre el módulo JS: sin errores.
Sin funciones duplicadas (`renderFocus`/`renderBoard`/`renderBoardColumnsHtml`
verificados únicos por nombre).

---

### Edge cases verificados

- Alguna zona vacía (filtro de categoría): **ok** — mismo `.empty-col` +
  mensaje "soltá acá" que Tablero, columna sigue siendo drop zone
- Las tres zonas vacías (`visible.length === 0`): **ok** — mismo mensaje
  `.board-empty-filter` que Tablero
- Arrastrar dentro de la misma zona: **ok** — `moveTask` solo se llama si
  `getTaskStatus(t) !== newStatus`, mismo guard que ya tenía Tablero
- Layout real en mobile: **no verificable sin navegador real** — mismo
  motivo de siempre (sin sesión logueada). El breakpoint reutiliza el mismo
  valor (640px) y mecanismo (CSS puro) que Tablero, que sí está en
  producción, lo cual baja el riesgo relativo a algo ya probado en el mundo
  real, aunque no específicamente para el grid de Focus.

---

### Observación menor — especificidad CSS: opacidad de "Hecho" vs. estado `.dragging`

`.focus-board [data-col-status="done"] .board-card` tiene mayor
especificidad CSS que `.board-card.dragging` (3 selectores de clase/atributo
contra 2). Al arrastrar una tarjeta de la zona "Hecho" en Focus, el
`opacity: 0.7` de la zona gana sobre el `opacity: 0.5` del estado
`.dragging` — la tarjeta no se ve tan tenue como en Tablero mientras se
arrastra. **No es una pérdida de feedback**: el anillo de acento
(`box-shadow: 0 0 0 2px var(--accent)`) que también aplica `.dragging` sigue
funcionando igual, sin conflicto de especificidad, así que el usuario igual
ve claramente qué tarjeta está arrastrando. No estaba especificado en el
brief cómo debían interactuar ambos estados — se documenta como observación
cosmética menor, no bloqueante.

---

### Seguridad

- [x] Sin `innerHTML` con datos de usuario nuevos — `renderBoardColumnsHtml`
      es exactamente el mismo código que ya sanitizaba con `escHtml()`,
      movido de lugar sin modificar
- [x] Firestore Security Rules no tocadas
- [x] Sin cambios en modelo de datos ni en funciones de escritura
      (`moveTask`, `updateTask`, `deleteTask` sin tocar)

---

### Recomendaciones

- Ninguna bloqueante. La observación de especificidad CSS queda a criterio
  del Ingeniero Jefe (fix trivial si se quiere: subir la especificidad de
  `.board-card.dragging` o bajar la de la regla de opacidad — no se tocó
  para no asumir una preferencia de diseño no pedida).
- Verificación visual/interactiva en navegador real pendiente, mismo caveat
  de toda la sesión.
