# Witchroam — Game Design Document

> **Working title:** *Witchroam*
> **Status:** Draft v0.1 — pre-production
> **Platform:** Browser (desktop and mobile), single-page, no install
> **Repository:** [dehaku/browsertester](https://github.com/dehaku/browsertester)
> **Audience:** Designers, developers, and Claude Code working in this repo

This document is the source of truth for what the game is and how it should behave. Every requirement has a stable ID (for example `FR-SURV-03`) so that issues, commits, and code comments can reference it. Numeric values are **starting points for tuning**, not final balance.

---

## Table of Contents

1. [Vision](#1-vision)
2. [Design Pillars](#2-design-pillars)
3. [Player Fantasy and Story Frame](#3-player-fantasy-and-story-frame)
4. [Core Loop](#4-core-loop)
5. [Survival Systems](#5-survival-systems)
6. [Magic and Gesture System](#6-magic-and-gesture-system)
7. [Spells](#7-spells)
8. [Exploration and World](#8-exploration-and-world)
9. [Day, Night, and Weather](#9-day-night-and-weather)
10. [Crafting and Building](#10-crafting-and-building)
11. [Controls: Mouse and Touch](#11-controls-mouse-and-touch)
12. [User Interface](#12-user-interface)
13. [Art Direction](#13-art-direction)
14. [Audio](#14-audio)
15. [Technical Design](#15-technical-design)
16. [Accessibility](#16-accessibility)
17. [Milestones and Acceptance Criteria](#17-milestones-and-acceptance-criteria)
18. [Working With This Document in Claude Code](#18-working-with-this-document-in-claude-code)
19. [Open Questions](#19-open-questions)
20. [Glossary](#20-glossary)

---

## 1. Vision

*Witchroam* is a 2D top-down survival exploration game about a witch who wakes in a wild land with no memory. She must keep her body and mind alive while she relearns magic. Magic is drawn, not typed: she traces glyphs in the air to light fires, coax plants from the soil, and conjure lights that push back the dark.

The game is built from simple geometric shapes. Clarity of shape and color carries the meaning. The mood is quiet, eerie, and warm: the tension between the cold outside and the fire you keep alive.

**The one-sentence pitch:** *Draw the spell, keep the fire alive, and remember who you were before the dark remembers you.*

### 1.1 Goals

- Be playable in a modern mobile or desktop browser with no install.
- Make casting feel physical and satisfying through gesture.
- Make survival pressure readable at a glance and tied to the magic system.
- Support a session of 10–25 minutes and a longer arc of several hours.

### 1.2 Non-Goals (for v1)

- No multiplayer or networking.
- No combat against humanoid enemies. Threats are environmental and psychological. (A small set of non-humanoid creatures may be added later.)
- No keyboard-only requirement. Keyboard shortcuts may exist as conveniences, never as the only path.
- No 3D rendering, no photorealistic art, no voice acting, no procedural text generation.

---

## 2. Design Pillars

These are the tie-breakers. When two ideas conflict, pick the one that serves more pillars.

| # | Pillar | Meaning in practice |
|---|--------|---------------------|
| P1 | **Magic is a gesture** | Casting uses the player's hand, not a menu. A spell should feel like it was drawn, not selected. |
| P2 | **Survival is a conversation** | The five bars talk to each other. Cold makes hunger worse, dark eats sanity, low morale weakens casting. |
| P3 | **Shape is meaning** | Every object, creature, and spell is legible as a simple geometric form before any color or label. |
| P4 | **Touch first, mouse equal** | Every action works with one finger or one mouse button. Nothing requires a hover state or a keyboard. |
| P5 | **Fire is the center** | The hearth is home, the reason to explore, and the place the game is most beautiful. Everything leads back to it. |

---

## 3. Player Fantasy and Story Frame

### 3.1 Premise

The witch wakes on cold ground at the edge of a clearing. Her memory is gone. Her hands remember shapes, though, and as she draws them in the air, the world answers. Scattered ruins, shrines, and grimoire pages reveal fragments of who she was and why the land has turned to frost, ash, and fog.

### 3.2 Story Beats (Outline)

Story is delivered through found objects, not long dialogue. There is no spoken dialogue in v1.

1. **Waking** — First hearth lit. Tutorial through action: keep warm, keep fed.
2. **First Glyph** — Discover the first grimoire page. Learn Ignite.
3. **The Pale Road** — Explore outward. Find ruins with scratched markings of other witches.
4. **The Hollow** — A fog-bound zone where sanity drains fastest. Discover why.
5. **Choice** — At the end of v1, the player chooses what to do with what they've learned (see §19, Open Questions).

### 3.3 Player Fantasy

- "I am rebuilding myself from scratch, one spell at a time."
- "My fire is the only thing I can count on. Keeping it alive is my job."
- "I drew that spell, and it worked. I feel like a real witch."

---

## 4. Core Loop

### 4.1 Moment to Moment (seconds)

1. Look at the HUD: which bar is falling fastest?
2. Move (tap to walk) toward a resource, shelter, or point of interest.
3. Interact (tap) or cast (draw a glyph, pick a target).
4. Observe the result: a bar changes, a shape appears, a sound plays.

### 4.2 Session (minutes)

1. Leave the hearth at dawn with a plan: food, water, something new to discover.
2. Explore outward, gather, and cast to solve problems (cold → fire, thirst → tide, dark → lumen).
3. Return to the hearth before dusk, or risk the night.
4. Rest, eat, and reflect (review grimoire, store items).

### 4.3 Long Arc (hours)

1. Discover new spells by finding grimoire pages.
2. Unlock biomes that are dangerous without the right spell (e.g., the Hollow requires Ward).
3. Expand the hearth into a sanctuary (more stones, better shelter, shrines).
4. Reach the story ending.

---

## 5. Survival Systems

### 5.1 Overview

Five meters (bars) are shown on the HUD. Each is a number from **0 to 100**.

| Bar | Icon (shape) | Starts at | What it represents |
|-----|--------------|-----------|---------------------|
| Food | Circle | 70 | Hunger; falls over time |
| Water | Droplet (teardrop) | 70 | Thirst; falls faster than food |
| Temperature | Diamond | 50 | Body warmth; 50 is comfortable |
| Morale | Square | 60 | Will to go on; affects casting |
| Sanity | Star (5-point) | 80 | Mental clarity; affects vision and cost of magic |

Vitality (health) is not a fifth bar on the HUD. It is a hidden derived value that drains when a survival bar is in its danger zone. It is shown as a ring around the witch's icon. Full vitality regenerates slowly when all bars are safe.

### 5.2 Rules

- **FR-SURV-01** All bars are clamped to `[0, 100]`.
- **FR-SURV-02** Bars update on a fixed simulation tick (see §15), so behavior is consistent across devices.
- **FR-SURV-03** Each bar has three bands: *Safe*, *Warning*, and *Danger*. Bands are shown by color and by the bar's outline style (solid, dashed, pulsing) so color is not the only signal.
- **FR-SURV-04** When any bar is in *Danger*, Vitality drains. Multiple dangers stack.
- **FR-SURV-05** Bars influence one another (see §5.5). The game never punishes a single bar in isolation without a visible cause.

### 5.3 Individual Bars

#### Food
- Drain: −1.0 per in-game minute awake, −0.5 per minute asleep.
- Sources: forage (berries, mushrooms, roots), cooked meat, bread at the hearth.
- Warning: below 35. Danger: below 15.
- Danger effect: Vitality −2 per minute.

#### Water
- Drain: −1.5 per in-game minute awake, −0.75 asleep. Hotter weather doubles drain.
- Sources: streams, wells, rain collected in a basin, puddles conjured by *Tide*.
- Warning: below 35. Danger: below 15.
- Danger effect: Vitality −3 per minute (thirst is more dangerous than hunger).

#### Temperature
- Body temperature drifts toward the **ambient temperature** each tick. Clothing and shelter slow the drift. Fire adds warmth within its radius.
- Ambient temperature = biome base + time-of-day modifier + weather modifier.
- Comfort band: 40–60. Warning: 20–39 or 61–80. Danger: below 20 or above 80.
- Danger effect: Vitality −2 per minute. Extreme cold also lowers Morale.

#### Morale
- Starts at 60. Changes slowly and never drops or rises more than 10 points per in-game hour from a single cause.
- **Gains:** sleeping in shelter (+8 per night), eating a warm meal (+3), a lit hearth at night (+1 per minute while near it), discovering a grimoire page or shrine (+10).
- **Losses:** sleeping in the cold (−8), hunger or thirst in Danger (−1 per minute), witnessing a Hollow event (−5), casting failures (−2 each).
- **Effect:** When Morale is below 30, casting has a *fizzle chance* that rises to 25% at 0. A fizzle wastes the cost and shows a sputter. Morale does not directly reduce Vitality.

#### Sanity
- Starts at 80. The witch is fragile at the start: she does not yet remember how to be steady.
- **Drains:** standing in darkness without a light source (−2 per minute), fog zones (−1 per minute, the Hollow is worse), sleeping badly, casting while Sanity is below the spell cost (see §6.5).
- **Restores:** standing within a light source's radius (+1 per minute), daylight (+0.5 per minute outdoors), sleeping in shelter (+10 per night), the *Still* spell (+10), reading grimoire pages (+5).
- **Effect:** Below 30, the screen edges darken and the view subtly distorts (vignette and gentle wobble). Below 10, hallucinated markers appear: false glyph hints and phantom shapes. These never block input.
- **Danger (0):** the witch "unravels." Vitality drains at 4 per minute until she is restored or she faints.

### 5.4 Death and Waking

- **FR-SURV-06** When Vitality reaches 0, the witch **wakes at the last hearth** ("Waking"). This matches the premise.
- **FR-SURV-07** On Waking, carried items are kept, but the world's dropped items in the area of death are lost. Bars are reset to Safe-low values (food 40, water 40, temperature 50, morale 40, sanity 50).
- **FR-SURV-08** Waking is not a game over in v1. A run-ending mode is a possible future option (see §19).

### 5.5 Couplings (How the Bars Talk)

These interactions are the heart of P2. Each is visible in the HUD through a small linked icon.

| Cause | Effect |
|-------|--------|
| Temperature in Danger | Food and Water drain 25% faster (the body burns more) |
| Food or Water in Danger | Morale drops 1 per minute |
| Darkness without light | Sanity drains; Temperature drifts toward the night ambient faster |
| Morale below 30 | Casting may fizzle; casting takes 0.5 s longer |
| Sanity below 30 | Casting cost is paid more often in Vitality (see §6.5) |
| Hearth lit at night | Morale and Sanity recover; Temperature moves to comfort |

---

## 6. Magic and Gesture System

### 6.1 Principles

- Magic is **drawn** with one continuous stroke (v1). A spell is recognized from the shape of the stroke, not from where it begins on screen.
- The game shows what it recognized, every time, so players can learn the shapes.
- Casting is a two-step action: **draw** the glyph, then **choose a target**. Target selection is a second tap, so a misread glyph is never a wasted cast on the wrong tile.
- Failure is informative: a wrong glyph shows the closest spell and its match score.

### 6.2 Casting Flow

1. **Open the casting pad.** Tap the *Focus* button (bottom center). The world dims slightly and movement pauses.
2. **Draw.** Press and drag anywhere on the screen. A glowing trail follows the finger or pointer.
3. **Release.** The trail is recognized. The matched spell name, a match score (as a percentage), and its glyph silhouette appear for about 0.8 s.
4. **Target.** A reticle shows the valid target range (4 tiles, or self for *Ward* and *Still*). Tap a tile to cast there. Tap *Focus* or outside range to cancel. Cancelling costs nothing.
5. **Resolve.** Cost is paid, the spell's effect plays, and the pad closes.

- **FR-MAG-01** The casting pad must work with one finger and one mouse button, no hover required.
- **FR-MAG-02** Releasing with fewer than 8 sampled points or a stroke shorter than 40 px is treated as a tap and does not cast.
- **FR-MAG-03** A cast can always be cancelled before the target is chosen.

### 6.3 Gesture Recognition

The recognizer is based on the **$1 Unistroke Recognizer** approach (Wobbrock, Wilson, and Li, 2007). It is simple, fast, and works well with few example templates, which suits a small spell list.

Processing pipeline for every stroke:

1. **Resample** the stroke to 64 points spaced evenly along its length.
2. **Rotate** so the vector from the centroid to the first point points at 0° (the *indicative angle*). Rotation is then refined by a golden-section search within ±45°.
3. **Scale** the bounding box to a 250 × 250 square (non-uniform scaling is allowed, so a tall "Z" and a wide "Z" both match).
4. **Translate** the centroid to the origin.
5. **Compare** against each spell template, processed through the same pipeline. Use the average point-to-point distance.
6. **Score:** `score = 1 − distance / halfDiagonal`, where `halfDiagonal = 0.5 × √(250² + 250²)`. The best template wins if `score ≥ threshold`.

- **FR-MAG-04** The default acceptance threshold is **0.80**. Forgiving mode lowers it to 0.70 (see §16). A stricter setting may be added later.
- **FR-MAG-05** Each spell has 1–3 templates. Templates are stored as JSON, one array of `{x, y}` points per template, in `src/magic/templates/`.
- **FR-MAG-06** Recognition runs on the simulation side of the game loop, never blocks input, and completes in under 2 ms for a 64-point stroke against the full template set.
- **FR-MAG-07** Multi-stroke glyphs are out of scope for v1. The design leaves room for them through a `strokes[]` data shape in the recognizer input.

### 6.4 Teaching the Gestures

- The **Grimoire** (see §12) shows each known spell's animated glyph, drawn at the speed a player should trace.
- A **Practice** page lets the player draw freely and see the score and nearest spell, without spending resources.
- A spell is **unlocked** when its grimoire page is found. Spells can be re-learned at any hearth by reading the page again.

### 6.5 Cost of Magic

Casting costs **Sanity**. This creates tension between thinking clearly and surviving.

- If Sanity is at or above the spell's cost, the cost is paid from Sanity.
- If Sanity is below the cost, the remainder is paid from **Vitality** ("overcasting"). The HUD shows this as a red flash on the Vitality ring before the cast resolves, so the player can cancel.

### 6.6 Failure and Fizzles

- A recognized glyph below threshold is a **Misread**. The closest spell and the score are shown. Nothing is spent.
- A recognized glyph above threshold can still **Fizzle** when Morale is below 30 (see §5.3). A fizzle spends half the cost and plays a sputter. It is never silent.

---

## 7. Spells

### 7.1 Spell Table (v1)

| Spell | Glyph (stroke shape) | Target | Cost (Sanity) | Effect |
|-------|----------------------|--------|---------------|--------|
| **Ignite** | Zigzag (a "Z" or lightning bolt, top-left to bottom-right) | Tile within 4 | 4 | Lights fuel (wood, dry grass) or creates a small fire on bare ground. Fires warm within 4 tiles and burn for 3 in-game minutes. |
| **Lumen** | Circle (counter-clockwise) | Tile within 4 | 2 | Conjures a floating light orb that lasts 6 in-game minutes, lights a radius of 5 tiles, and restores Sanity while in range. Orbs can be carried if the witch stands still for 2 s. |
| **Bloom** | Spiral (inward) | Tile within 4 (soil only) | 5 | Grows a plant from a seed on soil, or accelerates a sapling by one stage. Stages: sprout → bush → fruit. Fruit gives Food. |
| **Tide** | Wave (two humps, horizontal) | Tile within 4 | 3 | Conjures a water puddle (holds 25 water, evaporates in 4 in-game minutes). On a stream or well, it is a free refill of 10. |
| **Ward** | Triangle (any winding) | Self | 6 | Creates a 3-tile barrier around the witch for 20 seconds. Blocks Hollow fog and reduces weather damage by half. Required to enter the Hollow safely. |
| **Still** | Straight horizontal line | Self | 0 | Restores 10 Sanity and clears hallucinated markers for 30 seconds. Costs Morale 1 on cast so it cannot be spammed. |

- **FR-SPELL-01** Every spell has a single, unique glyph family in v1 (1–3 templates). Glyphs must be mutually distinct: each spell's templates must score below 0.70 against every other spell's templates, so no glyph is mistaken for another.
- **FR-SPELL-02** Spell data (name, glyph, target rule, cost, effect parameters) lives in data files (`src/magic/spells.json`), not hard-coded in logic.

### 7.2 Spell Discovery

| Spell | Found in |
|-------|----------|
| Still | Starting hearth (given at waking) |
| Ignite | First grimoire page, Waking clearing |
| Lumen | Pale Road ruin |
| Tide | Marsh shrine |
| Bloom | Frostmere orchard, ruin inner room |
| Ward | Sealed shrine at the Hollow's edge, outside the fog. Its seal is opened by a relic from the Frostmere ruins, so Ward is always reachable before the Hollow. |

### 7.3 Future Spells (Not in v1)

Ideas kept for later: *Frost* (cool a tile), *Gust* (move fog), *Mend* (repair a hearth), *Tether* (link two lights), multi-stroke glyphs for combined spells.

---

## 8. Exploration and World

### 8.1 Structure

- The world is a **tile grid** (32 × 32 px per tile at base zoom). The world is generated in **chunks** of 16 × 16 tiles.
- Generation is **seeded and deterministic**: the same seed produces the same world on every device.
- The world is large enough for roughly 20–40 minutes of exploration before the player has seen most of it. Exact size is open (see §19).

### 8.2 Biomes

| Biome | Shape language | Ambient temperature | Notes |
|-------|----------------|---------------------|-------|
| **Clearing** (start) | Circles and small squares; soft greens | Mild | Starting hearth, first forage |
| **Pine Wood** | Tall triangles for trees, dark green | Cool | Plentiful wood, mushrooms; shelter from wind |
| **Marsh** | Overlapping ovals, teal | Cool, damp | Water everywhere; reeds; slow movement |
| **Ash Plain** | Grey broken shapes, gaps | Warm (oddly) | Burned land; little food, lots of coal |
| **Frostmere** | Pale blue crystal polygons | Freezing | Needs warmth; rich in ruins and bloom sites |
| **Hollow** | Concentric rings, fog | Mild, but Sanity drains | Locked by fog until Ward is learned |

### 8.3 Points of Interest (POI)

- **Hearth:** the player's save point and base. Fire-lit, crafting station, Waking point.
- **Ruins:** contain grimoire pages and crafting materials. Some have sealed rooms.
- **Shrines:** each restores Morale and Sanity once per day. Some hold spell pages.
- **Forage Patches:** food and herbs that regrow over several in-game days.
- **Wells and Streams:** refill water. Wells need a bucket made from wood and clay.
- **Hollow Markers:** strange shapes that appear only in fog; they are the lore's clues.

- **FR-WORLD-01** Each biome has at least one of every resource type it can produce (food, water, fuel, material).
- **FR-WORLD-02** Every POI is reachable without any spell that is locked behind a later POI (no soft-locks).

### 8.4 Movement

- Tap a tile to walk there. The witch paths around solid tiles using A* (4-direction, with a path cap of 200 steps).
- Tap a POI or entity to walk to it and interact on arrival.
- Movement speed: 3 tiles per second on flat ground; halved in marsh and deep snow.

---

## 9. Day, Night, and Weather

### 9.1 Time

- A **day** lasts **10 real minutes** by default (tunable). The day is split into dawn, day, dusk, and night.
- Night is the main threat window. Light sources and the hearth matter most then.
- Time is visible as a sun-and-moon arc on the HUD and as a light tint on the world.

### 9.2 Weather

| Weather | Effect |
|---------|--------|
| **Clear** | Neutral. Daylight restores Sanity. |
| **Rain** | Water refills faster from a basin. Temperature −5 ambient. Fires burn at half strength in open ground. |
| **Fog** | Visibility drops. Sanity −1/min outdoors. Hollow markers appear more often. |
| **Frost** | Temperature −15 ambient. Plants stop growing. |
| **Storm** | Rain and wind. Ignite fails outdoors unless a shelter covers the target. |

- **FR-WEATHER-01** Weather changes on a schedule seeded by the world seed, so it is predictable enough to plan around, but still surprising.
- **FR-WEATHER-02** The HUD shows the next weather change 60 seconds before it happens.

---

## 10. Crafting and Building

Crafting is deliberately small in v1. It supports survival, not inventory management.

### 10.1 Resources

| Resource | Found in | Used for |
|----------|----------|----------|
| Wood | Pine Wood, Clearing | Fuel, shelter frames |
| Stone | Frostmere, Ruins | Hearth upgrades, basins |
| Fiber | Clearing, Marsh (reeds) | Rope, bandages |
| Clay | Marsh banks | Buckets, pots |
| Herbs | Forage patches | Tea (Morale), salve (Vitality) |
| Coal | Ash Plain | Long-burning fuel |

### 10.2 Recipes (v1)

| Item | Ingredients | Use |
|------|-------------|-----|
| Bucket | 2 Clay, 1 Fiber | Carries water; required for wells |
| Basin | 3 Stone | Collects rain |
| Rope | 3 Fiber | Required for some ruins |
| Tent | 4 Wood, 2 Fiber | Portable shelter; +Warmth, safer sleep |
| Hearth Upgrade | 6 Stone | Doubles fire radius; adds a second cooking spot |
| Tea | Herbs, Water | +5 Morale |
| Salve | Herbs, Fat | +10 Vitality over 10 seconds |

- **FR-CRAFT-01** Crafting happens only at the hearth or with a carried tent (for the tent, not the upgrades).
- **FR-CRAFT-02** The inventory has 12 slots. Carried items are lost only on Waking if dropped (see FR-SURV-07).

---

## 11. Controls: Mouse and Touch

**Every action has a one-pointer path.** There are no keyboard requirements. Keyboard shortcuts may be added for desktop convenience only.

| Action | Mouse | Touch |
|--------|-------|-------|
| Walk to tile | Click | Tap |
| Interact with POI or entity | Click on it | Tap on it |
| Open casting pad | Click *Focus* | Tap *Focus* |
| Draw a glyph | Press and drag, then release | One finger drag, then lift |
| Choose a spell target | Click tile | Tap tile |
| Cancel casting | Click *Focus* or outside range | Tap *Focus* or outside range |
| Open inventory | Click the bag icon | Tap the bag icon |
| Open grimoire | Click the book icon | Tap the book icon |
| Pan the camera (manual) | Right-drag or middle-drag | Two-finger drag |
| Zoom | Mouse wheel | Pinch |

- **FR-CTRL-01** Input is handled through a single pointer abstraction using the Pointer Events API, so mouse, pen, and touch share one code path.
- **FR-CTRL-02** Tap and click are distinguished by a 10 px movement tolerance and a 350 ms time limit. Longer holds are treated as a drag.
- **FR-CTRL-03** The camera follows the witch with a soft ease. Manual panning disables follow for 5 seconds.
- **FR-CTRL-04** Touch targets are at least 44 × 44 CSS px. The *Focus* button sits in the thumb zone on portrait phones.
- **FR-CTRL-05** Long-press on an entity shows its name and info. There is no hover-only information anywhere.

---

## 12. User Interface

### 12.1 HUD (Always Visible)

- **Five bars** in a cluster at the top-left, each with its shape icon, its value, and its band outline.
- **Vitality ring** around the witch's icon at the bottom-left or on the map.
- **Time and weather** at the top-right: sun-and-moon arc and the next weather icon.
- **Focus button** at the bottom-center. Large, obvious, and always reachable.
- **Bag and grimoire buttons** at the bottom-right.
- **Coupling hints** (§5.5): a small linked icon appears when two bars are affecting each other.

### 12.2 Screens

| Screen | Purpose |
|--------|---------|
| **Hearth** | Rest, craft, save, view the day's summary |
| **Inventory** | 12-slot grid; drag or tap to move, tap to use |
| **Grimoire** | Known spells with animated glyphs; Practice pad; lore pages |
| **Map** | Discovered tiles; POIs; hearth marker; weather front |
| **Settings** | Audio, forgiving mode, reduced motion, text size, reset save |

### 12.3 Feedback

- Every change to a bar shows a short floating shape (down = triangle, up = circle) near the bar.
- Every spell plays a distinct visual: the glyph traced in light, then a burst in the spell's color and shape.
- Every event that could be missed (weather change, bar entering Danger) has a visual *and* a sound cue.

---

## 13. Art Direction

### 13.1 Principles

- **Simple geometric shapes** only: circles, triangles, squares, polygons, rings, and lines. No textures.
- **Shape before color.** Each entity must be recognizable as a silhouette.
- **Light is the star.** The world is dark and muted; light sources bloom with a radial gradient.

### 13.2 Shape Language

| Element | Shape |
|---------|-------|
| Witch | Triangle hat, circle head, rectangle robe, small triangle staff |
| Trees | Stacked triangles (pine), circle clusters (clearing) |
| Rocks | Irregular convex polygons |
| Animals | Circles with small triangles for ears, or rectangles with a circle head |
| Water | Ovals and horizontal wave lines |
| Fire | Layered triangles, animated jitter |
| Light orbs | Concentric circles with soft glow |
| Hollow | Concentric rings, dashed outlines |

### 13.3 Palette (Working)

Colors are defined as tokens. Light and dark variants are required for the UI.

| Token | Hex | Use |
|-------|-----|-----|
| `--night-deep` | `#0d0f1f` | Background at night |
| `--night-mid` | `#1c2140` | Ground at night |
| `--day-ground` | `#3b5a3e` | Ground in daylight |
| `--ember` | `#ff9a3c` | Fire, warmth, Temperature |
| `--lumen` | `#ffe9a6` | Light orbs |
| `--tide` | `#4fc3c7` | Water, Water bar |
| `--bloom` | `#8bd17c` | Growth, Food bar |
| `--violet` | `#9d7bff` | Magic, Sanity, glyph trails |
| `--ash` | `#8a8a94` | Ash Plain, neutral UI |
| `--frost` | `#b9e4ff` | Frostmere, Frost |
| `--alarm` | `#ff5d6c` | Danger band, overcasting |

- **FR-ART-01** Bar colors must be paired with an outline style (solid, dashed, pulsing) so the state is readable without color.
- **FR-ART-02** Palette must pass WCAG AA contrast for all text on its background. Verify during UI work.

---

## 14. Audio

Audio is supportive, not required. The game must be fully playable with sound off.

- **Ambient:** wind, water, crackle of fire. Layers change with biome and weather.
- **Cues:** a soft chime for a grimoire page; a low tone for a bar entering Danger; a rising tone for a successful cast; a dull thud for a fizzle.
- **Glyph sound:** each stroke plays a pitch that rises with the stroke's speed.
- **Mixing:** separate volume sliders for music, ambience, and effects in Settings.
- **FR-AUDIO-01** Audio starts only after a user gesture (browser autoplay policy).

---

## 15. Technical Design

### 15.1 Platform and Stack

- **Language:** JavaScript (ES modules). TypeScript is optional and can be added later.
- **Rendering:** HTML5 Canvas 2D. Shapes are drawn with paths; no image assets are required.
- **Build:** None in v1. Files are served as static assets, so the game runs on GitHub Pages or any static host.
- **Storage:** `localStorage` for the save file, with a version number. Saves are written at the hearth and on Waking. Fall back gracefully if storage is unavailable.
- **Dependencies:** none required. Any library must be justified in a PR description.

### 15.2 Architecture

```
src/
  main.js              Entry point; boots the game and the loop
  core/
    loop.js            Fixed-timestep update, variable-rate render
    input.js           Pointer abstraction (mouse, pen, touch)
    camera.js          Follow, pan, zoom
    rng.js             Seeded random number generator
    save.js            Versioned save and load
  world/
    chunk.js           Chunk generation and storage
    biomes.js          Biome definitions
    pathfind.js        A* on the tile grid
    poi.js             Points of interest
  survival/
    stats.js           Five bars, couplings, Vitality
    weather.js         Weather schedule and effects
    time.js            Day and night
  magic/
    recognizer.js      $1 unistroke recognizer
    casting.js         Casting flow state machine
    spells.json        Spell definitions
    templates/         Glyph template point arrays
  render/
    draw.js            Shape primitives
    lighting.js        Light radii, darkness overlay
    hud.js             HUD layout and bars
  ui/
    grimoire.js
    inventory.js
    map.js
    settings.js
```

### 15.3 Simulation and Rendering

- **Fixed timestep:** the simulation advances at **20 ticks per second**. Rendering runs at the display rate.
- **Determinism:** all randomness goes through `rng.js` with the world seed. This keeps tests stable and makes bugs reproducible.
- **State:** a single game state object. Systems read and write it in a defined order: input → casting → world → survival → render.

### 15.4 Performance Targets

- 60 FPS on a mid-range phone (2021-era) in a standard mobile browser.
- Under 16 ms per frame for update and render combined on that device.
- Only chunks near the camera are kept in memory. Off-screen chunks are dropped after 30 seconds.
- Lighting is drawn once per frame into an offscreen canvas at half resolution.

### 15.5 Testing

- **Unit tests** for the recognizer (known gesture pairs), the survival couplings, and the save format.
- **Golden tests** for world generation: fixed seed → known tile hash.
- **Manual device checklist** for touch, including one-handed play on a phone.

---

## 16. Accessibility

- **FR-A11Y-01** Every interaction works with one pointer. No drag-only, hover-only, or multi-finger-only requirement, except for the optional two-finger camera pan.
- **FR-A11Y-02** *Forgiving mode* lowers the recognition threshold to 0.70 and gives a longer target window.
- **FR-A11Y-03** *Reduced motion* removes screen wobble, pulsing, and the Sanity distortion. Warnings stay as static outlines.
- **FR-A11Y-04** Bars use shape and outline as well as color (see FR-ART-01).
- **FR-A11Y-05** Text can be scaled from 100% to 150% in Settings. The HUD reflows.
- **FR-A11Y-06** Sound is never the only cue for a danger state.
- **FR-A11Y-07** The game never flashes more than three times per second.

---

## 17. Milestones and Acceptance Criteria

Each milestone is built and verified before the next starts. Each has checkable criteria so Claude Code and humans can confirm completion.

### M0 — Foundations
- Canvas renders at device pixel ratio with a fixed-timestep loop.
- Pointer abstraction distinguishes tap, click, drag, and long-press on mouse and touch.
- Camera follows the witch, pans, and zooms.
- **Done when:** a tap moves the witch on desktop Chrome and on a phone emulator, with no console errors.

### M1 — Gesture Casting
- $1 recognizer implemented with the pipeline in §6.3.
- Casting pad with draw → recognize → target → resolve flow (§6.2).
- Practice pad in the Grimoire shows the matched spell and score.
- Templates for all six v1 spells, each with a match score ≥ 0.85 against itself and < 0.70 against the others.
- **Done when:** unit tests for the recognizer pass, and each v1 glyph is recognized 9 times out of 10 in manual testing by someone other than its author.

### M2 — Survival Core
- Five bars update per §5. Bands and outlines render.
- Couplings from §5.5 are active.
- Vitality drain, Waking, and last-hearth save work.
- **Done when:** a player can survive one full day cycle with a plan, and fails predictably when they ignore a bar.

### M3 — World
- Seeded chunk generation for all six biomes.
- A* pathing, POIs, forage patches, and wells.
- Golden test for the seed `witchroam-0001` produces a stable tile hash.
- **Done when:** the world has no soft-locks (FR-WORLD-02 verified by a reachability test).

### M4 — Spells in the World
- All six spells work with their effects (§7.1).
- Light orbs, fires, puddles, and plants persist as world state.
- **Done when:** each spell has an in-world use the player can discover without reading documentation.

### M5 — Day, Night, Weather
- Day-night cycle, sun-and-moon HUD, light tint.
- Five weather types with the effects in §9.2.
- **Done when:** a night without a light source measurably drains Sanity, and a lit hearth measurably recovers Morale.

### M6 — Content and Story
- Grimoire pages for all v1 spells and a lore arc of at least 8 pages.
- Shrines, ruins, and the Hollow with Hollow Markers.
- **Done when:** the story beats in §3.2 can be reached in order, and each has an in-world clue.

### M7 — Polish
- Audio layers and cues (§14).
- Accessibility options (§16) all implemented and tested.
- Performance targets (§15.4) met on the reference device.
- **Done when:** the game is shareable at a stable URL and passes the manual device checklist.

---

## 18. Working With This Document in Claude Code

This document is written to be read by Claude Code and by people. To get the best results:

- **Reference IDs.** Ask for work by ID, for example "implement FR-MAG-03 and FR-MAG-02 in `src/magic/casting.js`."
- **Work by milestone.** Pick the current milestone in §17 and build only what it lists.
- **Keep tuning in data.** Numbers in this doc are starting values. Put them in `spells.json` and survival config files, and update this doc when a value is changed on purpose.
- **Update the doc when design changes.** A decision that changes a rule belongs here, with the same ID, in the same commit as the code change.
- **Record decisions.** Resolved questions move from §19 into the relevant section, with a one-line note of the decision and its date.

See also `CLAUDE.md` at the repository root for repo-level conventions.

---

## 19. Open Questions

These need a decision before the milestone that depends on them.

| # | Question | Needed by | Recommendation |
|---|----------|-----------|----------------|
| Q1 | Final game name? | M0 | **Resolved 2026-10-31:** renamed to *Witchroam* (formerly *Hearthlight*) |
| Q2 | Is death permanent (roguelite) or always Waking (cozy)? | M2 | Waking only in v1; revisit after playtests |
| Q3 | Single-stroke glyphs only, or multi-stroke from v1? | M1 | Single-stroke for v1 (see FR-MAG-07) |
| Q4 | World size: fixed map or endless chunks? | M3 | Fixed large map with a seeded layout, so the story can be placed |
| Q5 | What is the story ending? (Restore the land, delve into the Hollow, or leave) | M6 | Decide after the M3 playtest; plan for two endings |
| Q6 | Should Sanity cost of casting scale with Morale? | M1 | No for v1; keep the two couplings separate so each is easy to read |
| Q7 | Save slots, or one save only? | M2 | One save in v1 |
| Q8 | Is tap-to-move enough, or add a virtual joystick for fast play? | M0 | Tap-to-move in v1; test joystick as an option in M7 |
| Q9 | Does the day length (10 min) feel right in playtests? | M5 | Tune in M5 |

---

## 20. Glossary

| Term | Meaning |
|------|---------|
| **Bar** | One of the five survival meters: Food, Water, Temperature, Morale, Sanity |
| **Band** | The state of a bar: Safe, Warning, or Danger |
| **Casting pad** | The drawing surface opened by *Focus*; recognizes glyphs |
| **Coupling** | A rule where one bar changes another |
| **Fizzle** | A correctly drawn spell that fails because Morale is low |
| **Focus** | The button that opens the casting pad |
| **Glyph** | The drawn stroke that identifies a spell |
| **Grimoire** | The witch's spellbook, with spell glyphs, Practice, and lore |
| **Hearth** | The player's base and save point |
| **Hollow** | A fog-bound zone where Sanity drains fastest |
| **Misread** | A glyph below the recognition threshold |
| **Overcasting** | Paying a spell's cost from Vitality when Sanity is too low |
| **POI** | Point of interest: ruin, shrine, forage patch, well, or marker |
| **Vitality** | Hidden health value, drained by Danger bands |
| **Waking** | Respawn at the last hearth after Vitality reaches 0 |

---

*End of document.*
