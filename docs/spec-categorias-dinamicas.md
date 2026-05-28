# Spec: Categorías Dinámicas

**Agente:** Spec Writer
**Feature:** Categorías dinámicas
**Fecha:** 2026-05-28
**Stack afectado:** JS · CSS · HTML · Firestore · Firebase Security Rules

---

## 1. Cambios en modelo de datos

### 1.1 Nueva colección: `users/{uid}/categories/{catId}`

| Campo | Tipo | Descripción |
|---|---|---|
| `name` | string | Nombre de la categoría (texto libre del usuario) |
| `colorIndex` | number | Índice de color 0–9 (apunta a la paleta predefinida) |
| `createdAt` | number | Timestamp Unix (`Date.now()`) |

### 1.2 Seed — usuario m4verick (IDs explícitos)

Usar `setDoc(doc(..., catId), datos, { merge: true })` — idempotente por diseño.
Los IDs coinciden con los valores del campo `cat` de las tareas existentes → cero migración.

| catId | name | colorIndex |
|---|---|---|
| `'karate'` | `'karate'` | 0 |
| `'facultad'` | `'facultad'` | 1 |
| `'antel'` | `'antel'` | 2 |
| `'salud'` | `'salud'` | 3 |
| `'personal'` | `'personal'` | 4 |
| `'padres'` | `'padres'` | 5 |

### 1.3 Seed — usuarios nuevos

Detectado cuando el primer disparo del listener de categorías llega con `categories.length === 0`.
Usar `addDoc` (IDs autogenerados).

| name | colorIndex |
|---|---|
| `'trabajo'` | 0 |
| `'personal'` | 4 |

### 1.4 Impacto en tareas existentes

**Ninguno.** El campo `t.cat` de cada tarea ya contiene el `catId` correcto
(para m4verick: `'karate'`, `'facultad'`, etc. — que son los mismos IDs del seed).

### 1.5 Firebase Security Rules — cambio externo requerido

⚠️ **El Ingeniero Jefe debe actualizar las reglas en la consola de Firebase
antes de probar en producción.** El Backend Builder lo recordará al finalizar.

Regla a agregar (junto a la existente de `tasks`):

```
match /users/{userId}/categories/{catId} {
  allow read, write: if request.auth != null && request.auth.uid == userId;
}
```

---

## 2. Cambios en CSS

### 2.1 Paleta predefinida (10 colores)

Reemplazar las variables por nombre (`--karate`, `--facultad`, etc.) con una paleta numerada.
Cada color necesita dos variantes: texto (`--pN`) y fondo (`--pN-bg`).
Definir en `:root` (dark) **y** en `html[data-theme="light"]` (light).

Mapeo de colores nuevos ↔ existentes (para continuidad visual):

| Índice | Apodo | Equivale a |
|---|---|---|
| 0 | naranja-rojo | karate |
| 1 | azul | facultad |
| 2 | verde-teal | antel |
| 3 | rojo | salud |
| 4 | violeta | personal |
| 5 | ámbar | padres |
| 6 | rosa/magenta | nuevo |
| 7 | cian | nuevo |
| 8 | verde-lima | nuevo |
| 9 | índigo | nuevo |

Variables dark (`:root`): `--p0` / `--p0-bg` ... `--p9` / `--p9-bg`
Variables light (`html[data-theme="light"]`): mismas, con valores distintos

Valores numéricos exactos a cargo del Frontend Builder, respetando la filosofía visual
(saturación moderada, fondos oscuros en dark y fondos pastel en light).

### 2.2 Alias de backward compatibility

Mantener en ambos temas:
```
--salud:    var(--p3);
--salud-bg: var(--p3-bg);
```

Preserva las referencias en `.logout-btn:hover` y `.clear-btn:hover` sin tocarlas.
Los otros aliases (`--karate`, `--facultad`, etc.) se eliminan junto con sus clases `.cat-*`.

### 2.3 Clases de color de categoría

Eliminar las 6 clases estáticas:
```
.cat-karate / .cat-facultad / .cat-antel / .cat-salud / .cat-personal / .cat-padres
```

