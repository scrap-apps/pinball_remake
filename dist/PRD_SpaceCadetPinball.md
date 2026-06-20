# PRD: Space Cadet Pinball (Windows復刻版)

## PROJECT OVERVIEW

Recreate the classic Windows "3D Pinball - Space Cadet" as a faithful Windows `.exe` using Python + PyQt6 + Pygame (for physics/rendering) or pure PyQt6 with custom physics. The game must replicate the original's layout, rules, scoring, and sound effects as closely as possible.

---

## TECHNOLOGY STACK

- **Language**: Python 3.11
- **UI Framework**: PyQt6 (window/launcher shell)
- **Game Engine**: Pygame (game loop, rendering, collision, sound)
- **Physics**: Pymunk (Box2D-style 2D rigid body physics via Chipmunk)
- **Audio**: Pygame mixer (WAV sound effects)
- **Packaging**: PyInstaller → single `.exe`
- **Build**: `build.py` (fully automated)
- **Icons**: Generated via Pillow (no external assets required for icons)

---

## BUILD SYSTEM

### `build.py` requirements

```
1. Create virtual environment (.venv)
2. pip install: pygame, pymunk, PyQt6, PyInstaller, Pillow
3. Generate app icon (128x128 PNG → ICO) using Pillow:
   - Dark navy/space background
   - Silver pinball (circle) with motion blur streaks
   - Star field dots
   - "PINBALL" text in yellow pixel-style font
4. Run PyInstaller:
   --onefile --windowed --icon=icon.ico
   --add-data "assets:assets"  (if assets dir exists)
   --name SpaceCadetPinball
5. Confirm dist/SpaceCadetPinball.exe exists and is > 10MB
6. Print "BUILD SUCCESS: dist/SpaceCadetPinball.exe"
```

Task is NOT complete until `dist/SpaceCadetPinball.exe` launches and runs the game.

---

## WINDOW & DISPLAY

- Window title: `"3D Pinball for Windows - Space Cadet"`
- Default resolution: `600 × 900` px (portrait, matching original aspect ratio)
- Resizable: No
- Background: Deep space theme (dark navy `#0a0a1a`)
- Frame rate: 60 FPS locked
- Launch directly into game (no splash screen needed)

---

## GAME TABLE LAYOUT

Replicate the Space Cadet table faithfully. All coordinates are relative to a 600×900 canvas.

### Table Zones (top → bottom)

```
[DRAIN ZONE]         y: 850–900  — ball lost here
[FLIPPER ZONE]       y: 780–850  — left/right flippers
[LOWER PLAYFIELD]    y: 550–780  — bumpers, targets, lanes
[MIDDLE PLAYFIELD]   y: 300–550  — skill shot lane, flags, ramps
[UPPER PLAYFIELD]    y: 100–300  — launch lanes, top bumpers
[PLUNGER LANE]       x: 540–580, y: 200–850  — right edge launch lane
```

### Walls & Boundary

- Left wall: x=30, vertical
- Right wall: x=570, vertical (left of plunger lane)
- Plunger lane right wall: x=595
- Top wall: y=60, horizontal arc
- Angled side gutters: 30-degree slopes from lower corners toward flippers
- Side outlane channels: narrow lanes outside flippers leading to drain

---

## GAME ELEMENTS (faithfully replicated)

### Flippers
- **Left flipper**: pivot at `(180, 820)`, length 90px, rest angle −30°, activated angle +28°
- **Right flipper**: pivot at `(420, 820)`, length 90px, rest angle +30°, activated angle −28°
- Activation: Left flipper = `Z` or `Left Shift`; Right flipper = `/` or `Right Shift`
- Both: `Space` (nudge / both flip)
- Physics: fast angular velocity (800 deg/s), slight restitution on ball contact
- Visual: rounded rectangle, silver gradient, dark outline

### Plunger (Ball Launcher)
- Position: right lane, x=565, y=750–820
- Control: Hold `↓` to pull back, release to launch
- Power indicator: yellow bar grows while held (0–100%)
- Max power launches ball to top of table
- Visual: spring coil graphic at bottom, power bar on right

