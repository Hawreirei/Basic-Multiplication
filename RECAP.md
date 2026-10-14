# Times Table Rocket Race: Recap & Blueprint

This document covers what the app is, how it works, and how to reuse the same design for other learning materials (addition, fractions, vocabulary, capitals, science facts, and so on).

> **Sibling apps built from this blueprint:** [Percent Rocket Race](percentages/RECAP.md) (`percentages/index.html`), with fractions → % and % of an amount.

---

## 1. One-paragraph summary

**Times Table Rocket Race** is a single-file browser game (`index.html`, about 900 lines with no build step and no dependencies) where **1–4 players race side by side on one screen** answering multiple-choice multiplication questions (1×1 to 12×12) against a timer (30/60/90 s). Each correct answer moves that player's rocket up their track. Streaks give a score multiplier, fast answers give a speed bonus, and random "golden" questions are worth ×3. Every 5 correct answers charges the player's **hero power**. Some powers help yourself and some attack the leading player. At the end there is a **results dashboard** with a scoreboard, a 12×12 **mastery heatmap** of which facts each player got right or wrong, and a **local leaderboard** saved in the browser.

---

## 2. Tech & constraints (keep these for every sibling app)

| Decision | Why |
|---|---|
| **One `index.html`** (inline CSS + JS, IIFE, `"use strict"`) | Open by double-click, host anywhere (GitHub Pages, USB stick, classroom PC). |
| **No libraries, no images, no audio files** | Emoji are the graphics. Web Audio `OscillatorNode` beeps are the sound. Canvas draws the stars and confetti. |
| **`localStorage` only** (wrapped in try/catch) | Settings, personal best and leaderboard persist on that device. It still works in private mode, just without saving. |
| **Everything fits one screen, no page scroll** | Uses `clamp()`, `vh`/`vw`, flex/grid with `min-height:0`, and container queries (`cqw/cqh`) for the heatmap. |
| **Shared-device multiplayer** | Tap/touch (`pointerdown`) on a tablet or interactive whiteboard, or keyboard zones per player on a laptop. |
| **Kid-friendly** | Big buttons, encouraging copy ("you'll get it next time!"), no penalties below 0, `prefers-reduced-motion` respected. |

---

## 3. Screen flow

```
SETUP ──START──▶ COUNTDOWN (3‑2‑1‑GO) ──▶ PLAY (timer) ──time up──▶ RESULTS
  ▲                                        │  Esc / blur = PAUSE        │
  └──────────── "Change Players" ◀─────────┴──── "Play Again" (Enter) ──┘
```

`phase` variable: `"setup" | "countdown" | "play" | "results"`. Screens are `<section class="screen">` and `show(id)` toggles `.on`.

### 3.1 Setup screen (single card, no scrolling)
- Title, tagline, and 3 "how to play" step chips.
- **Players:** 1 / 2 / 3 / 4 buttons.
- **Round length:** 30 / 60 / 90 seconds.
- **Per-player card** (colored border per player): name input and a hero picker (6 emoji buttons) with a description of the selected hero.
  - Tapping a name opens that player's **own on-screen keyboard** (QWERTY + shift, backspace, clear, space, done). Several can be open at once so all kids can type simultaneously. Names are capped at 12 characters.
  - With 1 player, attack heroes are disabled and a note explains why.
- START button, Leaderboard button, and a keyboard-controls hint line.

### 3.2 Game screen
- Top bar: Pause, a shrinking gradient **timer bar** (pulses/outlined at ≤10 s, ticks at ≤5 s), and Mute.
- **Arena:** one panel per player in a CSS grid (`repeat(n, 1fr)`). Each panel has:
  - name + hero icon, big score (bumps on gain)
  - hero-power button with a fill bar ("❄ FREEZE · 3/5" → glowing "FREEZE! [key]")
  - streak counter + multiplier pill (×2, ×3, ×4)
  - a vertical **rocket track** (rocket height = score relative to the leader, and a 👑 crown on the leader)
  - **question box** (golden style, "fog" blur, 3-factor smaller font)
  - **4 answer buttons** in a 2×2 grid, each showing its keyboard key
  - a feedback line ("⚡ Lightning! +30", "6 × 7 = 42 — you'll get it next time!")
  - status badges (⚡×2 (3), 🦉 easy 2, 🛡️) and a FROZEN overlay

