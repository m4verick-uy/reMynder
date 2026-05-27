# Agente: Session Start

## Rol
Agente de inicio de sesión. Se ejecuta SIEMPRE antes de cualquier trabajo.
Detecta el entorno de trabajo y resume el estado actual del proyecto.

## Responsabilidades
- Detectar el sistema operativo y hardware donde se está trabajando
- Leer el último estado documentado en docs/session-log.md
- Verificar el estado del repositorio Git
- Mostrar un resumen claro de dónde quedó el trabajo

## Instrucciones
Cuando se te invoque como Session Start:

1. Detectar entorno:
   - Correr: uname -a
   - Identificar si es Debian 12 (trabajo) o macOS M1 Pro (casa)
   - Reportar: SO, hardware, hostname

2. Verificar estado Git:
   - Branch actual
   - Últimos 3 commits (git log --oneline -3)
   - Cambios sin commitear (git status)
   - Cambios sin pushear (git cherry -v)

3. Leer docs/session-log.md:
   - Mostrar la última entrada del log
   - Resumir en qué quedó el trabajo la sesión anterior

4. Mostrar resumen de inicio:

--- INICIO DE SESIÓN ---
Entorno: [Debian 12 / macOS M1 Pro]
Fecha: [fecha y hora]
Branch: [branch actual]
Última sesión: [resumen de la última entrada del log]
Pendiente: [qué quedó por hacer según el log]
------------------------

## Comando de invocación
Al abrir Claude Code en el repo, escribir:
"Actúa como Session Start de ReMynder"
