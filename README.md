# SPACE RUNNER

Juego web interactivo desarrollado con JavaScript, HTML5 y CSS3 enfocado en lógica de juego, renderizado dinámico y experiencia arcade en navegador. Implementé mecánicas interactivas y control de eventos en tiempo real.

🎮 **Jugar ahora:** https://space-runner-theta.vercel.app

## 🎮 Cómo jugar

### Controles PC
- **A / D** o **Flechas ← →** - Mover entre carriles
- **ESPACIO** - Disparar (3 direcciones, 5 en abanico con multishot) / Iniciar juego
- **ESC / P** - Pausar / reanudar
- **M** - Silenciar / activar audio

### Controles Táctil (Móvil/Tablet)
- **Swipe izquierda / derecha** - Cambiar de carril
- **Tocar zona central (25%-75% de la pantalla)** - Disparar
- **Tocar pantalla** - Iniciar o reiniciar partida
- **Botones en la esquina inferior derecha** - Pausa y silenciar

## 🎯 Objetivo

- Destruye enemigos:
  - Normal: +15 puntos
  - Shooter (dispara): +30 puntos
  - Speedster (cambia de carril, forma de rombo): +25 puntos
  - Tanque (3 golpes, con barra de vida): +50 puntos
  - Jefe (cada 500 puntos, 10 golpes, ojos que parpadean): +200 puntos
- Encadena eliminaciones para activar **combos** (multiplicador de puntaje x2/x3/x5)
- Recolecta powerups: +40 puntos
  - **(+)** = Agrega 1 vida
  - **(S)** = Escudo temporal (invulnerabilidad ~3s)
  - **(M)** = Multishot x5 temporal (~10 segundos)
- **GANA** al llegar a **1500 puntos**
- **PIERDE** al perder todas sus **3 vidas**

## 🌌 Características del Juego

- Fondo espacial con estrellas, planetas y asteroides en paralaje diferenciado por profundidad
- El tinte del fondo cambia de "zona" cada 500 puntos
- Música procedural generada en tiempo real (Web Audio API), con tempo que escala con la velocidad y el jefe
- Efectos de partículas variados según el evento (impacto, recolección, explosión)
- Sistema de combos y mensajes de racha
- Jefes con animación (parpadeo de ojos, pulso de aura)
- Sistema de puntuación con record guardado (`localStorage`) y botón de compartir puntaje
- Pausa y silencio accesibles por teclado y por botones táctiles
- Controles táctiles con detección de swipe horizontal
- Funciona en PC y dispositivos móviles

## 🛠️ Tecnologías Usadas

- **HTML5 Canvas** - Renderizado del juego
- **JavaScript ES6+** - Lógica del juego sin librerías
- **Web Audio API** - Música y efectos de sonido procedurales
- **localStorage** - Guardado de puntuación más alta y preferencia de silencio

## 📁 Estructura del Proyecto

```
space-runner/
├── index.html         # Juego completo (todo en un archivo)
├── vercel.json         # Cabeceras de seguridad para el despliegue
├── README.md            # Este archivo
├── CLAUDE.md / AGENTS.md  # Guía para agentes de código (Claude Code y otros)
├── PLAN.md               # Historial de desarrollo y backlog pendiente
├── docs/audit-report-*.md  # Informes de auditoría del proyecto
└── .claude/agents/         # Sub agentes personalizados para Claude Code
```

## 🚀 Cómo Ejecutar

### Opción 1: Abrir directamente
Simplemente abre el archivo `index.html` en cualquier navegador moderno.

### Opción 2: Servidor local
Si prefieres usar un servidor local:

```bash
# Python 3
python -m http.server 8000

# Node.js
npx http-server

# PHP
php -S localhost:8000
```

Luego abre `http://localhost:8000` en tu navegador.

### Opción 3: Despliegue en la nube
El juego está desplegado en **Vercel** (https://space-runner-theta.vercel.app), con `vercel.json` configurando cabeceras de seguridad (CSP, `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`). También funciona en Netlify o GitHub Pages arrastrando la carpeta o conectando el repositorio.

## 🔧 Solución de Problemas

### No se escucha la música
- El audio requiere interacción del usuario para funcionar
- Intenta presionar una tecla o tocar la pantalla antes de jugar
- Verifica que no esté silenciado (tecla `M` o el ícono de audio en el HUD)

### Los controles no responden
- Verifica que el juego esté en estado "playing" (no en pausa)
- Espera un momento entre pulsaciones (hay cooldown de movimiento)

### La pantalla está en blanco
- Verifica que tu navegador soporte HTML5 Canvas
- Prueba con Chrome, Firefox, Safari o Edge

## 📝 Notas

- El canvas se adapta al tamaño de la ventana/pantalla en tiempo real
- Se permite hacer zoom (pinch) por accesibilidad
- No requiere instalación ni dependencias
- Funciona sin conexión a internet (excepto por la fuente de letra del título/HUD)

---

Desarrollado por Nicolás Sarmiento
