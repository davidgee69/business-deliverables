# Eirene's Great Escape: Amsterdam → Home
**Design Spec — 2026-04-14**

## Overview

A single-file HTML5 canvas arcade game built as a surprise for Eirene, who is currently on a flight from Amsterdam back to the United States. Her dad built it at 5am using AI (vibe coding demo). She can open it in any mobile browser via a shared URL.

**Core mechanic:** Tap/click to fly up, gravity pulls down (Flappy Bird-style). Three geographic zones scroll left-to-right. Collect power-ups, dodge obstacles, get home.

**Delivery:** Single `game.html` file deployed to GitHub Pages. Dad texts Eirene the URL mid-flight.

---

## Architecture

- **Single self-contained HTML file** — no dependencies, no CDN, no install
- **HTML5 Canvas** for rendering (2D context)
- **Vanilla JS** — game loop via `requestAnimationFrame`
- **Touch + click events** — mobile-first (tap = flap)
- All assets drawn programmatically (no image files needed) or as Unicode emoji on canvas

---

## Game Structure

### Screens
1. **Start Screen** — "Hi Eirene! Dad built this at 5am using AI. Yes, he knows. Get home safe. — Dad 🤖" + TAP TO START button
2. **Game Screen** — scrolling map, HUD (score, lives as Bunny icons)
3. **Death Screen** — rotating Old Gregg / IASYL quotes + RETRY button
4. **Win Screen** — "EIRENE IS HOME. Bunny has been waiting." + confetti explosion + PLAY AGAIN button

### Three Zones (progress left → right)
| Zone | Name | Background | Vibe |
|------|------|------------|------|
| 1 | Europe / Amsterdam | Sky blue, flat green ground, windmills | Calm start |
| 2 | The Atlantic | Dark ocean blue, waves, fog | Old Gregg territory |
| 3 | America / Home | Warm sunset, clouds, thrift stores on the ground | Victory stretch |

Zone transitions happen at ~33% and ~66% of total distance.

---

## Player

- Small cartoon plane (drawn with canvas shapes)
- Tap/click = upward impulse
- Gravity constant pulls down
- Horizontal position fixed (~20% from left); world scrolls toward player
- 3 lives displayed as small Bunny cat icons in top-left HUD

---

## Obstacles

| Obstacle | Zone | Behavior | Notes |
|----------|------|----------|-------|
| Windmill blades | Europe | Rotating gaps, vertical pairs | Classic Flappy-style columns |
| Tulip petal bursts | Europe | Scattered projectiles from below | Gentle difficulty ramp |
| Old Gregg tentacles | Atlantic | Rise from bottom of screen | Main Atlantic hazard; "I'M OLD GREGG" warning flashes 1s before |
| Fog walls with gap | Atlantic | Horizontal fog bands with narrow passage | Visibility challenge |
| 🤖 Dad's AI robot | All zones | Floats across screen as hazard | On hit: plane wobbles, text flashes "Sorry, Eirene. Dad let the AI drive again." |

---

## Collectibles

| Item | Effect | Spawn zone |
|------|--------|------------|
| 🍜 Ramen bowl | Speed boost (3s) | All zones |
| 🧀 Mac & cheese | 2× score multiplier (5s) | All zones |
| 🥛 Bailey's cream | Invincibility (3s) + glow effect | Atlantic zone |
| 🐱 Bunny on a cloud | +1 life (max 3) | All zones |
| 👗 Thrift rack | +500 bonus points + "SCORE!" flash | America zone |

---

## Random Events

1. **Old Gregg Warning** — Atlantic zone only. Screen flashes green, text: "I'M OLD GREGG!" for 1.5s. Tentacle wave incoming.
2. **IASYL Chaos Moment** — Any zone, ~every 30s. Screen briefly flashes red, text: "STOP TALKING. STOP TALKING." for 2s. No damage — pure chaos energy.
3. **Hot Dog Suit Guy** — Runs across the middle of the screen randomly. Harmless. Gives 200 bonus points if the plane touches him. Tim Robinson energy.

---

## Death Screen Quotes (randomized)

- "ARE YOU PLAYING GAMES WITH OLD GREGG?!"
- "You're going to look at it."
- "I got the funk. Have you got the funk?"
- "This game was made with AI. That's probably why it's broken."
- "Do you want to come to a party at the end of the universe?"
- "STOP TALKING. STOP TALKING. (you crashed)"

---

## Win Condition & End Screen

- Player reaches the right edge of the America zone
- Plane lands with a little bounce animation
- Confetti explosion (canvas particle system)
- Text: **"EIRENE IS HOME. Bunny has been waiting."**
- Final score displayed
- "PLAY AGAIN" button

---

## HUD

- Top-left: 3 Bunny icons (lives)
- Top-right: Score (increments with distance + collectibles)
- Zone label: brief fade-in text when zone changes ("THE ATLANTIC — watch out.")
- Active power-up: small icon + countdown bar below score

---

## Scoring

| Event | Points |
|-------|--------|
| Distance scrolled | 1 pt/frame |
| Ramen collected | +100 |
| Mac & cheese collected | 2× multiplier active |
| Bailey's cream collected | +50 |
| Bunny collected | +200 |
| Thrift rack (America) | +500 |
| Hot dog suit guy touched | +200 |

---

## Technical Notes

- Target 60fps via `requestAnimationFrame`
- Responsive canvas: fills viewport, scales on resize
- Touch events on the full canvas element (no small buttons to tap)
- All text rendered with canvas `fillText` — no DOM overlays (cleaner on mobile)
- Particle system for confetti: ~80 particles, random colors, gravity + fade
- No localStorage, no external requests — fully offline-capable after first load

---

## Personalization Easter Eggs

- Start screen credits: "Made by Dad at 5am with AI ☕🤖"
- The AI robot obstacle is explicitly labeled "DAD'S AI"
- Death screen includes one self-deprecating AI joke
- Thrift racks in the America zone are labeled "GOODWILL"
- Bunny the cat appears both as a collectible and as the life icons — consistent character

---

## Spec Self-Review Notes

- No TBDs or placeholders — all mechanics fully defined
- Zones, obstacles, and collectibles are each scoped and non-overlapping
- Single HTML file constraint is honored throughout (no assets, no deps)
- Mobile-first approach confirmed (tap = flap, full-canvas touch)
- Win/lose/start screens all defined
- Scoring is complete and internally consistent
