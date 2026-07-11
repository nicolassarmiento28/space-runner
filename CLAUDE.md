# CLAUDE.md — Space Runner

Guía para Claude Code y otros asistentes de codificación que trabajen en este repositorio.

## Qué es este proyecto

Juego arcade tipo endless-runner + shooter, en un solo archivo HTML5 Canvas (`index.html`). JavaScript vanilla, sin frameworks, sin build, sin dependencias. Audio con Web Audio API (música procedural + efectos).

## Cómo ejecutar y probar

- Abrir `index.html` directamente en el navegador, o servir con `python -m http.server` / `npx http-server`.
- No hay build, lint ni tests automatizados. La verificación es manual: abrir el juego en el navegador y probar teclado, táctil y audio (el audio requiere interacción del usuario para iniciarse).
- Al hacer cambios de UI/gameplay, siempre probar el flujo real en el navegador antes de dar el trabajo por terminado.

## Estructura del repositorio

```
space-runner/
├── index.html       # Juego completo (HTML + CSS + JS en un solo archivo)
├── music.mp3         # Pista de fondo
├── README.md          # Documentación orientada al jugador
├── CLAUDE.md / AGENTS.md  # Esta guía (mismo contenido)
├── PLAN.md            # Plan de desarrollo
├── docs/               # Documentación adicional
└── .claude/agents/    # Sub agentes personalizados para Claude Code
```

## Arquitectura del juego (dentro de index.html)

- **Loop principal**: `requestAnimationFrame`, sin deltaTime (movimiento atado a fps).
- **Estados**: `'menu' | 'playing' | 'gameover' | 'win'`.
- **Sistema de carriles**: 3 carriles verticales, el jugador se mueve entre ellos.
- **Entidades**: jugador, balas, enemigos (`normal`, `shooter`), powerups (`health`, `shield`, `multishot`), partículas, fondo parallax (estrellas, planetas, asteroides).
- **Audio**: `<audio>` para música de fondo + Web Audio API para beats procedurales y SFX. El `AudioContext` se crea de forma perezosa en la primera interacción del usuario.
- **Persistencia**: puntaje máximo en `localStorage`.

## Convenciones de código

- `camelCase` para variables y funciones, `PascalCase` para clases, `const`/`let` (nunca `var`).
- Colisión AABB con padding (`checkCollision`).
- Temporizadores basados en frames (cooldowns, spawn rate, aumento de velocidad).
- Todo el código vive en `index.html`: no dividir en módulos ni añadir un build system salvo que el usuario lo pida explícitamente.

## Cómo trabajar en este repo

- No introducir dependencias, bundlers ni frameworks — el proyecto es deliberadamente de archivo único sin build.
- Antes de "arreglar" algo, verificar el estado actual del código (los bugs documentados en versiones anteriores pueden ya no aplicar tras refactors).
- Cambios de gameplay o visuales: probar en el navegador (escritorio y, si es relevante, táctil) antes de reportar como terminado.
- Mantener el estilo existente (nombres, patrones de eventos, manejo de audio) en vez de introducir convenciones nuevas.

## Sub agentes

Los sub agentes personalizados para Claude Code viven en `.claude/agents/`. Cada uno es un archivo Markdown con frontmatter (`name`, `description`, `tools`, opcionalmente `model`) que define un agente especializado invocable con el Agent tool.
