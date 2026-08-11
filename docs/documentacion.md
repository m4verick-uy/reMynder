# ReMynder — Documentación técnica

## Qué es

App web de gestión de tareas personales con autenticación Google. Cada usuario ve y gestiona únicamente sus propias tareas, organizadas en categorías dinámicas creadas por él mismo, filtrables por estado y categoría. Datos sincronizados en tiempo real entre dispositivos.

**URL de producción:** `https://remynder.vercel.app`

---

## Stack tecnológico

| Capa | Tecnología |
|---|---|
| Frontend | HTML + CSS + JavaScript vanilla (sin build, sin bundler) |
| Auth | Firebase Authentication — proveedor Google (OAuth 2.0) |
| Base de datos | Cloud Firestore (NoSQL, tiempo real) |
| Hosting | Vercel (deploy estático de archivo único) |
| Fuentes | Google Fonts — DM Sans + DM Mono |
| Firebase SDK | CDN `gstatic.com` v10.12.0 (módulos ES) |

No hay framework, no hay npm, no hay proceso de build. Un único `web/index.html` que el browser ejecuta directamente.

---

## Estructura del proyecto

```
reMynder/
├── web/
│   └── index.html        # Toda la app: HTML + CSS + JS
├── docs/
│   └── documentacion.md  # Este archivo
├── .agents/              # Agentes de la Software Factory
├── vercel.json           # Rewrite: /* → /web/index.html
└── .vercel/
    └── project.json      # Linkeo Vercel (projectId, orgId)
```

---

## Arquitectura

```
Browser
  │
  ├── web/index.html (todo el código)
  │     ├── CSS con variables para theming dark/light
  │     ├── HTML: #login-screen + #app + #settings-screen
  │     └── JS módulo:
  │           ├── Firebase Auth (Google login)
  │           ├── Firestore (categorías + tareas en tiempo real)
  │           └── Lógica UI (filtros, render, ajustes)
  │
  ├── Firebase Auth ──── Google OAuth ──── Google Cloud
  │
  └── Firestore (base de datos)
        └── users/{uid}/
              ├── tasks/{taskId}
              └── categories/{catId}
```

No hay backend propio. Firebase actúa como BaaS (Backend as a Service).

---

## Autenticación

### Flujo de login

1. Al cargar la página, `onAuthStateChanged` determina si hay sesión activa.
2. Si no hay sesión → muestra `#login-screen`, oculta `#app` y `#settings-screen`.
3. Si hay sesión → muestra `#app`, inicia `startListeners(uid)`.
4. El usuario hace clic en "Continuar con Google" → `signInWithPopup`.
5. Si el popup está bloqueado → fallback a `signInWithRedirect`.
6. Tras auth exitosa, `onAuthStateChanged` dispara con el usuario autenticado.

### Gestión de errores de auth

| Código de error | Comportamiento |
|---|---|
| `auth/popup-blocked` | Cae a `signInWithRedirect` automáticamente |
| `auth/popup-closed-by-user` | Muestra mensaje "Cerraste el popup. Intentá de nuevo." |
| `auth/cancelled-popup-request` | Igual que el anterior |
| Cualquier otro | Muestra `Error: <código>` en rojo bajo el botón |

### Datos del usuario disponibles post-login

```javascript
user.uid          // ID único del usuario (clave en Firestore)
user.displayName  // Nombre completo de Google
user.email        // Email
user.photoURL     // URL del avatar de Google
```

### Configuración requerida en servicios externos

**Firebase Console** (`console.firebase.google.com` → proyecto `pendientes-2c0ea`):
- Authentication → Settings → Authorized Domains → debe incluir el dominio de producción

