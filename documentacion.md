# Pendientes — Documentación técnica

## Qué es

App web de gestión de tareas personales con autenticación Google. Cada usuario ve y gestiona únicamente sus propias tareas, organizadas por categorías y filtrables por estado. Datos sincronizados en tiempo real entre dispositivos.

URL de producción: `https://pendientes-six.vercel.app`

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

No hay framework, no hay npm, no hay proceso de build. Un único archivo `index.html` que el browser ejecuta directamente.

---

## Estructura del proyecto

```
proyectos/
├── index.html          # Toda la app: HTML + CSS + JS en un único archivo
├── .vercel/
│   └── project.json    # Linkeo con proyecto Vercel (projectId, orgId)
└── documentacion.md    # Este archivo
```

---

## Arquitectura

```
Browser
  │
  ├── index.html (todo el código)
  │     ├── CSS con variables para theming dark/light
  │     ├── HTML: #login-screen + #app (uno visible a la vez)
  │     └── JS módulo:
  │           ├── Firebase Auth (Google login)
  │           ├── Firestore (tareas en tiempo real)
  │           └── Lógica UI (filtros, render, tema)
  │
  ├── Firebase Auth ──── Google OAuth ──── Google Cloud
  │
  └── Firestore (base de datos)
        └── users/{uid}/tasks/{taskId}
```

No hay backend propio. Firebase actúa como BaaS (Backend as a Service).

---

## Autenticación

### Flujo de login

1. Al cargar la página, `onAuthStateChanged` determina si hay sesión activa.
2. Si no hay sesión → muestra `#login-screen`, oculta `#app`.
3. Si hay sesión → muestra `#app`, oculta `#login-screen`, inicia listener de Firestore.
4. El usuario hace clic en "Continuar con Google" → `signInWithPopup`.
5. Si el popup está bloqueado por el browser → fallback a `signInWithRedirect`.
6. Tras auth exitosa, `onAuthStateChanged` dispara con el usuario autenticado y transiciona la UI.

### Gestión de errores de auth

| Código de error | Comportamiento |
|---|---|
| `auth/popup-blocked` | Cae a `signInWithRedirect` automáticamente |
| `auth/popup-closed-by-user` | Muestra mensaje "Cerraste el popup. Intentá de nuevo." |
| `auth/cancelled-popup-request` | Igual que el anterior |
| Cualquier otro | Muestra `Error: <código>` en rojo bajo el botón |
| Error de redirect (vuelta de Google) | Muestra `Error: <código>` si no es `auth/no-auth-event` |

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

**Google Cloud Console** (`console.cloud.google.com`):
- APIs & Services → Credentials → OAuth 2.0 Client ID
- Authorized JavaScript Origins → debe incluir `https://pendientes-six.vercel.app`

