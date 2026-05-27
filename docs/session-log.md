# Session Log — ReMynder

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
