# Stories — Edición de tareas

**Basado en:** `docs/research-edicion-tareas.md`

**Decisiones confirmadas por el Ingeniero Jefe:**
- Alcance: se puede editar texto **y** categoría de una tarea existente
- Punto de entrada: disponible tanto en vista Lista (`.task-item`) como en
  vista Board (`.board-card`)

---

### Historia 1 — Editar una tarea desde la lista
**Historia:** Como usuario, quiero editar el texto y la categoría de una
tarea existente desde la vista lista, para corregir errores de tipeo o
recategorizar sin tener que borrar y recrear la tarea.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] Cada `.task-item` tiene un ícono de editar, visible junto al ícono de
      eliminar existente
- [ ] Al tocar editar, esa tarea (y solo esa) se reemplaza por un form inline
      con: input de texto (valor actual precargado) y selector de categoría
      (categoría actual preseleccionada)
- [ ] Confirmar guarda los cambios en Firestore y vuelve a la vista normal
- [ ] Cancelar descarta los cambios y vuelve a la vista normal sin escribir
      nada
- [ ] Si había otra tarea en edición, abrir la edición de una nueva la
      cierra automáticamente (una sola tarea editable a la vez, igual que
      categorías)
- [ ] El texto no puede guardarse vacío — mismo mensaje de error inline que
      usa la edición de categorías ("El nombre no puede estar vacío" →
      equivalente para tarea)

**Edge cases:**
- Tarea ya completada (`done: true`) — debe poder editarse igual
- Editar y no cambiar nada — confirmar no debe fallar ni crear un no-op raro
- Llega un snapshot de Firestore (cambio en otro dispositivo) mientras el
  form de edición está abierto — no debe perderse el texto que el usuario
  está tipeando (ver riesgo #4 del research)

---

### Historia 2 — Editar una tarea desde el Board
**Historia:** Como usuario, quiero editar una tarea desde la vista Board
(Kanban), para no tener que cambiar a la vista lista solo para corregir el
texto o la categoría.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] Cada `.board-card` tiene un ícono de editar junto a los controles
      existentes (mover, eliminar)
- [ ] El form de edición inline reemplaza la tarjeta en su columna actual,
      sin mover la tarea de columna/status
- [ ] Mismo comportamiento de confirmar/cancelar/validación que en Historia 1
- [ ] Editar desde el Board no afecta el `status` (columna) de la tarea

**Edge cases:**
- Editar una tarjeta que está siendo arrastrada (drag & drop) — el ícono de
  editar no debe interferir con el gesto de arrastre existente
- Columna muy angosta en mobile — el form de edición debe seguir siendo
  usable (ver `.board-col` en layout mobile)

---

### Historia 3 — Cambiar la categoría de una tarea al editarla
**Historia:** Como usuario, quiero poder reasignar la categoría de una tarea
mientras la edito, para corregir una tarea mal categorizada sin perder su
texto ni su historial.

**Prioridad:** Alta (parte del mismo form de las historias 1 y 2, no es una
interacción separada)

**Criterios de aceptación:**
- [ ] El selector de categoría dentro del form de edición muestra la
      categoría actual de la tarea preseleccionada
- [ ] Cambiar la categoría y confirmar actualiza `cat` en Firestore
- [ ] Si la tarea queda visible o no después de guardar depende del filtro de
      categoría activo (`catFilter`) — comportamiento ya existente, no
      requiere lógica nueva

**Edge cases:**
- Reasignar a una categoría que luego se elimina — no aplica a esta feature
  (el bloqueo de borrado de categorías con tareas asociadas ya existe y sigue
  vigente)

---

## Fuera de alcance (explícitamente, para que no se asuma)

- Edición de `status` (columna del Board) desde el form de edición — eso ya
  existe vía drag & drop / botones mover, no se toca
- Edición masiva (múltiples tareas a la vez)
- Historial de cambios / undo
