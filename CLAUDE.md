# CLAUDE.md — ReMynder

Este archivo es el contexto compartido de todos los agentes que trabajan en este proyecto.
Léelo completo antes de hacer cualquier tarea. No asumas nada que no esté acá.

---

## Qué es ReMynder

App web de gestión de tareas personales con autenticación Google. Nació como lista de pendientes
personal y evoluciona hacia una **app web + iOS comercial**. Es el producto principal de una futura
empresa de software personal fundada en Uruguay, orientada a calidad Apple-like.

**URL de producción:** https://remynder.vercel.app
**Repositorio:** https://github.com/m4verick-uy/reMynder

---

## Visión del producto

- Empezó como to-do personal, se convierte en producto comercial
- Targets: app web (ya existe) + app iOS nativa (próxima etapa)
- Filosofía de producto: **calidad sobre velocidad**, diseño cuidado, experiencia pulida
- Referente de calidad: Apple (simplicidad, atención al detalle, consistencia)
- ReMynder es la punta de lanza de un ecosistema de productos

---

## Stack actual

| Capa | Tecnología |
|---|---|
| Frontend | HTML + CSS + JavaScript vanilla (sin framework, sin build) |
| Auth | Firebase Authentication — Google OAuth 2.0 |
| Base de datos | Cloud Firestore (NoSQL, tiempo real) |
| Hosting | Vercel (deploy estático) |
| Fuentes | Sin Google Fonts — pila de fuente de sistema (`-apple-system, BlinkMacSystemFont, 'SF Pro Text', system-ui, sans-serif`) |
| Firebase SDK | CDN gstatic.com v10.12.0 (módulos ES) |

**No hay backend propio. No hay npm. No hay proceso de build.**
Un único `web/index.html` que el browser ejecuta directamente.

---

## Estructura del proyecto (estado actual)

```
reMynder/
├── web/
│   ├── index.html         # Toda la app: HTML + CSS + JS en un único archivo
│   └── assets/            # Assets estáticos
├── docs/                  # Documentación técnica, session-log.md, y docs por feature
│                          # (research/stories/spec/test-report/validation-{feature}.md)
├── .agents/                # Definiciones de los agentes de la Software Factory
├── .vercel/
│   └── project.json       # Linkeo Vercel (projectId, orgId) — gitignored
├── vercel.json
├── README.md
└── CLAUDE.md
```

---

## Modelo de datos (Firestore)

```
users/
  {uid}/
    tasks/
      {taskId}/
        text:      string   — texto de la tarea
        done:      boolean  — completada o no
        cat:       string   — categoría (ver abajo)
        createdAt: number   — timestamp Unix (Date.now())
```

### Categorías actuales (hardcodeadas)

`karate` | `facultad` | `antel` | `salud` | `personal` | `padres`

Estas categorías son personales del creador. En la versión comercial serán **categorías dinámicas
creadas por el usuario**.

---

## Convenciones de código

### JavaScript
- Vanilla ES modules, sin transpilación
- Funciones async/await para todas las operaciones Firestore
- Estado en variables de módulo (`tasks`, `filter`, `catFilter`, `selectedCat`, `unsubscribe`)
- `render()` recalcula toda la UI desde el estado (no diff)
- Delegación de eventos en la lista de tareas (no listeners por ítem)
- Sanitización XSS con `escHtml()` antes de todo `innerHTML`

### CSS

**Regla de jerarquía visual (no negociable):** un control secundario no debe ser
descendiente de un elemento con tratamiento visual de acción primaria. Si comparten
contenedor, comparten herencia, y separarlos después cuesta más que estructurarlos
bien desde el principio.

Motivo: un selector como `.add-row button` (descendiente, sin `>`) captura *todo*
`<button>` anidado en `.add-row`, sin importar cuántos niveles de profundidad —
incluyendo controles secundarios que viven dentro de un wrapper hijo (`.input-wrap`,
un menú desplegable, etc.). Como esa clase de acción primaria suele tener más
especificidad que la clase propia del control secundario, gana la cascada aunque el
control secundario declare su propio `background`. Esto costó varias rondas de
debugging (el bug real vivía en el selector del contenedor, no en el componente).
Al escribir CSS para un botón de acción primaria, usar `>` (hijo directo) salvo que
el selector deba aplicar a descendientes intencionalmente — y si un control
secundario debe vivir dentro del mismo wrapper que uno primario, verificar
explícitamente que ningún selector del primario lo alcance por descendencia.

Ejemplo real en el código: `.add-row > button` (`web/index.html`).

- Variables CSS en `:root` para theming (dark por defecto, `html[data-theme="light"]` override)
- Superficies: `--bg`, `--surface`, `--menu-bg`, `--menu-highlight`
- Bordes: `--border` (hairline translúcido), `--border-md` (sólido, algo más marcado),
  `--border-card` (borde de `.task-item`, `.board-card` y el input de "Nueva tarea..."
  — en dark theme igual a `--border-md`, en light theme un gris más marcado porque
  `--border-md` queda demasiado sutil sobre `--surface` blanco)