**Firestore Security Rules** (Firebase Console → Firestore → Rules):
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/tasks/{taskId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
    match /users/{userId}/categories/{catId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

---

## Modelo de datos (Firestore)

### Colección: tareas

```
users/
  {uid}/
    tasks/
      {taskId}/
        text:      string   — texto de la tarea
        done:      boolean  — completada o no
        cat:       string   — ID de la categoría (catId)
        createdAt: number   — timestamp Unix (Date.now())
```

### Colección: categorías

```
users/
  {uid}/
    categories/
      {catId}/
        name:       string  — nombre de la categoría (texto libre del usuario)
        colorIndex: number  — índice de color 0–9 (paleta predefinida)
        createdAt:  number  — timestamp Unix (Date.now())
```

El campo `cat` de cada tarea almacena el `catId` del documento de categoría correspondiente. Para el usuario admin, los `catId` coinciden con los nombres históricos (`'karate'`, `'facultad'`, etc.) para cero migración de datos.

### Seed inicial de categorías

- **Usuario admin** (`ADMIN_UID` en el código): recibe sus 6 categorías con IDs explícitos vía `setDoc + merge:true` (idempotente).
- **Usuarios nuevos** (sin categorías en primer login): reciben "trabajo" (colorIndex: 0) y "personal" (colorIndex: 4) vía `addDoc`. Editables y eliminables.
- La detección es: primer disparo del listener de categorías con `categories.length === 0`.

---

## Sincronización en tiempo real

Se usan **dos listeners `onSnapshot`** paralelos por sesión:

```javascript
function startListeners(uid) {
  // Listener de categorías
  const qCats = query(collection(db, 'users', uid, 'categories'), orderBy('createdAt'));
  unsubCats = onSnapshot(qCats, snap => {
    categories = snap.docs.map(d => ({ id: d.id, ...d.data() }));
    // seed si es primer login sin categorías
    // actualiza selector, filtros y lista
  });

  // Listener de tareas
  const qTasks = query(collection(db, 'users', uid, 'tasks'), orderBy('createdAt'));
  unsubTasks = onSnapshot(qTasks, snap => {
    tasks = snap.docs.map(d => ({ id: d.id, ...d.data() }));
    render();
  });
}
```

Ambos listeners se cancelan en logout. Las variables `unsubCats` y `unsubTasks` son module-level.

---

## Operaciones sobre tareas

| Función | Operación Firestore | Descripción |
|---|---|---|
| `addTask()` | `addDoc` | Lee el input, usa `selectedCat` como categoría, escribe en Firestore |
| `toggleTask(id)` | `updateDoc` | Invierte el campo `done` |
| `updateTask(id, text, cat)` | `updateDoc` | Actualiza `text` y `cat`; valida que el texto no esté vacío. No toca `done`/`status`/`createdAt` |
| `deleteTask(id)` | `deleteDoc` | Elimina el documento |
| `clearDone()` | `writeBatch` | Elimina todas las tareas con `done: true` en una sola operación atómica |

## Operaciones sobre categorías

| Función | Operación Firestore | Descripción |
|---|---|---|
| `createCategory(name, colorIndex)` | `addDoc` | Crea categoría con nombre y color; valida que nombre no esté vacío |
| `updateCategory(id, name, colorIndex)` | `updateDoc` | Actualiza nombre y color; valida nombre |
| `deleteCategory(id)` | `deleteDoc` | Elimina si no tiene tareas; pide confirmación nativa |
| `seedCategories(uid)` | `setDoc+merge` / `addDoc` | Seed inicial según si es admin o usuario nuevo |

---

## Estado de la UI (variables JS)

```javascript
let tasks           = []     // array local de tareas (sincronizado por onSnapshot)
let categories      = []     // array local de categorías (sincronizado por onSnapshot)
let filter          = 'all'  // 'all' | 'pending' | 'done'
let catFilter       = null   // null | catId — filtro activo de categoría
let selectedCat     = null   // catId activo para nuevas tareas (null hasta primer snapshot)
let unsubTasks      = null   // función para cancelar listener de tareas
let unsubCats       = null   // función para cancelar listener de categorías
let categoriesReady = false  // true después del primer snapshot de categorías
let editingCatId    = null   // catId en edición inline, 'new' si se está creando, null si ninguno
let editingTaskId   = null   // taskId en edición inline, null si ninguna
let viewMode        = localStorage.getItem('viewMode') || 'board'  // 'board' | 'focus' | 'list' — persiste entre sesiones
```

No hay framework de estado. El estado vive en variables de módulo y `render()` recalcula todo desde `tasks` con los filtros activos.

---

## Funciones de renderizado

| Función | Qué hace |
|---|---|
| `render()` | Dispatcher según `viewMode` (`'board'` / `'focus'` / `'list'`); actualiza contadores y botón "limpiar" antes de delegar en `renderBoard()`/`renderFocus()`/`renderList()` |
| `renderBoardColumnsHtml(visible)` | Genera el HTML de las 3 columnas (por hacer/haciendo/hecho) — compartida entre `renderBoard()` y `renderFocus()`, mismo template de columna/tarjeta en ambas vistas |
| `renderBoard()` | Vista Tablero — 3 columnas en fila horizontal, carrusel + dots en mobile |
| `renderFocus()` | Vista Focus — mismas 3 columnas que Tablero, reacomodadas en grid (por hacer arriba a todo el ancho, haciendo/hecho abajo 50/50); "hecho" con opacidad reducida. Drag & drop vía el mismo `initBoardDnd()` que Tablero, sin carrusel/dots |
| `renderCatSelector()` | Genera los pills de categoría en el header para elegir categoría de nueva tarea |
| `renderNavCats()` | Genera los botones de filtro de categoría en la nav |
| `renderSettings()` | Genera la lista de categorías en la pantalla de ajustes |
| `renderCatForm(id, name, colorIndex)` | Genera el formulario inline de crear/editar categoría |
| `renderTaskForm(id, text, cat)` | Genera el formulario inline de editar tarea (texto + picker de categoría), usado tanto en lista como en Board |
| `renderCatPicker(currentCatId)` | Helper de `renderTaskForm`: lista de categorías seleccionables dentro del form de edición de tarea |
| `getCatBadge(catId)` | Helper que devuelve el HTML del badge de una tarea (busca en `categories[]`) |

El render es completo cada vez (no diff). Funciona bien dado el volumen esperado de tareas.

---

## Sistema de pantallas

La app tiene tres estados de UI excluyentes:

```
#login-screen   — pantalla de login (visible cuando no autenticado)
#app            — pantalla principal con tareas (visible cuando autenticado)
#settings-screen — pantalla de ajustes de categorías (visible desde ⚙ en el header)
```

Los tres divs empiezan con `display:none`. `onAuthStateChanged` decide cuál mostrar al cargar. `showSettings()` / `hideSettings()` gestiona la transición entre `#app` y `#settings-screen`.

### Estructura HTML de #settings-screen

```
#settings-screen (display:none por defecto)
  <header> (sticky)
    .header-top
      #settings-back-btn.back-btn   ← volver
      h2.settings-title             ajustes
  <main class="settings-main">
    .settings-section-title         categorías
    #categories-list                ← renderizado por renderSettings()
    #add-category-btn               + nueva categoría
```

---

## Sistema de colores (paleta predefinida)

Las categorías ya no usan variables CSS por nombre. El campo `colorIndex` (0–9) apunta a una paleta numerada:

| Índice | Dark (texto / fondo) | Light (texto / fondo) | Historia |
|---|---|---|---|
| 0 | #F0997B / #4A1B0C | #D85A30 / #FAECE7 | naranja-rojo |
| 1 | #85B7EB / #042C53 | #185FA5 / #E6F1FB | azul |
| 2 | #5DCAA5 / #04342C | #0F6E56 / #E1F5EE | verde-teal |
| 3 | #F09595 / #501313 | #A32D2D / #FCEBEB | rojo |
| 4 | #AFA9EC / #26215C | #534AB7 / #EEEDFE | violeta |
| 5 | #EF9F27 / #412402 | #854F0B / #FAEEDA | ámbar |
| 6 | #EC8EC4 / #3D1030 | #B52B7A / #FDE8F4 | rosa |
| 7 | #67D4E8 / #053140 | #0F6C82 / #E0F7FA | cian |
| 8 | #A3D96C / #1D3508 | #4A7C10 / #EFF7E0 | verde-lima |
| 9 | #7B9EEB / #0D1A4A | #2840A0 / #E8EDFC | índigo |

Las clases CSS son `.cat-p0` a `.cat-p9`. El alias `--salud: var(--p3)` se preserva para backward compatibility con estilos de chrome (botón salir, limpiar completadas).

---

## Seguridad XSS

Todo texto de usuario que se inserta en el DOM via `innerHTML` pasa por `escHtml()`:

```javascript
function escHtml(str) {
  return str
    .replace(/&/g,'&amp;')
    .replace(/</g,'&lt;')
    .replace(/>/g,'&gt;')
    .replace(/"/g,'&quot;');
}
```

En `renderNavCats()` se usa `btn.textContent = cat.name` (seguro por diseño, sin necesidad de escaping).

---

## Sistema de temas (dark / light)

Variables CSS en `:root` (dark) y `html[data-theme="light"]` (override). Toggle manual, persistido en `localStorage`. Script síncrono en `<head>` aplica el tema antes del render (anti-FOUC).

| Variable | Uso |
|---|---|
| `--bg` | Fondo de página |
| `--surface` | Fondo de cards, header, footer |
| `--border` | Bordes sutiles |
| `--border-md` | Bordes medianos (inputs, badges) |
| `--text` | Texto principal |
| `--muted` | Texto secundario |
| `--subtle` | Texto terciario / placeholders |
| `--p0`–`--p9` | Texto de cada color de categoría |
| `--p0-bg`–`--p9-bg` | Fondo de cada color de categoría |
| `--salud` | Alias para `var(--p3)` — backward compat |

---

## Event listeners

Centralizados al final del módulo JS. Se usa **delegación de eventos** donde los elementos son dinámicos:

| Contenedor | Datos leídos | Acción |
|---|---|---|
| `#task-list` | `data-action`, `data-id` | toggle / delete / edit-task / cancel-task / select-task-cat / confirm-task |
| `#board` | `data-action`, `data-id`, `data-status` | move-prev / move-next / delete / edit-task / cancel-task / select-task-cat / confirm-task (además de drag & drop nativo para mover entre columnas) |
| `#categories-list` | `data-action`, `data-id`, `data-color` | edit-cat / delete-cat / confirm-cat / cancel-cat / select-color |
| `nav` | `data-cat-filter` | setCatFilter |
| `#cat-selector` | `data-cat` | selectCat |
| `.view-switch-btn` (uno por botón, no delegado) | `data-view` | `setViewMode('board' \| 'focus' \| 'list')` |

---

## Deploy

Hosting en **Vercel** como sitio estático. El `vercel.json` redirige todas las rutas a `web/index.html`:

```json
{ "rewrites": [{ "source": "/(.*)", "destination": "/web/index.html" }] }
```

```bash
vercel --prod  # deploy a producción
```

Proyecto linkeado via `.vercel/project.json`:
```json
{
  "projectId":   "prj_N8LDIc2p7vuoR2I0MSLRpo2WZQkm",
  "orgId":       "team_KnSP45ZtOw9ajYFt2IYJFVzc",
  "projectName": "remynder"
}
```

---

## Desarrollo local

Firebase Auth bloquea `file://`. Usar servidor HTTP:

```bash
python3 -m http.server 3000
# o: npx serve .
```

Agregar `localhost` a Authorized Domains en Firebase Console.

---

## Servicios externos y credenciales

### Firebase (proyecto: `pendientes-2c0ea`)

```javascript
const firebaseConfig = {
  apiKey:            "AIzaSyAQM7IW_7XxpCXTJFDZGxUJye8M27O-8zc",
  authDomain:        "pendientes-2c0ea.firebaseapp.com",
  projectId:         "pendientes-2c0ea",
  storageBucket:     "pendientes-2c0ea.firebasestorage.app",
  messagingSenderId: "656621685090",
  appId:             "1:656621685090:web:029c4da97c27a805b0843b"
};
```

El `apiKey` de Firebase para apps web **no es un secreto**. La seguridad está en las **Firestore Security Rules**.

---

## Limitaciones conocidas (deuda técnica)

- **Sin orden personalizable**: las tareas siempre se muestran por fecha de creación.
- **Sin paginación**: si el usuario tiene muchas tareas, todas se cargan y renderizan.
- **Render completo**: cada cambio re-renderiza toda la lista (no diff/virtual DOM).
- **Feedback silencioso sin categorías**: si el usuario elimina todas sus categorías e intenta agregar una tarea, el input queda con texto pero no pasa nada (sin mensaje de error visible).
- **Tema solo accesible logueado**: el botón de tema está dentro de `#app`; en login el tema se aplica pero no hay botón visible.
