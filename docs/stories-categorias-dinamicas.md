# Stories: Categorías Dinámicas

**Agente:** Story Writer
**Feature:** Categorías dinámicas
**Fecha:** 2026-05-28
**Brief del Ingeniero Jefe:** categorías dinámicas con gestión vía ⚙ en header → pantalla de ajustes inline

---

## Checklist pre-stories ✅

- [x] ¿Qué pidió el Ingeniero Jefe? — crear, editar y eliminar categorías propias
- [x] ¿Límites numéricos? — sin límite por ahora (decisión tomada)
- [x] ¿Migración de datos? — m4verick: seed automático; nuevos usuarios: 2 categorías de ejemplo (decisión tomada)
- [x] ¿Modelo de negocio implícito? — ninguno en esta versión (decisión tomada)
- [x] ¿Punto de entrada en la UI? — ⚙ en el header → pantalla de ajustes (decisión tomada)
- [x] ¿Qué pasa al eliminar con tareas? — bloqueado con mensaje (decisión tomada)

---

## Historia 1 — Acceder a ajustes

**Historia:** Como usuario logueado, quiero tocar el ícono ⚙ en el header para entrar a la pantalla de ajustes donde puedo gestionar mis categorías.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] El ícono ⚙ es visible en el header cuando estoy logueado
- [ ] Al tocarlo entro a la pantalla de ajustes
- [ ] En la pantalla de ajustes veo la lista de mis categorías actuales
- [ ] Hay un botón o gesto claro para volver a la pantalla principal
- [ ] Al volver, estoy exactamente donde estaba (mismo filtro activo, misma lista)

**Edge cases:**
- El usuario no tiene categorías todavía: ve un mensaje vacío y el botón para crear la primera
- El ícono no debe aparecer en la pantalla de login

---

## Historia 2 — Ver mis categorías en ajustes

**Historia:** Como usuario en la pantalla de ajustes, quiero ver todas mis categorías listadas de forma clara para saber qué tengo y poder gestionarlas.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] Cada categoría muestra: ícono/dot de color + nombre + botón editar + botón eliminar
- [ ] Las categorías aparecen en el orden en que fueron creadas
- [ ] Al final de la lista hay un botón "+" para agregar una nueva categoría
- [ ] El diseño es limpio y minimalista

**Edge cases:**
- Lista larga: hace scroll sin romper el layout
- Nombres largos: se truncan o hacen wrap correctamente

---

## Historia 3 — Crear categoría

**Historia:** Como usuario en ajustes, quiero crear una nueva categoría con nombre y color para organizar mis tareas según mi vida.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] Al tocar "+" aparece un formulario inline (dentro de la misma pantalla de ajustes)
- [ ] Puedo escribir el nombre de la categoría
- [ ] Puedo elegir un color de una paleta visual predefinida
- [ ] Al confirmar, la nueva categoría aparece al instante en la lista
- [ ] La nueva categoría está disponible inmediatamente en el selector de tareas y en los filtros
- [ ] La categoría persiste si cierro y vuelvo a abrir la app

**Edge cases:**
- Nombre vacío: no se puede guardar, el campo se marca en error
- Nombre con solo espacios: se ignoran los espacios, si queda vacío no se guarda

---

## Historia 4 — Editar categoría

**Historia:** Como usuario en ajustes, quiero editar el nombre y el color de una categoría existente para mantener mi sistema organizado.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] Al tocar el botón editar de una categoría, el nombre y el color se vuelven editables inline
- [ ] Puedo cambiar el nombre y el color
- [ ] Al confirmar, los cambios se reflejan inmediatamente en toda la app (selector, filtros, badges de tareas)
- [ ] Las tareas que tenían esa categoría actualizan su badge automáticamente

**Edge cases:**
- Nombre vacío al editar: no se guarda, el campo se marca en error
- Cancelar edición: vuelve al estado original sin guardar cambios

---

## Historia 5 — Eliminar categoría sin tareas

**Historia:** Como usuario en ajustes, quiero eliminar una categoría que ya no uso cuando no tiene tareas asociadas.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] Al tocar el botón eliminar de una categoría sin tareas, se pide confirmación
- [ ] Al confirmar, la categoría desaparece de la lista, del selector y de los filtros
- [ ] La eliminación es inmediata y persiste

**Edge cases:**
- Cancelar la confirmación: la categoría no se elimina
- Eliminar la única categoría disponible: permitido, el usuario puede quedar sin categorías

---

## Historia 6 — Intentar eliminar categoría con tareas

**Historia:** Como usuario en ajustes, quiero recibir un mensaje claro cuando intento eliminar una categoría que tiene tareas, para entender por qué no puedo hacerlo.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] El botón eliminar de una categoría con tareas está visualmente diferenciado (deshabilitado o con indicador)
- [ ] Si el usuario intenta eliminarla, ve un mensaje que explica cuántas tareas tiene esa categoría
- [ ] El mensaje le indica qué debe hacer primero (reasignar o completar/eliminar las tareas)
- [ ] No se elimina nada

**Edge cases:**
- La categoría pasa de tener tareas a no tener (el usuario eliminó sus tareas): el botón eliminar se habilita automáticamente

---

## Historia 7 — Migración transparente (usuario admin)

**Historia:** Como usuario actual de ReMynder (m4verick), quiero que al actualizar la app mis categorías y tareas existentes sigan funcionando exactamente igual.

**Prioridad:** Alta — prerequisito de todo lo demás

**Criterios de aceptación:**
- [ ] Al entrar a la app actualizada, veo mis 6 categorías (karate, facultad, antel, salud, personal, padres) ya disponibles
- [ ] Todas mis tareas existentes siguen asociadas a su categoría correcta
- [ ] Los colores son iguales o muy similares a los actuales
- [ ] No tuve que hacer ninguna acción para que esto funcione

**Edge cases:**
- Si entro varias veces, las categorías no se duplican (migración idempotente)

---

## Historia 8 — Primera experiencia (usuario nuevo)

**Historia:** Como usuario nuevo de ReMynder, quiero encontrar categorías de ejemplo al entrar por primera vez para poder empezar a usar la app de inmediato.

**Prioridad:** Media

**Criterios de aceptación:**
- [ ] Al hacer login por primera vez veo 2 categorías ya creadas: "trabajo" y "personal"
- [ ] Puedo usar esas categorías desde el primer momento para crear tareas
- [ ] Puedo editar sus nombres y colores desde ajustes
- [ ] Puedo eliminarlas si no las quiero

**Edge cases:**
- Si el usuario elimina las 2 categorías de ejemplo y crea las propias, todo funciona normalmente

---

## Prioridad de implementación

| Historia | Prioridad | Dependencias |
|---|---|---|
| H7 — Migración transparente | Alta | Ninguna — va primero |
| H8 — Primera experiencia | Media | H7 (mismo mecanismo de seed) |
| H1 — Acceder a ajustes | Alta | — |
| H2 — Ver categorías en ajustes | Alta | H1 |
| H3 — Crear categoría | Alta | H2 |
| H4 — Editar categoría | Alta | H2 |
| H5 — Eliminar sin tareas | Alta | H2 |
| H6 — Bloqueo con tareas | Alta | H5 |