Agregar 10 clases numéricas:
```
.cat-p0 { background: var(--p0-bg); color: var(--p0); }
...
.cat-p9 { background: var(--p9-bg); color: var(--p9); }
```

### 2.4 Nuevos estilos: pantalla de ajustes

Clases nuevas a definir (valores a cargo del Frontend Builder respetando el sistema existente):

| Clase | Descripción |
|---|---|
| `.settings-main` | `<main>` de la pantalla de ajustes, mismo padding que el main principal |
| `.back-btn` | Botón "← volver" — estilo ghost, similar a `.logout-btn` |
| `.settings-title` | Título "ajustes" — DM Mono, mismo tamaño que `header h1` |
| `.cat-row` | Fila de categoría: `display:flex; align-items:center; gap:12px; padding:12px 0;` |
| `.cat-row + .cat-row` | Separador `border-top: 1px solid var(--border)` |
| `.cat-dot` | Círculo de color — 16px, `border-radius:50%` — recibe clase `.cat-pN` |
| `.cat-name` | Nombre de la categoría — DM Sans, `font-size:15px; flex:1` |
| `.icon-btn` | Botón ícono — 28×28px, ghost, igual que `.del-btn` existente |
| `.icon-btn.blocked` | Versión deshabilitada con `opacity:0.3; cursor:not-allowed` |
| `.cat-row-form` | Fila de formulario inline — `display:flex; gap:8px; align-items:center` |
| `.cat-form-input` | Input de nombre — mismo estilo que `.add-row input` |
| `.color-picker` | Contenedor de los 10 dots seleccionables — `display:flex; gap:8px; flex-wrap:wrap` |
| `.color-dot` | Dot de color seleccionable — 22px, `border-radius:50%; cursor:pointer; border:2px solid transparent` |
| `.color-dot.selected` | Dot activo — `border-color: currentColor; transform: scale(1.15)` |
| `.cat-blocked-msg` | Mensaje de bloqueo — DM Mono, `font-size:12px; color:var(--muted); margin-top:4px` |
| `.cat-error` | Error inline — `font-size:12px; color:var(--p3); margin-top:4px` |
| `.add-cat-btn` | Botón "+" al final de lista — mismo estilo que `.add-row button` pero `width:auto` |
| `.settings-empty` | Estado vacío — mismo estilo que `.empty` existente |

---

## 3. Cambios en HTML

### 3.1 Header — agregar botón ⚙

En `#app > header > .header-top > .user-info`, insertar **antes** del avatar:

```
button#settings-btn  (aria-label="Ajustes")
```

Estilo: idéntico a `.theme-btn` (mismo tamaño, ghost, hover). Ícono: SVG ⚙ de 14px.
**No aparece** en `#login-screen` (ya está fuera del scope del login).

### 3.2 Cat selector — vaciar HTML estático

El `<div class="cats-row" id="cat-selector">` queda **vacío** en el HTML fuente.
Los botones son renderizados dinámicamente por `renderCatSelector()`.

### 3.3 Nav — vaciar filtros de categoría

En `<nav>`, **mantener estáticos**:
- `#f-all`, `#f-pending`, `#f-done`
- `.sep`

**Eliminar** los 6 botones `#fc-karate`, `#fc-facultad`, `#fc-antel`, `#fc-salud`, `#fc-personal`, `#fc-padres`.

Los filtros de categoría se renderizan dinámicamente por `renderNavCats()` después del `.sep`.

### 3.4 Nueva pantalla: `#settings-screen`

Agregar como **hermano** de `#app` en `<body>` (no anidado dentro). `display:none` por defecto.

Estructura interna:

```
div#settings-screen  (display:none)
  header
    div.header-top
      button#settings-back-btn.back-btn   ← volver
      h2.settings-title                   ajustes
  main.settings-main
    div#categories-list                   ← renderizado dinámicamente
    button#add-category-btn.add-cat-btn   +
```

El `#add-category-btn` queda al final de `settings-main`, siempre visible.
El `#categories-list` se llena con `renderSettings()`.

---

## 4. Cambios en lógica JS

### 4.1 Imports de Firestore

