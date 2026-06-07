# AGENTS.md - Space Runner Game Project

## Project Overview
Single-file HTML5 Canvas game combining endless runner + shooter mechanics. 3-lane vertical scroller with space theme. Vanilla JS + Web Audio API for procedural audio. No build system, no dependencies.

---

## Build / Run / Test Commands

### Running
- Open `index.html` directly in any modern browser
- For local dev: `python -m http.server` or `npx http-server`
- Deploy: Vercel / Netlify / GitHub Pages (push to git)

### No Build
Pure static HTML. No npm, no build, no lint, no tests.

### Manual Testing
- Test in Chrome, Firefox, Safari, Edge
- Test on mobile (touch + keyboard)
- Verify audio after user interaction
- Check localStorage persistence for high score

---

## Code Analysis (index.html - 1022 lines)

### File Structure
```
space-runner/
├── index.html       # Complete game (all-in-one)
├── README.md        # Game documentation
├── AGENTS.md        # This file
├── PLAN.md          # Development plan
└── music.mp3        # Background music track
```

### Known Bugs / Issues

1. **`updateLanes()` defined twice** — lines 264 and 335. Second definition overwrites the first. No runtime issue but code duplication.

2. **Shield powerup does nothing** — `spawnEntity()` creates 'shield' type (line 363), but collision handler only processes 'health' and 'multishot' (lines 566-571). Shield pickup is silently consumed.

3. **`musicGain` never initialized** — `scheduleMusic()` connects oscillators to `musicGain` but it's never assigned to a `GainNode`. Procedural music (kick/snare/hihat/bass) will fail with silent audio errors.

4. **Collision padding bug with enemy bullets** — `checkCollision` uses padding `p=8`, but enemy bullets have `width:6, height:10`. Formula `b.x + b.width - p` = `b.x - 2` shifts collision box negatively. Small bullets may pass through player.

5. **No deltaTime** — All movement is frame-based (tied to 60fps). Game runs slower/faster on different refresh rates (120Hz monitors = 2x speed).

6. **LANE_WIDTH / LANE_START_X named as constants but are functions** — SCREAMING_SNAKE_CASE suggests constants, but they're arrow functions that compute dynamically.

7. **No swipe up/down on mobile** — Menu shows "DESLIZAR ARRIBA / ABAJO" as controls, but only horizontal swipe is implemented in `touchend` handler.

### Game Architecture

#### Rendering
- Canvas fills viewport (`resizeCanvas()`)
- `requestAnimationFrame` game loop
- Parallax background: stars → planets → asteroids
- 3-lane system: player moves between lanes 0, 1, 2
- HUD: score, lives (hearts), high score, speed bar, multishot indicator

#### Game States
- `'menu'` — Title screen with controls and objective
- `'playing'` — Active gameplay with update/draw loop
- `'gameover'` — Death screen, final score, high score check
- `'win'` — Victory screen at 1500 points

#### Audio System
- HTML `<audio>` element (`music.mp3`) for background track
- Web Audio API for procedural beat (kick, snare, hihat, space bass)
- `playSound()` for SFX: 'shoot', 'collect', 'hit', 'gameover'
- AudioContext created lazily on first user interaction

#### Entity System
- **Player**: Ship drawn with gradients, engine glow. Width: 50, Height: 45
- **Bullets**: 3-bullet spread normally, 5-bullet fan when multishot active
- **Enemies**: 'normal' (+15 pts) and 'shooter' (+30 pts, fires downward)
- **Powerups**: 'health' (extra life up to 5), 'shield' (broken), 'multishot' (x5 for ~10s)
- **Particles**: Explosion particles on enemy kill / player hit
- **Background**: 150 stars (parallax), 3 planets, 5 asteroids

#### Progression
- Speed increases: +0.05 every 60 frames, capped at 15
- Spawn rate increases as speed rises (interval: 50 → 25 frames)
- Win at 1500 points
- High score persisted via `localStorage('spaceRunnerHighScore')`

### Key Functions

