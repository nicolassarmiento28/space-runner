# Space Runner — Game Improvements Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Fix 4 bugs, add 8 gameplay/visual features, and refactor the codebase — all within `index.html`.

**Architecture:** All changes in a single file (`index.html`). No build system, no dependencies. Each task is self-contained and produces a working game.

**Tech Stack:** HTML5 Canvas, JavaScript ES6+, Web Audio API

**Files:**
- Modify: `C:\Users\Nicolas Sarmiento\Desktop\space-runner\index.html` (only file)

---

## Phase 1: Bug Fixes

### Task 1.1: Remove duplicate `updateLanes()`

**Files:**
- Modify: `index.html:335-339`

- [ ] **Remove the second `updateLanes()` definition**

Delete lines 335-339 (the second occurrence of `updateLanes()`):

```javascript
function updateLanes() {
    for (let i = 0; i < 3; i++) {
        game.player.lanePositions[i] = LANE_START_X() + i * LANE_WIDTH() - game.player.width / 2;
    }
}
```

Keep the first definition at lines 264-268 — it is identical.

- [ ] **Verify**

Open `index.html` in browser. Game loads, player appears centered, no console errors.

---

### Task 1.2: Initialize `musicGain` GainNode

**Files:**
- Modify: `index.html:63-64`, `index.html:77-84`

- [ ] **Add `musicGain` initialization in `startMusic()`**

Replace the `startMusic()` function (lines 77-84):

```javascript
function startMusic() {
    if (!audioCtx) initAudio();
    if (!musicGain) {
        musicGain = audioCtx.createGain();
        musicGain.gain.value = 0.3;
        musicGain.connect(audioCtx.destination);
    }
    stopMusic();
    bgMusic.currentTime = 0;
    const p = bgMusic.play();
    if (p) p.catch(() => {});
    musicPlaying = true;
    scheduleMusic();
}
```

Also ensure `scheduleMusic()` is called so procedural audio starts. Add the call at the end of `startMusic()`.

- [ ] **Verify**

Start a game, shoot, wait for procedural kick/snare/hihat. Open console — no `musicGain is null` errors. Audio plays correctly.

---

### Task 1.3: Implement Shield powerup

**Files:**
- Modify: `index.html:247`, `index.html:279`, `index.html:519`, `index.html:542`, `index.html:565-571`, `index.html:597-600`, `index.html:678-716`

- [ ] **Add shield properties to player object**

On line 247, add `shieldActive` and `shieldTimer`:

```javascript
player: {
    lane: 1, targetLane: 1, x: 0, y: 0, canMove: true,
    moveCooldown: 0, shootCooldown: 0, width: 50, height: 45,
    lanePositions: [0, 0, 0], multiShot: false, multiShotTimer: 0,
    shieldActive: false, shieldTimer: 0
},
```

- [ ] **Reset shield in initGame()**

After line 280 (`game.player.multiShotTimer = 0;`):

```javascript
game.player.shieldActive = false;
game.player.shieldTimer = 0;
```

- [ ] **Add shield collision handler**

In the powerup collision block (around line 565), add the shield case:

```javascript
if (checkCollision(game.player, p)) {
    if (p.type === 'health' && game.lives < 5) {
        game.lives++;
    } else if (p.type === 'multishot') {
        game.player.multiShot = true;
        game.player.multiShotTimer = 600;
    } else if (p.type === 'shield') {
        game.player.shieldActive = true;
        game.player.shieldTimer = 180;
    }
    // ... rest stays the same
```

- [ ] **Add shield timer decrement**

After the multishot timer block (after line 600):

```javascript
// Shield timer
if (game.player.shieldActive) {
    game.player.shieldTimer--;
    if (game.player.shieldTimer <= 0) game.player.shieldActive = false;
}
```

- [ ] **Make player invulnerable during shield**

On line 519, change collision check:

```javascript
if (!game.player.shieldActive && checkCollision(game.player, e)) {
```

On line 542, change enemy bullet collision:

```javascript
if (b.isEnemy && !game.player.shieldActive && checkCollision(game.player, b)) {
```

- [ ] **Draw shield visual on player**

In `drawPlayer()`, before `ctx.shadowBlur = 0;` at the end:

```javascript
// Shield aura
if (game.player.shieldActive) {
    ctx.shadowColor = COLORS.cyan;
    ctx.shadowBlur = 25;
    ctx.strokeStyle = COLORS.cyan;
    ctx.lineWidth = 3;
    ctx.beginPath();
    ctx.arc(px + 25, py + 22, 32, 0, Math.PI * 2);
    ctx.stroke();
    ctx.shadowBlur = 0;
}
```

- [ ] **Verify**

Play game. When shield powerup (circle with "S") is collected, a cyan ring appears around the ship for ~3 seconds. During that time, enemy collisions do not reduce lives.

---

### Task 1.4: Fix enemy bullet collision padding

