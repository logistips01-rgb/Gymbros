# GymBros — Notas de desarrollo

## Stack
- PWA single-file (`index.html`) — vanilla JS, sin build tools
- Firebase Firestore (compat SDK v10.12.0) para datos compartidos del grupo
- `localStorage` para datos privados: nutrición, peso corporal, sesión activa
- Service worker (`sw.js`) — cache-first, versión actual `gymbros-v39`
- Deploy: GitHub Pages (auto en push a `main`)
- IA: Groq API, modelo `llama-3.3-70b-versatile`, max 150 tokens (comentarios de sesión)

## Mejoras pendientes

### Alta prioridad
- [ ] **Soporte multi-grupo con contraseña**
  Permitir crear grupos de amigos aislados, cada uno accesible con un código + contraseña.
  - Estructura: `groups/{groupId}/sessions/`, `groups/{groupId}/users/`
  - Password hashed con Web Crypto SHA-256 en cliente
  - Pantalla de bienvenida: crear grupo / unirse con código
  - El groupId se guarda en `localStorage` como `gymbros_group`
  - El groupId sirve como código de invitación compartible por WhatsApp
  - Requiere migración de los datos actuales al nuevo grupo

### Menores / ya identificadas
- [ ] Backup de datos privados (nutrición/peso) en Firestore cifrado por usuario
- [ ] Exportar historial de sesiones a CSV/PDF

## Proyectos futuros
- [ ] **GymSister** — versión hermana de GymBros orientada a mujeres (mismo stack, paleta y tono adaptados)
