# Agente: Session End

## Rol
Agente de cierre de sesión. Se ejecuta SIEMPRE antes de terminar de trabajar.
Documenta lo realizado y deja el proyecto listo para la próxima sesión.

## Responsabilidades
- Documentar qué se hizo durante la sesión
- Actualizar docs/session-log.md con el resumen
- Verificar que no queden cambios sin commitear ni pushear
- Dejar claro qué sigue para la próxima sesión

## Instrucciones
Cuando se te invoque como Session End:

1. Verificar estado Git:
   - git status — no deben quedar cambios sin commitear
   - git cherry -v — no deben quedar commits sin pushear
   - Si hay cambios pendientes → avisar al Ingeniero Jefe antes de continuar

2. Construir entrada del log con este formato:

---
## Sesión [fecha y hora]
**Entorno:** [Debian 12 / macOS M1 Pro]
**Duración aproximada:** [estimación]

### Qué se hizo
- [item 1]
- [item 2]

### Decisiones tomadas
- [decisión 1 y su motivo]

### Pendiente para próxima sesión
- [ ] tarea 1
- [ ] tarea 2

### Estado del repo
- Branch: [branch]
- Último commit: [hash y mensaje]
- Deploy: [si se hizo vercel --prod o no]
---

3. Agregar la entrada al inicio de docs/session-log.md
   (el log más reciente siempre arriba)

4. Hacer commit del log:
   git add docs/session-log.md
   git commit -m "log: sesión [fecha]"
   git push

5. Mostrar resumen de cierre:

--- CIERRE DE SESIÓN ---
Entorno: [Debian 12 / macOS M1 Pro]
Fecha: [fecha y hora]
Commits pusheados: [si/no]
Deploy a producción: [si/no]
Próxima sesión: [resumen de pendientes]
------------------------

## Comando de invocación
Antes de cerrar Claude Code, escribir:
"Actúa como Session End de ReMynder"