**Files:**
- Modify: `index.html:609-613`

- [ ] **Change padding to be dynamic**

Replace the `checkCollision` function:

```javascript
function checkCollision(a, b) {
    const p = Math.min(8, Math.min(a.width, b.width) / 2);
    return a.x + p < b.x + b.width - p &&
           a.x + a.width - p > b.x + p &&
           a.y + p < b.y + b.height - p &&
           a.y + a.height - p > b.y + p;
}
```

- [ ] **Verify**

Play against shooter enemies. Enemy bullets (6×10) no longer pass through the player when they visually overlap. Collision feels fair.

---

## Phase 2: Gameplay Foundation

### Task 2.1: Implement vertical swipe on mobile

**Files:**
- Modify: `index.html:978-1002`

- [ ] **Add vertical swipe detection in touchend**

Replace the `touchend` handler (lines 978-1002):

```javascript
document.addEventListener('touchend', (e) => {
    e.preventDefault();
    if (game.state !== 'playing') return;

    const touch = e.changedTouches[0];
    const deltaY = touch.clientY - game.lastTouchY;
    const deltaX = touch.clientX - game.lastTouchX;
    const deltaTime = Date.now() - game.lastTouchTime;

    if (deltaTime < 300 && Math.abs(deltaX) > 30 && Math.abs(deltaX) > Math.abs(deltaY)) {
        if (deltaX < -30 && game.player.targetLane > 0) {
            game.player.targetLane--;
            game.player.moveCooldown = 0;
        } else if (deltaX > 30 && game.player.targetLane < 2) {
            game.player.targetLane++;
            game.player.moveCooldown = 0;
        }
    } else if (deltaTime < 300 && Math.abs(deltaY) > 30 && Math.abs(deltaY) > Math.abs(deltaX)) {
        if (deltaY < -30 && game.player.targetLane > 0) {
            game.player.targetLane--;
            game.player.moveCooldown = 0;
        } else if (deltaY > 30 && game.player.targetLane < 2) {
            game.player.targetLane++;
            game.player.moveCooldown = 0;
        }
    }

    game.player.moveCooldown = 0;
}, { passive: false });
```

- [ ] **Verify**

On mobile/device toolbar: swipe up moves to lane 0, swipe down moves to lane 2, swipe left/right also works. Horizontal swipes take priority when the angle is ambiguous.

---

### Task 2.2: Implement DeltaTime

**Files:**
- Modify: `index.html:411-607`, `index.html:907-915`

- [ ] **Add deltaTime to game loop**

Replace the `gameLoop()` function (lines 907-915):

```javascript
let lastTime = 0;

function gameLoop(timestamp) {
    try {
        const dt = lastTime ? Math.min((timestamp - lastTime) / 16.67, 3) : 1;
        lastTime = timestamp;
        update(dt);
        draw();
    } catch(e) {
        console.error('Game loop error:', e);
    }
    requestAnimationFrame(gameLoop);
}
```

- [ ] **Update `update()` to accept `dt` parameter**

Change function signature at line 411:

```javascript
function update(dt) {
```

- [ ] **Scale background movement by dt**

Replace the star update loop:

```javascript
for (let star of game.stars) {
    star.y += star.speed * (game.speed / 5) * dt;
    if (star.y > canvas.height) {
        star.y = 0;
        star.x = Math.random() * canvas.width;
    }
}
```

Same for planets:
```javascript
for (let planet of game.planets) {
    planet.y += planet.speed * (game.speed / 5) * dt;
    // ...
}
```

Same for asteroids:
```javascript
for (let ast of game.asteroids) {
    ast.y += ast.speed * (game.speed / 5) * dt;
    // ...
}
```

- [ ] **Scale enemy and powerup movement by dt**

In the enemies loop:
```javascript
e.y += game.speed * dt;
```

In the powerups loop:
```javascript
p.y += game.speed * dt;
```

- [ ] **Scale player lane interpolation by dt**

```javascript
game.player.x += (target - game.player.x) * 0.25 * dt;
```

- [ ] **Add time-based speed accumulator**

Replace the frame-based speed increment:

```javascript
// Speed (time-based instead of frame-based)
game.speedAccumulator += dt;
if (game.speedAccumulator >= 1) {
    game.speedAccumulator -= 1;
    if (game.speed < game.maxSpeed) game.speed += 0.05;
}
```

And add `speedAccumulator: 0` to the game object initialization (around line 258).

- [ ] **Scale spawn interval by dt**

```javascript
game.spawnTimer += dt;
const spawnInterval = Math.max(25, 50 - Math.floor(game.speed));
if (game.spawnTimer >= spawnInterval) {
    game.spawnTimer = 0;
    spawnEntity();
}
```

Add `spawnTimer: 0` to game object initialization.

- [ ] **Verify**

