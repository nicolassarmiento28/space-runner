# Auditoría Space Runner — 2026-07-11

## Resumen ejecutivo
El proyecto está sano para un juego estático de archivo único: no hay vulnerabilidades activas de seguridad (sin XSS, sin `eval`, sin fugas de secretos), pero le faltan cabeceras de seguridad básicas y tiene dos bugs de layout de alta severidad en pantallas de menú/victoria. En UX, el mayor bloqueante es que el texto de `drawMenu()` y `drawWin()` no escala con `canvas.height`, pudiendo cortarse en pantallas pequeñas. En lo visual/creativo no hay problemas, sino oportunidades: el paralaje es falso y las partículas/música no varían por contexto. Total: **2 hallazgos de severidad alta**, **5 de media**, el resto baja/informativa. Ningún bloqueante de seguridad para producción; sí hay un bloqueante de UX (texto cortado en pantallas pequeñas).

## Tabla consolidada de hallazgos

| # | Área | Hallazgo | Severidad | Ubicación | Recomendación |
|---|---|---|---|---|---|
| 1 | UX | `drawMenu()` usa coordenadas Y y tamaños de fuente fijos, sin escalar con `canvas.height`/`getScaleFactor()`; en pantallas <700px de alto el texto se corta | Alta | `drawMenu()` ~L1095-1189 | Escalar con `sf` y posicionar como proporción de `canvas.height`, igual que `drawHUD()` |
| 2 | UX | `drawWin()` tiene el mismo problema: coordenadas Y absolutas sin relación a `canvas.height` | Alta | `drawWin()` ~L1221-1247 | Centrar en base a `canvas.height/2`, como ya hace `drawGameOver()` |
| 3 | UX | No hay botón/gesto de pausa visible para jugadores táctiles (solo ESC/P en teclado) | Media | keydown ~L1281-1291, sin equivalente táctil | Ícono de pausa tocable en el HUD |
| 4 | UX | No hay control de silencio/volumen visible; `bgMusic.volume` fijo en código | Media | L38 | Ícono de mute persistido en `localStorage` |
| 5 | UX / Visual | Tipos de enemigo (normal, shooter, speedster) se distinguen solo por color y son todos rectángulos; un jugador daltónico puede confundirlos, y visualmente falta silueta distintiva (el jugador sí tiene forma de nave hecha con `moveTo`/`lineTo`) | Media | `drawEntities()` ~L923-971 | Dar forma geométrica distintiva por tipo (triángulo, rombo, alargado), no solo color — resuelve accesibilidad y refuerzo visual a la vez |
| 6 | UX | `comboMessage` se dibuja en el centro exacto de la pantalla, superponiéndose con la acción del juego | Media | `drawHUD()` ~L1086-1092 | Mover a posición fija en el HUD superior |
| 7 | UX | No hay indicador visual de `moveCooldown`; el jugador no percibe por qué a veces el input no responde | Media | `updatePlayer()` ~L470-486 | Destello/atenuación en la nave mientras `canMove === false` |
| 8 | Seguridad | Sin cabeceras de seguridad ni archivo de config de despliegue (`vercel.json`/`_headers`): falta CSP, `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy` | Media | Confirmado: no existe ninguno en el repo | Agregar `vercel.json` con esas cabeceras |
| 9 | Seguridad | Fuente cargada desde Google Fonts sin alojar localmente; expone IP del usuario a Google en cada carga | Baja | `index.html:7` | Alojar `Press+Start+2P.woff2` localmente vía `@font-face` |
| 10 | Visual | Paralaje falso: estrellas, planetas y asteroides escalan todos con la misma fórmula (`speed * game.speed/5`), sin diferenciación de profundidad real | Alto (impacto visual) | `updateBackground()` ~L443-468 | Multiplicadores de velocidad fijos y distintos por capa, independientes de `game.speed` |
| 11 | Visual | Partículas uniformes (siempre cuadrado 2-5px) sin variar por tipo de evento (explosión, power-up, golpe) | Alto (impacto visual) | `Particle` L231-252, `spawnParticle` L416-421 | Variantes de forma/comportamiento por contexto |
| 12 | Visual | Música procedural con BPM fijo (120) sin escalar con `game.speed` ni con `game.bossActive` | Alto (impacto visual) | `scheduleMusic()` L104-120 | Interpolar BPM 120→160 y añadir capa extra durante el jefe |
| 13 | Visual | Fondo sin progresión de color/zona a medida que sube el puntaje | Alto (impacto visual) | `COLORS.background` fijo, `draw()` L778 | Variar tinte de fondo por "zonas" de puntaje |
| 14 | Visual | Jefe estático salvo movimiento lateral; sin animación ligada a `frameCount` | Media | boss ~L943-957, L569-588 | Parpadeo/pulso de aura animado |
| 15 | Visual | SFX con frecuencias fijas, suenan repetitivos en partidas largas | Baja | `playSound()` L186-229 | Variación aleatoria leve de pitch |
| 16 | UX | Todo el texto largo usa la fuente pixelada `Press Start 2P`, poco legible en bloques largos | Baja | CSS L21, `drawMenu/GameOver/Win` | Usar sans-serif para bloques largos, reservar pixelada para títulos/HUD |
| 17 | UX | Zona táctil de disparo (25%-75%) no está señalada visualmente en el juego | Baja | `touchstart` L1338-1343 | Franja translúcida indicativa |
| 18 | Seguridad | `localStorage` (`spaceRunnerHighScore`) manejado correctamente (parseInt, solo `fillText`, nunca `innerHTML`) — sin riesgo actual | Informativa | Confirmado en L257, 610, 642, 674 | Ninguna acción; documentar que no es fuente de verdad si se agrega leaderboard online |
| 19 | Seguridad | Sin CI/CD ni verificación automática de HTML | Informativa | No existe `.github/workflows` | Mejora de proceso, no vulnerabilidad |
| 20 | UX | `user-scalable=no` fijado en el viewport | Baja (documentar) | meta viewport L5 | Limitación de accesibilidad conocida; no tocar sin decisión de producto |

