# Stories — Scroll interno en "Por hacer" (vista Focus)

**Basado en:** `docs/research-scroll-focus-backlog.md`

Sin preguntas abiertas de producto — brief prescriptivo, valor de "~4
tarjetas" derivado del box model real en el research.

---

### Historia 1 — Backlog acotado con scroll interno
**Historia:** Como usuario con muchas tareas pendientes, quiero que la zona
"Por hacer" de Focus tenga una altura máxima con scroll propio, para seguir
viendo "Haciendo" y "Hecho" sin importar cuántas tareas tenga el backlog.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] Con 4 tareas o menos en "Por hacer", la zona se contrae a su
      contenido — sin espacio vacío sobrante ni scrollbar visible
- [ ] Con más de 4, la zona mantiene su altura máxima (~4 tarjetas) y
      aparece scroll vertical interno para el resto
- [ ] El encabezado de la zona ("Por hacer" + contador) permanece siempre
      visible, no scrollea junto con las tarjetas
- [ ] "Haciendo" y "Hecho" quedan siempre visibles en su fila, sin importar
      cuántas tareas haya en "Por hacer"
- [ ] Tablero y Lista sin ningún cambio visible ni de comportamiento

**Edge cases:**
- Tareas con texto largo que envuelve a 2+ líneas — el límite de altura es
  aproximado por diseño (el brief lo pide como "~4"), menos tarjetas
  completas pueden caber si el texto es largo; no se considera un bug
- Backlog vacío — la zona sigue siendo drop zone visible (comportamiento ya
  existente de Tablero/Focus, no se toca)

---

### Historia 2 — Drag & drop con scroll
**Historia:** Como usuario, quiero poder arrastrar una tarjeta de "Por
hacer" que solo es visible después de scrollear, para reordenar o mover
tareas sin perder la funcionalidad de arrastre por el scroll agregado.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] Scrollear dentro de "Por hacer" para ver una tarjeta funciona con
      mouse, touch y teclado (scroll nativo del navegador, sin nada custom)
- [ ] Una vez visible, arrastrar esa tarjeta a "Haciendo"/"Hecho" (o
      reordenar dentro de "Por hacer") funciona igual que cualquier otra
      tarjeta — mismo mecanismo de `initBoardDnd`, sin código nuevo

**Edge cases:**
- Iniciar un arrastre mientras se está scrolleando — comportamiento nativo
  del navegador, no se agrega lógica custom que pueda interferir

---

## Fuera de alcance (explícitamente, para que no se asuma)

- Cualquier cambio a Tablero o Lista
- Cualquier cambio al modelo de datos
- Auto-scroll durante el drag (arrastrar hasta el borde de la zona para que
  scrollee sola) — no lo pide el brief, es scroll manual del usuario antes
  de arrastrar
