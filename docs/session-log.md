# Session Log — ReMynder

---

## Sesión 2026-06-11/12 (Debian 12)
**Entorno:** Debian 12 — topgun (x86_64)
**Duración aproximada:** ~3 horas

### Qué se hizo
- Tema Tokyo Night aplicado al dark mode (bg `#1a1b26`, texto `#c0caf5`, 10 acentos oficiales)
- Light mode reemplazado por neutro frío complementario (bg `#f0f0f5`, surface `#e4e4f0`)
- Feature Kanban board completa: toggle lista/tablero, 3 columnas (por hacer/haciendo/hecho), drag & drop con AbortController, botones ← → mobile, scroll-snap con dots indicator
- Hotfix: `class="board"` faltante en `<div id="board">` — las columnas caían verticales
- Color `#2bffdd` aplicado a headers de columnas del board (color fuente de terminal del usuario)
- Footer de atribución: "Powered by gechichure for Diamantina Labs" con mailto link
- Color `#2bffdd` en footer y headers del board para consistencia visual
- Título `remynder` → `reMynder` (capital M) + color `#2bffdd` en header y login screen
- 8 deploys a producción (`vercel --prod --force`)

### Decisiones tomadas
- **Tokyo Night light mode**: se eligió "neutro frío complementario" sobre Tokyo Night Day oficial — más limpio, mismos acentos pero en tono pastel frío
- **Kanban: toggle lista/tablero** — conviven, no reemplaza la lista
- **Kanban: drag & drop** como interacción principal, botones ← → como fallback mobile
- **Kanban: scroll horizontal snap** en mobile con dots indicator
- **Kanban: migración manual** — tareas sin `status` defaultean por lectura (`done ? 'done' : 'todo'`), sin batch migration a Firestore
- **AbortController** para cleanup de listeners DnD y dots en cada re-render del board
- **`#2bffdd`** como color de acento visual unificado (terminal font del usuario) — aplicado a títulos, headers de board y footer

### Pendiente para próxima sesión
- [ ] Validar visualmente en producción el Kanban board (drag & drop en desktop, scroll en mobile)
- [ ] Evaluar si el light mode neutro frío tiene buen contraste con `#2bffdd`
- [ ] Próxima feature del roadmap: edición de tareas o fechas de vencimiento

### Estado del repo
- Branch: main
- Último commit: `82c1a40 fix: title reMynder (capital M) + color #2bffdd`
- Deploy: ✅ múltiples `vercel --prod --force` — https://remynder.vercel.app

---

## Sesión 2026-05-28 (Debian 12)
**Entorno:** Debian 12
**Duración aproximada:** sesión larga (~4-5 hs)

### Qué se hizo

**Feature: Categorías dinámicas** (continuación y cierre)
- Spec Writer: creado `docs/spec-categorias-dinamicas.md`
- Backend Builder: implementado CRUD de categorías en Firestore + listeners + seed en `web/index.html`
- Frontend Builder: implementado CSS paleta 10 colores + pantalla de ajustes
- Test Verifier: `docs/test-report-categorias-dinamicas.md` (⚠️ Aprobado con observaciones)
- Validator: `docs/validation-categorias-dinamicas.md` (✅ Aprobado para deploy)
- Deploy a producción con `vercel --prod`

**Hotfixes visuales (post-deploy)**
- Dark mode badge contrast: `html[data-theme="dark"] .cat-pN` con color vibrante como fondo
- Reversión de light theme accidental: eliminadas reglas `html[data-theme="light"] .cat-pN` redundantes
- Light mode surface: ajuste iterativo de `--surface` → `#E2DDD6` (contraste visible sobre `#FFFFFF`)
- Light mode nav pills: fix de especificidad CSS — `.nav-btn` pisaba `.cat-pN` en cascade; resuelto con `html[data-theme="light"] .nav-btn.cat-pill:not(.active).cat-pN` + `opacity: 1`
- Light mode `--bg`/`--surface` swap: página blanca (`#FFFFFF`), cards cálidas (`#E2DDD6`)