### Bumpers (Pop Bumpers) — 3 units
| Name | Position | Points |
|------|----------|--------|
| Left Bumper | (160, 200) | 100 pts |
| Center Bumper | (270, 160) | 100 pts |
| Right Bumper | (380, 200) | 100 pts |
- Radius: 25px
- On hit: ball repelled outward at 1.5× incoming speed, flash white, play bump sound
- Visual: neon circle with glowing ring (cyan/yellow alternating on hit)
- After 75 total bumper hits: score multiplier activates (×2)

### Kickers (Slingshots) — 2 units
- Left kicker: triangle, vertices at `(60,650), (60,720), (130,685)`
- Right kicker: triangle, vertices at `(540,650), (540,720), (470,685)`
- On hit: kick ball away at 1.2× speed, flash, play kick sound
- Points: 50 per hit

### Targets — Inline Targets
- **Attack** targets: 3 drop targets in a row at y=350, x=200,250,300
  - Each target: 25×8px rectangle, standing upright
  - On hit: target drops flat, play drop sound, score 500 pts
  - All 3 hit → rank up + reset targets + 5000 bonus
- **Bonus** letter targets: spell "SPACE" — 5 circular targets at y=420
  - x positions: 180, 220, 260, 300, 340
  - On completion: light SPACE sign, 10000 pts, next ball gets +1 launch

### Ramps — 2 ramps
- **Left ramp**: curved guide from `(80, 580)` → loops to `(200, 120)` (top lanes)
  - Entry: funnel at bottom-left
  - Ball exits into top lane zone
  - Points: 2000 per traversal
- **Right ramp**: curved guide from `(520, 500)` → `(380, 100)`
  - Points: 2000 per traversal
- Ramps rendered as thick curved lines (light gray, 12px width)

### Lanes (Top Roll-Over Lanes) — 5 lanes
- At y=80, evenly spaced x=100 to x=500
- Labels: "1" "2" "3" "4" "5"
- Ball rolling through a lane lights it (yellow indicator)
- All 5 lit → 20000 pts + rank advance
- Left/Right flipper buttons rotate which lane is "active" (lit) while ball is in upper area

### Flags / Mission Indicators
- 9 flag positions around upper playfield (semi-circle arc)
- Flags light up as ranks advance
- Ranks: Recruit → Cadet → Private → Sergeant → Lieutenant → Captain → Commander → Colonel → General
- Each rank reached: 25000 pts bonus + visual fanfare

### Skill Shot Lane (center top)
- Narrow vertical lane at x=300, y=60–120
- If plunger launched ball passes through exact center: 75000 pts + "SKILL SHOT" text flash
- Hit detection: ball x within ±15px of center at y=80

### Gravity Well (Black Hole) — 1 unit
- Position: `(300, 450)`, radius 30px
- On ball entering: hold ball for 2 seconds (spinning animation), then eject randomly
- Points: 15000

### Wormhole Bumpers — 2 units
- Positions: `(120, 380)` and `(480, 380)`
- On hit: teleport ball to partner wormhole position
- Points: 3000

### Ball Saver (Tilt Guard)
- Active for first 5 seconds after ball launch
- If ball drains during this window: ball returned to plunger, no ball lost
- Indicator: blinking "BALL SAVER" text near drain

### Drain
- y > 870, between flippers (x: 180–420)
- Ball lost → life −1 → new ball launch sequence
- Drain sound + screen flash red briefly

---

## PHYSICS SPECIFICATION

Use **Pymunk** for all physics.

```python
# Pymunk setup
space = pymunk.Space()
space.gravity = (0, 900)   # downward gravity (px/s²)

# Ball
ball_body = pymunk.Body(1, pymunk.moment_for_circle(1, 0, 14))
ball_shape = pymunk.Circle(ball_body, 14)
ball_shape.elasticity = 0.6
ball_shape.friction = 0.4

# Flippers: kinematic bodies, rotated via angular velocity
# Walls: static segment shapes
# Bumpers: static circles with elasticity = 1.8 (super bouncy)
# Kickers: static poly shapes with custom collision handler (apply impulse)
```

- Physics step: `space.step(1/120)` × 2 per frame (sub-stepping for accuracy)
- Ball radius: 14px
- Ball mass: 1.0 kg (Pymunk units)
- Restitution: walls=0.4, bumpers=1.8, flippers=0.5
- Max ball speed cap: 1800 px/s (clamp velocity to prevent tunneling)

---

## SCORING SYSTEM

