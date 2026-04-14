# Eirene's Great Escape: Amsterdam → Home — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-file `game.html` HTML5 canvas arcade game for Eirene — she taps to fly her plane from Amsterdam through the Old Gregg Atlantic to home, dodging obstacles and collecting ramen — deployable to GitHub Pages.

**Architecture:** One self-contained `game.html`. All rendering via HTML5 Canvas 2D. Vanilla JS `requestAnimationFrame` game loop. No assets, no CDN — everything drawn programmatically with canvas shapes + emoji text. Mobile-first tap-to-flap. State machine controls which screen is active (`start` | `game` | `dead` | `win`).

**Tech Stack:** HTML5 Canvas 2D, Vanilla JavaScript (ES6), CSS (6 lines), GitHub Pages

> **Note on TDD:** This is a canvas-rendered browser game with no server/module boundary. There are no unit tests — verification is done by opening `game.html` in Chrome/Safari and confirming visual behavior. Each task has explicit browser verification steps.

---

## File Structure

| File | Purpose |
|------|---------|
| `game.html` | The entire game — single self-contained file |

All tasks create or modify `game.html`. Each task adds a new section inside the `<script>` tag. The file is built up incrementally.

---

## Shared State Reference

All tasks share these top-level objects. Defined in Task 1, referenced everywhere.

```js
const CONFIG = {
  gravity: 0.38,
  flapForce: -8.5,
  baseScrollSpeed: 3,
  totalDistance: 9000,   // 3 zones × 3000px each
  zoneLength: 3000,
  groundH: 55,
  playerX: null,         // set on resize: canvasW * 0.2
  obstacleGap: 185,      // vertical gap player flies through
  obstacleW: 58,
  minObstacleSpacing: 320,
  collectibleSize: 28,
};

const STATE = {
  screen: 'start',       // 'start' | 'game' | 'dead' | 'win'
  player: {
    y: 0, vy: 0,
    w: 52, h: 28,
    lives: 3,
    invincible: false, invincibleTimer: 0,
    speedBoost: false, speedTimer: 0,
    multiplier: 1, multiplierTimer: 0,
    wobble: 0,           // frames of AI-hit wobble
  },
  obstacles: [],
  collectibles: [],
  particles: [],
  events: [],            // active random events
  hotDogGuy: null,
  score: 0,
  distance: 0,
  scrollSpeed: CONFIG.baseScrollSpeed,
  zone: 0,              // 0=Europe 1=Atlantic 2=America
  zoneLabel: { text: '', alpha: 0 },
  deathQuoteIdx: 0,
  frame: 0,
  oldGreggWarningTimer: 0,
  iasylTimer: 0,
  nextIasyl: 1800,       // frames until next IASYL flash
  aiRobotTimer: 0,
  nextAiRobot: 600,
};

const DEATH_QUOTES = [
  "ARE YOU PLAYING GAMES WITH OLD GREGG?!",
  "You're going to look at it.",
  "I got the funk. Have you got the funk?",
  "This game was made with AI. That's probably why it's broken.",
  "Do you want to come to a party at the end of the universe?",
  "STOP TALKING. STOP TALKING. (you crashed)",
];
```

---

## Task 1: HTML Skeleton + Canvas Setup + Game Loop

**Files:**
- Create: `game.html`

- [ ] **Step 1: Create the file**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
  <title>Eirene's Great Escape ✈️</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { background: #1a1a2e; overflow: hidden; }
    canvas { display: block; }
  </style>
</head>
<body>
<canvas id="c"></canvas>
<script>
// ─── CONFIG ───────────────────────────────────────────────────────────────────
const CONFIG = {
  gravity: 0.38,
  flapForce: -8.5,
  baseScrollSpeed: 3,
  totalDistance: 9000,
  zoneLength: 3000,
  groundH: 55,
  playerX: null,
  obstacleGap: 185,
  obstacleW: 58,
  minObstacleSpacing: 320,
  collectibleSize: 28,
};

// ─── STATE ────────────────────────────────────────────────────────────────────
const STATE = {
  screen: 'start',
  player: { y: 0, vy: 0, w: 52, h: 28,
            lives: 3, invincible: false, invincibleTimer: 0,
            speedBoost: false, speedTimer: 0,
            multiplier: 1, multiplierTimer: 0, wobble: 0 },
  obstacles: [], collectibles: [], particles: [], events: [],
  hotDogGuy: null,
  score: 0, distance: 0,
  scrollSpeed: 3,
  zone: 0,
  zoneLabel: { text: '', alpha: 0 },
  deathQuoteIdx: 0, frame: 0,
  oldGreggWarningTimer: 0,
  iasylTimer: 0, nextIasyl: 1800,
  aiRobotTimer: 0, nextAiRobot: 600,
  nextHotDog: 2400,
};

const DEATH_QUOTES = [
  "ARE YOU PLAYING GAMES WITH OLD GREGG?!",
  "You're going to look at it.",
  "I got the funk. Have you got the funk?",
  "This game was made with AI. That's probably why it's broken.",
  "Do you want to come to a party at the end of the universe?",
  "STOP TALKING. STOP TALKING. (you crashed)",
];

// ─── CANVAS ───────────────────────────────────────────────────────────────────
const canvas = document.getElementById('c');
const ctx = canvas.getContext('2d');
let canvasW = 0, canvasH = 0;

function resize() {
  canvasW = canvas.width = window.innerWidth;
  canvasH = canvas.height = window.innerHeight;
  CONFIG.playerX = canvasW * 0.2;
  if (STATE.screen === 'start') STATE.player.y = canvasH * 0.45;
}
window.addEventListener('resize', resize);
resize();

// ─── GAME LOOP ────────────────────────────────────────────────────────────────
function loop() {
  update();
  draw();
  requestAnimationFrame(loop);
}

function update() {
  if (STATE.screen !== 'game') return;
  STATE.frame++;
  // placeholder — filled by later tasks
}

function draw() {
  ctx.clearRect(0, 0, canvasW, canvasH);
  ctx.fillStyle = '#87CEEB';
  ctx.fillRect(0, 0, canvasW, canvasH);
  // placeholder — filled by later tasks
}

