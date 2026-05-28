# Test Report: Categorías Dinámicas

**Agente:** Test Verifier
**Feature:** Categorías dinámicas
**Fecha:** 2026-05-28
**Estado general:** ⚠️ Aprobado con observaciones

> La implementación es correcta y completa. Hay **1 ítem de configuración** que bloquea el deploy a producción (ADMIN_UID), **2 mejoras menores de UX** y **1 acción externa** (Firebase Security Rules). No hay bugs críticos de lógica.

---

## Criterios de aceptación — Historia por historia

### H7 — Migración transparente (m4verick)
- [x] Al entrar a la app actualizada ve sus 6 categorías → `seedCategories` usa `setDoc + merge:true` con IDs explícitos `'karate'…'padres'` ✅
- [x] Todas las tareas existentes siguen asociadas a su categoría → el campo `t.cat` coincide con `catId` — cero migración de datos ✅
- [x] Los colores son iguales o muy similares → p0=karate, p1=facultad, p2=antel, p3=salud, p4=personal, p5=padres ✅
- [x] No tuvo que hacer ninguna acción → seed automático al primer login sin categorías ✅
- [x] Edge: si entra varias veces, no se duplican → `setDoc + merge:true` es idempotente por diseño ✅
- ⚠️ **BLOQUEO DE DEPLOY**: `const ADMIN_UID = 'REEMPLAZAR_CON_UID_DE_M4VERICK'` (línea 724) — sin el UID real, m4verick recibirá "trabajo" y "personal" en lugar de sus 6 categorías. Debe resolverse antes del deploy.

### H8 — Primera experiencia (nuevo usuario)
- [x] Al primer login ve "trabajo" y "personal" → `addDoc` en el camino `else` de `seedCategories` ✅
- [x] Puede usarlas desde el primer momento → listener de categorías las disponibiliza automáticamente ✅
- [x] Puede editar nombres y colores → `updateCategory` implementado ✅
- [x] Puede eliminarlas → `deleteCategory` implementado ✅
- [x] Edge: si las elimina y crea las propias, todo funciona normalmente ✅

### H1 — Acceder a ajustes
- [x] Ícono ⚙ visible en el header cuando logueado → `#settings-btn` dentro de `#app` que solo se muestra con usuario autenticado ✅
- [x] Al tocarlo entra a la pantalla de ajustes → `showSettings()` oculta `#app` y muestra `#settings-screen` ✅
- [x] Ve la lista de sus categorías actuales → `showSettings()` llama `renderSettings()` ✅
- [x] Hay botón claro para volver → `#settings-back-btn` con texto "← volver" ✅
- [x] Al volver, mismo filtro activo y misma lista → `hideSettings()` llama `render()` que recalcula desde estado actual (filter/catFilter preservados) ✅
- [x] Edge: el ⚙ NO aparece en login → correcto, `#settings-btn` está dentro de `#app` ✅

### H2 — Ver categorías en ajustes
- [x] Cada categoría muestra dot de color + nombre + botón editar + botón eliminar → `renderSettings()` genera cada fila ✅
- [x] Aparecen en orden de creación → query con `orderBy('createdAt')` ✅
- [x] Al final hay botón "+" → `#add-category-btn` en `settings-main`, siempre visible ✅
- [x] Diseño limpio y minimalista → consistente con el sistema de variables CSS existente ✅
- [x] Edge: lista larga hace scroll → `settings-main` sin overflow hidden ✅
- [x] Edge: nombres largos → `.cat-name` tiene `overflow:hidden; text-overflow:ellipsis; white-space:nowrap` ✅

### H3 — Crear categoría
- [x] Al tocar "+" aparece formulario inline → `editingCatId = 'new'; renderSettings()` ✅
- [x] Puede escribir nombre → `cat-form-input` con `maxlength="30"` ✅
- [x] Puede elegir color de paleta visual predefinida → 10 color-dots (p0–p9) ✅
- [x] Al confirmar, la nueva categoría aparece al instante → onSnapshot actualiza `categories[]` y re-renderiza ✅
- [x] Disponible inmediatamente en selector y filtros → listener llama `renderCatSelector()` y `renderNavCats()` ✅
- [x] Persiste si cierra y vuelve → Firestore ✅
- [x] Edge: nombre vacío no se guarda → validación en event handler + `createCategory` ✅
- [x] Edge: nombre con solo espacios → `name.trim()` antes de guardar ✅

