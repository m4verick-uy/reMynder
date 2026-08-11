# Stories — Layout de la vista Focus

**Basado en:** `docs/research-layout-focus.md`

Sin preguntas abiertas de producto — el brief del Ingeniero Jefe es
prescriptivo y coincide con el sistema ya construido en Tablero (checklist
del Story Writer revisado: sin límites numéricos nuevos, sin migración de
datos, sin modelo de negocio, punto de entrada ya existente vía el
selector de vista de la feature anterior).

---

### Historia 1 — Disposición espacial de Focus
**Historia:** Como usuario, quiero que la vista Focus muestre "Por hacer" en
una fila superior a todo el ancho y "Haciendo"/"Hecho" compartiendo la fila
inferior en mitades iguales, para que el backlog domine visualmente y las
otras dos etapas queden secundarias pero visibles sin necesidad de scroll
horizontal ni de cambiar de vista.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] "Por hacer" ocupa el 100% del ancho del contenedor en una fila
      superior, tarjetas en lista vertical simple (una por fila, ancho
      completo)
- [ ] "Haciendo" y "Hecho" comparten la fila inferior en dos mitades fijas
      50/50
- [ ] Las tarjetas de "Hecho" tienen opacidad reducida (~0.7) sin ocultarse
- [ ] El encabezado de cada zona (nombre + contador) usa el mismo
      tratamiento visual que en Tablero — sin componente nuevo
- [ ] El contenedor de Focus se centra a 1200px en monitores anchos, igual
      que Tablero/Lista/nav
- [ ] Ningún cambio visible en las vistas Tablero ni Lista

**Edge cases:**
- Alguna de las tres zonas sin tareas (filtrada por categoría o
  genuinamente vacía) — la zona debe seguir visible como drop zone
  descubrible, no colapsar ni desaparecer
- Las tres zonas vacías a la vez (ej. filtro de categoría sin tareas) —
  mismo mensaje que ya usa Tablero en ese caso ("Sin tareas en {categoría}")

---

### Historia 2 — Drag & drop idéntico a Tablero
**Historia:** Como usuario, quiero poder arrastrar tareas entre "Por
hacer", "Haciendo" y "Hecho" en Focus exactamente igual que en Tablero, para
no tener que aprender una interacción nueva ni cambiar de vista para mover
una tarea.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] Arrastrar una tarjeta a otra zona cambia su `status`, mismo mecanismo
      que Tablero (misma función, sin reimplementación)
- [ ] Mismo resaltado visual de la zona de destino al arrastrar por encima
- [ ] Zonas vacías conservan su drop zone visible y arrastrable
- [ ] Editar una tarea inline (ícono de editar, feature de sesión anterior)
      funciona igual en Focus que en Tablero — consecuencia de reusar el
      mismo componente de tarjeta, no requiere lógica nueva

**Edge cases:**
- Arrastrar dentro de la misma zona (sin cambiar de status) — no debe
  disparar una escritura innecesaria a Firestore (mismo comportamiento que
  ya tiene Tablero hoy)

---

### Historia 3 — Responsive: apilado vertical
**Historia:** Como usuario en un dispositivo angosto, quiero ver las tres
zonas de Focus apiladas verticalmente en orden (Por hacer → Haciendo →
Hecho), para poder usar la vista sin scroll horizontal ni gestos de swipe.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] Por debajo del breakpoint mobile, las tres zonas se apilan en una sola
      columna, en el orden Por hacer → Haciendo → Hecho
- [ ] No aparece el carrusel horizontal ni los dots de navegación que usa
      Tablero en mobile — Focus no los necesita ni los muestra
- [ ] El drag & drop sigue funcionando apilado (misma Historia 2, sin
      distinción por viewport)

**Edge cases:**
- Verificación visual real en mobile no disponible en este entorno (sin
  navegador logueado) — queda pendiente de prueba manual por el Ingeniero
  Jefe, igual que en features anteriores de esta sesión

---

## Fuera de alcance (explícitamente, para que no se asuma)

- Cualquier cambio a Tablero o Lista
- Cualquier cambio al modelo de datos (`status`, `cat`, etc.)
- Un toggle/selector nuevo — la alternancia entre vistas ya existe
- Comportamiento de negocio adicional en Focus más allá de lo que Tablero
  ya tiene (filtros, ordenamiento especial, etc. no pedidos en el brief)
