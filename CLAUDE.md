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
| Fuentes | Google Fonts — DM Sans + DM Mono |
| Firebase SDK | CDN gstatic.com v10.12.0 (módulos ES) |

**No hay backend propio. No hay npm. No hay proceso de build.**
Un único `index.html` que el browser ejecuta directamente.

---

## Estructura del proyecto (estado actual)

```
reMynder/
├── index.html          # Toda la app: HTML + CSS + JS en un único archivo
├── .vercel/
│   └── project.json    # Linkeo Vercel (projectId, orgId)
└── documentacion.md    # Documentación técnica del proyecto
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

- Variables CSS en `:root` para theming (dark por defecto, light override)
- Naming: `--bg`, `--surface`, `--border`, `--border-md`, `--text`, `--muted`, `--subtle`
- Cada categoría tiene `--{cat}` (texto) y `--{cat}-bg` (fondo)
- Clases de componentes: `.task-item`, `.check-btn`, `.badge`, `.cat-pill`, `.nav-btn`
- Radios: `--radius` (12px), `--radius-sm` (8px), `--radius-pill` (999px)
- ⚠️ Esta lista de variables/clases quedó desactualizada tras el rediseño visual de
  julio 2026 (acento teal, tokens de superficie separados, selector de categoría
  tipo Recordatorios de iOS). Verificar contra `web/index.html` antes de asumir
  nombres — pendiente una pasada de actualización completa de esta sección.

### HTML
- Dos root divs: `#login-screen` y `#app` — ambos `display:none` al inicio
- `onAuthStateChanged` decide cuál mostrar (evita FOUC de autenticación)
- Anti-FOUC de tema: script síncrono en `<head>` aplica `data-theme` antes del render

### Tipografía
- `DM Mono` para títulos, badges, contadores, botones secundarios
- `DM Sans` para texto de tareas, inputs, botones primarios

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
2. **Sin edición de tareas** — solo agregar, completar, eliminar
3. **Sin orden personalizable** — siempre por `createdAt` ascendente
4. **Sin paginación** — todas las tareas se cargan y renderizan juntas
5. **Render completo** — cada cambio re-renderiza toda la lista
6. **Todo en un archivo** — `index.html` contiene HTML + CSS + JS

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