### H4 — Editar categoría
- [x] Al tocar editar, nombre y color se vuelven editables inline → `editingCatId = cat.id; renderSettings()` con valores pre-cargados ✅
- [x] Puede cambiar nombre y color ✅
- [x] Al confirmar, cambios reflejados inmediatamente en toda la app → `updateCategory` → listener → `renderCatSelector()`, `renderNavCats()`, `render()` ✅
- [x] Las tareas actualizan su badge automáticamente → `render()` usa `getCatBadge(t.cat)` que lookupea en `categories[]` actualizado ✅
- [x] Edge: nombre vacío al editar → misma validación que crear ✅
- [x] Edge: cancelar edición → `editingCatId = null; renderSettings()` sin llamar `updateCategory` ✅

### H5 — Eliminar categoría sin tareas
- [x] Al tocar eliminar (sin tareas), se pide confirmación → `confirm()` nativo ✅
- [x] Al confirmar, desaparece de la lista, del selector y de los filtros → `deleteDoc` → listener → renderizados dinámicos ✅
- [x] La eliminación es inmediata y persiste → Firestore ✅
- [x] Edge: cancelar confirmación → la categoría no se elimina (`return` en `deleteCategory`) ✅
- [x] Edge: eliminar la única categoría disponible → `selectedCat` queda `null`, `addTask` tiene guard ✅

### H6 — Intentar eliminar categoría con tareas
- [x] El botón eliminar está visualmente diferenciado → clase `blocked` con `opacity:0.25; pointer-events:none` ✅
- [x] Si intenta eliminarla, ve un mensaje con cantidad de tareas → `.cat-blocked-msg` se renderiza en `renderSettings()` ✅
- [x] El mensaje indica qué debe hacer primero → "reasigná o eliminá las tareas primero" ✅
- [x] No se elimina nada → `deleteCategory` retorna temprano si `taskCount > 0` ✅
- [x] Edge: categoría pasa de tener tareas a no tener → listener de tareas llama `renderSettings()` cuando la pantalla está visible ✅

---

## Regresiones detectadas

Ninguna.

- `addTask()` ✅ — lógica idéntica, usa `selectedCat` dinámico
- `toggleTask()`, `deleteTask()`, `clearDone()` ✅ — sin cambios
- `setFilter()` ✅ — sin cambios
- `render()` ✅ — misma lógica de filtrado, solo cambia la generación del badge
- Login / logout ✅ — logout cancela ambos listeners y resetea todo el estado
- Theme toggle ✅ — sin cambios
- Filtros "Todas", "Pendientes", "Completadas" ✅ — sin cambios
- `--salud` en `.logout-btn:hover` y `.clear-btn:hover` ✅ — alias preservado: `--salud: var(--p3)`

---

## Edge cases verificados

| Caso | Resultado |
|---|---|
| `selectedCat = null` al agregar tarea | OK — `addTask` retorna sin feedback silencioso (ver Observaciones) |
| Seed doble por doble disparo del listener | OK — flag `categoriesReady` previene dentro de la sesión |
| Race condition: tasks render antes que categories | OK — `getCatBadge` usa fallback `cat-p0 + escHtml(t.cat)` |
| Editar y presionar volver sin confirmar | OK — `editingCatId` no se resetea en `hideSettings`; al volver a ajustes el form sigue abierto (ver Observaciones) |
| ADMIN_UID incorrecto o placeholder | FALLA — m4verick recibe "trabajo"+"personal" en lugar de sus 6 categorías |
| Categories listener llega tras tasks listener | OK — categories vacío no rompe render, usa fallback |
| `confirm()` bloqueado por el browser | Edge no manejado — `confirm()` puede bloquearse en algunos contextos (iframe, etc.). Aceptable para la versión actual. |

---

## Seguridad