Open game on a 120Hz display (or use Chrome DevTools → Rendering → "Enable Frame Rendering Stats"). Game speed should be identical to 60Hz behavior. Movement feels consistent.

---

### Task 2.3: Implement pause screen

**Files:**
- Modify: `index.html:240-259` (game state), `index.html:917-941` (keydown), `index.html:615-672` (draw)

- [ ] **Add pause toggle in keydown handler**

Within the `keydown` listener, add before the Space check:

```javascript
if ((e.code === 'Escape' || e.code === 'KeyP') && (game.state === 'playing' || game.state === 'paused')) {
    e.preventDefault();
    if (game.state === 'playing') {
        game.state = 'paused';
        stopMusic();
    } else {
        game.state = 'playing';
        setTimeout(() => startMusic(), 100);
    }
    return;
}
```

- [ ] **Add `drawPause()` function**

After `drawWin()` (after line 905):

```javascript
function drawPause() {
    ctx.fillStyle = 'rgba(0, 0, 20, 0.85)';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    ctx.textAlign = 'center';

    ctx.font = '22px "Press Start 2P"';
    ctx.fillStyle = COLORS.white;
    ctx.fillText('PAUSA', canvas.width / 2, canvas.height / 2 - 60);

    ctx.font = '9px "Press Start 2P"';
    ctx.fillStyle = COLORS.lightBlue;
    ctx.fillText('ESC / P PARA CONTINUAR', canvas.width / 2, canvas.height / 2 + 10);
    ctx.fillText('ESPACIO PARA REINICIAR', canvas.width / 2, canvas.height / 2 + 40);
}
```

- [ ] **Call drawPause() in draw()**

After `drawHUD()` (after line 667), add:

```javascript
if (game.state === 'paused') drawPause();
```

- [ ] **Handle Space to restart from pause**

Modify the Space key handler at line 922 to check for 'paused':

```javascript
if (e.code === 'Space') {
    e.preventDefault();
    if (game.state === 'menu' || game.state === 'gameover' || game.state === 'win' || game.state === 'paused') {
        game.state = 'playing';
        initGame();
        setTimeout(() => startMusic(), 100);
    } else if (game.state === 'playing' && (!game.player.shootCooldown || game.player.shootCooldown === 0)) {
        shoot();
    }
}
```

- [ ] **Verify**

Press Escape during gameplay. Game pauses. Press Escape again — game resumes. Press Space — game restarts.

---

## Phase 3: Visual Polish

### Task 3.1: Screen shake + hit flash

**Files:**
- Modify: `index.html:240-259`, `index.html:270-293`, `index.html:519-537`, `index.html:540-558`, `index.html:615-672`, `index.html:678-716`

- [ ] **Add screenShake and camera properties**

To game object (around line 258):
```javascript
screenShake: 0,
```

To player in game object:
```javascript
// Already in player object — add:
hitFlash: 0,
```

- [ ] **Reset in initGame()**
```javascript
game.screenShake = 0;
game.player.hitFlash = 0;
```

- [ ] **Trigger shake on player hit**

In the player-enemy collision (line 523, after `game.lives--`):
```javascript
game.screenShake = 10;
game.player.hitFlash = 20;
```

In the enemy-bullet-player collision (line 546, after `game.lives--`):
```javascript
game.screenShake = 10;
game.player.hitFlash = 20;
```

- [ ] **Apply screen shake in draw()**

At the beginning of the `draw()` function, after clearing canvas (after line 618):

```javascript
// Screen shake
let shakeX = 0, shakeY = 0;
if (game.screenShake > 0) {
    shakeX = (Math.random() - 0.5) * game.screenShake * 0.8;
    shakeY = (Math.random() - 0.5) * game.screenShake * 0.8;
    game.screenShake--;
    ctx.save();
    ctx.translate(shakeX, shakeY);
}
```

At the very end of `draw()`, before the state-based draws, add restore:
```javascript
if (game.screenShake > 0 || shakeX !== 0) {
    // We saved at the start; restore at the very end
}
```

Actually, simpler approach — wrap the entire draw content in save/restore:

Restructure `draw()` from:

```javascript
function draw() {
    ctx.fillStyle = COLORS.background;
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    // ... all content ...
}
```

To:

```javascript
function draw() {
    ctx.save();
    if (game.screenShake > 0) {
        const sx = (Math.random() - 0.5) * game.screenShake * 0.8;
        const sy = (Math.random() - 0.5) * game.screenShake * 0.8;
        ctx.translate(sx, sy);
        game.screenShake--;
    }

    ctx.fillStyle = COLORS.background;
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    // ... all content ...

    ctx.restore();
}
```

- [ ] **Apply hit flash alpha in drawPlayer()**

At the start of `drawPlayer()`, after the shadow setup:

```javascript
if (game.player.hitFlash > 0) {
    ctx.globalAlpha = game.player.hitFlash % 4 < 2 ? 0.3 : 1.0;
}
```