**Firestore Security Rules** (Firebase Console → Firestore → Rules):
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/tasks/{taskId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```
Cada usuario solo puede leer y escribir sus propias tareas.

---

## Modelo de datos (Firestore)

### Colección

```
users/
  {uid}/           ← ID de usuario de Firebase Auth
    tasks/
      {taskId}/    ← ID autogenerado por Firestore
        text:      string   — texto de la tarea
        done:      boolean  — completada o no
        cat:       string   — categoría (ver lista abajo)
        createdAt: number   — timestamp Unix (Date.now())
```

### Categorías disponibles

| Key | Label visible |
|---|---|
| `karate` | karate |
| `facultad` | facultad |
| `antel` | antel |
| `salud` | salud |
| `personal` | personal |
| `padres` | padres |

Las categorías están hardcodeadas en el array `CATS` del JS. Para agregar una nueva categoría hay que: (1) agregarla al array `CATS`, (2) agregar su entrada en `LABELS`, (3) agregar las variables CSS `--nueva` y `--nueva-bg` en ambos temas, (4) agregar la clase `.cat-nueva` en CSS, (5) agregar los botones en el HTML del selector y la nav.

### Query de tareas

Las tareas se ordenan por `createdAt` (ascendente) con `orderBy`. Firestore requiere un índice compuesto para queries combinadas; en este caso la query es simple (solo `orderBy`), así que no requiere índice adicional.

---

## Sincronización en tiempo real

Se usa `onSnapshot` de Firestore, que mantiene una conexión WebSocket abierta y dispara el callback cada vez que cambia cualquier documento en la colección del usuario.

```javascript
function startListener(uid) {
  if (unsubscribe) unsubscribe();  // cancela listener anterior si existe
  const q = query(collection(db, 'users', uid, 'tasks'), orderBy('createdAt'));
  unsubscribe = onSnapshot(q, snap => {
    tasks = snap.docs.map(d => ({ id: d.id, ...d.data() }));
    render();
  });
}
```

El listener se inicia al hacer login y se cancela al hacer logout (`unsubscribe()`). La variable `unsubscribe` es module-level para poder cancelarlo desde el callback de `onAuthStateChanged`.

---

## Operaciones sobre tareas

Todas son async y operan sobre Firestore directamente. La UI se actualiza sola gracias a `onSnapshot`.

| Función | Operación Firestore | Descripción |
|---|---|---|
| `addTask()` | `addDoc` | Lee el input, limpia el campo, escribe en Firestore |
| `toggleTask(id)` | `updateDoc` | Invierte el campo `done` |
| `deleteTask(id)` | `deleteDoc` | Elimina el documento |
| `clearDone()` | `writeBatch` + `batch.delete` | Elimina todas las tareas con `done: true` en una sola operación atómica |

---

## Estado de la UI (variables JS)

```javascript
let tasks       = []      // array local de tareas (sincronizado por onSnapshot)
let filter      = 'all'   // 'all' | 'pending' | 'done'
let catFilter   = null    // null | 'karate' | 'facultad' | ...
let selectedCat = 'karate'// categoría activa para nuevas tareas
let unsubscribe = null    // función para cancelar el listener de Firestore
```

No hay framework de estado. El estado vive en variables de módulo y `render()` recalcula todo desde `tasks` con los filtros activos.

---

## Función render()

Función central de la UI. Se llama cada vez que `onSnapshot` recibe datos o cambia un filtro. Recalcula todo:

1. Aplica `filter` (all/pending/done) y `catFilter` para obtener `visible[]`
2. Actualiza el contador del header (`hcount`) con tareas pendientes
3. Actualiza el contador del footer (`counter`) con `done/total`
4. Muestra/oculta el botón "limpiar completadas"
5. Renderiza el HTML de la lista via `innerHTML` (template string)

El render es completo cada vez (no diff). Funciona bien dado el volumen esperado de tareas.

### Seguridad XSS

El texto de las tareas se sanitiza con `escHtml()` antes de insertarse en el DOM via `innerHTML`:

```javascript
function escHtml(str) {
  return str
    .replace(/&/g,'&amp;')
    .replace(/</g,'&lt;')
    .replace(/>/g,'&gt;')
    .replace(/"/g,'&quot;');
}
```

---

## Sistema de temas (dark / light)

### Mecanismo

Se usa el atributo `data-theme` en el elemento `<html>` para activar uno u otro tema via CSS:

```css
:root { /* dark — default */ }
html[data-theme="light"] { /* light — override */ }
```

No se usa `@media (prefers-color-scheme)`. El tema es completamente manual y persistido en `localStorage`.

### Variables CSS

| Variable | Uso |
|---|---|
| `--bg` | Fondo de página |
| `--surface` | Fondo de cards, header, footer |
| `--border` | Bordes sutiles |
| `--border-md` | Bordes medianos (inputs, badges) |
| `--text` | Texto principal |
| `--muted` | Texto secundario |
| `--subtle` | Texto terciario / placeholders |

Cada categoría tiene dos variables: `--{cat}` (color del texto/badge) y `--{cat}-bg` (fondo del badge). Sus valores cambian entre temas.

### Prevención de flash (FOUC)

Un script síncrono en `<head>` — antes de cualquier CSS — aplica el tema antes de que el browser renderice la página:

```html
<script>
  document.documentElement.dataset.theme = localStorage.getItem('theme') || 'dark';
</script>
```

### Toggle

El botón circular con ícono sol/luna en el header llama a `toggleTheme()`:
- En dark → muestra ícono sol (click pasa a light)
- En light → muestra ícono luna (click pasa a dark)

---

## Estructura HTML

```
<html data-theme="dark|light">
  <head>
    <script> — aplica tema antes de render (anti-FOUC)
    <link>   — Google Fonts
    <style>  — todos los estilos inline
  <body>
    #login-screen   — pantalla de login (display:none por defecto)
      .login-card
        #login-sub  — texto de estado / errores
        #login-btn  — botón "Continuar con Google"

    #app            — pantalla principal (display:none por defecto)
      <header>      — sticky top
        .header-top
          h1 + #hcount
          .user-info
            #theme-btn   — toggle dark/light
            #user-avatar — foto de perfil Google
            #user-name   — nombre del usuario
            #logout-btn  — cerrar sesión
        .add-row
          #new-task    — input de nueva tarea
          #add-btn     — botón agregar
        #cat-selector — pills para elegir categoría de nueva tarea
      <nav>         — filtros de visualización
        #f-all, #f-pending, #f-done — filtros por estado
        #fc-{cat}                   — filtros por categoría
      <main>
        #task-list  — lista de tareas (renderizada por JS)
      <footer>
        #counter    — "X/Y listas"
        #clear-btn  — limpiar completadas (oculto si no hay)

    <script type="module"> — toda la lógica JS
```

Los dos root divs (`#login-screen` y `#app`) empiezan con `display:none`. `onAuthStateChanged` decide cuál mostrar. Esto evita el parpadeo de la pantalla equivocada mientras Firebase inicializa.

---

## Event listeners

Los listeners están centralizados al final del módulo JS. Para la lista de tareas se usa **delegación de eventos** (un solo listener en `#task-list` que lee `data-action` y `data-id` de los botones hijos) en lugar de listeners por cada tarea:

```javascript
document.getElementById('task-list').addEventListener('click', e => {
  const btn = e.target.closest('[data-action]');
  if (!btn) return;
  const { action, id } = btn.dataset;
  if (action === 'toggle') toggleTask(id);
  if (action === 'delete') deleteTask(id);
});
```

También se captura Enter en el input de nueva tarea para agregar sin usar el ratón.

---

## Deploy

Hosting en **Vercel** como sitio estático.

```bash
vercel        # deploy preview
vercel --prod # deploy producción
```

El proyecto está linkeado via `.vercel/project.json`:
```json
{
  "projectId": "prj_N8LDIc2p7vuoR2I0MSLRpo2WZQkm",
  "orgId":     "team_KnSP45ZtOw9ajYFt2IYJFVzc",
  "projectName": "pendientes"
}
```

Vercel sirve el `index.html` directamente sin configuración adicional. No hay `vercel.json`, no hay `package.json`.

---

## Desarrollo local

Sin servidor de desarrollo requerido. Abrir `index.html` directamente en el browser funciona, pero Firebase Auth bloquea el dominio `file://`. Para desarrollar localmente:

```bash
# Opción 1: servidor HTTP simple con Python
python3 -m http.server 3000

# Opción 2: con npx
npx serve .
```

Luego agregar `localhost` a los **Authorized Domains** en Firebase Console.

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

El `apiKey` de Firebase para apps web **no es un secreto**: está diseñado para ser público y visible en el cliente. La seguridad real está en las **Firestore Security Rules** que restringen qué operaciones puede hacer cada usuario autenticado.

---

## Limitaciones conocidas

- **Categorías hardcodeadas**: agregar una nueva requiere cambios en JS, CSS y HTML.
- **Sin edición de tareas**: solo se puede agregar, completar y eliminar.
- **Sin orden personalizable**: las tareas siempre se muestran por fecha de creación.
- **Sin paginación**: si el usuario tiene muchas tareas, todas se cargan y renderizan.
- **Render completo**: cada cambio re-renderiza toda la lista (no diff/virtual DOM).
- **Tema solo accesible logueado**: el botón de tema está dentro de `#app`; en la pantalla de login el tema se aplica pero no hay botón visible para cambiarlo.
