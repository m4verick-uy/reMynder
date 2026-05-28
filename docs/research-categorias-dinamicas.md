# Research: Categorías Dinámicas

**Agente:** Researcher
**Feature:** Permitir al usuario crear, editar y eliminar sus propias categorías
**Fecha:** 2026-05-28

---

## 1. Resumen del problema

ReMynder tiene 6 categorías hardcodeadas (`karate`, `facultad`, `antel`, `salud`,
`personal`, `padres`) — personales del creador. Para ser un producto comercial,
cada usuario debe gestionar sus propias categorías.

**Brief del Ingeniero Jefe:**
- Usuario admin (m4verick): sus 6 categorías se migran automáticamente, sin tocarlas
- Usuarios nuevos: arrancan con 2 categorías de ejemplo ("trabajo" y "personal"), editables
- Sin límite de categorías por ahora
- No se puede eliminar una categoría que tenga tareas asociadas
- UI: ícono ⚙ en el header → pantalla de ajustes → gestión inline de categorías

---

## 2. Archivos relevantes

### `web/index.html` — único archivo de la app

**CSS líneas 25–30 (dark) y 42–47 (light):** Variables por categoría hardcodeadas:
```css
--karate: #F0997B; --karate-bg: #4A1B0C; --karate-dark: #4A1B0C;
/* × 6 categorías × 2 temas = 36 declaraciones */
```

**CSS líneas 237–242:** Clases de color por categoría:
```css
.cat-karate   { background: var(--karate-bg);   color: var(--karate);   }
/* × 6 categorías */
```

**HTML líneas 449–456:** Selector de categoría — 6 botones estáticos con `data-cat`.

**HTML líneas 463–469:** Nav filtros — 6 botones estáticos con `id="fc-{nombre}"`.

**JS líneas 502–503:** Constantes hardcodeadas:
```javascript
const CATS   = ['karate','facultad','antel','salud','personal','padres'];
const LABELS = { karate:'karate', ... };
```

**JS línea 508:** `selectedCat = 'karate'` — inicializado con nombre hardcodeado.

**JS líneas 631–637:** `setCatFilter` itera sobre `CATS` buscando elementos por ID estático.

**JS líneas 696–698:** Event listeners individuales por categoría usando `CATS.forEach`.

**JS línea 671:** Badge renderizado con `cat-${t.cat}` y `LABELS[t.cat]`.

---

## 3. Patrones y convenciones a respetar

- **`onSnapshot` para tiempo real:** categorías necesitan su propio listener
- **`render()` recalcula todo desde estado:** selector, filtros y badges deben derivarse de `categories[]`
- **Delegación de eventos:** los clicks en categorías dinámicas usan delegación, no listeners individuales
- **`escHtml()` obligatorio:** nombres de categoría son texto libre del usuario → XSS risk
- **Variables CSS en `:root`:** sistema de colores actual está 100% en CSS; las categorías dinámicas necesitan paleta predefinida
- **Firestore Security Rules:** `allow read, write: if request.auth.uid == userId` — ya cubre subcolecciones bajo `users/{uid}/`
- **DM Mono** para labels/badges; **DM Sans** para inputs y texto; respetar `--radius`, `--radius-sm`, `--radius-pill`
- **Filosofía visual:** minimalista, Apple-like, sin decoración innecesaria

---

## 4. Riesgos y conflictos detectados

### 🔴 Colores dinámicos vs. CSS estático
Las clases `.cat-karate` etc. no pueden existir para categorías creadas en runtime.
**Solución:** paleta predefinida numerada (10 opciones), el usuario elige un color al crear.
CSS define `.palette-0` a `.palette-9` con variantes dark/light.

### 🟡 Migración de datos del usuario admin
m4verick tiene tareas con `cat: 'karate'` etc. La nueva colección `categories/{catId}`
debe usar los mismos valores como IDs: `setDoc(doc(..., 'karate'), {...})`.
Así las tareas existentes no requieren ningún cambio.

### 🟡 Nuevos usuarios — categorías de ejemplo
Al primer login sin categorías: crear "trabajo" y "personal" con IDs autogenerados.
Esto es diferente al seed de m4verick (que usa IDs explícitos).

### 🟡 selectedCat inicializado como `'karate'`
Debe pasar a `null` e inicializarse con el primer ID de `categories[]` al cargar.

### 🟡 Dos listeners Firestore paralelos
Actualmente hay uno (`unsubscribe`). Necesita `unsubTasks` + `unsubCats`.
Ambos deben cancelarse en logout. `selectedCat` debe resetearse a `null` en logout.

### 🟡 No se puede eliminar categoría con tareas — requiere consulta
Antes de permitir eliminar, verificar `tasks.filter(t => t.cat === id).length > 0`.
Si hay tareas → bloquear con mensaje explicativo (no `confirm()`).

### 🟢 Pantalla de ajustes — nueva ruta de UI
La app actualmente tiene una sola pantalla (`#login-screen` / `#app`).
Se agrega un tercer estado: `#settings-screen` o modo ajustes dentro de `#app`.
El header sticky debe mostrar el ⚙ solo cuando el usuario está logueado.

---

## 5. Recomendaciones para los agentes

### Para el Story Writer
El Ingeniero Jefe ya definió todas las decisiones de producto necesarias:
- Punto de entrada: ⚙ en el header
- UI de gestión: pantalla/modo de ajustes con lista inline
- Migración m4verick: automática y transparente
- Usuarios nuevos: 2 categorías de ejemplo ("trabajo", "personal")
- Sin límite de categorías
- No eliminar si tiene tareas → bloquear con mensaje
- Estilo: minimalista Apple (Steve Jobs era)

### Para el Spec Writer
- Nueva colección Firestore: `users/{uid}/categories/{catId}`
- Campos: `name` (string), `colorIndex` (0–9), `createdAt` (number)
- Seed admin: `setDoc` con IDs explícitos (`'karate'`, `'facultad'`, etc.)
- Seed nuevos usuarios: `addDoc` para "trabajo" (colorIndex: 0) y "personal" (colorIndex: 4)
- Ambos seeds deben ser idempotentes
- Nueva variable JS: `categories = []`, `unsubCats = null`
- `selectedCat` reseteado en logout
- Paleta CSS: 10 colores numerados con variantes dark/light
- Pantalla de ajustes: tercer estado de la app (o panel dentro de `#app`)

### Para los Builders
- Detectar si usuario es nuevo: `categories.length === 0` en primer disparo del listener
- Distinguir seed admin (IDs hardcodeados) vs. seed nuevos usuarios (IDs autogenerados):
  usar `user.uid` específico o simplemente aplicar seed a cualquier usuario nuevo (sin categorías)
- La pantalla de ajustes debe ser accesible desde el ⚙ del header y tener botón "← volver"
- Gestión inline: cada fila muestra dot de color + nombre + ✎ editar + ✕ eliminar (si no tiene tareas)

---

## Archivos a modificar

| Archivo | Secciones |
|---|---|
| `web/index.html` | CSS (paletas), HTML (selector, nav, settings screen), JS (estado, listeners, CRUD, render) |
| `docs/documentacion.md` | Modelo de datos, estado JS, arquitectura |