And restore at the end (before `ctx.shadowBlur = 0`):
```javascript
ctx.globalAlpha = 1.0;
```

- [ ] **Decrement hitFlash in update()**

Somewhere in the update function (e.g., with the other timer decrements):
```javascript
if (game.player.hitFlash > 0) game.player.hitFlash--;
```

- [ ] **Verify**

Get hit by an enemy. Screen shakes briefly. Player flashes red/transparent. Both effects decay smoothly.

---

### Task 3.2: Lane lines + thruster effect

**Files:**
- Modify: `index.html:674-676`, `index.html:678-716`

- [ ] **Implement `drawRoad()`**

Replace the empty `drawRoad()`:

```javascript
function drawRoad() {
    const laneW = LANE_WIDTH();
    const startX = LANE_START_X();
    ctx.strokeStyle = 'rgba(255, 255, 255, 0.12)';
    ctx.lineWidth = 2;
    ctx.setLineDash([15, 18]);
    ctx.lineDashOffset = -game.frameCount * 3;

    for (let i = 0; i <= 3; i++) {
        const x = startX + i * laneW - laneW / 2;
        ctx.beginPath();
        ctx.moveTo(x, 0);
        ctx.lineTo(x, canvas.height);
        ctx.stroke();
    }
    ctx.setLineDash([]);
}
```

- [ ] **Add dynamic thruster flame in drawPlayer()**

Replace the static engine rectangles (lines 706-713) with:

```javascript
// Engines with dynamic flame
const flameHeight = 10 + Math.random() * 6;
const flameOffset = Math.random() * 2;

// Left engine
ctx.fillStyle = COLORS.orange;
ctx.fillRect(px + 8, py + 35, 8, 10);
ctx.fillStyle = COLORS.yellow;
ctx.fillRect(px + 10 + flameOffset, py + 38 - flameOffset, 4, 4);
// Flame trail
ctx.fillStyle = 'rgba(255, 136, 0, 0.4)';
ctx.fillRect(px + 9, py + 45, 6, flameHeight);

// Right engine
ctx.fillStyle = COLORS.orange;
ctx.fillRect(px + 34, py + 35, 8, 10);
ctx.fillStyle = COLORS.yellow;
ctx.fillRect(px + 36 - flameOffset, py + 38 - flameOffset, 4, 4);
ctx.fillStyle = 'rgba(255, 136, 0, 0.4)';
ctx.fillRect(px + 35, py + 45, 6, flameHeight);
```

- [ ] **Verify**

Lane lines appear as dashed vertical lines scrolling downward. Engine flames flicker randomly each frame. Feeling of speed is enhanced.

---

### Task 3.3: HUD redesign

**Files:**
- Modify: `index.html:769-797`

- [ ] **Replace `drawHUD()`**

Replace the entire `drawHUD()` function:

```javascript
function drawHUD() {
    // Score with glow
    ctx.shadowColor = COLORS.white;
    ctx.shadowBlur = 10;
    ctx.font = '14px "Press Start 2P"';
    ctx.fillStyle = COLORS.white;
    ctx.textAlign = 'left';
    ctx.fillText(`PUNTOS: ${game.score}`, 15, 25);
    ctx.shadowBlur = 0;

    // Hearts drawn with Canvas (not Unicode)
    for (let i = 0; i < game.lives; i++) {
        const hx = canvas.width - 30 - i * 25;
        const hy = 20;
        const s = 10;
        ctx.shadowColor = COLORS.red;
        ctx.shadowBlur = 6;
        ctx.fillStyle = COLORS.red;
        ctx.beginPath();
        ctx.moveTo(hx, hy + s/4);
        ctx.bezierCurveTo(hx - s/2, hy - s/4, hx - s, hy + s/3, hx, hy + s);
        ctx.bezierCurveTo(hx + s, hy + s/3, hx + s/2, hy - s/4, hx, hy + s/4);
        ctx.fill();
        ctx.shadowBlur = 0;
    }

    // High score
    ctx.font = '10px "Press Start 2P"';
    ctx.fillStyle = COLORS.lightBlue;
    ctx.textAlign = 'left';
    ctx.fillText(`MEJOR: ${game.highScore}`, 15, 45);

    // Speed bar with gradient glow
    const barX = 15, barY = 55, barW = 80, barH = 8;
    ctx.fillStyle = '#222';
    ctx.fillRect(barX, barY, barW, barH);

    const speedRatio = game.speed / game.maxSpeed;
    const grad = ctx.createLinearGradient(barX, barY, barX + barW, barY);
    grad.addColorStop(0, '#00ccff');
    grad.addColorStop(1, '#ffffff');
    ctx.shadowColor = '#00ccff';
    ctx.shadowBlur = 8;
    ctx.fillStyle = grad;
    ctx.fillRect(barX, barY, barW * speedRatio, barH);
    ctx.shadowBlur = 0;

    // Multishot indicator
    if (game.player.multiShot) {
        ctx.shadowColor = COLORS.orange;
        ctx.shadowBlur = 10;
        ctx.font = '9px "Press Start 2P"';
        ctx.fillStyle = COLORS.orange;
        ctx.textAlign = 'left';
        ctx.fillText('MULTI x5', 15, 82);
        ctx.shadowBlur = 0;
    }

    // Shield indicator
    if (game.player.shieldActive) {
        ctx.shadowColor = COLORS.cyan;
        ctx.shadowBlur = 10;
        ctx.font = '9px "Press Start 2P"';
        ctx.fillStyle = COLORS.cyan;
        ctx.textAlign = 'left';
        ctx.fillText('SHIELD', 15, 97);
        ctx.shadowBlur = 0;
    }
}
```

