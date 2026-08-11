# Research — Scroll interno en "Por hacer" (vista Focus)

**Feature solicitada:** la zona "Por hacer" de Focus cap su altura a ~4
tarjetas; con más, scroll vertical interno solo ahí. "Haciendo"/"Hecho"
siempre visibles. Drag & drop debe seguir funcionando con scroll.

---

## 1. Resumen del problema

Hoy `.board-col-body` (línea 899) no tiene límite de altura — crece con el
contenido. En Focus, la zona "todo" ocupa su propia fila de CSS Grid
(`grid-template-areas: "todo todo"`, feature anterior), y esa fila se
autodimensiona por contenido: con muchas tareas, la fila crece y empuja la
fila de abajo (`"doing done"`) fuera del viewport — exactamente el bug que
describe el brief.

---

## 2. Cálculo de la altura (~4 tarjetas)

No hay que inventar un número — se puede derivar del box model real de
`.board-card` (línea 908) en el caso de una tarjeta de una sola línea:

| Elemento | Alto |
|---|---|
| `.board-card` padding (12px + 12px) | 24px |
| `.board-card-text` (font-size 15px, line-height 1.6, 1 línea) | 24px |
| gap interno (`.board-card { gap: 8px }`) | 8px |
| `.board-card-footer` (altura dominada por los botones de 24×24px de `.board-card-actions`) | 24px |
| borde 0.5px × 2 | ~1px |
| **Subtotal por tarjeta** | **~81px** |

Con 4 tarjetas: `4 × 81px = 324px`, más 3 gaps de `.board-col-body`
(`gap: 10px` → 30px) y el padding propio de `.board-col-body`
(`padding: 10px 0` → 20px):

```
324 + 30 + 20 = 374px
```

Se propone **380px** (374px + ~6px de margen por redondeo/render entre
navegadores). Es una aproximación — el brief mismo lo pide con "~4
tarjetas" — y solo es exacta para tarjetas de una línea; texto largo que
envuelve a 2+ líneas mostrará menos de 4 tarjetas completas dentro de esa
altura, lo cual es inherente a cualquier límite fijo en px y no se puede
evitar sin JS de medición (fuera de alcance, no lo pide el brief).

---

## 3. Dónde aplicar el límite

**No en `.board-col` completo** — el header ("Por hacer" + contador) debe
seguir visible siempre, no scrollear junto con las tarjetas (así se lee el
título y el conteo de un vistazo, coherente con el propósito de Focus). El
límite va en **`.board-col-body`** específicamente, que es hermano de
`.board-col-header` dentro de `.board-col` — acotar su altura no afecta al
header, que queda fijo arriba de la zona de scroll.

Selector a usar, mismo patrón que ya usa la feature de layout de Focus para
scopear reglas solo a "todo" dentro de Focus (`.focus-board
[data-col-status="done"] .board-card { opacity: 0.7 }`):

```css
.focus-board [data-col-status="todo"] .board-col-body {
  max-height: 380px;
  overflow-y: auto;
}
```

`max-height` + `overflow-y: auto` resuelve **los dos casos del brief con
una sola regla**, sin JS: con ≤4 tarjetas el contenido mide menos que
380px y no aparece scroll (se contrae a su contenido); con más, el
contenido excede el límite y el navegador agrega scroll automáticamente.
No hace falta lógica condicional en JS para contar tareas.

---

## 4. Drag & drop con scroll — confirmado que no requiere código nuevo

El drag & drop de Focus ya usa `initBoardDnd(focusEl)` tal cual (reutilizado
de Tablero, feature anterior). Arrastrar una tarjeta que requiere scroll
para alcanzarse es un caso que el HTML5 drag & drop nativo maneja solo:
- El usuario scrollea el contenedor (`overflow-y: auto` ya soporta scroll
  nativo con mouse/touch/teclado, sin nada especial)
- Una vez la tarjeta es visible, iniciar el drag funciona igual que
  cualquier otra tarjeta — el scroll de un ancestro no interfiere con
  `draggable="true"` ni con los listeners de `initBoardDnd`
- `e.target.closest('.board-col')` (usado en `dragover`/`drop`, línea
  ~1764-1786) resuelve por ancestría del DOM, no por posición visual —
  scrollear no cambia la ancestría, así que detectar la columna de destino
  sigue funcionando igual con el contenido scrolleado

**Conclusión: esta feature es 100% CSS, no requiere cambios de JS.**

---

## 5. Riesgos o conflictos detectados

1. Ninguno técnico — la regla está scopeada específicamente a
   `.focus-board [data-col-status="todo"]`, no toca `.board-col-body` en
   Tablero ni en ningún otro contexto.
2. `.board-col-header`/`.board-col.empty-col`/`.board-col.drag-over` no se
   tocan — siguen aplicando sobre `.board-col` completo, sin relación con
   el nuevo límite de `.board-col-body`.
3. Si "Por hacer" está vacío o tiene pocas tareas, `.board-col-body` ya
   tiene `min-height: 80px` (línea 905) para mantener la zona de drop
   descubrible — el nuevo `max-height: 380px` no entra en conflicto con
   ese `min-height` existente (80px < 380px siempre).

---

## 6. Recomendaciones para los siguientes agentes

- **Story Writer**: sin preguntas abiertas — brief prescriptivo con un
  valor de referencia claro ("~4 tarjetas"), ya derivado arriba.
- **Spec Writer**: una sola regla CSS nueva, scopeada, sin cambios de JS.
- **Test Verifier**: confirmar que el selector no alcanza a Tablero
  (`.board` no tiene la clase `.focus-board`, y la vista Lista no usa
  `.board-col-body` en absoluto) y que `min-height`/`max-height` de
  `.board-col-body` no entran en conflicto.
