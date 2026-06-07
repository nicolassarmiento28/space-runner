# Space Runner — Plan de Desarrollo

> **Para workers agentic:** Usar superpowers:subagent-driven-development o superpowers:executing-plans para implementar este plan tarea por tarea. Pasos usan sintaxis `- [ ]` para tracking.

**Objetivo:** Evolucionar Space Runner de un prototipo funcional a un juego pulido, corrigiendo bugs conocidos y añadiendo features que mejoren la experiencia de juego.

**Arquitectura:** Single-file HTML5 Canvas + Vanilla JS + Web Audio API. Sin build system ni dependencias externas. Todo el código vive en `index.html`.

**Tech Stack:** HTML5 Canvas, JavaScript ES6+, Web Audio API, localStorage

---

## Fase 1: Corrección de Bugs

### Tarea 1.1: Eliminar `updateLanes()` duplicada

**Archivos:**
- Modificar: `index.html:264-268` (primera definición), `index.html:335-339` (segunda definición)

- [ ] **Eliminar la segunda definición de `updateLanes()`**

La función está definida dos veces (línea 264 y línea 335). La segunda definición sobrescribe a la primera. Encontrar el bloque:

```javascript
function updateLanes() {
    for (let i = 0; i < 3; i++) {
        game.player.lanePositions[i] = LANE_START_X() + i * LANE_WIDTH() - game.player.width / 2;
    }
}
```

en la línea 335 (segunda ocurrencia) y eliminarlo. La primera definición (línea 264) es idéntica y debe conservarse.

- [ ] **Verificar que no haya error**

Abrir `index.html` en el navegador, verificar que el menú carga sin errores en la consola y que el player aparece centrado.

### Tarea 1.2: Inicializar `musicGain` correctamente

**Archivos:**
- Modificar: `index.html:64`

- [ ] **Asignar `musicGain` a un GainNode**

Reemplazar línea 64:
```javascript
let musicGain = null;
```
por:
```javascript
let musicGain = null;
```

Y en la función `startMusic()`, después de chequear `audioCtx`, inicializar `musicGain`:

Encontrar (después de la línea 86):
```javascript
function startMusicInternal() {
    startMusic();
}
```