## Mejoras rápidas (alto impacto, bajo esfuerzo)
1. Escalar y reposicionar proporcionalmente el texto de `drawMenu()` y `drawWin()` con `getScaleFactor()`/`canvas.height` — corrige el bug más grave detectado.
2. Dar velocidades de paralaje fijas y distintas a estrellas/planetas/asteroides, independientes de `game.speed` — mayor sensación de profundidad con un cambio mínimo.
3. Variar la forma de las partículas según el evento (explosión angular vs. brillo de power-up) — refuerza la lectura "bueno/malo" sin nuevos assets.
4. Añadir un ícono de pausa y uno de mute tocables en el HUD — paridad táctil/teclado.
5. Agregar un `vercel.json` con cabeceras de seguridad básicas (CSP, `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`).

## Hallazgos que requieren decisión de producto o de diseño (no deben resolverse de forma autónoma)
- Rediseñar siluetas de enemigos por tipo (accesibilidad daltonismo + refuerzo visual) — implica tocar sprites.
- Sustituir/combinar la fuente pixelada por una más legible en bloques largos de texto — impacta la identidad retro del juego.
- Habilitar `user-scalable` o dar alternativa de tamaño de texto/alto contraste.
- Progresión visual del fondo por "zonas" de puntaje y escalado dinámico de BPM/capas musicales — cambios de diseño de contenido, no solo de código.
- Función de compartir puntaje al finalizar partida (alcance no definido).

## Lista de verificación previa al despliegue
1. Grep de `innerHTML`, `eval(`, `fetch(` nuevos antes de cada commit.
2. Confirmar que sigue faltando (o ya se agregó) `vercel.json`/`_headers` con cabeceras de seguridad — hueco conocido hasta que se implemente.
3. Verificar HTTPS forzado en el panel de la plataforma de hosting si se usa dominio propio.
4. Si se toca `localStorage`, confirmar que el valor se siga tratando como número y solo se pinte vía Canvas.
5. `grep -rEni "api[_-]?key|secret|token=|password"` sobre archivos modificados antes de commitear.
