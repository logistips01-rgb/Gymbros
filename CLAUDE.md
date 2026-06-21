# GymBros — Notas de desarrollo

## Reglas de desarrollo
- **Siempre** subir la versión del service worker (`sw.js`) en el mismo commit que cualquier cambio a `index.html`. Formato: `gymbros-vN` incrementando N.
- **Siempre** sincronizar el número de versión visible en el header del HTML (`vN` junto al logo) con la versión del SW. Son el mismo número. El usuario lo usa para verificar que la actualización ha llegado a su móvil.

## Stack
- PWA single-file (`index.html`) — vanilla JS, sin build tools
- Firebase Firestore (compat SDK v10.12.0) para datos compartidos del grupo
- `localStorage` para datos privados: nutrición, peso corporal, sesión activa
- Service worker (`sw.js`) — cache-first, versión actual `gymbros-v59`
- Deploy: GitHub Pages (auto en push a `main`)
- IA: Groq API, modelo `llama-3.3-70b-versatile`, max 150 tokens (comentarios de sesión)

## Mejoras pendientes

### Alta prioridad
- [x] **Soporte multi-grupo con contraseña + sistema piramidal de referidos** ✅ implementado
  - Grupos aislados con código + contraseña
  - Creación solo mediante enlace `?invite=CODIGO` (pirámide de referidos)
  - Límite de 4 generaciones (originales = Gen 1)
  - `generation` + `referredBy` guardados en Firestore por grupo
  - Contador de usuarios activos por generación visible solo para Gen 1
  - Migración automática de datos antiguos al primer grupo

### Menores / ya identificadas
- [ ] Backup de datos privados (nutrición/peso) en Firestore cifrado por usuario
- [ ] Exportar historial de sesiones a CSV/PDF

## Proyectos futuros
- [ ] **GymSister** — versión hermana de GymBros orientada a mujeres (mismo stack, paleta y tono adaptados)