Modificar `startMusic()` para inicializar `musicGain`:
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
}
```

- [ ] **Verificar audio procedural**

Iniciar juego, disparar, esperar que la música procedural suene (kick/snare/hihat/bass). Verificar que no haya errores de `musicGain is null` en la consola.

### Tarea 1.3: Implementar powerup de escudo (shield)

**Archivos:**
- Modificar: `index.html:560-588` (bloque de colisión de powerups)

- [ ] **Agregar lógica para 'shield' en el collision handler**

En el bloque `if (checkCollision(game.player, p))` dentro del loop de powerups (línea 565), agregar el caso faltante:

```javascript
if (checkCollision(game.player, p)) {
    if (p.type === 'health' && game.lives < 5) {
        game.lives++;
    } else if (p.type === 'multishot') {
        game.player.multiShot = true;
        game.player.multiShotTimer = 600;
    } else if (p.type === 'shield') {
        // Shield: invulnerabilidad temporal (~3 segundos)
        game.player.shieldActive = true;
        game.player.shieldTimer = 180; // ~3s a 60fps
    }
    // ...
```

Además, agregar la propiedad `shieldActive: false` y `shieldTimer: 0` al objeto `game.player` en la inicialización (línea 247):

```javascript
player: {
    lane: 1, targetLane: 1, x: 0, y: 0, canMove: true,
    moveCooldown: 0, shootCooldown: 0, width: 50, height: 45,
    lanePositions: [0, 0, 0], multiShot: false, multiShotTimer: 0,
    shieldActive: false, shieldTimer: 0
},
```

Y en `initGame()` (línea 270), agregar:
```javascript
game.player.shieldActive = false;
game.player.shieldTimer = 0;
```

- [ ] **Agregar update del timer de shield en el game loop**

Dentro de `update()`, después del bloque de multishot timer (línea 597-600):
```javascript
// Shield timer
if (game.player.shieldActive) {
    game.player.shieldTimer--;
    if (game.player.shieldTimer <= 0) game.player.shieldActive = false;
}
```

- [ ] **Agregar invulnerabilidad cuando shield está activo**

En la colisión player-enemigo (línea 519), agregar condición:
```javascript
if (!game.player.shieldActive && checkCollision(game.player, e)) {
```

Y en la colisión enemy-bullet-player (línea 542):
```javascript
if (b.isEnemy && !game.player.shieldActive && checkCollision(game.player, b)) {
```

- [ ] **Dibujar indicador visual de shield en el player**

En `drawPlayer()`, al final antes de `ctx.shadowBlur = 0;`:
```javascript
if (game.player.shieldActive) {
    ctx.shadowColor = COLORS.cyan;
    ctx.shadowBlur = 20;
    ctx.strokeStyle = COLORS.cyan;
    ctx.lineWidth = 3;
    ctx.beginPath();
    ctx.arc(px + 25, py + 22, 30, 0, Math.PI * 2);
    ctx.stroke();
    ctx.shadowBlur = 0;
}
```

### Tarea 1.4: Corregir colisión de enemy bullets

**Archivos:**
- Modificar: `index.html:609-613` (`checkCollision`)

- [ ] **Ajustar padding basado en tamaño del objeto pequeño**

Modificar `checkCollision` para usar padding dinámico:
```javascript
function checkCollision(a, b) {
    const p = Math.min(8, Math.min(a.width, b.width) / 2);
    return a.x + p < b.x + b.width - p &&
           a.x + a.width - p > b.x + p &&
           a.y + p < b.y + b.height - p &&
           a.y + a.height - p > b.y + p;
}
```

### Tarea 1.5: Implementar swipe vertical en móvil

**Archivos:**
- Modificar: `index.html:978-1002` (touchend handler)

- [ ] **Agregar detección de swipe vertical**

Modificar la lógica en `touchend` para soportar swipes verticales:
```javascript
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
```

---

## Fase 2: Mejoras de Gameplay

### Tarea 2.1: Implementar deltaTime para movimiento consistente

**Archivos:**
- Modificar: `index.html` (game loop, update, movement)

- [ ] **Agregar deltaTime al game loop**

```javascript
let lastTime = 0;

function gameLoop(timestamp) {
    try {
        const dt = Math.min((timestamp - lastTime) / 16.67, 3); // capped at 3x
        lastTime = timestamp;
        update(dt);
        draw();
    } catch(e) {
        console.error('Game loop error:', e);
    }
    requestAnimationFrame(gameLoop);
}
```

- [ ] **Modificar `update()` para recibir dt y escalar movimientos**

```javascript
function update(dt) {
    // ...
    // Player movement interpolation escalado por dt
    if (game.player.lane !== game.player.targetLane) {
        // ...
        game.player.x += (target - game.player.x) * 0.25 * dt;
        // ...
    }
    // Enemies
    for (let i = game.enemies.length - 1; i >= 0; i--) {
        const e = game.enemies[i];
        e.y += game.speed * dt;
        // ...
    }
    // Speed increment basado en dt
    game.speedAccumulator += dt;
    if (game.speedAccumulator >= 1) {
        game.speedAccumulator -= 1;
        if (game.speed < game.maxSpeed) game.speed += 0.05;
    }
}
```

### Tarea 2.2: Mejorar feedback visual (screen shake, hit flash)

**Archivos:**
- Modificar: `index.html` (draw, update)

- [ ] **Agregar screen shake en colisiones**

```javascript
// En update(), cuando el player recibe daño:
game.screenShake = 10; // frames de shake
// game.screenShakeDuration = 10; y game.screenShakeIntensity = 5;

// En draw(), aplicar translación:
if (game.screenShake > 0) {
    const shakeX = (Math.random() - 0.5) * game.screenShake * 0.5;
    const shakeY = (Math.random() - 0.5) * game.screenShake * 0.5;
    ctx.save();
    ctx.translate(shakeX, shakeY);
    game.screenShake--;
}
// Al final de draw(), restaurar:
if (game.screenShake > 0) {
    ctx.restore();
}
```

- [ ] **Hit flash en el player al recibir daño**

```javascript
// En update():
if (game.player.hitFlash > 0) game.player.hitFlash--;

// En drawPlayer():
const alpha = game.player.hitFlash > 0 && game.player.hitFlash % 4 < 2 ? 0.3 : 1.0;
ctx.globalAlpha = alpha;
// ... resto del dibujo ...
ctx.globalAlpha = 1.0;

// En initGame():
game.player.hitFlash = 0;

// En colisión:
game.player.hitFlash = 20;
```

### Tarea 2.3: Líneas de carril (road)

**Archivos:**
- Modificar: `index.html:674-676` (`drawRoad()`)

- [ ] **Implementar `drawRoad()` con líneas de carril animadas**

```javascript
function drawRoad() {
    const laneW = LANE_WIDTH();
    const startX = LANE_START_X();
    ctx.strokeStyle = 'rgba(255, 255, 255, 0.15)';
    ctx.lineWidth = 2;
    ctx.setLineDash([20, 15]);
    ctx.lineDashOffset = -game.frameCount * 2;
    
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

---

## Fase 3: Nuevas Features

### Tarea 3.1: Sistema de vidas visual (corazones animados)

**Archivos:**
- Modificar: `index.html:769-797` (`drawHUD`)

- [ ] **Dibujar corazones como sprites canvas (no texto Unicode)**

```javascript
function drawHearts(count) {
    for (let i = 0; i < count; i++) {
        const hx = canvas.width - 25 - i * 22;
        const hy = 20;
        const s = 8;
        ctx.fillStyle = COLORS.red;
        ctx.shadowColor = COLORS.red;
        ctx.shadowBlur = 5;
        ctx.beginPath();
        ctx.moveTo(hx, hy + s/4);
        ctx.bezierCurveTo(hx - s/2, hy - s/2, hx - s, hy + s/2, hx, hy + s);
        ctx.bezierCurveTo(hx + s, hy + s/2, hx + s/2, hy - s/2, hx, hy + s/4);
        ctx.fill();
        ctx.shadowBlur = 0;
    }
}
```

### Tarea 3.2: Añadir más variedad de enemigos

**Archivos:**
- Modificar: `index.html:341-383` (`spawnEntity`)

- [ ] **Agregar enemy tipo 'tank' (más HP, más lento)**

```javascript
// En spawnEntity, después de shooter:
} else if (rand < 0.50) {
    game.enemies.push({
        lane: lane,
        x: game.player.lanePositions[lane],
        y: -60,
        width: 55,
        height: 50,
        type: 'tank',
        hp: 3,
        shootTimer: Math.random() * 120
    });
```

- [ ] **Agregar lógica para tank en update()**

```javascript
// En el loop de colisión bullet-enemigo:
if (e.type === 'tank') {
    e.hp--;
    if (e.hp <= 0) {
        game.enemies.splice(j, 1);
        game.score += 50;
        spawnParticle(e.x + e.width/2, e.y + e.height/2, COLORS.purple, 25);
    }
    game.bullets.splice(i, 1);
    break;
}
```

- [ ] **Dibujar tank en drawEntities()**

```javascript
// En drawEntities:
if (e.type === 'tank') {
    ctx.fillStyle = '#553355';
    ctx.fillRect(e.x, e.y, e.width, e.height);
    ctx.fillStyle = COLORS.purple;
    ctx.fillRect(e.x + 3, e.y + 3, e.width - 6, 8);
    // HP indicator
    ctx.fillStyle = COLORS.green;
    ctx.fillRect(e.x + 5, e.y + e.height - 5, (e.width - 10) * (e.hp / 3), 3);
}
```

### Tarea 3.3: Pantalla de pausa

**Archivos:**
- Modificar: `index.html` (game state, keydown handler, draw)

- [ ] **Agregar estado 'paused'**

```javascript
// En keydown, toggle con Escape o P:
if ((e.code === 'Escape' || e.code === 'KeyP') && game.state === 'playing') {
    game.state = 'paused';
    stopMusic();
    return;
} else if ((e.code === 'Escape' || e.code === 'KeyP') && game.state === 'paused') {
    game.state = 'playing';
    setTimeout(() => startMusic(), 100);
    return;
}
```

- [ ] **Dibujar overlay de pausa**

```javascript
function drawPause() {
    ctx.fillStyle = 'rgba(0, 0, 20, 0.8)';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    ctx.font = '22px "Press Start 2P"';
    ctx.fillStyle = COLORS.white;
    ctx.textAlign = 'center';
    ctx.fillText('PAUSA', canvas.width / 2, canvas.height / 2);
    ctx.font = '10px "Press Start 2P"';
    ctx.fillText('ESC/P PARA CONTINUAR', canvas.width / 2, canvas.height / 2 + 60);
}
```

Y en `draw()`, después de `drawHUD()`:
```javascript
if (game.state === 'paused') drawPause();
```

---

## Fase 4: Pulido y Optimización

### Tarea 4.1: Renombrar LANE_WIDTH / LANE_START_X

**Archivos:**
- Modificar: `index.html:261-262`

- [ ] **Renombrar a lowerCamelCase**

```javascript
const getLaneWidth = () => canvas.width / 3;
const getLaneStartX = () => canvas.width / 6;
```

Actualizar todas las referencias en el código.

### Tarea 4.2: Pooling de partículas

**Archivos:**
- Modificar: `index.html` (Particle class, spawn, update)

- [ ] **Implementar object pooling para partículas**

```javascript
// Límite máximo
const MAX_PARTICLES = 100;

function spawnParticle(x, y, color, count) {
    for (let i = 0; i < count; i++) {
        if (game.particles.length >= MAX_PARTICLES) break;
        game.particles.push(new Particle(x, y, color));
    }
}
```

### Tarea 4.3: Mejorar la música procedural

**Archivos:**
- Modificar: `index.html:90-164` (scheduling y sonidos)

- [ ] **Variar BPM con la velocidad del juego**

```javascript
function scheduleMusic() {
    if (!audioCtx || !musicPlaying || game.state !== 'playing') return;
    
    const bpm = 120 + game.speed * 2; // Acelera con la velocidad
    const secondsPerBeat = 60 / bpm;
    // ...
}
```

---

## Resumen de Prioridades

| Prioridad | Tarea | Esfuerzo | Impacto |
|-----------|-------|----------|---------|
| 🔴 Crítica | 1.1 - updateLanes duplicada | 1 min | Media |
| 🔴 Crítica | 1.2 - musicGain no inicializado | 5 min | Alta |
| 🔴 Crítica | 1.3 - Shield no funciona | 10 min | Alta |
| 🔴 Crítica | 1.4 - Colisión enemy bullets | 5 min | Media |
| 🟡 Media | 1.5 - Swipe vertical móvil | 5 min | Alta |
| 🟡 Media | 2.1 - DeltaTime | 15 min | Alta |
| 🟡 Media | 2.2 - Screen shake / hit flash | 10 min | Media |
| 🟢 Baja | 2.3 - Líneas de carril | 5 min | Baja |
| 🟢 Baja | 3.1 - Corazones animados | 10 min | Media |
| 🟢 Baja | 3.2 - Enemigos tank | 15 min | Media |
| 🟢 Baja | 3.3 - Pantalla de pausa | 10 min | Media |
| 🟢 Baja | 4.1 - Renombrar constantes | 10 min | Baja |
| 🟢 Baja | 4.2 - Pooling partículas | 5 min | Baja |
| 🟢 Baja | 4.3 - Música variable | 5 min | Baja |

---

## Testing

Cada tarea debe verificarse manualmente:
1. Abrir `index.html` en navegador
2. Verificar consola (F12) sin errores
3. Jugar partida completa probando la feature modificada
4. Verificar en móvil (device toolbar) cuando aplique