- [ ] **Verify**

Hearts render as actual heart shapes. Speed bar has cyan glow. Multishot and shield indicators are visible with glow effects.

---

## Phase 4: Content Expansion

### Task 4.1: New enemy types — Tank & Speedster

**Files:**
- Modify: `index.html:341-383`, `index.html:411-607`, `index.html:718-763`

- [ ] **Update spawn weights in `spawnEntity()`**

Replace the spawn logic:

```javascript
function spawnEntity() {
    const lane = Math.floor(Math.random() * 3);
    const rand = Math.random();

    if (rand < 0.10) {
        game.powerups.push({
            lane: lane, x: game.player.lanePositions[lane], y: -60,
            width: 35, height: 40, type: 'health'
        });
    } else if (rand < 0.20) {
        game.powerups.push({
            lane: lane, x: game.player.lanePositions[lane], y: -60,
            width: 35, height: 40, type: 'shield'
        });
    } else if (rand < 0.30) {
        game.powerups.push({
            lane: lane, x: game.player.lanePositions[lane], y: -60,
            width: 35, height: 40, type: 'multishot'
        });
    } else if (rand < 0.55) {
        // Tank enemy
        game.enemies.push({
            lane: lane, x: game.player.lanePositions[lane], y: -60,
            width: 55, height: 50, type: 'tank', hp: 3,
            shootTimer: Math.random() * 80
        });
    } else if (rand < 0.75) {
        // Speedster enemy
        const speedsterLane = Math.floor(Math.random() * 3);
        game.enemies.push({
            lane: speedsterLane, x: game.player.lanePositions[speedsterLane], y: -60,
            width: 35, height: 30, type: 'speedster',
            moveDir: Math.random() > 0.5 ? 1 : -1,
            moveTimer: 0
        });
    } else {
        // Normal or shooter
        game.enemies.push({
            lane: lane, x: game.player.lanePositions[lane], y: -60,
            width: 45, height: 40,
            type: Math.random() > 0.5 ? 'shooter' : 'normal',
            shootTimer: Math.random() * 80
        });
    }
}
```

- [ ] **Add tank and speedster logic in `update()`**

In the enemies update loop (around line 500), add before the collision check:

```javascript
// Tank: needs 3 hits
if (e.type === 'tank') {
    e.shootTimer++;
    if (e.shootTimer > 60 && Math.random() > 0.97) {
        game.bullets.push({
            x: e.x + e.width / 2 - 3, y: e.y + e.height,
            vx: 0, vy: 6, width: 6, height: 10, isEnemy: true
        });
        e.shootTimer = 40;
    }
}

// Speedster: moves between lanes
if (e.type === 'speedster') {
    e.moveTimer++;
    if (e.moveTimer > 30) {
        e.moveTimer = 0;
        const nextLane = e.lane + e.moveDir;
        if (nextLane < 0 || nextLane > 2) {
            e.moveDir *= -1;
        } else {
            e.lane = nextLane;
        }
    }
    // Interpolate to target lane position
    const targetX = game.player.lanePositions[e.lane];
    e.x += (targetX - e.x) * 0.1;
}
```

- [ ] **Update bullet-enemy collision for tank HP**

In the bullet-enemy collision (around line 474-495), replace the logic:

```javascript
if (b.x < e.x + e.width && b.x + b.width > e.x &&
    b.y < e.y + e.height && b.y + b.height > e.y) {
    game.bullets.splice(i, 1);
    if (e.type === 'tank') {
        e.hp--;
        if (e.hp <= 0) {
            game.enemies.splice(j, 1);
            game.score += 50;
            playSound('collect');
            spawnParticle(e.x + e.width/2, e.y + e.height/2, COLORS.purple, 25);
            checkWin();
        }
    } else {
        game.enemies.splice(j, 1);
        game.score += e.type === 'shooter' ? 30 : e.type === 'speedster' ? 25 : 15;
        playSound('collect');
        spawnParticle(e.x + e.width/2, e.y + e.height/2, COLORS.yellow, 15);
        checkWin();
    }
    break;
}
```