```
Pop Bumper hit:          100 pts  (×2 after 75 hits)
Slingshot hit:            50 pts
Drop Target:             500 pts
Drop Target set clear:  5000 pts
SPACE letter:           1000 pts each
SPACE complete:        10000 pts
Lane lit:               1000 pts
All lanes lit:         20000 pts
Ramp traversal:         2000 pts
Gravity Well:          15000 pts
Wormhole:               3000 pts
Rank advance bonus:    25000 pts
Skill Shot:            75000 pts
```

- High score: top 5 scores saved to `highscores.json` in same dir as exe
- Score displayed top-center in yellow, digital font style

---

## HUD (Heads-Up Display)

Overlay rendered on Pygame surface, top strip (y: 0–60):

```
[BALL: 3]  [SCORE: 0000000]  [HIGH: 0000000]  [RANK: CADET]
```

- Font: monospaced pixel font (use pygame.font with a system monospace or bundled bitmap font)
- Color: yellow `#FFD700` on dark background
- Ball indicator: 3 ball icons (filled = remaining)
- Multiplier badge: shown when active (×2, ×3)

Bottom strip (y: 860–900):
```
[BALL SAVER active indicator]  [TILT warning]
```

---

## SOUNDS

Generate all sounds **programmatically** using numpy + pygame (no external audio files needed).

### Sound generation functions

```python
import numpy as np
import pygame

def generate_tone(freq, duration, volume=0.5, wave='sine', sample_rate=44100):
    t = np.linspace(0, duration, int(sample_rate * duration), False)
    if wave == 'sine':
        wave_data = np.sin(2 * np.pi * freq * t)
    elif wave == 'square':
        wave_data = np.sign(np.sin(2 * np.pi * freq * t))
    elif wave == 'noise':
        wave_data = np.random.uniform(-1, 1, len(t))
    wave_data = (wave_data * volume * 32767).astype(np.int16)
    stereo = np.column_stack([wave_data, wave_data])
    return pygame.sndarray.make_sound(stereo)
```

### Sound Library

| Sound Name | Description | Generation |
|------------|-------------|------------|
| `snd_launch` | Spring release whoosh | Descending sweep 800→200 Hz, 0.3s, sine |
| `snd_bumper` | Pop bumper hit | Short burst 800 Hz square, 0.1s + decay |
| `snd_flipper` | Flipper thwack | 150 Hz square, 0.08s |
| `snd_drain` | Ball lost | Descending 400→100 Hz, 0.8s, sine |
| `snd_target` | Drop target hit | 600 Hz sine, 0.12s |
| `snd_target_clear` | All targets cleared | Ascending arpeggio C-E-G, 0.4s |
| `snd_ramp` | Ramp traversal | Ascending sweep 300→900 Hz, 0.5s |
| `snd_rank` | Rank advance fanfare | 3-note melody, 0.6s |
| `snd_skill` | Skill shot | High-pitched 1200 Hz, 0.3s + echo |
| `snd_wormhole` | Teleport | Sci-fi warble 200–800 Hz oscillate, 0.4s |
| `snd_kick` | Slingshot kick | Sharp 300 Hz, 0.06s |
| `snd_tilt` | Tilt warning | 200 Hz buzz, 0.5s |
| `snd_ballsaver` | Ball saver active | Double beep 880 Hz, 0.2s |
| `snd_complete` | Game over | Descending 3-note, 1.0s |

All sounds generated at startup; stored in a dict `sounds = {}`.

---

## VISUAL STYLE

### Color Palette

| Element | Color |
|---------|-------|
| Background | `#0a0a1a` (deep space) |
| Table walls | `#1a2a4a` (dark blue-gray) |
| Ball | `#c0c0d0` (silver, with specular highlight) |
| Flippers | `#8090b0` (steel blue-gray) |
| Bumpers (inactive) | `#204060` with `#4080ff` ring |
| Bumpers (active) | `#ffffff` flash → `#00ffff` |
| Ramps | `#607090` |
| Targets (up) | `#ff4040` |
| Targets (hit) | `#202020` |
| HUD text | `#ffd700` |
| Score text | `#ffd700` |
| Rank lights | `#00ff80` |
| Star field | White dots, varied opacity |

### Ball Rendering

Draw ball as:
1. Base circle: `#c0c0d0`
2. Highlight circle (offset -3,-3, radius 5): `#ffffff` semi-transparent
3. Shadow arc (offset +2,+2): `#404040`

### Star Field Background

