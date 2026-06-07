# Space Runner — Game Improvements Design

> **Approved:** 2026-06-07
> **Status:** Spec complete, ready for implementation planning

**Goal:** Evolve Space Runner from a functional prototype to a polished game with better visuals, deeper gameplay, and cleaner code.

**Architecture:** Single-file HTML5 Canvas + Vanilla JS + Web Audio API. No build system. All changes within `index.html`.

**Tech Stack:** HTML5 Canvas, JavaScript ES6+, Web Audio API, localStorage

---

## Scope

Three dimensions of improvement, all in a single file:

1. **Visual** — HUD redesign, screen shake, hit flash, lane lines, menu polish, thruster effects
2. **Gameplay** — vertical swipe, shield powerup fix, new enemies (tank/speedster/boss), deltaTime, combo system, pause screen
3. **Code** — bug fixes (updateLanes duplicate, musicGain null, shield broken, collision padding), refactoring (rename constants, object pooling, split update()), performance optimizations

---

## Visual Improvements

### 1. HUD Redesign
- Hearts drawn with Canvas bezier curves instead of Unicode `♥` — consistent cross-platform rendering
- Speed bar with cyan-to-white gradient and subtle glow pulse
- "Score pop" floating numbers when earning points
- Multishot indicator with animated border
- Combo multiplier display when active

### 2. Screen Shake
- On player hit: 10-frame shake with decaying intensity
- On enemy kill: brief 3-frame micro-shake
- Implemented via `ctx.save()` / `ctx.translate(shakeX, shakeY)` / `ctx.restore()` in draw loop

### 3. Hit Flash
- Player sprite flashes (alpha toggles every 2 frames) for 20 frames after taking damage
- `game.player.hitFlash` counter decrements each frame

### 4. Lane Lines
- Vertical dashed lane dividers with animated `lineDashOffset`
- Semi-transparent white (`rgba(255,255,255,0.15)`) — visible but subtle
- Only drawn during `'playing'` state

### 5. Menu Enhancements
- Animated player ship preview drifting in background
- Controls displayed as visual icons (keyboard + touch)
- Difficulty select: Easy / Normal / Hard (affects spawn rate and speed cap)
- High score displayed with glow effect

### 6. Dynamic Thruster Effect
- Engine flame flickers using `Math.random()` offset each frame
- Small particles trail downward from engines
- Flame size varies with current speed

---

## Gameplay Improvements

### 1. Vertical Swipe on Mobile
- `touchend` handler checks both `deltaX` and `deltaY`
- Swipe threshold: 30px within 300ms
- Vertical swipe up = move up (lane--), down = move down (lane++)
- Menu already claims this works — now it actually will

### 2. Shield Powerup (Fix)
- `shieldActive` flag + `shieldTimer` (180 frames = ~3s at 60fps)
- During shield: player is immune to all damage
- Visual: cyan energy ring drawn around ship (`ctx.arc` with glow)
- Timer decrements each frame; auto-deactivates when timer hits 0
- Properties added to `game.player` and reset in `initGame()`

### 3. New Enemy Types

#### Tank
- HP: 3, size: 55×50, score: +50
- Visual: larger, purple tint, mini HP bar on top
- Collision: bullet removes 1 HP instead of instant kill
- Shoots occasionally like shooter type

#### Speedster
- No HP (1-hit kill), score: +25
- Moves horizontally between lanes (oscillates)
- Smaller size (35×30), faster fall speed
- Red/orange color scheme

#### Boss
- Spawns every 500 points
- HP: 10, size: 80×70, score: +200
- Pattern: moves side-to-side, shoots burst of 3 bullets
- Visual: large, dark red/purple with glow
- Spawns particle explosion on death

### 4. DeltaTime
- `gameLoop(timestamp)` receives timestamp from `requestAnimationFrame`
- `dt = Math.min((timestamp - lastTime) / 16.67, 3)` — capped at 3x slowdown
- All movement multiplied by `dt`
- Frame-based counters (cooldowns, timers) remain frame-based but adjusted
- Speed increment changes from frame-based to time-based accumulator