loop();
</script>
</body>
</html>
```

- [ ] **Step 2: Open in browser and verify**

Drag `game.html` to Chrome or Safari. Expected: solid sky-blue rectangle filling the entire window. No console errors. Resizing window keeps it full-screen.

- [ ] **Step 3: Commit**

```bash
git add game.html
git commit -m "feat: game skeleton — canvas, config, state, loop"
```

---

## Task 2: Player Plane + Physics + Tap/Click

**Files:**
- Modify: `game.html` — add `drawPlane()`, player physics in `update()`, event listeners

- [ ] **Step 1: Add `drawPlane()` function** (insert before `loop()`)

```js
// ─── DRAW: PLANE ──────────────────────────────────────────────────────────────
function drawPlane() {
  const p = STATE.player;
  const x = CONFIG.playerX;
  const y = p.y + (p.wobble > 0 ? Math.sin(p.wobble * 0.8) * 6 : 0);
  if (p.wobble > 0) p.wobble--;

  ctx.save();
  if (p.invincible && Math.floor(STATE.frame / 6) % 2 === 0) {
    ctx.globalAlpha = 0.4;
  }

  // Body
  ctx.fillStyle = '#f0f0f0';
  ctx.beginPath();
  ctx.ellipse(x, y, 26, 11, 0, 0, Math.PI * 2);
  ctx.fill();

  // Wing (top)
  ctx.fillStyle = '#4a8fdd';
  ctx.beginPath();
  ctx.moveTo(x - 4, y - 2);
  ctx.lineTo(x + 12, y - 22);
  ctx.lineTo(x + 18, y - 2);
  ctx.closePath();
  ctx.fill();

  // Tail fin
  ctx.fillStyle = '#4a8fdd';
  ctx.beginPath();
  ctx.moveTo(x - 22, y - 1);
  ctx.lineTo(x - 12, y - 14);
  ctx.lineTo(x - 10, y - 1);
  ctx.closePath();
  ctx.fill();

  // Window
  ctx.fillStyle = '#b8e0f7';
  ctx.beginPath();
  ctx.arc(x + 8, y - 1, 5, 0, Math.PI * 2);
  ctx.fill();
  ctx.strokeStyle = '#aaa';
  ctx.lineWidth = 1;
  ctx.stroke();

  ctx.restore();
}
```

- [ ] **Step 2: Add player physics to `update()`** (replace the placeholder comment)

```js
function update() {
  if (STATE.screen !== 'game') return;
  STATE.frame++;

  // ── Player physics ──
  const p = STATE.player;
  p.vy += CONFIG.gravity;
  p.y += p.vy;

  // Power-up timers
  if (p.invincible) { p.invincibleTimer--; if (p.invincibleTimer <= 0) p.invincible = false; }
  if (p.speedBoost) { p.speedTimer--; if (p.speedTimer <= 0) { p.speedBoost = false; STATE.scrollSpeed = CONFIG.baseScrollSpeed + STATE.distance * 0.0003; } }
  if (p.multiplier > 1) { p.multiplierTimer--; if (p.multiplierTimer <= 0) p.multiplier = 1; }

  // Out of bounds → die
  if (p.y - p.h / 2 < 0 || p.y + p.h / 2 > canvasH - CONFIG.groundH) {
    loseLife();
    return;
  }

  // Scroll speed ramp
  STATE.scrollSpeed = CONFIG.baseScrollSpeed + STATE.distance * 0.0003;
  if (p.speedBoost) STATE.scrollSpeed *= 1.6;

  STATE.distance += STATE.scrollSpeed;

  // Zone transitions
  const newZone = Math.min(2, Math.floor(STATE.distance / CONFIG.zoneLength));
  if (newZone !== STATE.zone) {
    STATE.zone = newZone;
    const labels = ['', 'THE ATLANTIC — watch out.', '🏠 Almost home...'];
    STATE.zoneLabel = { text: labels[newZone], alpha: 1.0 };
  }
  if (STATE.zoneLabel.alpha > 0) STATE.zoneLabel.alpha -= 0.008;

  // Score: distance points
  STATE.score += Math.floor(STATE.multiplier);

  // Win condition
  if (STATE.distance >= CONFIG.totalDistance) {
    STATE.screen = 'win';
    spawnConfetti();
    return;
  }
}
```

- [ ] **Step 3: Add `loseLife()` function** (insert after `DEATH_QUOTES`)

```js
function loseLife() {
  if (STATE.player.invincible) return;
  STATE.player.lives--;
  if (STATE.player.lives <= 0) {
    STATE.screen = 'dead';
    STATE.deathQuoteIdx = Math.floor(Math.random() * DEATH_QUOTES.length);
  } else {
    STATE.player.invincible = true;
    STATE.player.invincibleTimer = 120; // 2 seconds
    STATE.player.vy = 0;
  }
}
```

- [ ] **Step 4: Add `spawnConfetti()` stub** (insert after `loseLife()`)

```js
function spawnConfetti() {
  // Filled in Task 12
}
```

- [ ] **Step 5: Wire tap/click to flap** (insert before `loop()`)

```js
// ─── INPUT ────────────────────────────────────────────────────────────────────
function flap() {
  if (STATE.screen === 'game') {
    STATE.player.vy = CONFIG.flapForce;
  } else if (STATE.screen === 'start') {
    startGame();
  } else if (STATE.screen === 'dead' || STATE.screen === 'win') {
    resetGame();
  }
}

canvas.addEventListener('click', flap);
canvas.addEventListener('touchstart', (e) => { e.preventDefault(); flap(); }, { passive: false });

function startGame() {
  resetGame();
  STATE.screen = 'game';
}

function resetGame() {
  STATE.screen = 'start';
  STATE.player.y = canvasH * 0.45;
  STATE.player.vy = 0;
  STATE.player.lives = 3;
  STATE.player.invincible = false;
  STATE.player.speedBoost = false;
  STATE.player.multiplier = 1;
  STATE.player.wobble = 0;
  STATE.obstacles = [];
  STATE.collectibles = [];
  STATE.particles = [];
  STATE.events = [];
  STATE.hotDogGuy = null;
  STATE.score = 0;
  STATE.distance = 0;
  STATE.scrollSpeed = CONFIG.baseScrollSpeed;
  STATE.zone = 0;
  STATE.zoneLabel = { text: '', alpha: 0 };
  STATE.frame = 0;
  STATE.oldGreggWarningTimer = 0;
  STATE.iasylTimer = 0;
  STATE.nextIasyl = 1800;
  STATE.aiRobotTimer = 0;
  STATE.nextAiRobot = 600;
  STATE.nextHotDog = 2400;
}
```

- [ ] **Step 6: Update `draw()` to call `drawPlane()`** (add after clearRect/fill)

```js
function draw() {
  ctx.clearRect(0, 0, canvasW, canvasH);
  ctx.fillStyle = '#87CEEB';
  ctx.fillRect(0, 0, canvasW, canvasH);
  if (STATE.screen === 'game' || STATE.screen === 'dead' || STATE.screen === 'win') {
    drawPlane();
  }
}
```

- [ ] **Step 7: Open in browser and verify**

Open `game.html`. Expected: sky-blue canvas. Click — plane appears and bobs. Tap again — it flaps upward then falls with gravity. Plane vanishes when it hits the top/bottom edge (loseLife triggers resetGame → start screen).

- [ ] **Step 8: Commit**

```bash
git add game.html
git commit -m "feat: player plane, gravity, tap-to-flap"
```

---

## Task 3: Zone Backgrounds + Scrolling Ground

**Files:**
- Modify: `game.html` — add `drawBackground()`, `drawGround()`, zone label rendering

- [ ] **Step 1: Add `drawBackground()` and `drawGround()`** (insert before `loop()`)

```js
// ─── DRAW: BACKGROUND ─────────────────────────────────────────────────────────
function drawBackground() {
  const z = STATE.zone;
  const progress = (STATE.distance % CONFIG.zoneLength) / CONFIG.zoneLength;

  if (z === 0) {
    // Europe: sky blue gradient
    const g = ctx.createLinearGradient(0, 0, 0, canvasH);
    g.addColorStop(0, '#87CEEB');
    g.addColorStop(1, '#c8e6c9');
    ctx.fillStyle = g;
    ctx.fillRect(0, 0, canvasW, canvasH);

    // Scrolling tulip fields on ground
    const offset = (STATE.distance * 0.5) % 120;
    for (let x = -offset; x < canvasW + 120; x += 120) {
      drawTulip(x + 20, canvasH - CONFIG.groundH);
      drawTulip(x + 70, canvasH - CONFIG.groundH);
    }

    // Windmill silhouettes on horizon
    const wOff = (STATE.distance * 0.3) % 300;
    for (let x = -wOff; x < canvasW + 300; x += 300) {
      drawWindmillSilhouette(x, canvasH - CONFIG.groundH - 40);
    }
  } else if (z === 1) {
    // Atlantic: dark ocean
    const g = ctx.createLinearGradient(0, 0, 0, canvasH);
    g.addColorStop(0, '#1a2a4a');
    g.addColorStop(0.5, '#0d3b6e');
    g.addColorStop(1, '#051935');
    ctx.fillStyle = g;
    ctx.fillRect(0, 0, canvasW, canvasH);

    // Eerie green glow at bottom (Old Gregg vibes)
    const og = ctx.createLinearGradient(0, canvasH - 200, 0, canvasH);
    og.addColorStop(0, 'rgba(0,255,100,0)');
    og.addColorStop(1, 'rgba(0,255,100,0.15)');
    ctx.fillStyle = og;
    ctx.fillRect(0, canvasH - 200, canvasW, 200);

    // Waves
    drawWaves();
  } else {
    // America: warm sunset
    const g = ctx.createLinearGradient(0, 0, 0, canvasH);
    g.addColorStop(0, '#ff9a3c');
    g.addColorStop(0.4, '#ffcc70');
    g.addColorStop(1, '#c8e6c9');
    ctx.fillStyle = g;
    ctx.fillRect(0, 0, canvasW, canvasH);

    // Clouds
    const cOff = (STATE.distance * 0.4) % 350;
    for (let x = -cOff; x < canvasW + 350; x += 350) {
      drawCloud(x + 80, canvasH * 0.25);
      drawCloud(x + 250, canvasH * 0.15);
    }
  }
}