Generate 150 random star positions at startup. Draw as white circles radius 1–2px with varying alpha (50–200). Stars do NOT move (static backdrop).

### Table Decorations

- NASA/space insignia decals (drawn with simple pygame shapes — rocket silhouette, planet rings, stars)
- Neon glow effect on bumpers: draw same circle 3× with increasing radius and decreasing alpha
- Lane indicators: small triangles above each lane, light yellow when active

### Tilt Effect

On near-tilt (nudge): briefly offset entire table render by ±5px (one-frame shake)
On actual tilt: flash "TILT" in red, disable flippers for 3 seconds

---

## CONTROLS

| Action | Key(s) |
|--------|--------|
| Left flipper | `Z`, `Left Shift` |
| Right flipper | `/`, `Right Shift` |
| Both flippers | `Space` |
| Plunger pull | `↓` (hold) |
| Plunger release | Release `↓` |
| Nudge left | `X` |
| Nudge right | `.` |
| Nudge up | `↑` |
| Pause | `P`, `Escape` |
| New game | `F2` |

Nudge: applies small random impulse to ball (±80, ±60 px/s). After 3 nudges in 10 seconds → TILT.

---

## GAME FLOW

```
[Launch exe]
    ↓
[Table displayed, ball in plunger lane]
    ↓
[Player holds ↓ to charge, releases to launch]
    ↓
[Ball in play — physics, collisions, scoring]
    ↓
[Ball drains] → [Life -1] → [If lives > 0: new ball] → [If lives = 0: Game Over]
    ↓
[Game Over screen]
    - Final score displayed
    - High score check / entry (type initials, max 3 chars)
    - Press F2 for new game
```

### Ball Count: 3 per game

### Pause Screen

Semi-transparent dark overlay over game, "PAUSED" text center, "P to resume".

---

## FILE STRUCTURE

```
SpaceCadetPinball/
├── build.py
├── main.py
├── game/
│   ├── __init__.py
│   ├── table.py         # Table layout, static bodies
│   ├── ball.py          # Ball physics body
│   ├── flippers.py      # Flipper mechanics
│   ├── bumpers.py       # Pop bumpers, slingshots
│   ├── targets.py       # Drop targets, lane targets
│   ├── ramps.py         # Ramp detection
│   ├── scoring.py       # Score tracking, ranks
│   ├── sounds.py        # Procedural sound generation
│   ├── hud.py           # HUD rendering
│   └── renderer.py      # All pygame draw calls
└── highscores.json      # Created at runtime
```

---

## CURSOR IMPLEMENTATION NOTES

- Do NOT use any external image/audio asset files. All visuals drawn with pygame primitives. All sounds generated with numpy.
- Pymunk `Space` is the physics authority. Never manually set ball position except for respawn.
- Flipper collision: use `pymunk.Segment` shapes attached to kinematic flipper body; drive angle via `body.angular_velocity` each frame.
- Ramp detection: use pymunk sensor segments (no collision response, just callbacks) to detect ball crossing the ramp entry/exit lines.
- `build.py` must run completely headless (no interactive prompts).
- Entry point: `main.py` → `python main.py` launches the pygame window directly (no PyQt6 wrapper needed unless you want a launcher shell).
- All game logic in `game/` directory, `main.py` only contains the main loop.
- Task NOT complete until `python build.py` succeeds AND `dist/SpaceCadetPinball.exe` launches, shows the table, and ball physics work correctly.

---

## ACCEPTANCE CRITERIA

- [ ] `.exe` launches without terminal window
- [ ] Pinball table renders with all elements (walls, flippers, bumpers, targets, ramps)
- [ ] Ball launches from plunger with power-dependent speed
- [ ] Ball physics feel natural (gravity, bouncing, flipper response)
- [ ] All 3 pop bumpers repel ball correctly
- [ ] Both slingshots activate on contact
- [ ] Drop targets drop and reset after full set cleared
- [ ] Ramps redirect ball to upper table
- [ ] HUD shows score, ball count, rank
- [ ] Scoring increments correctly for each element
- [ ] All 14 sound effects play on correct events
- [ ] 3-ball game cycle: drain → next ball → game over
- [ ] High score saved/loaded from `highscores.json`
- [ ] F2 starts new game
- [ ] P pauses/resumes
- [ ] Nudge with tilt detection works
- [ ] 60 FPS maintained during normal play
