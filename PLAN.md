# Space Runner — Estado del Desarrollo

**Arquitectura:** Single-file HTML5 Canvas + Vanilla JS + Web Audio API. Sin build system ni dependencias externas. Todo el código vive en `index.html`.

**Tech Stack:** HTML5 Canvas, JavaScript ES6+, Web Audio API, localStorage

**Despliegue:** Vercel (producción en https://space-runner-theta.vercel.app), con `vercel.json` para cabeceras de seguridad.

---

## Completado

Este plan original (fases 1-4: bugs críticos, deltaTime, feedback visual, líneas de carril, corazones animados, enemigos variados, pausa, pooling de partículas, música variable) **ya está implementado en `index.html`**. Resumen de lo que existe hoy:

- `updateLanes()` única, `musicGain` inicializado correctamente, colisión con padding dinámico, swipe horizontal en móvil.
- Movimiento y timers escalados por `dt` (deltaTime) en todo el loop.
- Screen shake, hit flash del jugador, corazones dibujados en Canvas (no Unicode).
- Líneas de carril animadas (`drawRoad`).
- Enemigos: `normal`, `shooter` (triángulo invertido), `speedster` (rombo), `tank` (barra de HP), `boss` (cada 500 pts, con parpadeo de ojos y pulso de aura).
- Powerups `health`, `multishot` y `shield` (escudo con invulnerabilidad temporal) totalmente funcionales.
- Sistema de combos con multiplicador y mensajes de racha.
- Pausa (`ESC`/`P` en teclado, botón táctil) y silencio (`M` en teclado, botón táctil, persistido en `localStorage`).
- Pooling de partículas (`MAX_PARTICLES`), con formas distintas según el evento (impacto, recolección, explosión).
- Música procedural con BPM escalado según velocidad/jefe; SFX con variación de pitch.

Adicionalmente, se ejecutó una auditoría completa (UX/UI, visual/creativo, seguridad — ver `docs/audit-report-2026-07-11.md`) y se implementaron todos sus hallazgos accionables:

- Menú y pantalla de victoria escalan con el tamaño de canvas (ya no se cortan en pantallas pequeñas).
- Paralaje diferenciado por capa (planetas lentos, asteroides rápidos).
- Progresión de tinte de fondo por zonas de puntaje (cada 500 pts).
- Tipografía sans-serif para bloques de texto largo, pixel font reservada a títulos/HUD.
- Botón de compartir puntaje en game over/victoria (Web Share API con fallback a portapapeles).
- Zoom habilitado (`user-scalable` ya no bloqueado) por accesibilidad.
- `vercel.json` con `Content-Security-Policy`, `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`.

---

## Backlog pendiente

Hallazgos de la auditoría marcados como de baja prioridad o que requieren decisión de diseño adicional, sin implementar todavía:

- [ ] Alojar la fuente `Press Start 2P` localmente (`.woff2`) en vez de cargarla desde Google Fonts, para evitar exponer la IP del usuario en cada carga.
- [ ] Agregar un workflow simple de CI (GitHub Actions) que valide el HTML antes de cada push a `main`.
- [ ] Evaluar reemplazar los power-ups (círculo + letra) por íconos vectoriales manteniendo el estilo pixelado.
- [ ] Evaluar variedad de mezcla de enemigos que escale con el puntaje/velocidad, no solo con umbrales fijos.

Ninguno de estos es bloqueante para producción.

---

## Testing

Cada cambio debe verificarse manualmente:
1. Abrir `index.html` en navegador (o correr `python -m http.server` y navegar a `localhost`)
2. Verificar consola (F12) sin errores
3. Jugar partida completa probando la feature modificada
4. Verificar en móvil (device toolbar o dispositivo real) cuando aplique