- Texto derivado del fondo, no del componente: `--text` (primario), `--muted` (secundario,
  translúcido), `--subtle` (terciario — iconos/dots inactivos, texto tachado), `--fill`
  (relleno translúcido para hover/estado neutro, ej. `.nav-btn` inactivo)
- Acento — **único token de marca**: `--accent` (+ `--accent-fill` para fondos translúcidos,
  `--accent-contrast` para texto/ícono encima de `--accent`). `--accent-contrast` se recalibra
  por tema porque el mismo accent no da contraste AA en ambos fondos (dark: contraste oscuro
  sobre teal claro; light: blanco sobre teal oscuro). Cambiar `--accent` en los dos bloques
  (`:root` y `html[data-theme="light"]`) alcanza para toda la UI — no hay accent hardcodeado
  fuera de esos dos lugares
- Paleta de categoría: `--p0` a `--p9`, mapeo fijo por índice — no por nombre de variable como
  antes (`--p2` = antel, `--p3` = salud, etc.; ver comentario `seedCategories` en
  `web/index.html` para el mapeo completo). El color de categoría vive *solo* en dots
  (`.cat-dot`, `.cat-select-dot`, `.color-dot`, `.badge::before`) y nunca en píldoras de
  filtro (`.nav-btn`), que son siempre mono-color (acento si está activa, `--fill` si no)
- Radios: `--radius` (10px), `--radius-sm` (8px), `--radius-pill` (999px)
- Clases de componentes por área — lista no exhaustiva, `web/index.html` es la fuente de verdad:
  - Login: `.login-screen`, `.login-card`, `.login-title`, `.login-btn`
  - Header/nav: `header`, `.user-info`, `.nav-btn` (`.active`)
  - Agregar tarea: `.add-row`, `.input-wrap`, `.cat-select-btn` (`.has-value`), `.cat-menu`,
    `.cat-menu-item` (`.selected`)
  - Lista: `.task-item` (`.done`), `.check-btn` (`.done`), `.badge`
  - Board (Kanban): `.board`, `.board-col` (`.empty-col`, `.drag-over`), `.board-card`
    (`.dragging`), `.board-dots`
  - Ajustes/categorías: `#categories-list`, `.cat-row`, `.color-picker`, `.color-dot` (`.selected`)
- Layout: `.add-row`, `nav` y `.board` se centran a `max-width: 1200px` en viewports ≥1200px
  vía media query; `header`/`footer` quedan siempre a ancho completo (patrón Apple)

### HTML
- Tres root divs: `#login-screen`, `#app` y `#settings-screen` — los tres `display:none` al inicio
- `onAuthStateChanged` decide cuál mostrar (evita FOUC de autenticación)
- Anti-FOUC de tema: script síncrono en `<head>` aplica `data-theme` antes del render

### Tipografía
- Fuente única de sistema (`--font`), sin Google Fonts: `-apple-system, BlinkMacSystemFont,
  'SF Pro Text', system-ui, sans-serif` — no hay distinción de familia por tipo de texto
- La jerarquía tipográfica se logra con tamaño/peso/color (`--text`/`--muted`/`--subtle`), no
  con fuentes distintas

---

## Reglas de seguridad (Firestore)

```
match /users/{userId}/tasks/{taskId} {
  allow read, write: if request.auth != null && request.auth.uid == userId;
}
```

Cada usuario solo puede leer y escribir sus propias tareas. Esta regla NO debe relajarse.

---

## Limitaciones conocidas (deuda técnica)

Estas son áreas de mejora conocidas y aceptadas. No son bugs, son decisiones de diseño iniciales:

1. **Categorías hardcodeadas** — agregar una nueva requiere cambios en JS, CSS y HTML
2. **Sin orden personalizable** — siempre por `createdAt` ascendente
3. **Sin paginación** — todas las tareas se cargan y renderizan juntas
4. **Render completo** — cada cambio re-renderiza toda la lista
5. **Todo en un archivo** — `index.html` contiene HTML + CSS + JS

---

## Roadmap de producto (dirección conocida)

Estas son las próximas evoluciones del producto. Cualquier decisión técnica debe ser compatible
con este roadmap:

- [ ] Categorías dinámicas creadas por el usuario (no hardcodeadas)
- [ ] Edición de tareas existentes
- [ ] Fechas de vencimiento y recordatorios
- [ ] App iOS nativa
- [ ] Modelo freemium / monetización
- [ ] Nombre comercial: **ReMynder** (el repo actualmente se llama "pendientes")

---

## Servicios externos

### Firebase (proyecto: `pendientes-2c0ea`)
- Console: console.firebase.google.com
- Auth: Firebase Authentication con proveedor Google
- DB: Cloud Firestore
- **El `apiKey` de Firebase es público por diseño** — la seguridad está en las Security Rules