Agregar `setDoc` a los imports existentes (actualmente faltan).
El resto de los imports (`addDoc`, `deleteDoc`, `doc`, `updateDoc`, `onSnapshot`,
`query`, `orderBy`, `writeBatch`, `collection`) se mantienen.

### 4.2 Eliminar constantes hardcodeadas

Eliminar completamente:
```javascript
const CATS   = ['karate','facultad','antel','salud','personal','padres'];
const LABELS = { karate:'karate', ... };
```

### 4.3 Variables de estado — cambios

| Variable | Estado actual | Estado nuevo |
|---|---|---|
| `let selectedCat = 'karate'` | inicializado | `let selectedCat = null` |
| `let unsubscribe = null` | renombrar | `let unsubTasks = null` |
| *(nueva)* | — | `let categories = []` |
| *(nueva)* | — | `let unsubCats = null` |
| *(nueva)* | — | `let categoriesReady = false` |
| *(nueva)* | — | `let editingCatId = null` |

### 4.4 `onAuthStateChanged` — bloque de logout

En el bloque `else` (usuario no autenticado), agregar:
```
unsubCats cancelado y a null
categories = []
selectedCat = null
categoriesReady = false
```
Renombrar todas las referencias `unsubscribe` → `unsubTasks`.

### 4.5 Función: `startListeners(uid)` — reemplaza `startListener(uid)`

Inicia **dos listeners en paralelo**:

**Listener A — categorías:**
- Query: `collection(db, 'users', uid, 'categories')` ordenado por `createdAt` ASC
- Callback:
  1. Actualizar `categories[]` desde el snapshot
  2. Si `!categoriesReady && categories.length === 0` → llamar `seedCategories(uid)`
  3. `categoriesReady = true`
  4. Si `selectedCat === null && categories.length > 0` → `selectedCat = categories[0].id`
  5. Llamar `renderCatSelector()`
  6. Llamar `renderNavCats()`
  7. Llamar `render()`
  8. Si `#settings-screen` está visible → llamar `renderSettings()`
- Almacenar en `unsubCats`

**Listener B — tareas:**
- Query: igual al actual (`collection(db, 'users', uid, 'tasks')` por `createdAt` ASC)
- Callback: actualizar `tasks[]`, llamar `render()` (y `renderSettings()` si visible)
- Almacenar en `unsubTasks`

### 4.6 Función: `seedCategories(uid)`

Distingue dos caminos según el UID:

**Camino A — usuario admin (m4verick):**
- Condición: `uid === '<UID_DE_M4VERICK>'` (Backend Builder inserta el UID real)
- Acción: `setDoc` con `{ merge: true }` para cada una de las 6 categorías (IDs explícitos, colorIndex 0–5, `createdAt: Date.now()`)
- Los setDoc con merge:true son no-ops si el documento ya existe con todos los campos → idempotente

**Camino B — usuarios nuevos (cualquier otro uid):**
- Acción: `addDoc` para "trabajo" (colorIndex: 0) y "personal" (colorIndex: 4), ambos con `createdAt: Date.now()`
- El listener de categorías traerá los nuevos documentos automáticamente

Ambos caminos son async. Los errores deben capturarse con try/catch y loguearse a consola.

### 4.7 Función: `createCategory(name, colorIndex)`

```
name = name.trim()
si name === '' → mostrar error en el form, no guardar
addDoc(collection(db, 'users', uid, 'categories'), { name, colorIndex, createdAt: Date.now() })
editingCatId = null
renderSettings()    ← el listener actualizará categories[], render() se dispara
```

No llamar `render()` manualmente — el listener lo hace.

### 4.8 Función: `updateCategory(id, name, colorIndex)`

```
name = name.trim()
si name === '' → mostrar error en el form, no guardar
updateDoc(doc(db, 'users', uid, 'categories', id), { name, colorIndex })
editingCatId = null
```

### 4.9 Función: `deleteCategory(id)`

```
const taskCount = tasks.filter(t => t.cat === id).length
si taskCount > 0 → mostrar .cat-blocked-msg (cuántas tareas) + instrucción; no proceder
si taskCount === 0:
  confirm(`¿Eliminar "${cat.name}"?`)   ← confirm() nativo es aceptable aquí
  si confirma:
    si selectedCat === id → selectedCat = categories.find(c => c.id !== id)?.id || null
    deleteDoc(doc(db, 'users', uid, 'categories', id))
```