### 5. Combo System
- `game.combo` counter: increments on each enemy kill, resets on player hit
- Multiplier: x1 (0-4 kills), x2 (5-9), x3 (10-19), x5 (20+)
- Score awarded = base points × multiplier
- Visual: combo counter pulses in HUD, screen message at new tiers ("x2 COMBO!")
- Resets to 0 on player hit or shield activation

### 6. Pause Screen
- `'paused'` state added to game state machine
- Toggle with Escape or P key
- Overlay: dark semi-transparent background, "PAUSA" title
- Options: Continue, Reiniciar, Salir al menú
- Music stops on pause, resumes on continue

---

## Code Improvements

### Bug Fixes

| # | Bug | Location | Fix |
|---|-----|----------|-----|
| 1 | `updateLanes()` defined twice | Lines 264 & 335 | Remove second definition |
| 2 | `musicGain` never initialized | Line 64, used in playKick/playSnare/etc | Create GainNode in startMusic() |
| 3 | Shield spawned but never handled | Lines 566-571 | Add shield case in collision handler |
| 4 | Enemy bullet collision padding | Lines 609-613 | Dynamic padding: `Math.min(8, min(a.width,b.width)/2)` |

### Refactoring

#### Rename SCREAMING_SNAKE_CASE functions
- `LANE_WIDTH` → `getLaneWidth` (it's an arrow function, not a constant)
- `LANE_START_X` → `getLaneStartX`
- Update all references (currently 5 usages)

#### Object Pooling for Particles
- Hard cap: `MAX_PARTICLES = 100`
- `spawnParticle()` checks array length before pushing
- Consider reusing dead particles from the pool

#### Split update() function
Current monolithic `update()` (~200 lines) split into:
- `updatePlayer(dt)` — movement interpolation, cooldowns
- `updateEnemies(dt)` — enemy movement, shooting, off-screen cull
- `updateBullets(dt)` — bullet movement, off-screen cull
- `updateCollisions()` — player-enemy, bullet-enemy, player-powerup
- `updatePowerups(dt)` — powerup movement, collision
- `updateParticles()` — existing particle update loop
- `updateBackground(dt)` — stars, planets, asteroids
- `updateSpawning()` — spawn timer logic
- `updateSpeed()` — speed increment logic
- `updateCombo()` — combo decay/persistence

Each function is called in sequence from `update(dt)`.

### Performance Optimizations
- DeltaTime fixes the biggest perf issue (120Hz = 2x speed)
- Disable `shadowBlur` on mobile (detected via `'ontouchstart' in window`)
- Limit particles to 100 max (already proposed in pooling)
- Batch canvas state changes: minimize `save()`/`restore()` calls

---

## Implementation Order

### Phase 1: Critical Bugs
1. ~updateLanes() duplicate (2 min)
2. ~musicGain initialization (5 min)
3. Shield powerup (15 min)
4. Enemy bullet collision (3 min)

### Phase 2: Gameplay Foundation
5. Vertical swipe (5 min)
6. DeltaTime (20 min)
7. Pause screen (10 min)

### Phase 3: Visual Polish
8. HUD redesign (15 min)
9. Screen shake + hit flash (10 min)
10. Lane lines + thruster effect (10 min)
11. Menu enhancements (15 min)

### Phase 4: Content Expansion
12. New enemies: Tank, Speedster, Boss (30 min)
13. Combo system (10 min)

### Phase 5: Code Quality
14. Rename constants (5 min)
15. Object pooling (5 min)
16. Split update() (15 min)
17. Performance tweaks (5 min)

---

## Testing Checklist

- [ ] Game loads without console errors
- [ ] Shield powerup grants temporary invulnerability
- [ ] Vertical swipe works in both directions on mobile
- [ ] Game runs at same speed on 60Hz and 120Hz displays
- [ ] Pause/unpause works correctly (Escape/P)
- [ ] Tank enemy requires 3 hits and shows HP
- [ ] Speedster enemy moves between lanes
- [ ] Boss spawns at 500-point intervals
- [ ] Combo counter increments and resets correctly
- [ ] Screen shake only on player hit
- [ ] Lane lines are visible but subtle
- [ ] HUD shows accurate score, lives, speed, combo
- [ ] Procedural music plays without errors
- [ ] `localStorage` high score persists across sessions
- [ ] Touch controls work on mobile (no default behavior conflicts)