### Vercel
- Proyecto: `pendientes` (projectId: `prj_N8LDIc2p7vuoR2I0MSLRpo2WZQkm`)
- **Integración Git↔Vercel activa desde 2026-07-20** (GitHub `m4verick-uy/reMynder` conectado).
  **Production Branch confirmada: `main`** (verificado vía API — `link.productionBranch: "main"`).
  Cada push a `main` o `develop` dispara un build automático. **Ya NO se necesita `vercel --prod`
  ni `vercel` manual** — el flujo viejo (deploy manual vía CLI) quedó obsoleto con esta conexión.
- No requiere configuración adicional (`vercel.json`, `package.json`)
- **Entorno de desarrollo:** branch `develop`. Push automático → preview en
  **https://remynder-dev.vercel.app**, registrado como Domain del proyecto con
  `gitBranch: "develop"` (vía API `POST /v10/projects/{id}/domains`), así que siempre trackea el
  último deploy de `develop` sin re-aliasing manual. Merge a `main` + push → producción automática.
  Comparte el mismo proyecto Firebase/Firestore que producción (sin aislamiento de datos) —
  decisión tomada el 2026-07-15 por simplicidad, dado que es un solo usuario (el creador).
  `remynder-dev.vercel.app` ya está en Authorized Domains en Firebase Console.
- **Regla de oro para cualquier agente:** antes de asumir que hay que deployar manualmente,
  correr `vercel project inspect remynder` o revisar Settings → Git en el dashboard para
  confirmar si la integración sigue activa y cuál es la Production Branch. Si esto cambia de
  nuevo, actualizar esta sección — no dejar que quede desactualizada otra vez.

---

## Desarrollo local

```bash
# Sin servidor de desarrollo requerido.
# Firebase Auth bloquea file://, usar servidor HTTP:

python3 -m http.server 3000
# o
npx serve .

# Luego agregar localhost a Authorized Domains en Firebase Console
```

---

## Estándares de calidad (NO negociables)

Estos estándares aplican a TODO el trabajo en este proyecto:

1. **El código debe ser legible** — cualquier dev debe entenderlo sin preguntar
2. **Sin regresiones** — cada cambio debe preservar el comportamiento existente
3. **Seguridad primero** — sanitización XSS en todo `innerHTML`, Security Rules intactas
4. **Diseño consistente** — usar las variables CSS existentes, no inventar nuevas sin justificación
5. **Mobile-first** — la app debe funcionar perfectamente en móvil
6. **Sin dependencias innecesarias** — agregar una lib requiere justificación explícita
7. **Documentar cambios** — actualizar `documentacion.md` con cada cambio arquitectural

---

## Para los agentes: cómo trabajar en este proyecto

- **Leer este archivo completo** antes de cualquier tarea
- **Preguntar antes de asumir** si algo no está claro aquí
- **Respetar el stack actual** — no proponer migraciones a frameworks sin pedido explícito
- **Pensar en el roadmap** — toda solución debe ser compatible con la evolución prevista
- **Priorizar calidad** — este producto tiene aspiraciones comerciales, no es un prototipo

---

## Comando de ayuda

Cuando el usuario escriba "workflow", mostrar el flujo de trabajo 
completo de la Software Factory tal como está documentado en este archivo.

---

## Flujo de trabajo — Software Factory

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  COMANDOS DE SESIÓN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

INICIO (siempre primero):
"Actúa como Session Start de ReMynder"

CIERRE (siempre al final):
"Actúa como Session End de ReMynder"

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TIPOS DE TAREA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

FEATURE — algo nuevo para el usuario:
"Actúa como Orchestrator de ReMynder.
Feature: [descripción]"

HOTFIX — algo roto:
"Actúa como Orchestrator de ReMynder.
Hotfix [visual/lógico/datos]: [descripción]
Límites: [qué NO tocar]"

REFACTOR — mejorar sin cambiar comportamiento:
"Actúa como Orchestrator de ReMynder.
Refactor: [qué y por qué]
Límites: el comportamiento visible no debe cambiar"

RESEARCH — explorar antes de decidir:
"Actúa como Orchestrator de ReMynder.
Research: [pregunta]
Solo análisis, sin implementar nada."

DECISION — criterio antes de actuar:
"Actúa como Orchestrator de ReMynder.
Decisión pendiente: [contexto y opciones]
Dame tu recomendación antes de implementar."

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  REGLAS DE ORO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

- Sin Session Start no se trabaja
- Sin Validator aprobado no hay deploy
- Sin Session End no se cierra
- Toda tarea entra SIEMPRE por el Orchestrator
- Nunca directo a Claude Code sin pasar por el Orchestrator

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  AGENTES DISPONIBLES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

.agents/session-start.md
.agents/orchestrator.md
.agents/researcher.md
.agents/story-writer.md
.agents/spec-writer.md
.agents/backend-builder.md
.agents/frontend-builder.md
.agents/test-verifier.md
.agents/validator.md
.agents/session-end.md