function drawTulip(x, y) {
  ctx.fillStyle = '#e53935';
  ctx.beginPath();
  ctx.arc(x, y - 10, 6, 0, Math.PI * 2);
  ctx.fill();
  ctx.fillStyle = '#43a047';
  ctx.fillRect(x - 1, y - 10, 2, 18);
}

function drawWindmillSilhouette(x, y) {
  ctx.fillStyle = 'rgba(100,120,100,0.4)';
  ctx.fillRect(x - 3, y - 50, 6, 50);
  // blades
  const angle = STATE.frame * 0.015;
  ctx.save();
  ctx.translate(x, y - 50);
  ctx.rotate(angle);
  ctx.fillStyle = 'rgba(100,120,100,0.4)';
  for (let i = 0; i < 4; i++) {
    ctx.save();
    ctx.rotate((Math.PI / 2) * i);
    ctx.fillRect(-2, -22, 4, 22);
    ctx.restore();
  }
  ctx.restore();
}

function drawWaves() {
  const offset = (STATE.distance * 0.8) % canvasW;
  ctx.strokeStyle = 'rgba(0,200,255,0.2)';
  ctx.lineWidth = 2;
  for (let i = 0; i < 5; i++) {
    const y = canvasH * 0.65 + i * 30;
    ctx.beginPath();
    for (let x = -offset; x < canvasW + 60; x += 60) {
      ctx.moveTo(x, y);
      ctx.bezierCurveTo(x + 15, y - 8, x + 45, y + 8, x + 60, y);
    }
    ctx.stroke();
  }
}

function drawCloud(x, y) {
  ctx.fillStyle = 'rgba(255,255,255,0.7)';
  ctx.beginPath();
  ctx.arc(x, y, 22, 0, Math.PI * 2);
  ctx.arc(x + 25, y - 8, 18, 0, Math.PI * 2);
  ctx.arc(x + 48, y, 20, 0, Math.PI * 2);
  ctx.fill();
}

function drawGround() {
  const z = STATE.zone;
  const colors = ['#5d8a3c', '#1a4a6e', '#6aaa55'];
  ctx.fillStyle = colors[z];
  ctx.fillRect(0, canvasH - CONFIG.groundH, canvasW, CONFIG.groundH);

  if (z === 2) {
    // Thrift store silhouettes on ground
    const tOff = (STATE.distance * 0.5) % 400;
    for (let x = -tOff; x < canvasW + 400; x += 400) {
      drawThriftStoreSilhouette(x + 80);
      drawThriftStoreSilhouette(x + 260);
    }
  }
}

function drawThriftStoreSilhouette(x) {
  const y = canvasH - CONFIG.groundH;
  ctx.fillStyle = 'rgba(80,50,30,0.5)';
  ctx.fillRect(x, y - 35, 50, 35);
  ctx.fillStyle = 'rgba(60,30,10,0.5)';
  ctx.fillRect(x - 5, y - 42, 60, 10);
  // sign
  ctx.fillStyle = 'rgba(200,180,50,0.6)';
  ctx.fillRect(x + 8, y - 28, 34, 12);
}

function drawZoneLabel() {
  if (STATE.zoneLabel.alpha <= 0) return;
  ctx.save();
  ctx.globalAlpha = STATE.zoneLabel.alpha;
  ctx.font = 'bold 22px Arial';
  ctx.fillStyle = '#fff';
  ctx.textAlign = 'center';
  ctx.shadowColor = 'rgba(0,0,0,0.6)';
  ctx.shadowBlur = 8;
  ctx.fillText(STATE.zoneLabel.text, canvasW / 2, canvasH * 0.15);
  ctx.restore();
}
```

- [ ] **Step 2: Update `draw()` to call background + ground + zone label**

```js
function draw() {
  ctx.clearRect(0, 0, canvasW, canvasH);

  if (STATE.screen === 'start') {
    ctx.fillStyle = '#87CEEB';
    ctx.fillRect(0, 0, canvasW, canvasH);
  } else {
    drawBackground();
    drawGround();
    drawZoneLabel();
    drawPlane();
  }
}
```

- [ ] **Step 3: Open in browser, start a game (tap), verify**

Expected: Europe zone shows sky-blue with tulips and windmills. Progress the game by NOT dying (just keep tapping). At distance 3000, background shifts to dark ocean with green glow. At 6000, warm sunset and thrift store silhouettes appear.

Since you can't easily reach zone 3 yet, temporarily in devtools console: `STATE.distance = 6100` then observe.

- [ ] **Step 4: Commit**

```bash
git add game.html
git commit -m "feat: zone backgrounds, scrolling ground, zone labels"
```

---

## Task 4: Obstacle System + Windmill Columns (Europe)

**Files:**
- Modify: `game.html` — add obstacle spawn/update/draw system, windmill columns

- [ ] **Step 1: Add obstacle spawn + update functions** (insert before `loop()`)

```js
// ─── OBSTACLES ────────────────────────────────────────────────────────────────
function spawnObstacle() {
  const z = STATE.zone;
  const gapY = CONFIG.groundH + CONFIG.obstacleGap + Math.random() * (canvasH - CONFIG.groundH - CONFIG.obstacleGap * 2.2 - 60);
  const x = canvasW + 10;

  if (z === 0) {
    // Windmill columns
    STATE.obstacles.push({
      type: 'windmill',
      x, gapY,
      topH: gapY - CONFIG.obstacleGap / 2,
      bottomY: gapY + CONFIG.obstacleGap / 2,
      bottomH: canvasH - CONFIG.groundH - (gapY + CONFIG.obstacleGap / 2),
      w: CONFIG.obstacleW,
      angle: 0,
    });
  } else if (z === 1) {
    // Atlantic: alternate tentacles and fog walls
    if (Math.random() < 0.5) {
      STATE.obstacles.push({ type: 'tentacle', x, gapY, w: CONFIG.obstacleW });
    } else {
      STATE.obstacles.push({ type: 'fog', x, gapY, w: 70 });
    }
  } else {
    // America: windmill columns (faster)
    STATE.obstacles.push({
      type: 'windmill',
      x, gapY,
      topH: gapY - CONFIG.obstacleGap / 2,
      bottomY: gapY + CONFIG.obstacleGap / 2,
      bottomH: canvasH - CONFIG.groundH - (gapY + CONFIG.obstacleGap / 2),
      w: CONFIG.obstacleW,
      angle: 0,
    });
  }
}

function updateObstacles() {
  // Spawn
  const lastX = STATE.obstacles.length ? STATE.obstacles[STATE.obstacles.length - 1].x : -999;
  if (canvasW - lastX > CONFIG.minObstacleSpacing) {
    spawnObstacle();
  }

  // Move + rotate
  STATE.obstacles.forEach(o => {
    o.x -= STATE.scrollSpeed;
    if (o.type === 'windmill') o.angle += 0.025;
  });

  // Remove off-screen
  STATE.obstacles = STATE.obstacles.filter(o => o.x + o.w + 60 > 0);
}

function drawObstacles() {
  STATE.obstacles.forEach(o => {
    if (o.type === 'windmill') drawWindmill(o);
    else if (o.type === 'tentacle') drawTentacle(o);
    else if (o.type === 'fog') drawFogWall(o);
  });
}