### 3.3 Results dashboard (3 cards, fits screen)
1. **Scoreboard:** ranked rows with medal, avatar, name, rank title (Cadet → Pilot → Captain → Commander → Galaxy Legend, normalized per minute), score, animated bar, and stat pills (✔ correct, 🎯 accuracy %, 🔥 best streak, ⚡ avg time, ✨ powers used, ⭐ NEW BEST).
2. **Mastery heatmap:** a 13×13 grid (axes 1–12). Green = always right, yellow = right and wrong, red = only wrong, grey = not seen. Below it: "X of 78 facts tried" plus a **"Practice: 7×8, 6×9…"** list. Click a player row or chip to switch whose map is shown.
3. **Leaderboard:** top 10 per round length (tabs 30s/60s/90s), the current game's entries highlighted, with dates.
- Buttons: Play Again / Change Players. Confetti + victory jingle.
- Responsive: under 1000 px the heatmap is hidden, and under 760 px the cards stack.

---

## 4. Game rules & numbers (tunable constants)

| Rule | Value |
|---|---|
| Base points per correct answer | **10** |
| Speed bonus | **+5** if < 2 s, **+2** if < 4 s |
| Streak multiplier | `min(4, 1 + floor(streak/5))`, so ×2 at 5, ×3 at 10, ×4 at 15 |
| Golden question | needs streak ≥ 3, 18% chance, 4 s global cooldown. **×3 points**, counts 2 toward the power charge |
| Wrong answer | streak resets (unless shielded), correct answer revealed, 0.9 s pause, no point loss |
| Next question delay | 140 ms after correct, 900 ms after wrong |
| Power charge | **5** correct answers (`CHARGE_NEED`) |

Formula: `gain = (10 + speed) × multiplier × (golden ? 3 : 1) × (turbo ? 2 : 1)`

### Heroes (`CHARS` array)
| Hero | Power | Type | Effect |
|---|---|---|---|
| 🤖 Turbo Bot | TURBO | self | Next 5 correct answers score ×2 |
| 🦉 Wise Owl | EASY | self | Next 4 questions use small numbers (≤5) |
| 🛡️ Guardian | SHIELD | self | Blocks the next attack **or** saves your streak once |
| 🧙 Ice Wizard | FREEZE | attack | Leader can't answer for 3 s |
| 👽 Fog Alien | FOG | attack | Leader's question blurred for 3 s |
| 🦹 Sneaky Thief | STEAL | attack | Takes up to 40 points from the leader |

Attacks always target the **highest-scoring opponent** (random among ties). This rubber-band mechanic keeps games close.

---

## 5. Learning design (the part to reproduce carefully)

1. **Gentle difficulty ramp:** the max factor starts at 5 and grows by 1 every 3 correct answers up to 12. After factor 8 is unlocked, 45% of questions force one "hard" factor (6–12). Questions with ×1 are mostly avoided.
2. **Spaced retry of mistakes:** wrong facts go into `p.missed`, and each new question has a 30% chance of re-asking one.
3. **No immediate repeats:** the same fact (order-independent key, e.g. `"6x7"`) is never asked twice in a row.
4. **Smart distractors** (wrong options come from common mistakes, not random numbers):
   - neighbour facts (change one factor by ±1 or +2: 6×8, 7×7 for 6×7)
   - ±1, ±2, ±10 of the answer, answer ± a factor
   - digit swap (42 → 24)
   - fallback `ans + 3k` until there are 4 unique positive options
5. **Variety:** after 6 correct answers there is a 15% chance of a 3-factor question (each 2–5, product ≤ 100).
6. **Immediate, kind feedback:** the correct answer is always shown after a mistake.
7. **Mastery tracking per fact:** `p.hist[key] = {r, w}` drives the heatmap and the "Practice:" list.

---

## 6. Controls

| Player | Answer keys (4 options) | Power key |
|---|---|---|
| P1 | `1 2 3 4` | `5` |
| P2 | `7 8 9 0` | `-` |
| P3 | `Z X C V` | `A` |
| P4 | `B N M ,` | `.` |

Global keys: `Esc` = pause, `Enter` = start / play again. The game auto-pauses when the window loses focus. Touch uses `pointerdown` with `preventDefault()` so multi-touch works on whiteboards and tablets.

---

## 7. Persistence (`localStorage` keys)

| Key | Content |
|---|---|
| `ttrr` | settings: `{players, length, sound, names[4], chars[4]}` |
| `ttrr_leaderboard` | array of `{id, n:name, i:icon, s:score, p:playerCount, l:length, t:time}`, keeping the best 50 per round length |
| `best_{N}p_{len}` | personal best per player-count + length (drives the "NEW BEST" pill) |