**Documentación de la Software Factory**
- `CLAUDE.md`: sección "Flujo de trabajo" reescrita como cheatsheet con separadores visuales
- `CLAUDE.md`: comando de ayuda `workflow` para mostrar el cheatsheet
- `.agents/orchestrator.md`: sección "Hotfix — flujo abreviado" agregada
- `.agents/orchestrator.md`: sección "Tipos de tareas y formatos" con ejemplos reales (FEATURE, HOTFIX, REFACTOR, RESEARCH, DECISION)
- `CLAUDE.md`: "Regla de entrada única" — toda tarea entra por el Orchestrator

### Decisiones tomadas

- **Paleta numerada (p0–p9)**: reemplaza las variables CSS por nombre de categoría (`--karate`, etc.) — permite categorías dinámicas sin migración
- **`--bg`/`--surface` swap en light mode**: la arquitectura correcta es página blanca, surfaces cálidas (no al revés)
- **Especificidad CSS para dark mode**: `html[data-theme="dark"] .cat-pN` en lugar de override de light mode — no tocar light theme para arreglar dark
- **`vercel --prod --force`**: necesario cuando el edge CDN de Vercel cachea versión vieja (`x-vercel-cache: HIT`)

### Pendiente para próxima sesión
- [ ] Validar visualmente en producción el light mode (cards + nav pills) con el usuario
- [ ] Próxima feature del roadmap: edición de tareas o fechas de vencimiento
- [ ] Evaluar si `--surface: #E2DDD6` es el tono definitivo o necesita ajuste fino

### Estado del repo
- Branch: main
- Último commit: `58a3a22 docs: set workflow as help command`
- Deploy: ✅ `vercel --prod --force` — https://remynder.vercel.app

---

## Sesión 2026-05-27 16:26 (Debian 12)
**Entorno:** Debian 12 — topgun (x86_64)
**Duración aproximada:** ~4 horas

### Qué se hizo
- Reorganización del proyecto: `index.html` → `web/`, `documentacion.md` → `docs/`
- Copia de `CLAUDE.md` desde `~/Descargas/`
- Creación de `README.md`
- Renombrado del proyecto de "pendientes" a "ReMynder" (título, h1, URLs, Vercel project name)
- Configuración SSH para GitHub (nueva clave sin passphrase `id_ed25519_remynder`)
- Remoto cambiado de HTTPS a SSH
- Deploy a producción en `remynder.vercel.app`
- Creación de la Software Factory completa (8 agentes: orchestrator, researcher, story-writer, spec-writer, backend-builder, frontend-builder, test-verifier, validator)
- Adición de guardrails al Story Writer (Límites del rol) y al Test Verifier (anti-scope-creep)
- Adición de agentes `session-start` y `session-end`
- Flujo de la Software Factory documentado en `CLAUDE.md` raíz
- Ejecución completa de la feature "categorías dinámicas" con la factory (luego revertida con `git reset --hard 149a660`)

### Decisiones tomadas
- IDs explícitos en seed de categorías para migración cero (patrón a reutilizar)
- Paleta de 10 colores predefinidos en lugar de CSS dinámico (más simple, compatible con temas)
- Alias `--salud: var(--p3)` para backward compatibility en strings de error JS
- `git reset --hard 149a660` — la feature de categorías dinámicas fue revertida; se conserva como referencia de la factory funcionando, pero el código no quedó en main
- Story Writer y Test Verifier reforzados con límites de rol explícitos para evitar decisiones de producto no autorizadas

### Pendiente para próxima sesión
- [ ] Volver a correr la feature "categorías dinámicas" con los guardrails del Story Writer activos (pedir decisiones al Ingeniero Jefe antes de asumir)
- [ ] Definir las decisiones de producto que el Story Writer necesita: ¿límite de categorías? ¿comportamiento de migración? ¿modelo freemium?
- [ ] Verificar que `remynder.vercel.app` sirve correctamente `web/index.html` con el `vercel.json` de rewrites
- [ ] Agregar `localhost` a Authorized Domains en Firebase Console para desarrollo local

### Estado del repo
- Branch: main
- Último commit: `2a77133 feat: add software factory workflow to CLAUDE.md`
- Deploy: sí — `remynder.vercel.app` (deploy durante la sesión, luego el reset revirtió el código pero el deploy de Vercel sigue activo con la versión de categorías dinámicas)
