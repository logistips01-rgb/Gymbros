# GymBros — Notas de desarrollo

## Reglas de desarrollo
- **Siempre** subir la versión del service worker (`sw.js`) en el mismo commit que cualquier cambio a `index.html`.
- **Siempre** sincronizar el número de versión visible en el header del HTML (`vN` junto al logo) con la versión del SW. El usuario lo usa para verificar que la actualización ha llegado a su móvil.
- **Siempre** actualizar `./sw.js?v=N` en el registro del SW para forzar actualización en GitHub Pages (que cachea sw.js con 1h de TTL).

### Esquema de versiones
- Actual: secuencial `v90`, `v91`… hasta `v99`
- Al llegar a `v100` → reiniciar a `v1.0` (semantic versioning)
- Luego `v1.1`, `v1.2`… `v1.99`
- Al llegar a `v1.99` → saltar a `v2.1`
- Formato en header y SW: `v1.0`, `v1.1`… `v2.1` (siempre con la `v` delante)

## Stack
- PWA single-file (`index.html`) — vanilla JS, sin build tools
- Firebase Firestore (compat SDK v10.12.0) para datos compartidos del grupo
- `localStorage` para datos privados: nutrición, peso corporal, recetas, sesión activa, Groq API key
- Service worker (`sw.js`) — cache-first, versión actual `gymbros-v90`
- Deploy: GitHub Pages (auto en push a `main`), rama de desarrollo `claude/amazing-gauss-7i0yds`
- IA: Groq API, modelo `llama-3.3-70b-versatile`, max 600 tokens. Key en `localStorage('bw_groq_key')`, accedida via `getIaKey()`

## Seguridad crítica
- **NUNCA** hacer backup de la Groq API key (`bw_groq_key` / `IA_KEY_STORAGE`) a Firestore ni ningún almacenamiento remoto. Solo vive en el dispositivo.

## Estructura de datos

### Firestore
- `groups/{groupCode}` — datos del grupo (nombre, contraseña hash, generation, destacado, reto)
- `groups/{groupCode}/profiles/{profileId}` — perfil del usuario incluyendo `_priv` (backup privado cifrado) y `pinHash`

### localStorage (privado por dispositivo)
- `gymbros_nutri_{profileId}` — nutrición (comidas, objetivos, perfil físico)
- `gymbros_bw_{profileId}` — historial de peso corporal
- `gymbros_recipes_{profileId}` — recetas personalizadas
- `bw_groq_key` — Groq API key (NUNCA subir a Firestore)
- `gymbros_notif` — preferencia de notificaciones (`'1'` = activadas)
- `gymbros_active_session` — sesión activa en curso

### Perfil (Firestore)
- `profile.muscles[].exercises[].sessions[]` — historial de entrenos libres `{ id, date, sets[] }`
- `profile.gymSessions[]` — sesiones finalizadas (rutinas + sesiones libres) `{ id, date, routineId, routineName, exercises[] }`
- `profile.routines[]` — plantillas de rutinas
- `profile.pinHash` — SHA-256 del PIN salteado con profileId

## Funcionalidades implementadas

### Entrenamiento
- [x] Grupos musculares + ejercicios con tipos: normal, bodyweight, cardio
- [x] Historial por ejercicio con gráfico SVG de progresión
- [x] Rutinas: crear, editar, iniciar sesión, historial completo
- [x] Sesión libre (sin rutina) con barra flotante y resumen al finalizar
- [x] Timer de descanso flotante, siempre visible sin scroll
- [x] Récords personales automáticos (🏆 toast al superar 1RM est.) — rutinas y entrenos libres
- [x] Kirk analiza estancamiento (3 sesiones sin subir peso) y da consejo
- [x] Kirk comenta cada sesión al finalizarla (rutinas y libres)
- [x] Aviso de grupo muscular olvidado (+10 días) en tarjetas del home

### Rutinas y sesión
- [x] Guardar progreso de rutina (recupera sets al reabrir)
- [x] Inputs de series con grid CSS correcto (sin overflow)
- [x] Comparativa vs sesión anterior en el historial

### Social / Grupo
- [x] Multi-grupo con contraseña + sistema piramidal de referidos (Gen 1-4)
- [x] Ejercicio destacado del mes (admin Gen 1 lo fija, todos ven su progreso)
- [x] Reto del mes con seguimiento automático
- [x] Rankings por disciplina (fuerza/calistenia/cardio) con progresión %
- [x] Logros (16 achievements, 3 tiers) con banner motivacional al entrar
- [x] Sesiones recientes de bros visible en tab Rutinas

### Nutrición
- [x] Registro diario de comidas con macros
- [x] Scanner de código de barras (BarcodeDetector + Open Food Facts)
- [x] TDEE calculado con Mifflin-St Jeor + multiplicador de actividad
- [x] Balance real vs TDEE (no vs objetivo ajustado)
- [x] Calorías gym estimadas y descontadas del balance
- [x] Calendario semanal Lun-Dom + histórico de semanas y meses
- [x] Recetas personalizadas guardadas en localStorage
- [x] Historial de peso corporal con gráfico

### Privacidad y seguridad
- [x] PIN por perfil (SHA-256 salteado con profileId) — protege datos privados
- [x] Backup automático de datos privados en Firestore (campo `_priv`) — excluye siempre la Groq key
- [x] Restauración del backup solo si la clave localStorage no existe (no sobreescribe)
- [x] Admin Gen 1 puede resetear PIN de cualquier perfil desde panel de administración

### PWA / Técnico
- [x] Service worker cache-first con `updateViaCache:'none'` + `?v=N` para forzar update en GitHub Pages
- [x] Auto-recarga al activar nuevo SW (`controllerchange`)
- [x] Botón "Forzar actualización SW" en configuración
- [x] Banner de instalación PWA nativo (Android: `beforeinstallprompt`, iOS: instrucciones)
- [x] Notificaciones locales: racha en peligro + bros entrenaron hoy

### IA (Kirk)
- [x] Chat libre con Kirk en tab IA
- [x] Resumen semanal Kirk (lunes)
- [x] Comentario de sesión al finalizar
- [x] Consejo de estancamiento por ejercicio
- [x] Contexto nutricional y físico enviado a Kirk

## Conteo de sesiones — regla anti-doble-conteo
- `ex.sessions`: entrenos libres guardados por ejercicio
- `p.gymSessions`: sesiones finalizadas (rutinas + sesiones libres con `routineId=null`)
- Al calcular volumen/días: excluir `ex.sessions` solo para días con sesión libre finalizada (`!routineId`), porque en ese caso los sets están duplicados. Las rutinas NO duplican `ex.sessions`.
- `getAllSessionDates(p)`: siempre deduplica por fecha — 1 día entrenado = 1 sesión.

## Pendientes
- [ ] Exportar historial de sesiones a CSV/PDF

## Proyectos futuros
- [ ] **GymSister** — versión hermana de GymBros orientada a mujeres (mismo stack, paleta y tono adaptados)