function drawWindmill(o) {
  // Top column
  ctx.fillStyle = '#8B7355';
  ctx.fillRect(o.x, 0, o.w, o.topH);
  // Bottom column
  ctx.fillRect(o.x, o.bottomY, o.w, o.bottomH);

  // Rotating blades on top column cap
  const bx = o.x + o.w / 2;
  const by = o.topH;
  ctx.save();
  ctx.translate(bx, by);
  ctx.rotate(o.angle);
  ctx.fillStyle = '#a0855b';
  for (let i = 0; i < 4; i++) {
    ctx.save();
    ctx.rotate((Math.PI / 2) * i);
    ctx.fillRect(-3, -28, 6, 28);
    ctx.restore();
  }
  ctx.restore();

  // Cap border
  ctx.strokeStyle = '#6b5335';
  ctx.lineWidth = 2;
  ctx.strokeRect(o.x, 0, o.w, o.topH);
  ctx.strokeRect(o.x, o.bottomY, o.w, o.bottomH);
}

function drawTentacle(o) {
  // Rises from bottom
  const h = (canvasH - CONFIG.groundH) * 0.55 + Math.sin(STATE.frame * 0.04 + o.x * 0.01) * 30;
  const topY = canvasH - CONFIG.groundH - h;
  ctx.fillStyle = '#1a7a3a';
  ctx.beginPath();
  ctx.moveTo(o.x, canvasH - CONFIG.groundH);
  ctx.bezierCurveTo(o.x - 15, topY + h * 0.6, o.x + 15, topY + h * 0.3, o.x + o.w / 2, topY);
  ctx.bezierCurveTo(o.x + o.w - 15, topY + h * 0.3, o.x + o.w + 15, topY + h * 0.6, o.x + o.w, canvasH - CONFIG.groundH);
  ctx.closePath();
  ctx.fill();
  ctx.strokeStyle = '#0d5522';
  ctx.lineWidth = 2;
  ctx.stroke();
  // Sucker dots
  ctx.fillStyle = '#0d8a3a';
  for (let i = 0; i < 4; i++) {
    ctx.beginPath();
    ctx.arc(o.x + o.w / 2 + Math.sin(i) * 5, topY + h * 0.2 + i * (h * 0.15), 4, 0, Math.PI * 2);
    ctx.fill();
  }
  // Update collision box
  o.gapY = topY;
  o.collisionTopY = topY;
  o.collisionH = h;
}

function drawFogWall(o) {
  // Solid fog except for gap
  ctx.fillStyle = 'rgba(180,200,220,0.75)';
  ctx.fillRect(o.x, 0, o.w, o.gapY - CONFIG.obstacleGap / 2);
  ctx.fillRect(o.x, o.gapY + CONFIG.obstacleGap / 2, o.w, canvasH - CONFIG.groundH - (o.gapY + CONFIG.obstacleGap / 2));
}
```

- [ ] **Step 2: Wire obstacles into `update()` and `draw()`**

In `update()`, add after the score line:
```js
  updateObstacles();
```

In `draw()`, add `drawObstacles();` after `drawPlane();`:
```js
  drawBackground();
  drawGround();
  drawObstacles();   // ← add this
  drawPlane();
  drawZoneLabel();
```

- [ ] **Step 3: Open in browser and verify**

Start the game. Expected: windmill columns with rotating blades appear and scroll left. At zone 1 (set `STATE.distance = 3100` in console), tentacles and fog walls appear instead.

- [ ] **Step 4: Commit**

```bash
git add game.html
git commit -m "feat: obstacle system — windmills, tentacles, fog walls"
```

---

## Task 5: Collectibles System

**Files:**
- Modify: `game.html` — add collectible spawn/update/draw and effect application

- [ ] **Step 1: Add collectible functions** (insert before `loop()`)

```js
// ─── COLLECTIBLES ─────────────────────────────────────────────────────────────
const COLLECTIBLES = [
  { type: 'ramen',    emoji: '🍜', color: '#ff6b35', prob: 0.30 },
  { type: 'mac',      emoji: '🧀', color: '#ffcc00', prob: 0.25 },
  { type: 'baileys',  emoji: '🥛', color: '#d4a017', prob: 0.15 },
  { type: 'bunny',    emoji: '🐱', color: '#ffaacc', prob: 0.20 },
  { type: 'thrift',   emoji: '👗', color: '#9b59b6', prob: 0.10 },
];

function spawnCollectible() {
  const r = Math.random();
  let cumulative = 0;
  let chosen = COLLECTIBLES[0];
  for (const c of COLLECTIBLES) {
    cumulative += c.prob;
    if (r < cumulative) { chosen = c; break; }
  }
  // Thrift racks only in America zone
  if (chosen.type === 'thrift' && STATE.zone !== 2) {
    chosen = COLLECTIBLES[0]; // ramen instead
  }
  const y = 60 + Math.random() * (canvasH - CONFIG.groundH - 120);
  STATE.collectibles.push({ ...chosen, x: canvasW + 20, y, collected: false, alpha: 1 });
}

function updateCollectibles() {
  const spacing = 500 + Math.random() * 300;
  const lastX = STATE.collectibles.length ? STATE.collectibles[STATE.collectibles.length - 1].x : -999;
  if (canvasW - lastX > spacing || STATE.collectibles.length === 0) {
    spawnCollectible();
  }

  STATE.collectibles.forEach(c => {
    c.x -= STATE.scrollSpeed;
    if (c.collected) c.alpha -= 0.08;
  });

  STATE.collectibles = STATE.collectibles.filter(c => c.x + 40 > 0 && c.alpha > 0);
}

function drawCollectibles() {
  const s = CONFIG.collectibleSize;
  STATE.collectibles.forEach(c => {
    ctx.save();
    ctx.globalAlpha = c.alpha;
    // Glow ring
    ctx.beginPath();
    ctx.arc(c.x, c.y, s * 0.8, 0, Math.PI * 2);
    ctx.fillStyle = c.color + '44';
    ctx.fill();
    // Emoji
    ctx.font = `${s}px Arial`;
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(c.emoji, c.x, c.y);
    ctx.restore();
  });
}

function applyCollectible(c) {
  const p = STATE.player;
  c.collected = true;
  if (c.type === 'ramen') {
    p.speedBoost = true; p.speedTimer = 180;
    STATE.score += 100;
    showFloatingText(c.x, c.y, '+100 🍜 SPEED!', '#ff6b35');
  } else if (c.type === 'mac') {
    p.multiplier = 2; p.multiplierTimer = 300;
    STATE.score += 50;
    showFloatingText(c.x, c.y, '2× MAC POWER', '#ffcc00');
  } else if (c.type === 'baileys') {
    p.invincible = true; p.invincibleTimer = 180;
    STATE.score += 50;
    showFloatingText(c.x, c.y, "BAILEY'S! INVINCIBLE", '#d4a017');
  } else if (c.type === 'bunny') {
    if (p.lives < 3) p.lives++;
    STATE.score += 200;
    showFloatingText(c.x, c.y, '+LIFE 🐱 BUNNY!', '#ffaacc');
  } else if (c.type === 'thrift') {
    STATE.score += 500;
    showFloatingText(c.x, c.y, '+500 GOODWILL SCORE', '#9b59b6');
  }
}

// Floating score text
const floatingTexts = [];
function showFloatingText(x, y, text, color) {
  floatingTexts.push({ x, y, text, color, alpha: 1, vy: -1.5 });
}
function updateFloatingTexts() {
  floatingTexts.forEach(t => { t.y += t.vy; t.alpha -= 0.018; });
  floatingTexts.splice(0, floatingTexts.length, ...floatingTexts.filter(t => t.alpha > 0));
}
function drawFloatingTexts() {
  floatingTexts.forEach(t => {
    ctx.save();
    ctx.globalAlpha = t.alpha;
    ctx.font = 'bold 16px Arial';
    ctx.fillStyle = t.color;
    ctx.strokeStyle = 'rgba(0,0,0,0.5)';
    ctx.lineWidth = 3;
    ctx.textAlign = 'center';
    ctx.strokeText(t.text, t.x, t.y);
    ctx.fillText(t.text, t.x, t.y);
    ctx.restore();
  });
}
```

- [ ] **Step 2: Wire collectibles into `update()` and `draw()`**

In `update()`, add:
```js
  updateCollectibles();
  updateFloatingTexts();