- [x] `escHtml()` aplicado en todo `innerHTML` que inserta datos de usuario
  - `renderCatSelector()`: `escHtml(cat.name)` ✅
  - `renderNavCats()`: `btn.textContent = cat.name` ✅ (textContent es safe)
  - `renderSettings()`: `escHtml(cat.name)` en `.cat-name` ✅
  - `renderCatForm()`: `value="${escHtml(currentName)}"` ✅
  - `getCatBadge()`: `escHtml(cat.name)` y `escHtml(catId)` ✅
  - `render()`: `escHtml(t.text)` ✅ (sin cambios)
  - Mensajes de error: `textContent` (no `innerHTML`) ✅
  - `confirm()` con `cat.name`: dialog nativo, no renderiza HTML ✅
- [x] Firestore Security Rules no modificadas en el código ✅
- [x] No hay API keys ni secrets nuevos expuestos ✅
- [x] Login/logout siguen funcionando correctamente ✅
- [x] Datos de usuario aislados: queries usan `users/${uid}/categories` con `uid = auth.currentUser.uid` ✅
- ⚠️ **Acción externa requerida**: agregar regla de Security Rules para `categories` en Firebase Console antes del deploy a producción

---

## Problemas encontrados

| # | Problema | Severidad | Descripción |
|---|---|---|---|
| 1 | `ADMIN_UID` sin reemplazar | **CRÍTICO para deploy** | Línea 724: `const ADMIN_UID = 'REEMPLAZAR_CON_UID_DE_M4VERICK'`. Sin el UID real, la H7 (migración de m4verick) falla: recibirá "trabajo"+"personal" en lugar de sus 6 categorías. Bloquea `vercel --prod`. No bloquea pruebas con otros usuarios. |
| 2 | `hideSettings()` no resetea `editingCatId` | Menor | Si el usuario abandona ajustes mientras tiene un formulario inline abierto, al volver a ajustes el formulario sigue visible. No es destructivo, pero puede ser confuso. Corregir con `editingCatId = null` en `hideSettings()`. |
| 3 | Feedback silencioso si `selectedCat === null` | Menor | Si el usuario no tiene categorías y toca "+ Agregar" o presiona Enter, la tarea no se crea y no hay mensaje de error. El input queda con texto. Podría mostrar un hint como "creá una categoría primero". |
| 4 | Firebase Security Rules — acción externa | Bloqueante para categorías en producción | La nueva colección `users/{uid}/categories` no tiene reglas en Firebase Console. Los read/write fallarán en producción hasta que se actualicen. El Backend Builder lo documentó; requiere acción manual del Ingeniero Jefe. |

---

## Recomendaciones

1. **Antes del deploy** (obligatorio):
   - Obtener el UID de m4verick e insertar en línea 724
   - Actualizar Firebase Security Rules para cubrir `categories`

2. **Corrección menor — `hideSettings()`** (recomendado):
   ```javascript
   function hideSettings() {
     editingCatId = null;   // ← agregar esta línea
     document.getElementById('settings-screen').style.display = 'none';
     document.getElementById('app').style.display = '';
     render();
   }
   ```

3. **Corrección menor — feedback cuando no hay categorías** (opcional):
   Si `!selectedCat` en `addTask`, limpiar el input y mostrar brevemente "creá una categoría primero" (o deshabilitar visualmente el botón "+ Agregar" cuando `categories.length === 0`).

4. **Flujo de prueba recomendado antes del deploy**:
   - [ ] Resolver ADMIN_UID y Security Rules
   - [ ] Entrar como m4verick → verificar que aparecen las 6 categorías
   - [ ] Verificar que las tareas existentes siguen asociadas a sus categorías
   - [ ] Crear una nueva categoría, crear tareas con ella, editar, eliminar
   - [ ] Probar bloqueo al intentar eliminar categoría con tareas
   - [ ] Verificar que el tema claro/oscuro funciona en la pantalla de ajustes
   - [ ] Probar en mobile (viewport 375px)

---

## Anti-scope-creep ✅

- No hay cambios al modelo de tareas (sin campos nuevos)
- No hay nueva autenticación ni proveedores
- No hay límites de categorías implementados (decisión del Ingeniero Jefe: sin límite por ahora)
- No hay modelo freemium ni restricciones de plan
- No hay edición de tareas
- Solo las 8 historias definidas por el Ingeniero Jefe