Extract `checkWin()` as a helper to avoid duplication:

```javascript
function checkWin() {
    if (game.score >= 1500) {
        game.state = 'win';
        stopMusic();
        if (game.score > game.highScore) {
            game.highScore = game.score;
            localStorage.setItem('spaceRunnerHighScore', game.highScore);
        }
    }
}
```

Replace all inline win checks with `checkWin()` calls.

- [ ] **Add tank drawing in `drawEntities()`**

In the enemy draw section (around line 720), add:

```javascript
// In the enemy drawing loop, inside the for...of:
if (e.type === 'tank') {
    ctx.fillStyle = '#553355';
    ctx.fillRect(e.x, e.y, e.width, e.height);
    ctx.fillStyle = COLORS.purple;
    ctx.fillRect(e.x + 3, e.y + 3, e.width - 6, 8);
    // HP bar
    const hpRatio = e.hp / 3;
    ctx.fillStyle = hpRatio > 0.5 ? COLORS.green : hpRatio > 0.25 ? COLORS.orange : COLORS.red;
    ctx.fillRect(e.x + 4, e.y + e.height - 6, (e.width - 8) * hpRatio, 4);
} else if (e.type === 'speedster') {
    ctx.fillStyle = '#553333';
    ctx.fillRect(e.x, e.y, e.width, e.height);
    ctx.fillStyle = COLORS.orange;
    ctx.fillRect(e.x + 3, e.y + 3, e.width - 6, 6);
} else {
    // Original normal/shooter drawing
    ctx.fillStyle = e.type === 'shooter' ? '#664433' : '#443344';
    ctx.fillRect(e.x, e.y, e.width, e.height);
    ctx.fillStyle = e.type === 'shooter' ? COLORS.orange : COLORS.red;
    ctx.fillRect(e.x + 5, e.y + 5, e.width - 10, 10);
    if (e.type === 'shooter') {
        ctx.fillStyle = COLORS.yellow;
        ctx.fillRect(e.x + e.width/2 - 3, e.y + e.height - 5, 6, 5);
    }
}
```

- [ ] **Verify**

Play the game. Tank enemies (large, purple, 3 HP bars) appear and take 3 hits. Speedster enemies (small, orange) move between lanes. All enemies are destroyable and award correct points.

---

### Task 4.2: Boss enemy

**Files:**
- Modify: `index.html:341-383`, `index.html:411-607`, `index.html:718-763`

- [ ] **Add boss state to game object**

Add to game object:
```javascript
bossActive: false,
bossSpawnScore: 500,
```

Reset in `initGame()`:
```javascript
game.bossActive = false;
game.bossSpawnScore = 500;
```

- [ ] **Add boss spawn check in update()**

At the end of `update()`, add:

```javascript
// Boss spawn check
if (!game.bossActive && game.score >= game.bossSpawnScore) {
    game.bossActive = true;
    game.bossSpawnScore += 500;
    game.enemies.push({
        lane: 1,
        x: game.player.lanePositions[1],
        y: -80,
        width: 80,
        height: 70,
        type: 'boss',
        hp: 10,
        moveDir: 1,
        shootTimer: 0
    });
}
```

- [ ] **Add boss behavior in update()**

In the enemies update loop:

```javascript
if (e.type === 'boss') {
    // Move side to side
    e.x += e.moveDir * 2 * dt;
    if (e.x > game.player.lanePositions[2] + 20) e.moveDir = -1;
    if (e.x < game.player.lanePositions[0] - 20) e.moveDir = 1;

    // Shoot burst
    e.shootTimer++;
    if (e.shootTimer > 40) {
        e.shootTimer = 0;
        for (let a = -1; a <= 1; a++) {
            game.bullets.push({
                x: e.x + e.width / 2 - 3 + a * 15,
                y: e.y + e.height,
                vx: a * 1.5,
                vy: 5,
                width: 6,
                height: 10,
                isEnemy: true
            });
        }
    }
}
```

- [ ] **Handle boss death in bullet-enemy collision**

In the tank HP logic, add a boss case:

```javascript
if (e.type === 'boss') {
    e.hp--;
    if (e.hp <= 0) {
        game.enemies.splice(j, 1);
        game.score += 200;
        game.bossActive = false;
        playSound('collect');
        spawnParticle(e.x + e.width/2, e.y + e.height/2, COLORS.purple, 50);
        checkWin();
    }
}
```

- [ ] **Draw boss in drawEntities()**

In the enemy drawing section:

```javascript
if (e.type === 'boss') {
    ctx.shadowColor = COLORS.purple;
    ctx.shadowBlur = 20;
    ctx.fillStyle = '#552244';
    ctx.fillRect(e.x, e.y, e.width, e.height);
    // Eyes / details
    ctx.fillStyle = COLORS.red;
    ctx.fillRect(e.x + 15, e.y + 20, 15, 10);
    ctx.fillRect(e.x + 50, e.y + 20, 15, 10);
    ctx.fillStyle = COLORS.yellow;
    ctx.fillRect(e.x + e.width/2 - 5, e.y + e.height - 10, 10, 10);
    // HP bar
    const hpRatio = e.hp / 10;
    ctx.fillStyle = COLORS.red;
    ctx.fillRect(e.x + 2, e.y + 2, (e.width - 4) * hpRatio, 5);
    ctx.shadowBlur = 0;
}
```

- [ ] **Verify**

Play until 500 points. A large purple boss appears with 10 HP, moves side-to-side, and fires 3-bullet bursts. Killing it awards 200 points and causes a big explosion.

---

### Task 4.3: Combo system

**Files:**
- Modify: `index.html:240-259`, `index.html:270-293`, `index.html:769-797`

- [ ] **Add combo properties to game object**

```javascript
combo: 0,
comboTimer: 0,
comboMessage: '',
comboMessageTimer: 0,
```

Reset in `initGame()`:
```javascript
game.combo = 0;
game.comboTimer = 0;
game.comboMessage = '';
game.comboMessageTimer = 0;
```

- [ ] **Update combo on enemy kill**

In the bullet-enemy collision, whenever an enemy is killed:

```javascript
game.combo++;
game.comboTimer = 120; // 2 seconds to get next kill
```

- [ ] **Reset combo on player hit**

Wherever lives are reduced:

```javascript
if (game.combo > 1) {
    game.comboMessage = `COMBO x${getComboMultiplier()} ROTO!`;
    game.comboMessageTimer = 60;
}
game.combo = 0;
game.comboTimer = 0;
```

- [ ] **Add combo multiplier function**

```javascript
function getComboMultiplier() {
    if (game.combo >= 20) return 5;
    if (game.combo >= 10) return 3;
    if (game.combo >= 5) return 2;
    return 1;
}
```

- [ ] **Use multiplier in score calculation**

When awarding score:

```javascript
game.score += basePoints * getComboMultiplier();
```

- [ ] **Add combo timer decrement in update()**

```javascript
if (game.comboTimer > 0) {
    game.comboTimer--;
    if (game.comboTimer === 0) {
        game.combo = 0;
    }
}
if (game.comboMessageTimer > 0) game.comboMessageTimer--;
```

- [ ] **Draw combo info in HUD**

In `drawHUD()`, add:

```javascript
// Combo indicator
if (game.combo >= 5) {
    ctx.shadowColor = COLORS.yellow;
    ctx.shadowBlur = 10;
    ctx.font = '12px "Press Start 2P"';
    ctx.fillStyle = COLORS.yellow;
    ctx.textAlign = 'center';
    ctx.fillText(`x${getComboMultiplier()} COMBO`, canvas.width / 2, 25);
    ctx.shadowBlur = 0;
}

// Combo message
if (game.comboMessageTimer > 0) {
    ctx.font = '10px "Press Start 2P"';
    ctx.fillStyle = COLORS.red;
    ctx.textAlign = 'center';
    ctx.fillText(game.comboMessage, canvas.width / 2, canvas.height / 2 - 50);
}
```

- [ ] **Verify**

Kill 5 enemies in a row without getting hit. "x2 COMBO" appears. Kill 10 for x3, 20 for x5. Get hit — combo resets, message "COMBO x5 ROTO!" appears briefly.

---

## Phase 5: Code Quality

### Task 5.1: Rename LANE_WIDTH and LANE_START_X

**Files:**
- Modify: `index.html:261-262` and all references

- [ ] **Rename the functions**

```javascript
const getLaneWidth = () => canvas.width / 3;
const getLaneStartX = () => canvas.width / 6;
```

- [ ] **Update all references**

Replace `LANE_WIDTH()` with `getLaneWidth()` (appears in `updateLanes()` and `drawRoad()`).
Replace `LANE_START_X()` with `getLaneStartX()` (same locations).

### Task 5.2: Object pooling for particles

**Files:**
- Modify: `index.html:385-389`

- [ ] **Add particle cap**

```javascript
const MAX_PARTICLES = 100;

function spawnParticle(x, y, color, count) {
    for (let i = 0; i < count; i++) {
        if (game.particles.length >= MAX_PARTICLES) break;
        game.particles.push(new Particle(x, y, color));
    }
}
```

### Task 5.3: Split monolithic update()

**Files:**
- Modify: `index.html:411-607`

- [ ] **Refactor into focused functions**

Split `update()` into:

```javascript
function update(dt) {
    if (game.state !== 'playing') return;
    game.frameCount++;

    updateBackground(dt);
    updatePlayer(dt);
    updateEnemies(dt);
    updateBullets(dt);
    updatePowerups(dt);
    updateParticles();
    updateTimers();
    updateSpawning(dt);
    updateSpeed(dt);
}
```

