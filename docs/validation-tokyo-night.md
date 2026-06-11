# Validation — Tokyo Night theme

## Decisión: APROBADO para deploy

### Cambios implementados
- `web/index.html` líneas 13–58 — único bloque CSS modificado
- Dark mode → Tokyo Night Night palette oficial
- Light mode → neutro frío complementario (elección del usuario)

### Checklist de seguridad
- [x] XSS: `escHtml()` intacto, sin cambios en innerHTML
- [x] Firestore Security Rules: sin cambios
- [x] Auth: sin cambios
- [x] `--salud` alias preservado en ambos modos

### Checklist de regresiones
- [x] Radii CSS sin cambios
- [x] Tipografía sin cambios
- [x] Layout sin cambios
- [x] JS sin cambios
- [x] HTML sin cambios
- [x] Todos los event listeners intactos
- [x] Cat-pN dark overrides funcionan sin cambio estructural
- [x] Nav pill light overrides funcionan sin cambio estructural

### Compatibilidad roadmap
- [x] Paleta numerada p0–p9 preservada (categorías dinámicas futuras sin impacto)
- [x] `--salud` alias preservado (backward compat)
- [x] Theme toggle (dark/light) funciona igual

### Paleta dark (Tokyo Night Night)
bg `#1a1b26` · surface `#24283b` · text `#c0caf5` · muted `#565f89` · subtle `#3b4261`

### Paleta light (neutro frío)
bg `#f0f0f5` · surface `#e4e4f0` · text `#24283b` · muted `#6b7090` · subtle `#a8aac0`