### 4.10 Función: `renderCatSelector()`

Genera los botones del `#cat-selector` desde `categories[]`:
- Por cada cat: `<button class="cat-pill cat-p{cat.colorIndex} {selected?'selected':''}" data-cat="{cat.id}">{escHtml(cat.name)}</button>`
- Si `categories.length === 0`: mostrar placeholder deshabilitado "sin categorías"

### 4.11 Función: `renderNavCats()`

Elimina del `<nav>` todos los elementos con `data-cat-filter` (los dinámicos).
Agrega después del `.sep`:
- Por cada cat: `<button class="nav-btn cat-pill cat-p{cat.colorIndex} {catFilter===cat.id?'active':''}" data-cat-filter="{cat.id}">{escHtml(cat.name)}</button>`

### 4.12 Función: `renderSettings()`

Construye el `#categories-list`:

Si `categories.length === 0`: `<div class="settings-empty">sin categorías. usá + para crear la primera.</div>`

Por cada categoría:
- Si `editingCatId === cat.id`: renderiza `.cat-row-form` (input nombre pre-relleno + `.color-picker` con dot seleccionado + botón confirmar + botón cancelar)
- Si no: renderiza `.cat-row` (`.cat-dot.cat-p{colorIndex}` + `.cat-name` + botón editar + botón eliminar)

El botón eliminar:
- Si `tasks.filter(t => t.cat === cat.id).length > 0`: clase `icon-btn blocked`, `aria-disabled="true"`, muestra `.cat-blocked-msg` debajo de la fila
- Si no tiene tareas: `icon-btn` normal con `data-action="delete-cat"` y `data-id={cat.id}`

### 4.13 Modificar `render()`

Cambios en la generación del badge de cada tarea:

```
// ANTES:
`<span class="badge cat-${t.cat}">${LABELS[t.cat] || t.cat}</span>`

// DESPUÉS:
const catObj = categories.find(c => c.id === t.cat);
const catName = catObj ? escHtml(catObj.name) : escHtml(t.cat);
const catClass = catObj !== undefined ? `cat-p${catObj.colorIndex}` : 'cat-p0';
`<span class="badge ${catClass}">${catName}</span>`
```

El fallback `cat-p0` y `escHtml(t.cat)` cubre el race condition donde `categories[]` aún está vacío.

### 4.14 Modificar `selectCat(cat)`

Eliminar el `querySelectorAll` manual. Simplemente:
```
selectedCat = cat;
renderCatSelector();
```

### 4.15 Modificar `setCatFilter(cat)`

Eliminar la iteración sobre `CATS`. El estado:
```
catFilter = catFilter === cat ? null : cat;
renderNavCats();
render();
```

### 4.16 Funciones de navegación

**`showSettings()`:**
```
document.getElementById('app').style.display = 'none';
document.getElementById('settings-screen').style.display = '';
renderSettings();
```

**`hideSettings()`:**
```
document.getElementById('settings-screen').style.display = 'none';
document.getElementById('app').style.display = '';
render();
```

### 4.17 Nuevos event listeners

**Nuevos listeners directos:**
- `#settings-btn` → `showSettings()`
- `#settings-back-btn` → `hideSettings()`
- `#add-category-btn` → `editingCatId = 'new'; renderSettings()`

**Delegación en `#categories-list`:**
```
data-action="edit-cat"    → editingCatId = data-id; renderSettings()
data-action="delete-cat"  → deleteCategory(data-id)
data-action="confirm-cat" → si editingCatId === 'new': createCategory(...) sino: updateCategory(editingCatId, ...)
data-action="cancel-cat"  → editingCatId = null; renderSettings()
data-action="select-color"→ actualizar colorIndex seleccionado en el form activo
```

**Delegación en `<nav>`:**
Cambiar el `CATS.forEach(c => document.getElementById('fc-' + c)...)` por un listener único en `<nav>`:
```
e.target.closest('[data-cat-filter]') → setCatFilter(dataset.catFilter)
```

---

## 5. Archivos a modificar

