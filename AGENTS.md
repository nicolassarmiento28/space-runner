# AGENTS.md - Space Runner Game Project

## Project Overview
This is a single-file HTML5 Canvas game (space-runner) combining endless runner and shooter mechanics. It uses vanilla JavaScript with Web Audio API for procedural audio. No build system or dependencies - runs directly in browser.

---

## Build / Run / Test Commands

### Running the Game
- Open `index.html` directly in any modern browser
- For local development: use any HTTP server (e.g., `python -m http.server`)
- Deploy to Vercel/Netlify/GitHub Pages by pushing to git

### No Build Required
This is a pure static HTML file. No:
- `npm install`
- `npm run build`
- `npm test`
- Linting or type checking

### Manual Testing
- Test in Chrome, Firefox, Safari, Edge
- Test on mobile (touch controls)
- Test keyboard + touch input
- Verify audio works (Web Audio API requires user interaction)

---

## Code Style Guidelines

### General Principles
- Keep it simple - single HTML file with embedded CSS/JS
- No external dependencies except Google Fonts CDN
- ES6+ JavaScript (const/let, arrow functions, template literals)
- 80-character line limit where practical

### HTML Structure
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Game Title</title>
    <!-- External fonts only -->
    <link href="..." rel="stylesheet">
    <style>
        /* CSS here */
    </style>
</head>
<body>
    <canvas id="gameCanvas"></canvas>
    <script>
        // JS here
    </script>
</body>
</html>
```

### JavaScript Style

#### Constants & Variables
- Use `const` for values that won't change
- Use `let` for mutable values
- Avoid `var`

#### Functions
- Use arrow functions for callbacks and short functions
- Use `function` declarations for game methods

#### Naming Conventions
```javascript
// Variables: camelCase
let gameScore = 0;
const maxSpeed = 15;

// Classes (if used): PascalCase
class Particle { }

// Constants: SCREAMING_SNAKE_CASE
const LANE_WIDTH = 150;
const LANE_START_X = 75;

// Canvas elements: descriptive names
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
```

#### Game Object Structure
```javascript
const game = {
    state: 'menu',        // 'menu', 'playing', 'gameover', 'win'
    score: 0,
    lives: 3,
    player: {
        lane: 1,
        targetLane: 1,
        x: 0,
        y: 0,
        canMove: true,
        moveCooldown: 0,
        shootCooldown: 0
    },
    // ... other properties
};
```

#### Event Handlers
```javascript
document.addEventListener('keydown', (e) => {
    initAudio(); // Always init audio on user interaction
    
    if (e.code === 'Space') {
        e.preventDefault();
        // Handle space key
    }
});
```

### CSS Guidelines
- Use flexbox for centering
- Avoid !important
- Use CSS custom properties for colors if needed
- Keep styles in the `<style>` tag at top of file

### Error Handling

#### Audio Initialization
```javascript
function initAudio() {
    if (!audioCtx) {
        try {
            audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        } catch (e) {
            console.log('Web Audio not supported:', e);
        }
    }
    // Resume if suspended (mobile browsers)
    if (audioCtx && audioCtx.state === 'suspended') {
        audioCtx.resume().catch(() => {});
    }
}
```

#### Safe Property Access
```javascript
// Use optional chaining or defaults
const score = game?.score ?? 0;
const lives = game.lives || 3;

// Check before array access
if (game.enemies && game.enemies.length > 0) { }
```

### Performance Guidelines
- Use `requestAnimationFrame` for game loop
- Object pooling for particles (don't create new objects every frame)
- Limit particle count to ~100
- Use simple collision detection (AABB)
- Avoid creating objects in the game loop

### Mobile/Touch Support
```javascript
// Always prevent default on touch to avoid scrolling
canvas.addEventListener('touchstart', (e) => {
    e.preventDefault();
    // Handle touch
}, { passive: false });
```

### Code Organization (Suggested Order)
1. Canvas and context setup
2. Constants and colors
3. Audio system (initAudio, startMusic, playSound)
4. Game state object
5. Helper functions (initGame, spawnEntity, etc.)
6. Update function
7. Draw functions
8. Event listeners
9. Game loop
10. Initialization

### Comments
- Use comments for complex logic only
- Avoid obvious comments
- Document game mechanics and algorithms

---

## File Structure
```
space-runner/
├── index.html    # Complete game (HTML + CSS + JS)
└── README.md     # Game documentation
```

---

## Common Issues & Solutions

### Audio not working on mobile
- Always call `initAudio()` on first user interaction
- Resume AudioContext if suspended

### Controls not responding
- Check `canMove` flag before allowing movement
- Use `moveCooldown` to prevent spam

### Game runs too fast/slow
- Use `requestAnimationFrame` (not setInterval)
- Frame-rate independent movement (multiply by deltaTime)

---

## Portability
This single-file approach allows the game to run anywhere HTML5 is supported. No build steps, no dependencies, no configuration needed.