## Test Report — Selector de vista unificado (Tablero | Focus | Lista)

**Feature:** Segmented control de 3 vistas + shell de la vista Focus
**Estado general:** ⚠️ Aprobado con observaciones

---

### Criterios de aceptación

**Historia 1 — Selector unificado**
- [x] Segmented control en `.user-info`, antes de `#theme-btn`/`#settings-btn`
- [x] `#view-toggle-btn` y toda su lógica asociada (`toggleViewMode`, `updateViewBtn`, listener, sep huérfano) eliminados — verificado con grep, cero referencias colgantes
- [x] Orden fijo Tablero → Focus → Lista en el markup
- [x] Default `'board'` sin preferencia guardada (`localStorage.getItem('viewMode') || 'board'`)
- [x] `setViewMode(mode)` cambia la vista inmediatamente (llama `render()` de forma síncrona)
- [x] Mutuamente excluyentes — `.active` se asigna por comparación exacta `dataset.view === viewMode`, solo puede matchear un botón a la vez
- [x] Persistencia entre sesiones — mismo mecanismo `localStorage` de antes, sin cambios
- [x] Estilo: fondo `var(--surface)`, borde `var(--border)`, radio 9px, padding 3px; activo con `var(--accent)`/`var(--accent-contrast)` peso 500; inactivo `var(--muted)`/transparente — coincide con el brief
- [x] No comparte clase con `.nav-btn` — `.view-switch`/`.view-switch-btn` son clases nuevas e independientes; confirmado que no hay ninguna regla `button {}` global que pudiera afectarlas por cascada

**Historia 2 — Shell de Focus**
- [x] `#focus-view` se muestra/oculta con el mismo patrón que `#task-list`/`#board` dentro de `render()`
- [x] Contenido placeholder honesto ("el modo focus está en construcción."), reutiliza `.empty` sin CSS nuevo
- [x] `renderFocus()` no toca `tasks`/`categories`/Firestore
- [x] Sin side effects cruzados — `boardDndController`/`dotsController` se abortan correctamente al entrar a Focus, igual que al entrar a Lista
- [x] Recargar con `viewMode: 'focus'` guardado abre directo en Focus — el fallback `|| 'board'` solo aplica cuando no hay valor guardado, un `'focus'` guardado se respeta

---

### Regresiones detectadas y corregidas durante la verificación

**Bug real encontrado — `renderNavCats()` dependía de `#view-toggle-btn` para insertar los pills de categoría**

No estaba en el research original. `renderNavCats()` (línea ~1841) usaba
`document.getElementById('view-toggle-btn').previousElementSibling` como
punto de referencia para `nav.insertBefore(...)`. Al sacar ese botón del
DOM, `getElementById` devuelve `null` y `null.previousElementSibling` tira
excepción — **esto rompía el render de categorías en cada snapshot de
Firestore**, no solo el selector de vista. Corregido cambiando a
`nav.appendChild(btn)`: como ya no queda nada después de los filtros de
categoría en `<nav>` (el sep+botón viejo se eliminaron), appendear al final
preserva exactamente el mismo orden visual que antes.

Se corrigió en el mismo pase, antes de cerrar la feature — no llegó a
producción.

**Sin otras regresiones.** Revisado `renderList`, `renderBoard`,
`renderTaskForm`, drag & drop, edición de categorías/tareas, tema —
ninguno depende de `viewMode` fuera del dispatch de `render()`, que sigue
cubriendo los mismos casos (board/lista) más el nuevo (focus). `node --check`
sobre el módulo JS extraído: sin errores de sintaxis.

---

### Edge cases verificados

- Cambiar de vista repetidas veces rápido: **ok** — cada click es
  síncrono, no hay estado intermedio que pueda dejar dos segmentos activos
- `viewMode: 'focus'` persistido y recarga de página: **ok**, ver arriba
- Usuario con cero categorías: **observación menor, no bloqueante** — ver
  abajo
- Layout en mobile (pantallas angostas): **no verificable sin navegador
  real** — mitigación con `flex-wrap` aplicada en `.header-top` y
  `.user-info` según lo especificado, pero no se pudo confirmar visualmente
  (misma limitación de Auth ya reportada en features anteriores de esta
  sesión)

---

### Observación menor — separador colgante con cero categorías

Antes de esta feature, `<nav>` siempre terminaba en `#view-toggle-btn`, así
que el `<div class="sep" id="status-sep"></div>` nunca podía ser el último
hijo visible. Ahora que ese botón no existe, si un usuario borra todas sus
categorías (posible vía Ajustes, `deleteCategory` lo permite si no tiene
tareas asociadas), `#status-sep` queda como el último elemento de `<nav>` —
una línea vertical de 1px sin nada después. Es un artefacto cosmético
menor, de alcance ajeno a esta feature (afecta el estado "cero categorías",
no el selector de vista), y no estaba pedido en el brief. Se documenta para
que el Ingeniero Jefe decida si amerita un hotfix aparte (ej. ocultar
`#status-sep` cuando `categories.length === 0`).

---

### Seguridad

- [x] Sin `innerHTML` con texto de usuario nuevo en esta feature — el único
      `innerHTML` nuevo (`renderFocus`) es un string estático, sin
      interpolación de datos
- [x] Firestore Security Rules no tocadas
- [x] No hay datos de usuario expuestos — `viewMode` es preferencia de UI
      local, no dato de negocio
- [x] Login/logout sin cambios — `viewMode` no se resetea en logout,
      correcto (es preferencia de dispositivo, mismo criterio que el tema)

---

### Recomendaciones

- Ninguna bloqueante. La observación del separador colgante queda a
  criterio del Ingeniero Jefe para un hotfix aparte.
- Verificación visual/interactiva en navegador real pendiente (mismo
  motivo de siempre: sin acceso a sesión logueada con Google en este
  entorno) — recomendado antes de mergear a `main`.