| Archivo | Qué cambia |
|---|---|
| `web/index.html` | CSS (paleta, settings), HTML (header ⚙, selector/nav vacíos, settings screen), JS (estado, listeners, CRUD, renders) |
| `docs/documentacion.md` | Modelo de datos (nueva colección `categories`), estado JS, arquitectura de pantallas |

## 6. Archivos a crear

| Archivo | Propósito |
|---|---|
| `docs/spec-categorias-dinamicas.md` | Este documento |

---

## 7. Riesgos técnicos

| Riesgo | Mitigación |
|---|---|
| Race condition: `render()` de tareas llega antes que el snapshot de categorías | Badge usa `escHtml(t.cat)` + `cat-p0` como fallback; cuando llega `categories[]`, `render()` se vuelve a llamar y corrige |
| Seed doble: listener se dispara dos veces antes de que Firestore confirme los setDoc | Flag `categoriesReady` impide que `seedCategories` se llame más de una vez por sesión |
| `selectedCat` apunta a ID eliminado | `deleteCategory` reasigna `selectedCat` antes de borrar el documento |
| `renderSettings()` stale mientras el usuario está en ajustes | El listener de categorías llama `renderSettings()` cuando `#settings-screen` está visible |
| Nombre de categoría con `<`, `>`, `&`, `"` en el DOM | Todas las inserciones usan `escHtml()` — `renderCatSelector()`, `renderNavCats()`, `renderSettings()`, `render()` |
| Clase CSS del badge derivada de nombre (podría romper si el nombre tiene espacios) | Las clases son `cat-p0`–`cat-p9` (índice numérico). El nombre del usuario no entra en la clase CSS |
| Listener de categorías y listener de tareas disparan `render()` en paralelo | Vanilla JS es single-thread; los snapshots no se solapan. No hay riesgo de concurrencia |

---

## 8. Criterios de done (técnicos)

- [ ] Import `setDoc` agregado a los imports de Firestore
- [ ] Constantes `CATS` y `LABELS` eliminadas del código
- [ ] `selectedCat` inicializado a `null`, nunca más `'karate'`
- [ ] `startListeners(uid)` reemplaza `startListener(uid)` y gestiona dos subscriptions
- [ ] `unsubTasks` y `unsubCats` se cancelan en logout
- [ ] `categories = []`, `selectedCat = null`, `categoriesReady = false` reseteados en logout
- [ ] Firestore collection `users/{uid}/categories` creada correctamente vía seed
- [ ] Seed de m4verick usa `setDoc + merge:true` con IDs explícitos
- [ ] Seed de nuevos usuarios usa `addDoc`
- [ ] Ambos seeds son idempotentes (varias ejecuciones = mismo resultado)
- [ ] CSS: variables `--p0` a `--p9` + `--pN-bg` definidas en dark y light
- [ ] CSS: alias `--salud` → `var(--p3)` preservado
- [ ] CSS: clases `.cat-p0` a `.cat-p9` reemplazan `.cat-karate` etc.
- [ ] HTML: `#cat-selector` vacío en el fuente; `renderCatSelector()` lo llena
- [ ] HTML: nav sin botones `#fc-*` estáticos; `renderNavCats()` los agrega
- [ ] HTML: `#settings-screen` existe como hermano de `#app`
- [ ] Botón ⚙ en header abre `#settings-screen`
- [ ] Botón "← volver" cierra `#settings-screen` y muestra `#app`
- [ ] `renderSettings()` muestra: dot de color + nombre + editar + eliminar (o bloqueado)
- [ ] Crear categoría: nombre vacío no guarda, muestra error inline
- [ ] Editar categoría: cancelar no guarda cambios
- [ ] Eliminar con tareas: muestra mensaje bloqueante con cantidad de tareas
- [ ] Eliminar sin tareas: `confirm()` + eliminación inmediata y persistente
- [ ] Badge en `render()` usa `cat-p{colorIndex}` y `escHtml(cat.name)`
- [ ] Firebase Security Rules actualizadas para `categories` (acción manual del Ingeniero Jefe)
- [ ] `docs/documentacion.md` actualizado con nueva colección y estado JS
