# Eirene's Great Escape — Project Memory

## What This Is

A single-file HTML5 canvas arcade game built as a surprise for Eirene (dad's daughter), who flew from Amsterdam back to the US. Built at 5am using AI ("vibe coding"). Deployed to GitHub Pages so she can open it in a mobile browser via a shared URL.

**Live URL:** `https://davidgee69.github.io/business-deliverables/game.html`  
**Cache bust:** append `?v=N` (currently at v15) when testing after deploys.

---

## Repository

- **Repo:** `https://github.com/davidgee69/business-deliverables` (public)
- **Branch:** `claude/cranky-mahavira`
- **PR:** #2 (open, merging into main deploys to GitHub Pages)
- **Working file:** `game.html` — the entire game is one self-contained HTML file, no deps, no CDN

### Local paths
- Worktree: `/Users/apple/Desktop/family dinner easy decider/.claude/worktrees/cranky-mahavira/`
- Main project: `/Users/apple/Desktop/family dinner easy decider/`

---

## About Eirene (personalization context)

- Daughter's name: **Eirene** (spelled exactly this way)
- Cat: **Bunny Buns** (🐱) — appears as the life icon and a collectible
- Loves: **Old Gregg** (Mighty Boosh), **I Think You Should Leave** (Tim Robinson)
- Loves: **ramen** 🍜, **mac & cheese** 🧀, **Bailey's** 🥛, **thrifting** (Goodwill)
- Teases dad about using AI too much — hence "DAD'S AI" robot obstacle and self-deprecating start screen text

---

## Game Architecture

**Single file:** `game.html`  
**Engine:** Vanilla JS, `requestAnimationFrame` game loop, HTML5 Canvas 2D  
**State machine:** `start` | `instructions` | `game` | `dead` | `win`

### Key globals
```js
CONFIG          // flapForce: -4.2, obstacleW, groundH: 50, totalDistance, zoneLength, etc.
STATE           // screen, player, obstacles, collectibles, score, distance, zone,
                // gameFrames, currentLevel, levelUpFlash, scrollSpeed, frame, etc.
canvasW, canvasH  // set by resize(), always = window.innerWidth/Height
isMobile()      // returns true if touch device
isPortrait()    // returns true if height > width
planeScale()    // returns canvasW/1000 clamped 0.5–1.4
```

### Player object
```js
STATE.player = {
  y, vy, w, h,        // position, velocity, size (w/h updated each frame by drawPlane)
  lives: 9,           // starts at 9, max 9
  wobble,             // visual wobble on hit
  invincible,         // bool — true for 150 frames after losing a life
  invincibleTimer,
  speedBoost, speedTimer,
  multiplier, multiplierTimer,
}
```

---

## Difficulty System

Frame-based (time-in-game, not distance). `getDifficulty()` returns 1–5:

```js
function getDifficulty() {
  const f = STATE.gameFrames;
  if (f < 3600)  return 1;   // ~1 min
  if (f < 7200)  return 2;   // ~2 min
  if (f < 10800) return 3;   // ~3 min
  if (f < 14400) return 4;   // ~4 min
  return 5;
}
```

### Per-level values
| Level | Gravity | Speed | Obstacle gap | Obstacle spacing | Collectible spacing |
|-------|---------|-------|--------------|------------------|---------------------|
| 1 | 0.045 | 1.1 | N/A (no obstacles) | N/A | 160px |
| 2 | 0.058 | 1.3 | 560px | 860px | 220px |
| 3 | 0.075 | 1.6 | 450px | 740px | 340px |
| 4 | 0.10  | 2.0 | 330px | 580px | 480px |
| 5 | 0.14  | 2.6 | 240px | 460px | 620px |

All gravity values are multiplied by `(canvasH / 800)` to be screen-size-relative.  
`flapForce` = `CONFIG.flapForce * (canvasH / 800) * (isMobile() ? 0.65 : 1.0)`

Level-up flash: `STATE.levelUpFlash = { text: '...', alpha: 1.0 }` — fades out at -0.005/frame.

---

## Three Zones (progress left → right)

| Zone | Index | Background | Obstacles |
|------|-------|------------|-----------|
| Amsterdam/Europe | 0 | Sky blue, windmills, tulips | Windmill gaps |
| The Atlantic | 1 | Dark ocean, waves, fog | Old Gregg tentacles, fog walls |
| America/Home | 2 | Sunset, clouds, thrift stores | Windmill gaps |

Zone transition at `distance >= CONFIG.zoneLength` per zone.

---

## Collectibles

| Emoji | Effect | Points |
|-------|--------|--------|
| 🍜 Ramen | Speed boost 3s | +100 |
| 🧀 Mac & cheese | 2× score multiplier 5s | +0 |
| 🥛 Bailey's | Invincibility 3s | +50 |
| 🐱 Bunny Buns | +1 life (max 9) | +200 |
| 👗 Thrift rack | Bonus | +500 |

Pulsing ▲▼ arrows drawn toward nearby uncollected collectibles via `drawCollectibleArrows()`.

---

## Mobile Responsiveness

- **Portrait mode blocked:** `drawRotatePrompt()` overlay covers everything + blocks all taps when `isMobile() && isPortrait()`
- **Landscape required for mobile play**
- **Plane size:** `planeScale() = canvasW/1000` clamped 0.5–1.4; collision box (`p.w`, `p.h`) updated each frame in `drawPlane()`
- **Flap force:** 35% weaker on mobile (`* 0.65`)
- **Instructions screen:** All font sizes relative to `canvasH` (e.g., `canvasH * 0.10` for big text)

---

## Instructions Screen (7 pages, tap-through)

Pages 0–6. Skip zone: tap `y > canvasH * 0.88` on any page → starts game immediately.

Page content:
- 0: TAP ANYWHERE to fly up / Like Flappy Bird!
- 1: Keep tapping to stay up / Fly through the GAP
- 2: 🦑 Dodge Old Gregg's tentacles
- 3: 🍜🥛 Ramen = speed / Bailey's = invincible
- 4: 🐱 Bunny Buns = extra life
- 5: 🐱🤖 Avoid DAD'S AI
- 6: 🌭 Hot dog guy from I Think You Should Leave = bonus

---

## Screens

### Start screen
"Hi Eirene! Dad built this at 5am using AI. Yes, he knows. Get home safe. — Dad 🤖"

### Death screen
Randomized Old Gregg / IASYL quotes. RETRY button.

### Win screen
"EIRENE IS HOME. Bunny has been waiting." + confetti particle system + PLAY AGAIN

### Rotate prompt (mobile portrait only)
Fullscreen overlay: big ↻ icon + "Turn your phone sideways!" — blocks all input until rotated

---

## HUD

- **Lives:** Row of 9 🐱 icons — full opacity if alive, 20% opacity if lost. Responsive size (`canvasW * 0.052`)
- **Score:** Top-right, bold white with shadow
- **Progress bar:** Center top, small plane ✈️ icon tracks position
- **Power-up bar:** Below score — label + countdown bar (Bailey's, Ramen, Mac)
- **Level-up flash:** Yellow bold centered text when level increases, fades over ~200 frames

---

## Random Events

- **Old Gregg Warning** (Atlantic zone): Screen flashes green, "I'M OLD GREGG!" for 1.5s, tentacle wave incoming
- **IASYL Chaos** (any zone, ~30s): Screen flashes red, "STOP TALKING. STOP TALKING." for 2s, no damage
- **Hot Dog Suit Guy** (Tim Robinson): Runs across screen randomly. Touch him = +200 pts + "FROM I THINK YOU SHOULD LEAVE" flash

---

## Key Functions Reference

```
getDifficulty()         → 1-5 based on STATE.gameFrames
getDynamicGap()         → obstacle gap for current level
planeScale()            → visual/collision scale factor
flap()                  → applies scaled flapForce to player.vy
loseLife()              → decrements lives, resets plane to center y=canvasH*0.45,
                          sets invincible=true for 150 frames
drawPlane()             → draws scaled plane, updates p.w and p.h
drawHUD()               → lives icons, score, progress bar, power-up bar
drawInstructionsScreen()→ 7-page responsive instructions
drawRotatePrompt()      → portrait-mode mobile blocker overlay
drawLevelUpFlash()      → yellow level-up message with fade
drawRandomEventOverlays()→ Old Gregg flash, IASYL flash
spawnConfetti()         → win screen particle system (~80 particles)
resetGame()             → resets all STATE for retry
handleTap(x, y)         → routes taps; blocked during portrait mode on mobile
```

---

## Deployment

Push to `claude/cranky-mahavira` → PR #2 auto-triggers GitHub Pages deploy from `main` (must merge PR or push directly to main).

**To deploy a change:**
```bash
cd "/Users/apple/Desktop/family dinner easy decider/.claude/worktrees/cranky-mahavira"
git add game.html
git commit -m "your message"
git push origin claude/cranky-mahavira
# then merge PR #2 on GitHub, or push directly to main
```

**To test:** Open `https://davidgee69.github.io/business-deliverables/game.html?v=N` (increment N each time to bust cache)

---

## Current Status (as of session end)

Latest commit: `89acee7` — "fix: fully responsive instructions screen"  
Cache-bust version: `?v=15`  
Branch: `claude/cranky-mahavira` — **PR #2 open, needs merge to deploy**

### Known remaining issues / possible improvements
- Further difficulty tuning possible (9 lives + no obstacles in level 1 is very forgiving)
- Win screen / landing animation could be more dramatic
- Sound effects would be great but not implemented (single-file constraint makes this harder)
- The mobile flap force (0.65×) may need further tuning once tested in landscape on iPhone