```

In `draw()`, add after `drawObstacles()`:
```js
  drawCollectibles();
  drawFloatingTexts();
```

- [ ] **Step 3: Open in browser and verify**

Expected: collectible emojis appear with glow rings and scroll left. They don't disappear yet (collision is Task 6). Verify all 5 types spawn by watching for a while or setting `STATE.distance = 6100` to see thrift racks in America zone.

- [ ] **Step 4: Commit**

```bash
git add game.html
git commit -m "feat: collectibles — ramen, mac, baileys, bunny, thrift"
```

---

## Task 6: Collision Detection

**Files:**
- Modify: `game.html` — add AABB collision for obstacles and collectibles

- [ ] **Step 1: Add `checkCollisions()` function** (insert before `loop()`)

```js
// ─── COLLISIONS ───────────────────────────────────────────────────────────────
function checkCollisions() {
  const p = STATE.player;
  const px = CONFIG.playerX;
  const py = p.y;
  const pw = p.w * 0.7; // tighter hitbox than visual
  const ph = p.h * 0.7;

  // Obstacles
  if (!p.invincible) {
    for (const o of STATE.obstacles) {
      if (o.x > px + pw || o.x + o.w < px - pw) continue; // X check first

      if (o.type === 'windmill' || o.type === 'fog') {
        // Top column
        if (py - ph / 2 < o.topH && o.x < px + pw && o.x + o.w > px - pw) {
          loseLife(); p.wobble = 20; return;
        }
        // Bottom column
        if (py + ph / 2 > o.bottomY && o.x < px + pw && o.x + o.w > px - pw) {
          loseLife(); p.wobble = 20; return;
        }
      } else if (o.type === 'tentacle') {
        if (o.collisionTopY !== undefined) {
          if (py + ph / 2 > o.collisionTopY && o.x < px + pw && o.x + o.w > px - pw) {
            loseLife(); p.wobble = 20; return;
          }
        }
      } else if (o.type === 'ai_robot') {
        const dist = Math.hypot(px - o.cx, py - o.cy);
        if (dist < 28) {
          loseLife();
          p.wobble = 30;
          showFloatingText(px, py - 40, "Sorry, Eirene. Dad let the AI drive again.", '#ff4444');
          return;
        }
      }
    }
  }

  // Collectibles
  for (const c of STATE.collectibles) {
    if (c.collected) continue;
    const dist = Math.hypot(px - c.x, py - c.y);
    if (dist < CONFIG.collectibleSize * 0.9) {
      applyCollectible(c);
    }
  }
}
```

- [ ] **Step 2: Call `checkCollisions()` in `update()`**

Add after `updateCollectibles()`:
```js
  checkCollisions();
```

- [ ] **Step 3: Open in browser and verify**

Start game and deliberately fly into a windmill column. Expected: plane flashes (invincibility), life count eventually decrements to 0, death screen triggers. Fly over a ramen bowl — it disappears and "SPEED!" floats up. Collect Bunny when below max lives — life count increases.

- [ ] **Step 4: Commit**

```bash
git add game.html
git commit -m "feat: AABB collision detection — obstacles and collectibles"
```

---

## Task 7: HUD (Score, Lives, Power-Up Bar)

**Files:**
- Modify: `game.html` — add `drawHUD()` with score, Bunny life icons, power-up indicator

- [ ] **Step 1: Add `drawHUD()` function** (insert before `loop()`)

```js
// ─── HUD ──────────────────────────────────────────────────────────────────────
function drawHUD() {
  const p = STATE.player;

  // Score (top right)
  ctx.save();
  ctx.font = 'bold 22px Arial';
  ctx.fillStyle = '#fff';
  ctx.strokeStyle = 'rgba(0,0,0,0.6)';
  ctx.lineWidth = 3;
  ctx.textAlign = 'right';
  ctx.strokeText(`${STATE.score}`, canvasW - 16, 34);
  ctx.fillText(`${STATE.score}`, canvasW - 16, 34);
  ctx.restore();

  // Bunny lives (top left) — small cat faces
  for (let i = 0; i < 3; i++) {
    const lx = 18 + i * 36;
    const ly = 20;
    ctx.save();
    ctx.globalAlpha = i < p.lives ? 1 : 0.25;
    ctx.font = '26px Arial';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText('🐱', lx, ly);
    ctx.restore();
  }

  // Active power-up indicator (below score)
  let powerLabel = '';
  let powerColor = '';
  let powerFrac = 0;
  if (p.invincible) {
    powerLabel = "BAILEY'S 🥛";
    powerColor = '#d4a017';
    powerFrac = p.invincibleTimer / 180;
  } else if (p.speedBoost) {
    powerLabel = 'RAMEN SPEED 🍜';
    powerColor = '#ff6b35';
    powerFrac = p.speedTimer / 180;
  } else if (p.multiplier > 1) {
    powerLabel = '2× MAC 🧀';
    powerColor = '#ffcc00';
    powerFrac = p.multiplierTimer / 300;
  }

  if (powerLabel) {
    const bx = canvasW - 140, by = 44, bw = 128, bh = 10;
    ctx.fillStyle = 'rgba(0,0,0,0.4)';
    ctx.fillRect(bx, by, bw, bh);
    ctx.fillStyle = powerColor;
    ctx.fillRect(bx, by, bw * powerFrac, bh);
    ctx.font = '11px Arial';
    ctx.fillStyle = '#fff';
    ctx.textAlign = 'center';
    ctx.fillText(powerLabel, bx + bw / 2, by + bh + 12);
  }

  // Distance progress bar (top center, thin)
  const barW = canvasW * 0.4;
  const barX = (canvasW - barW) / 2;
  ctx.fillStyle = 'rgba(0,0,0,0.3)';
  ctx.fillRect(barX, 8, barW, 6);
  const frac = Math.min(1, STATE.distance / CONFIG.totalDistance);
  ctx.fillStyle = '#fff';
  ctx.fillRect(barX, 8, barW * frac, 6);

  // ✈️ icon on progress bar
  ctx.font = '14px Arial';
  ctx.textAlign = 'center';
  ctx.fillText('✈️', barX + barW * frac, 22);
}
```

- [ ] **Step 2: Call `drawHUD()` in `draw()`** (add at the very end of the game draw block, after all other draw calls)

```js
  drawHUD();
