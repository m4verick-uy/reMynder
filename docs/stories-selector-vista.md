# Stories — Selector de vista unificado (Tablero | Focus | Lista)

**Basado en:** `docs/research-selector-vista.md`

**Decisión confirmada por el Ingeniero Jefe:** "Focus" es una vista nueva a
construir en esta feature (shell/placeholder), sin comportamiento propio —
el comportamiento se define en una feature futura aparte.

---

### Historia 1 — Selector unificado de 3 segmentos
**Historia:** Como usuario, quiero un único control siempre visible con las
tres vistas disponibles (Tablero, Focus, Lista), para cambiar de vista con
un solo toque y sin ambigüedad sobre cuál está activa.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] Un segmented control de 3 segmentos en el header, arriba a la derecha,
      junto a los íconos de tema y ajustes — reemplaza al `#view-toggle-btn`
      actual, que se elimina por completo (markup, CSS específico si lo
      hubiera, JS de wiring)
- [ ] Orden fijo: Tablero, Focus, Lista
- [ ] Al cargar sin preferencia guardada, Tablero es la vista activa por
      defecto
- [ ] Tocar un segmento cambia la vista mostrada inmediatamente
- [ ] Solo un segmento activo a la vez (mutuamente excluyentes)
- [ ] La preferencia de vista elegida se recuerda entre sesiones
      (persistencia ya existente vía `localStorage`, extendida a 3 valores)
- [ ] Estilo: contenedor con fondo/borde/radio/padding según el brief;
      segmento activo con fondo de acento y texto de alto contraste, peso
      500; segmentos inactivos con texto secundario y fondo transparente
- [ ] El componente no comparte clase ni estilos con las píldoras de filtro
      de categoría/estado (`.nav-btn`) — son controles visualmente distintos

**Edge cases:**
- Pantallas angostas (mobile): el selector no debe desbordar ni tapar otros
  elementos del header (avatar, nombre, botón salir)
- Cambiar de vista repetidamente y rápido no debe dejar más de un segmento
  marcado como activo

---

### Historia 2 — Shell de la vista Focus
**Historia:** Como usuario, quiero poder entrar a una vista "Focus" desde el
selector, para que exista como destino navegable aunque su comportamiento
todavía no esté definido.

**Prioridad:** Alta (bloquea la Historia 1 — sin esto, el tercer segmento no
tiene a dónde apuntar)

**Criterios de aceptación:**
- [ ] Existe un contenedor de vista Focus, mostrado/ocultado igual que
      `#task-list` y `#board` según el `viewMode` activo
- [ ] El contenido es un placeholder simple y honesto (algo como "en
      construcción"), sin simular funcionalidad que no existe
- [ ] No implementa ningún comportamiento de negocio — no filtra tareas, no
      lee ni escribe Firestore, no depende de `tasks`/`categories` más allá
      de lo mínimo para no romper el resto del render
- [ ] Entrar y salir de Focus no afecta el estado de Tablero ni Lista (sin
      side effects cruzados)

**Edge cases:**
- Recargar la página con `viewMode: 'focus'` guardado — debe abrir
  directamente en Focus, no en Tablero

---

## Fuera de alcance (explícitamente, para que no se asuma)

- Cualquier comportamiento, filtro, o lógica de negocio de la vista Focus —
  queda para una feature futura, según lo confirmado por el Ingeniero Jefe
- Cambios al contenido de Tablero o Lista
- Rediseño de las píldoras de filtro de categoría/estado existentes