> For a sibling app, **change the key prefix** (e.g. `addr_`, `vocab_`) so apps on the same domain don't share data.

---

## 8. Visual design system

- **Theme:** dark space background (radial gradients + animated canvas stars). Glassy cards (`#ffffff18` → `#ffffff0a`, 1px `#ffffff2a` border, radius 22–28 px).
- **Tokens:** `--good #3ddc84`, `--bad #ff5d73`, `--gold #ffd43b`, `--muted #a9b0d9`, `--text #f4f6ff`.
- **Player colors:** red `#ff5d73`, blue `#4dabf7`, yellow `#ffd43b`, green `#51cf66`. Each panel or card sets `--c` and children use `color-mix()` for glows.
- **Big CTA button:** orange→pink gradient pill with a 3D bottom shadow that presses down on `:active`.
- **Motion:** bump (score), shake (wrong), glow (golden), float-up (+points), pop (countdown), rise (results rows), and a banner for power announcements. All of it is disabled under `prefers-reduced-motion`.
- **Font:** `"Trebuchet MS", "Segoe UI", system-ui`. Numbers use `font-variant-numeric: tabular-nums`.

---

## 9. Code map: generic vs. subject-specific

This is the key part for cloning. Most of the file is a **reusable game engine**. Only a few parts know about multiplication.

| Section in `index.html` | Reuse as-is? | Notes |
|---|---|---|
| CSS tokens, layout, panels, results, keyboard | ✅ | Restyle the theme only if wanted |
| `CONFIG` (colors, keysets, lengths, `CHARS`) | ✅ (tweak) | Rename heroes and powers to fit the theme |
| `store`, `settings`, audio `sfx`, stars, confetti | ✅ | Change the storage key prefix |
| On-screen keyboard (`openKb`/`closeKb`…) | ✅ | |
| Setup screen `buildSetup()` | ✅ | Change title, tagline and step text |
| **`makeQuestion(p)`** | ❌ **replace** | The content engine (see the contract below) |
| `buildArena`, `answer`, `activate`, timer, flow | ✅ | Works with any `{txt, ans, opts}` |
| Owl "EASY" power | ⚠️ adapt | Needs an "easy" mode in your question generator |
| Fog power / `.qbox.tri` style | ✅ / ⚠️ | `tri` = long question → smaller font |
| **Mastery heatmap** in `buildResults` | ❌ **replace** | 12×12 grid fits times tables only (see options below) |
| `titleFor(score)` thresholds + rank names | ⚠️ retune | Score per minute depends on how hard the content is |
| Leaderboard | ✅ | |

### The content "plug-in" contract

Everything else works as long as the question generator returns:

```js
// makeQuestion(p) must return:
{
  txt:    "6 × 7",          // what to show in the question box (string)
  ans:    42,               // correct answer (number or string)
  opts:   [42, 48, 36, 24], // exactly 4 unique options, shuffled, containing ans
  key:    "6x7",            // stable ID of the "fact" (for mastery + no-repeat + missed retry)
  golden: false,            // keep the existing golden logic
  f:      [6, 7]            // (multiplication only) used for 3-factor styling; replace with e.g. `long: true`
}
```

It may read from the player object `p`:
- `p.correct`: for the difficulty ramp
- `p.missed`: re-ask mistakes (store **keys** or item objects)
- `p.last`: avoid an immediate repeat
- `p.fx.easy > 0`: Owl power is active, so give an easy item

And `answer()` records `p.hist[key] = {r, w}`. The mastery view only needs to turn `hist` into a picture.

### Mastery view options for other materials
| Material | Mastery visual |
|---|---|
| Addition/subtraction 0–10 or 0–20 | Same N×N heatmap (just change the size) |
| Division | 12×12 grid indexed by divisor × quotient |
| Number bonds / doubles / squares | A single row/strip of cells |
| Vocabulary, capitals, symbols, dates | **List of item chips** colored green/yellow/red, sorted by mistakes, or a grid of N cards grouped by category |
| Fractions / decimals | Grid by denominator × numerator |

---

## 10. How to build a sibling app (step by step)