Each function contains the relevant block of code from the original `update()`. The blocks are already logically separated with comments in the existing code.

Define each function inline above `update()`:

```javascript
function updateBackground(dt) {
    // Stars, planets, asteroids (already separated by comments)
    for (let star of game.stars) { /* ... */ }
    for (let planet of game.planets) { /* ... */ }
    for (let ast of game.asteroids) { /* ... */ }
}

function updatePlayer(dt) {
    // Lane interpolation, cooldowns
    if (game.player.lane !== game.player.targetLane) { /* ... */ }
}

function updateEnemies(dt) {
    // Enemy movement, shooting, collision checks
    for (let i = game.enemies.length - 1; i >= 0; i--) { /* ... */ }
}

function updateBullets(dt) {
    // Bullet movement, enemy hits, player hits
    for (let i = game.bullets.length - 1; i >= 0; i--) { /* ... */ }
}

function updatePowerups(dt) {
    for (let i = game.powerups.length - 1; i >= 0; i--) { /* ... */ }
}

function updateParticles() {
    for (let i = game.particles.length - 1; i >= 0; i--) { /* ... */ }
}

function updateTimers() {
    if (game.player.multiShot) { /* ... */ }
    if (game.player.shieldActive) { /* ... */ }
    if (game.player.hitFlash > 0) game.player.hitFlash--;
    if (game.comboTimer > 0) { game.comboTimer--; if (game.comboTimer === 0) game.combo = 0; }
    if (game.comboMessageTimer > 0) game.comboMessageTimer--;
}

function updateSpawning(dt) {
    game.spawnTimer += dt;
    const interval = Math.max(25, 50 - Math.floor(game.speed));
    if (game.spawnTimer >= interval) {
        game.spawnTimer = 0;
        spawnEntity();
    }
    // Boss spawn
    if (!game.bossActive && game.score >= game.bossSpawnScore) { /* ... */ }
}

function updateSpeed(dt) {
    game.speedAccumulator += dt;
    if (game.speedAccumulator >= 1) {
        game.speedAccumulator -= 1;
        if (game.speed < game.maxSpeed) game.speed += 0.05;
    }
}
```

### Task 5.4: Menu enhancements

**Files:**
- Modify: `index.html:799-847`

- [ ] **Add animated ship preview in background**

At the start of `drawMenu()`, before the overlay:

```javascript
function drawMenu() {
    // Draw a small player ship drifting as preview
    const previewX = canvas.width / 2 - 25 + Math.sin(Date.now() / 2000) * 100;
    const previewY = 280;
    ctx.save();
    ctx.translate(previewX, previewY);
    ctx.shadowColor = COLORS.lightBlue;
    ctx.shadowBlur = 20;
    // Ship body (mini version)
    const g = ctx.createLinearGradient(0, 0, 50, 45);
    g.addColorStop(0, '#667788');
    g.addColorStop(0.5, '#889999');
    g.addColorStop(1, '#556677');
    ctx.fillStyle = g;
    ctx.beginPath();
    ctx.moveTo(25, 0);
    ctx.lineTo(50, 25);
    ctx.lineTo(42, 45);
    ctx.lineTo(8, 45);
    ctx.lineTo(0, 25);
    ctx.closePath();
    ctx.fill();
    ctx.shadowBlur = 0;
    ctx.restore();

    // ... rest of existing drawMenu content ...
}
```

Adjust the Y positions of text elements to account for the preview ship.

- [ ] **Verify**

Menu screen shows a player ship drifting left-right in the background behind the text. Visual is more engaging than static text.

---

## Summary: All Tasks

| # | Task | Est. Time |
|---|------|-----------|
| 1.1 | Remove duplicate updateLanes() | 1 min |
| 1.2 | Initialize musicGain | 3 min |
| 1.3 | Implement shield powerup | 10 min |
| 1.4 | Fix enemy bullet collision | 3 min |
| 2.1 | Vertical swipe | 5 min |
| 2.2 | DeltaTime | 15 min |
| 2.3 | Pause screen | 8 min |
| 3.1 | Screen shake + hit flash | 10 min |
| 3.2 | Lane lines + thrusters | 8 min |
| 3.3 | HUD redesign | 10 min |
| 4.1 | Tank & Speedster enemies | 15 min |
| 4.2 | Boss enemy | 12 min |
| 4.3 | Combo system | 8 min |
| 5.1 | Rename constants | 3 min |
| 5.2 | Object pooling | 3 min |
| 5.3 | Split update() | 10 min |
| 5.4 | Menu enhancements | 10 min |

**Total estimated time:** ~2 hours

---

## Testing

No test framework exists for this project. Each task is verified manually:
1. Open `index.html` in Chrome
2. Open DevTools console (F12) — watch for errors
3. Execute the gameplay scenario for the feature being tested
4. Verify on mobile viewport (Ctrl+Shift+M in Chrome) for touch features