| Function | Location | Purpose |
|---|---|---|
| `initAudio()` | 68-75 | Create AudioContext on first interaction |
| `startMusic()` | 77-84 | Play HTML audio track |
| `scheduleMusic()` | 90-106 | Schedule procedural beats (scheduling loop) |
| `playSound(type)` | 172-215 | Play SFX ('shoot','collect','hit','gameover') |
| `Particle` class | 217-238 | Explosion particles with fade-out |
| `initGame()` | 270-293 | Reset all state for new game |
| `spawnEntity()` | 341-383 | Spawn enemies/powerups with weighted random |
| `shoot()` | 391-409 | Fire player bullets (3 or 5 spread) |
| `update()` | 411-607 | Main update logic (movement, collisions, spawning) |
| `checkCollision(a, b)` | 609-613 | AABB collision with padding |
| `draw()` | 615-672 | Main render function |
| `drawRoad()` | 674-676 | Placeholder (empty - no lane lines) |
| `drawPlayer()` | 678-716 | Draw ship with gradient, cockpit, engines |
| `drawMenu()` | 799-847 | Title screen with controls |
| `drawGameOver()` | 849-877 | Game over overlay |
| `drawWin()` | 879-905 | Victory overlay |
| `gameLoop()` | 907-915 | requestAnimationFrame loop |

### Controls

#### Keyboard (keydown)
- `Space` — Start game / Shoot
- `ArrowUp` / `W` — Move up (decrease lane, min 0)
- `ArrowDown` / `S` — Move down (increase lane, max 2)
- Cooldown: 3 frames between lane changes

#### Touch
- `touchstart` — Start game / Shoot (if in center 25%-75% zone)
- `touchend` — Swipe detection: horizontal swipe changes lane
- Swipe threshold: 30px, within 300ms, horizontal > vertical distance

---

## Code Style Conventions

### Naming
- Variables: camelCase (`gameScore`, `maxSpeed`)
- Classes: PascalCase (`class Particle`)
- Constants: SCREAMING_SNAKE_CASE (but current code uses functions, not constants)
- Canvas elements: descriptive (`canvas`, `ctx`)

### Patterns
- Arrow functions for callbacks and short logic
- `function` declarations for game methods
- `const`/`let`, no `var`
- Object literal for game state
- AABB collision with padding for forgiveness
- Frame-based timers (cooldown, spawn interval, speed increase)

### Audio Initialization (Required Pattern)
```javascript
function initAudio() {
    if (!audioCtx) {
        try {
            audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        } catch (e) { console.log('Web Audio not supported:', e); }
    }
    if (audioCtx && audioCtx.state === 'suspended') {
        audioCtx.resume().catch(() => {});
    }
}
```
Always call `initAudio()` on first user interaction (keydown, touchstart, click).

### Event Listener Pattern
```javascript
document.addEventListener('keydown', (e) => {
    initAudio();
    if (e.code === 'Space') {
        e.preventDefault();
        // Handle
    }
});
```

### Touch Handling
```javascript
canvas.addEventListener('touchstart', (e) => {
    e.preventDefault();  // Always prevent default
    // Handle
}, { passive: false });
```

### Entity Collision (AABB with padding)
```javascript
function checkCollision(a, b) {
    const p = 8;
    return a.x + p < b.x + b.width - p &&
           a.x + a.width - p > b.x + p &&
           a.y + p < b.y + b.height - p &&
           a.y + a.height - p > b.y + p;
}
```

---

## Performance Guidelines
- `requestAnimationFrame` (not setInterval)
- Frame-rate dependent updates (no deltaTime — known limitation)
- Array mutation with `splice()` in reverse loops
- Particle limit via lifecycle (life decreases 0.04/frame)
- Shadow blur for glow effects (moderate perf cost)

---

## Common Issues

### Audio not working on mobile
- AudioContext requires user gesture
- Must be resumed if suspended after browser tab switch

### Controls not responding
- Check `game.state === 'playing'`
- `moveCooldown` prevents spam
- `canMove` flag disabled during lane transition animation

### Shield powerup feels broken
- It is broken — shield type is spawned but never implemented in collision handler

### Game runs too fast/slow  
- No deltaTime means 120Hz displays run at 2x speed
- Frame-based cooldowns break on non-60fps displays

---

## Portability
Runs anywhere with HTML5 Canvas + Web Audio. Chrome, Firefox, Safari, Edge. Mobile + Desktop. No internet needed (except Google Fonts on first load). Single file deployment.