1. **Copy** `index.html` into a new repo or folder.
2. **Rename:** `<title>`, `<h1>`, tagline, step chips, storage keys (`ttrr*` → `xyz*`), and the `rsub` "times-table diagram" tooltip.
3. **Replace the content engine:** write `makeQuestion(p)` for the new material, following the contract above. For fact-list subjects, define a data array such as:
   ```js
   const ITEMS = [
     {key:"fr",  q:"Capital of France?", a:"Paris",  level:1, wrong:["Lyon","Rome","Berlin"]},
     {key:"jp",  q:"Capital of Japan?",  a:"Tokyo",  level:1, wrong:["Kyoto","Seoul","Osaka"]},
     // ...
   ];
   ```
   Then pick by `level ≤ 1 + floor(p.correct/3)`, use 30% retry from `p.missed`, no repeat of `p.last`, and options = answer + 3 distractors. Hand-written distractors are best. Otherwise pull same-category answers from other items.
4. **Adapt the Owl power** so "easy" means level-1 items only.
5. **Replace the mastery card** with a visual that fits (see the table above), and update the "X of 78 facts tried" count to `ITEMS.length`.
6. **Retune** `titleFor()` thresholds and rank names to fit the theme (e.g. Word Explorer → Wordsmith → Lexicon Legend).
7. **Check the font size** of long questions or answers: add a class like `.qbox.long` / `.ans.long` with a smaller `clamp()`.
8. **Optional reskin:** change the theme (ocean 🐠, jungle 🐒, racing 🏎️). Swap the `ROCKETS` emoji, background gradients and hero names, and keep the mechanics.
9. **Test:** 1 player and 4 players, a phone in portrait, a laptop with the keyboard, a tablet with multi-touch, a round that ends mid-answer, and a long player name.

---

## 11. Ready-to-paste prompt for building a sibling app

> Build a single-file `index.html` educational game (no dependencies, inline CSS/JS, emoji graphics, Web Audio beeps, localStorage) called **"[NAME]"** for practicing **[MATERIAL, e.g. addition within 20 / English–Indonesian vocabulary / world capitals]** for **[AGE GROUP]**.
>
> Copy the design and mechanics of the existing *Times Table Rocket Race* (see `RECAP.md`):
> - 1–4 players race side by side on one shared screen. Each has their own panel with a vertical progress track, a question, 4 multiple-choice answers (tap, or keyboard zones `1234/5`, `7890/-`, `ZXCV/A`, `BNM,/.`), score, streak multiplier (×2/×3/×4 every 5), speed bonus (+5 under 2 s, +2 under 4 s), and golden ×3 questions.
> - Round length 30/60/90 s, 3-2-1-GO countdown, pause on Esc or window blur, mute button.
> - Six heroes with a power charged by 5 correct answers: Turbo (next 5 ×2), Owl (next 4 easy), Shield (block an attack or save a streak), Freeze, Fog, Steal (attacks hit the current leader, disabled in 1-player mode).
> - Setup screen fits one screen without scrolling: player count, round length, per-player name with its own on-screen keyboard (several open at once), and a hero picker.
> - Adaptive questions: start easy and unlock harder items every 3 correct answers, re-ask missed items 30% of the time, never repeat the same item twice in a row, and use smart distractors based on common mistakes.
> - Results dashboard that fits the screen: scoreboard (rank title, score bar, correct/accuracy/best streak/avg time/powers used/NEW BEST), a **mastery view showing [DESCRIBE: e.g. a grid of all items colored green/yellow/red/grey]** with a "Practice:" list, and a local top-10 leaderboard per round length.
> - Theme: **[THEME, e.g. ocean race with fish]**, dark background, glassy cards, per-player colors red/blue/yellow/green, kid-friendly and encouraging copy, `prefers-reduced-motion` support.
>
> Content: **[PASTE ITEM LIST or describe the generator rules + difficulty levels]**.

Tip: building it in the same order as the original keeps each step small and easy to review:
1. Core race game (setup → countdown → play → results)
2. Heroes & powers
3. Content tweaks (easy mode, variety questions)
4. Results fit-to-screen + local leaderboard
5. Results card layout (scoreboard | mastery | leaderboard)
6. Setup fits one screen
7. Per-player on-screen keyboards (several open at once)

---

## 12. Ideas that would fit any version

- Teacher mode: pick which tables or items are included (e.g. only ×6, ×7, ×8).
- Practice mode with no timer that only drills the "Practice:" list from the last game.
- Export results (CSV/print) for teachers.
- Sound/voice reading the question (Web Speech API) for younger kids or language learning.
- PWA manifest + service worker so it works offline as an installed app.