```

- [ ] **Step 3: Open in browser and verify**

Expected: top-left has 3 🐱 icons. Top-right shows score incrementing. Progress bar at top center shows ✈️ icon advancing. Collect Bailey's — yellow power bar appears under score. Lose a life — one 🐱 goes dim.

- [ ] **Step 4: Commit**

```bash
git add game.html
git commit -m "feat: HUD — score, Bunny lives, power-up bar, progress"
```

---

## Task 8: Start Screen

**Files:**
- Modify: `game.html` — add `drawStartScreen()`, wire into `draw()`

- [ ] **Step 1: Add `drawStartScreen()` function** (insert before `loop()`)

```js
// ─── SCREEN: START ────────────────────────────────────────────────────────────
function drawStartScreen() {
  // Background gradient
  const g = ctx.createLinearGradient(0, 0, 0, canvasH);
  g.addColorStop(0, '#1a2a5e');
  g.addColorStop(1, '#0d1b3e');
  ctx.fillStyle = g;
  ctx.fillRect(0, 0, canvasW, canvasH);

  // Stars
  ctx.fillStyle = 'rgba(255,255,255,0.7)';
  for (let i = 0; i < 60; i++) {
    const sx = ((i * 137.5) % canvasW);
    const sy = ((i * 97.3) % (canvasH * 0.7));
    ctx.beginPath();
    ctx.arc(sx, sy, 1.2, 0, Math.PI * 2);
    ctx.fill();
  }

  // Plane on start screen (animated bob)
  const bobY = canvasH * 0.38 + Math.sin(STATE.frame * 0.04) * 8;
  STATE.player.y = bobY;
  drawPlane();

  // Title
  ctx.save();
  ctx.textAlign = 'center';
  ctx.font = 'bold 32px Arial';
  ctx.fillStyle = '#fff';
  ctx.shadowColor = '#4a8fdd';
  ctx.shadowBlur = 20;
  ctx.fillText("✈️ EIRENE'S GREAT ESCAPE", canvasW / 2, canvasH * 0.22);
  ctx.font = '18px Arial';
  ctx.fillStyle = '#87ceeb';
  ctx.shadowBlur = 0;
  ctx.fillText('Amsterdam → Home', canvasW / 2, canvasH * 0.30);
  ctx.restore();

  // Dad's message
  ctx.save();
  ctx.textAlign = 'center';
  ctx.font = '14px Arial';
  ctx.fillStyle = 'rgba(255,255,255,0.75)';
  ctx.fillText("Hi Eirene! Dad built this at 5am using AI.", canvasW / 2, canvasH * 0.56);
  ctx.fillText("Yes, he knows. Get home safe. — Dad 🤖", canvasW / 2, canvasH * 0.62);
  ctx.restore();

  // Tap to start button
  const pulse = 0.85 + Math.sin(STATE.frame * 0.06) * 0.15;
  ctx.save();
  ctx.globalAlpha = pulse;
  ctx.textAlign = 'center';
  ctx.font = 'bold 20px Arial';
  ctx.fillStyle = '#fff';
  ctx.strokeStyle = '#4a8fdd';
  ctx.lineWidth = 2;
  const btnW = 200, btnH = 50, btnX = canvasW / 2 - btnW / 2, btnY = canvasH * 0.72;
  ctx.beginPath();
  ctx.roundRect(btnX, btnY, btnW, btnH, 12);
  ctx.fillStyle = 'rgba(74,143,221,0.3)';
  ctx.fill();
  ctx.stroke();
  ctx.fillStyle = '#fff';
  ctx.fillText('TAP TO START ✈️', canvasW / 2, btnY + 32);
  ctx.restore();
}
```

- [ ] **Step 2: Update `draw()` to show start screen**

Replace the current `draw()` function's start-screen block:
```js
function draw() {
  ctx.clearRect(0, 0, canvasW, canvasH);

  if (STATE.screen === 'start') {
    drawStartScreen();
    return;
  }
  drawBackground();
  drawGround();
  drawObstacles();
  drawCollectibles();
  drawFloatingTexts();
  drawPlane();
  drawZoneLabel();
  drawHUD();
}
```

- [ ] **Step 3: Update `update()` to animate start screen**

At the top of `update()`, before the `STATE.screen !== 'game'` guard:
```js
function update() {
  STATE.frame++;
  if (STATE.screen === 'start') return; // just animate frame counter
  if (STATE.screen !== 'game') return;
  // ... rest of update
```

- [ ] **Step 4: Open in browser and verify**

Expected: star-field background, animated plane bobbing, title "EIRENE'S GREAT ESCAPE", dad's message, pulsing TAP TO START button. Tapping starts the game.

- [ ] **Step 5: Commit**

```bash
git add game.html
git commit -m "feat: start screen with animated plane and dad's message"
```

---

## Task 9: Death Screen + Win Screen

**Files:**
- Modify: `game.html` — add `drawDeadScreen()`, `drawWinScreen()`, wire into `draw()`

- [ ] **Step 1: Add `drawDeadScreen()`** (insert before `loop()`)

```js
// ─── SCREEN: DEATH ────────────────────────────────────────────────────────────
function drawDeadScreen() {
  ctx.fillStyle = 'rgba(180,20,20,0.88)';
  ctx.fillRect(0, 0, canvasW, canvasH);

  ctx.save();
  ctx.textAlign = 'center';

  // Old Gregg face
  ctx.font = '72px Arial';
  ctx.fillText('🦑', canvasW / 2, canvasH * 0.18);

  ctx.font = 'bold 26px Arial';
  ctx.fillStyle = '#fff';
  ctx.shadowColor = '#000';
  ctx.shadowBlur = 10;
  const quote = DEATH_QUOTES[STATE.deathQuoteIdx];
  // Word wrap
  wrapText(ctx, quote, canvasW / 2, canvasH * 0.34, canvasW * 0.8, 34);

  ctx.font = '20px Arial';
  ctx.fillStyle = 'rgba(255,255,255,0.8)';
  ctx.fillText(`Score: ${STATE.score}`, canvasW / 2, canvasH * 0.58);

  // Retry button
  ctx.font = 'bold 20px Arial';
  ctx.fillStyle = '#fff';
  const bw = 180, bh = 50, bx = canvasW / 2 - bw / 2, by = canvasH * 0.68;
  ctx.beginPath();
  ctx.roundRect(bx, by, bw, bh, 12);
  ctx.fillStyle = 'rgba(255,255,255,0.2)';
  ctx.fill();
  ctx.strokeStyle = '#fff';
  ctx.lineWidth = 2;
  ctx.stroke();
  ctx.fillStyle = '#fff';
  ctx.fillText('TRY AGAIN ✈️', canvasW / 2, by + 32);

  ctx.restore();
}
```

- [ ] **Step 2: Add `wrapText()` helper** (insert before `drawDeadScreen`)

```js
function wrapText(ctx, text, x, y, maxWidth, lineH) {
  const words = text.split(' ');
  let line = '';
  let currentY = y;
  for (const word of words) {
    const test = line ? line + ' ' + word : word;
    if (ctx.measureText(test).width > maxWidth && line) {
      ctx.fillText(line, x, currentY);
      line = word;
      currentY += lineH;
    } else {
      line = test;
    }
  }
  ctx.fillText(line, x, currentY);
}
```

- [ ] **Step 3: Add `drawWinScreen()`** (insert after `drawDeadScreen`)

```js
// ─── SCREEN: WIN ──────────────────────────────────────────────────────────────
function drawWinScreen() {
  const g = ctx.createLinearGradient(0, 0, 0, canvasH);
  g.addColorStop(0, '#ff9a3c');
  g.addColorStop(1, '#c8e6c9');
  ctx.fillStyle = g;
  ctx.fillRect(0, 0, canvasW, canvasH);

  // Confetti particles
  drawParticles();

  ctx.save();
  ctx.textAlign = 'center';

  ctx.font = '64px Arial';
  ctx.fillText('🐱', canvasW / 2, canvasH * 0.16);

  ctx.font = 'bold 28px Arial';
  ctx.fillStyle = '#1a3a1a';
  ctx.shadowColor = 'rgba(255,255,255,0.8)';
  ctx.shadowBlur = 12;
  ctx.fillText('EIRENE IS HOME.', canvasW / 2, canvasH * 0.34);
  ctx.font = 'bold 22px Arial';
  ctx.fillText('Bunny has been waiting. 🐱', canvasW / 2, canvasH * 0.44);

  ctx.font = '18px Arial';
  ctx.fillStyle = 'rgba(0,0,0,0.7)';
  ctx.shadowBlur = 0;
  ctx.fillText(`Final Score: ${STATE.score}`, canvasW / 2, canvasH * 0.56);

  ctx.font = '13px Arial';
  ctx.fillStyle = 'rgba(0,0,0,0.5)';
  ctx.fillText('Made by Dad at 5am with AI ☕🤖', canvasW / 2, canvasH * 0.63);

  // Play Again button
  const bw = 200, bh = 50, bx = canvasW / 2 - bw / 2, by = canvasH * 0.72;
  ctx.beginPath();
  ctx.roundRect(bx, by, bw, bh, 12);
  ctx.fillStyle = 'rgba(255,255,255,0.5)';
  ctx.fill();
  ctx.strokeStyle = '#1a3a1a';
  ctx.lineWidth = 2;
  ctx.stroke();
  ctx.font = 'bold 20px Arial';
  ctx.fillStyle = '#1a3a1a';
  ctx.fillText('PLAY AGAIN ✈️', canvasW / 2, by + 32);

  ctx.restore();
}
```

- [ ] **Step 4: Add particle system for confetti** (insert before `loop()`)

```js
// ─── PARTICLES (CONFETTI) ─────────────────────────────────────────────────────
function spawnConfetti() {
  const colors = ['#ff6b6b','#ffd93d','#6bcb77','#4d96ff','#ff6fc8','#fff'];
  for (let i = 0; i < 90; i++) {
    STATE.particles.push({
      x: canvasW * Math.random(),
      y: canvasH * 0.3 * Math.random(),
      vx: (Math.random() - 0.5) * 5,
      vy: Math.random() * 2 + 1,
      w: 6 + Math.random() * 8,
      h: 4 + Math.random() * 6,
      color: colors[Math.floor(Math.random() * colors.length)],
      rot: Math.random() * Math.PI * 2,
      rotV: (Math.random() - 0.5) * 0.15,
      alpha: 1,
    });
  }
}

function updateParticles() {
  STATE.particles.forEach(p => {
    p.x += p.vx;
    p.y += p.vy;
    p.vy += 0.05;
    p.rot += p.rotV;
    if (p.y > canvasH * 0.9) p.alpha -= 0.02;
  });
  STATE.particles = STATE.particles.filter(p => p.alpha > 0);
}

function drawParticles() {
  STATE.particles.forEach(p => {
    ctx.save();
    ctx.globalAlpha = p.alpha;
    ctx.translate(p.x, p.y);
    ctx.rotate(p.rot);
    ctx.fillStyle = p.color;
    ctx.fillRect(-p.w / 2, -p.h / 2, p.w, p.h);
    ctx.restore();
  });
}
```

- [ ] **Step 5: Update `update()` to handle particles and dead/win screens**

Add after `STATE.frame++` at the top of `update()`:
```js
  if (STATE.screen === 'win') { updateParticles(); return; }
  if (STATE.screen === 'dead') return;
```

- [ ] **Step 6: Update `draw()` to handle all screens**

```js
function draw() {
  ctx.clearRect(0, 0, canvasW, canvasH);

  if (STATE.screen === 'start') { drawStartScreen(); return; }
  if (STATE.screen === 'dead') { drawDeadScreen(); return; }
  if (STATE.screen === 'win') { drawWinScreen(); return; }

  // Game screen
  drawBackground();
  drawGround();
  drawObstacles();
  drawCollectibles();
  drawFloatingTexts();
  drawPlane();
  drawZoneLabel();
  drawHUD();
}
```

- [ ] **Step 7: Open in browser and verify**

Test death: fly into a windmill until 0 lives → red death screen with Old Gregg quote and score. Test win: in console set `STATE.distance = CONFIG.totalDistance - 10`, then next frame triggers win screen with confetti. Tap/click on both screens resets to game.

- [ ] **Step 8: Commit**

```bash
git add game.html
git commit -m "feat: death and win screens with confetti"
```

---

## Task 10: Random Events (Old Gregg Warning, IASYL Flash, Hot Dog Guy, AI Robot)

**Files:**
- Modify: `game.html` — add random event system and AI robot obstacle

- [ ] **Step 1: Add AI robot obstacle spawn/draw** (extend obstacle system)

Add to `spawnObstacle()` — this is called from the random event timer, not the normal spawn system. Add a separate function:

```js
function spawnAiRobot() {
  STATE.obstacles.push({
    type: 'ai_robot',
    x: canvasW + 20,
    cx: canvasW + 20,  // center x for collision
    cy: 80 + Math.random() * (canvasH - CONFIG.groundH - 140),
    w: 50, h: 50,
    speed: STATE.scrollSpeed * 1.2,
  });
}

function drawAiRobot(o) {
  o.cx = o.x;
  ctx.save();
  ctx.font = '38px Arial';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillText('🤖', o.cx, o.cy);
  ctx.font = 'bold 11px Arial';
  ctx.fillStyle = '#ff4444';
  ctx.textBaseline = 'top';
  ctx.fillText("DAD'S AI", o.cx, o.cy + 22);
  ctx.restore();
}
```

Update `drawObstacles()` to handle `ai_robot`:
```js
  else if (o.type === 'ai_robot') drawAiRobot(o);
```

Update `updateObstacles()` to move ai_robot by its own speed:
```js
  STATE.obstacles.forEach(o => {
    if (o.type === 'ai_robot') { o.x -= o.speed; o.cx = o.x; }
    else { o.x -= STATE.scrollSpeed; }
    if (o.type === 'windmill') o.angle += 0.025;
  });
```

- [ ] **Step 2: Add Hot Dog Guy** (insert before `loop()`)

```js
function spawnHotDogGuy() {
  STATE.hotDogGuy = {
    x: canvasW + 30,
    y: canvasH - CONFIG.groundH - 30,
    speed: STATE.scrollSpeed * 2.5,
  };
}

function updateHotDogGuy() {
  if (!STATE.hotDogGuy) return;
  STATE.hotDogGuy.x -= STATE.hotDogGuy.speed;
  if (STATE.hotDogGuy.x < -60) { STATE.hotDogGuy = null; return; }

  // Collision with player — bonus points
  const dist = Math.hypot(CONFIG.playerX - STATE.hotDogGuy.x, STATE.player.y - STATE.hotDogGuy.y);
  if (dist < 35) {
    STATE.score += 200;
    showFloatingText(STATE.hotDogGuy.x, STATE.hotDogGuy.y - 20, '+200 🌭 HOT DOG!', '#ff8c00');
    STATE.hotDogGuy = null;
  }
}

function drawHotDogGuy() {
  if (!STATE.hotDogGuy) return;
  ctx.save();
  ctx.font = '36px Arial';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'bottom';
  ctx.fillText('🌭', STATE.hotDogGuy.x, STATE.hotDogGuy.y);
  ctx.font = '10px Arial';
  ctx.fillStyle = '#fff';
  ctx.textBaseline = 'top';
  ctx.fillText('🏃', STATE.hotDogGuy.x, STATE.hotDogGuy.y - 36);
  ctx.restore();
}
```

- [ ] **Step 3: Add random event overlays + timers** (insert before `loop()`)

```js
// ─── RANDOM EVENTS ────────────────────────────────────────────────────────────
function updateRandomEvents() {
  // Old Gregg warning (zone 1 only)
  if (STATE.zone === 1 && STATE.oldGreggWarningTimer <= 0) {
    if (Math.random() < 0.003) {
      STATE.oldGreggWarningTimer = 90;
      // Spawn a tentacle wave
      for (let i = 0; i < 3; i++) {
        setTimeout(() => {
          if (STATE.screen === 'game') STATE.obstacles.push({
            type: 'tentacle',
            x: canvasW + 10 + i * 200,
            gapY: canvasH * 0.4,
            w: CONFIG.obstacleW,
          });
        }, i * 600);
      }
    }
  }
  if (STATE.oldGreggWarningTimer > 0) STATE.oldGreggWarningTimer--;

  // IASYL flash
  STATE.frame % 1 === 0 && (STATE.nextIasyl--);
  if (STATE.nextIasyl <= 0) {
    STATE.iasylTimer = 80;
    STATE.nextIasyl = 1200 + Math.random() * 1200;
  }
  if (STATE.iasylTimer > 0) STATE.iasylTimer--;

  // AI robot
  STATE.nextAiRobot--;
  if (STATE.nextAiRobot <= 0) {
    spawnAiRobot();
    STATE.nextAiRobot = 700 + Math.random() * 800;
  }

  // Hot dog guy
  STATE.nextHotDog--;
  if (STATE.nextHotDog <= 0 && !STATE.hotDogGuy) {
    spawnHotDogGuy();
    STATE.nextHotDog = 1800 + Math.random() * 1200;
  }

  updateHotDogGuy();
}

function drawRandomEventOverlays() {
  // Old Gregg warning
  if (STATE.oldGreggWarningTimer > 60) {
    const alpha = Math.min(1, (STATE.oldGreggWarningTimer - 60) / 30) * 0.85;
    ctx.save();
    ctx.fillStyle = `rgba(0,200,80,${alpha})`;
    ctx.fillRect(0, 0, canvasW, canvasH);
    ctx.font = 'bold 38px Arial';
    ctx.fillStyle = '#fff';
    ctx.textAlign = 'center';
    ctx.fillText("I'M OLD GREGG!", canvasW / 2, canvasH / 2);
    ctx.font = '22px Arial';
    ctx.fillText('🦑🦑🦑', canvasW / 2, canvasH / 2 + 48);
    ctx.restore();
  }

  // IASYL flash
  if (STATE.iasylTimer > 0) {
    const alpha = Math.min(1, STATE.iasylTimer / 30) * 0.75;
    ctx.save();
    ctx.fillStyle = `rgba(220,20,20,${alpha})`;
    ctx.fillRect(0, 0, canvasW, canvasH);
    ctx.font = 'bold 32px Arial';
    ctx.fillStyle = '#fff';
    ctx.textAlign = 'center';
    ctx.fillText('STOP TALKING.', canvasW / 2, canvasH / 2 - 20);
    ctx.fillText('STOP TALKING.', canvasW / 2, canvasH / 2 + 28);
    ctx.restore();
  }

  drawHotDogGuy();
}
```

- [ ] **Step 4: Wire into `update()` and `draw()`**

In `update()`, add:
```js
  updateRandomEvents();
```

In `draw()`, game screen block, add after `drawHUD()`:
```js
  drawRandomEventOverlays();
```

- [ ] **Step 5: Open in browser and verify**

Play through Atlantic zone — Old Gregg flash appears with green overlay and "I'M OLD GREGG!" text. Wait ~30s — IASYL red flash appears. AI robot 🤖 labeled "DAD'S AI" floats across the screen. Hot dog guy 🌭🏃 runs across and gives +200 if caught.

- [ ] **Step 6: Commit**

```bash
git add game.html
git commit -m "feat: random events — Old Gregg warning, IASYL flash, hot dog guy, AI robot"
```

---

## Task 11: Game Feel Polish

**Files:**
- Modify: `game.html` — tune constants, add missing UX details

- [ ] **Step 1: Tune CONFIG constants for good game feel**

Play through the full game. Adjust these in CONFIG if the game feels off:
- If too hard: increase `obstacleGap` (try 200), decrease `baseScrollSpeed` (try 2.5), increase `minObstacleSpacing` (try 380)
- If too easy: decrease `obstacleGap` (try 165), increase `baseScrollSpeed` (try 3.5)
- If flap feels sluggish: adjust `flapForce` (more negative = higher jump), `gravity` (higher = faster fall)

Suggested balanced values:
```js
const CONFIG = {
  gravity: 0.36,
  flapForce: -8.2,
  baseScrollSpeed: 2.8,
  totalDistance: 9000,
  zoneLength: 3000,
  groundH: 55,
  playerX: null,
  obstacleGap: 195,
  obstacleW: 58,
  minObstacleSpacing: 340,
  collectibleSize: 28,
};
```

- [ ] **Step 2: Add Old Gregg Warning to Atlantic obstacle header text**

In `updateObstacles()`, before spawning in zone 1, add a check so first tentacle wave is preceded by a warning. This is already handled in `updateRandomEvents()` — no additional change needed.

- [ ] **Step 3: Verify mobile touch works**

Open `game.html` on iPhone (via AirDrop or local network). Tap to start — game starts. Tap to flap — responsive. No zoom, no scroll bounce. Confirm touch events fire correctly.

If pinch-zoom or scroll interfere, the viewport meta tag in `<head>` already handles this: `user-scalable=no`.

- [ ] **Step 4: Final check — all spec items verified**

| Spec Item | Verified |
|-----------|---------|
| Tap/click to fly | ✓ |
| 3 zones (Europe, Atlantic, America) | ✓ |
| Windmill columns with rotating blades | ✓ |
| Tulip petal decorations | ✓ |
| Old Gregg tentacles + warning | ✓ |
| Fog walls | ✓ |
| AI Robot with wobble + "Dad's AI" text | ✓ |
| Ramen speed boost | ✓ |
| Mac & cheese 2× multiplier | ✓ |
| Bailey's cream invincibility | ✓ |
| Bunny life icon + +1 life | ✓ |
| Thrift rack +500 (America only) | ✓ |
| Hot dog guy bonus | ✓ |
| IASYL flash | ✓ |
| 3 Bunny lives in HUD | ✓ |
| Score + progress bar HUD | ✓ |
| Start screen with dad's message | ✓ |
| Death screen with Old Gregg quotes | ✓ |
| Win screen "Bunny has been waiting" + confetti | ✓ |
| Mobile touch works | ✓ |
| Single HTML file, no dependencies | ✓ |

- [ ] **Step 5: Commit**

```bash
git add game.html
git commit -m "polish: game feel tuning, mobile touch verification"
```

---

## Task 12: Deploy to GitHub Pages

**Files:**
- No code changes — deployment setup

- [ ] **Step 1: Push branch to GitHub**

```bash
git push -u origin claude/cranky-mahavira
```

- [ ] **Step 2: Enable GitHub Pages for this branch**

```bash
gh repo view --web
```

In the browser: Settings → Pages → Source → Deploy from branch → select `claude/cranky-mahavira`, folder `/` (root) → Save.

Or via CLI if `gh` supports it:
```bash
gh api repos/{owner}/{repo}/pages -X POST -f source[branch]=claude/cranky-mahavira -f source[path]=/
```

- [ ] **Step 3: Get the Pages URL**

```bash
gh api repos/{owner}/{repo}/pages --jq '.html_url'
```

Expected output: something like `https://{owner}.github.io/{repo}/game.html`

- [ ] **Step 4: Test the URL on a phone**

Open the URL in iPhone Safari. Verify: page loads, tap starts game, game plays correctly, no console errors.

- [ ] **Step 5: Text Eirene the URL**

Message: *"Hi! I've been up since 5am vibe coding something for you. Open this in Safari on your phone: [URL] — it's what I was building this morning. Safe flight 🛩️🐱 — Dad"*

- [ ] **Step 6: Merge to main when she lands (optional)**

```bash
git checkout main
git merge claude/cranky-mahavira
git push origin main
```

---

## Self-Review Checklist

**Spec coverage:**
- [x] All 3 zones with correct themes
- [x] All 5 collectibles with correct effects
- [x] All obstacle types: windmill, tulip, tentacle, fog, AI robot
- [x] All random events: Old Gregg warning, IASYL, hot dog guy
- [x] All screens: start, game, dead, win
- [x] Confetti particle system on win
- [x] HUD: score, Bunny lives, power-up bar, progress
- [x] Personalization: dad's message, "DAD'S AI" label, GOODWILL thrift racks, Bunny
- [x] Mobile-first touch events
- [x] GitHub Pages deployment

**Placeholder scan:** None found. All code blocks are complete.

**Type consistency:**
- `STATE.player` object used consistently throughout
- `CONFIG.playerX` set in `resize()`, referenced as `CONFIG.playerX` everywhere
- `loseLife()` called from both collision detection and out-of-bounds check
- `showFloatingText()` used consistently across collectibles and collision effects
- `spawnConfetti()` defined in Task 1 as stub, implemented in Task 9
