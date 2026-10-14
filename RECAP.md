# Maths Rocket Race: Recap & Blueprint

This document covers what the game is, how it works, and how to add more subjects or modes to it (addition, division, decimals, vocabulary, capitals, and so on) without building a new game.

---

## 1. One-paragraph summary

**Maths Rocket Race** (formerly *Times Table Rocket Race*) is a single-file browser game (`index.html`, about 1,080 lines with no build step and no dependencies). **1–4 players race side by side on one screen**, answering multiple-choice questions against a timer (30/60/90 s). Each correct answer moves that player's rocket up their track. Streaks give a score multiplier, fast answers give a speed bonus, and random "golden" questions are worth ×3. Every 5 correct answers charges the player's **hero power**. Some powers help yourself and some attack the leading player. At the end there is a **results dashboard** with a scoreboard, a **mastery chart** of which facts each player got right or wrong, and a **local leaderboard** saved in the browser.

### Subjects & modes

| Subject | Mode | Example question | Answers |
|---|---|---|---|
| **× Multiplication** | times tables 1×1 to 12×12 | `6 × 7` | `42` |
| **% Percentages** | **Fractions → %** | stacked **¾ = ?** | `75%`, `12.5%`, `33⅓%` |
| | **% of an amount** | `25% of 80` | `20` |

On the setup screen, **Subject** has two buttons (× multiplication | % percentages). Choosing Percentages shows its two modes on a row underneath (↳ ¾ → % | % of 80). The last percentages mode chosen is remembered.

---

## 2. Tech & constraints (keep these when adding features)

| Decision | Why |
|---|---|
| **One `index.html`** (inline CSS + JS, IIFE, `"use strict"`) | Open by double-click, host anywhere (GitHub Pages, USB stick, classroom PC). |
| **No libraries, no images, no audio files** | Emoji are the graphics. Web Audio `OscillatorNode` beeps are the sound. Canvas draws the stars and confetti. |
| **`localStorage` only** (wrapped in try/catch) | Settings, personal best and leaderboard persist on that device. It still works in private mode, just without saving. |
| **Everything fits one screen, no page scroll** | Uses `clamp()`, `vh`/`vw`, flex/grid with `min-height:0`, container queries, and JS `fit()` to shrink long question or answer text. |
| **Shared-device multiplayer** | Tap/touch (`pointerdown`) on a tablet or interactive whiteboard, or keyboard zones per player on a laptop. |
| **Kid-friendly** | Big buttons, encouraging copy, the worked answer shown after a mistake, no penalties below 0, `prefers-reduced-motion` respected. |

---

## 3. Screen flow

```
SETUP ──START──▶ COUNTDOWN (3‑2‑1‑GO) ──▶ PLAY (timer) ──time up──▶ RESULTS
  ▲                                        │  Esc / blur = PAUSE        │
  └──────────── "Change Players" ◀─────────┴──── "Play Again" (Enter) ──┘
```

`phase` variable: `"setup" | "countdown" | "play" | "results"`. Screens are `<section class="screen">` and `show(id)` toggles `.on`.

### 3.1 Setup screen (single card, no scrolling)
- Title, tagline, and 3 "how to play" chips. Step 1 changes with the mode.
- **Subject:** × multiplication / % percentages. With percentages, a **↳ mode row** appears: ¾ → % / % of 80.
- **Players:** 1 / 2 / 3 / 4. **Round length:** 30 / 60 / 90 seconds.
- **Per-player card** (colored border per player): name input and a hero picker (6 emoji buttons) with a description of the selected hero.
  - Tapping a name opens that player's **own on-screen keyboard**. Several can be open at once. Names are capped at 12 characters.
  - With 1 player, attack heroes are disabled and a note explains why.
- START button, Leaderboard button, and a keyboard-controls hint line.

### 3.2 Game screen
- Top bar: Pause, a shrinking **timer bar** (pulses at ≤10 s, ticks at ≤5 s), and Mute.
- **Arena:** one panel per player. Each panel has:
  - name + hero icon, big score (bumps on gain)
  - hero-power button with a fill bar
  - streak counter + multiplier pill (×2, ×3, ×4)
  - a vertical **rocket track** (rocket height = score relative to the leader, and a 👑 crown on the leader)
  - **question box** (golden style, "fog" blur). Fractions are drawn stacked.
  - **4 answer buttons** (2×2) with their keyboard key
  - a feedback line: "⚡ Lightning! +30", or after a mistake, the worked answer (`9/12 = 3/4 = 75%`, `15% of 60 = 9 (15% = 10% + 5%)`)
  - status badges (⚡×2, 🦉 easy, 🛡️) and a FROZEN overlay

### 3.3 Results dashboard (3 cards, fits screen)
1. **Scoreboard:** medal, avatar, name, rank title, score, animated bar, and stat pills (✔ correct, 🎯 accuracy, 🔥 best streak, ⚡ avg time, ✨ powers used, ⭐ NEW BEST).
2. **Mastery chart:** switch players with the chips. The colors are the same in every mode: **green** = always right, **yellow** = right and wrong, **red** = only wrong, **grey** = not asked.
   - Multiplication: a **12×12 heatmap**, "X of 78 facts tried", and a Practice list (`7×8, 6×9…`).
   - Percentages: a **fact wall** with one chip per fact in level order, showing the fact and its answer or shortcut (`3/4 · 75%`, `25% · ¼`). The footer shows "X of 32 / 23 facts tried" and a Practice list.
3. **Leaderboard:** top 10, with tabs per **mode** (× tables / ¾ → % / % of 80) and per **round length**.
- Buttons: Play Again / Change Players. Confetti + victory jingle.

---

## 4. Game rules & numbers

| Rule | Value |
|---|---|
| Base points per correct answer | **10** |
| Speed bonus | **+5** if < 2 s, **+2** if < 4 s |
| Streak multiplier | `min(4, 1 + floor(streak/5))`, so ×2 at 5, ×3 at 10, ×4 at 15 |
| Golden question | needs streak ≥ 3, 18% chance, 4 s cooldown. **×3 points**, counts 2 toward the power charge |
| Wrong answer | streak resets (unless shielded), worked answer shown, 0.9 s pause, no point loss |
| Power charge | **5** correct answers |

Formula: `gain = (10 + speed) × multiplier × (golden ? 3 : 1) × (turbo ? 2 : 1)`

**Rank titles** (points per minute, set per mode in `MODES[mode].ranks`):

| | Pilot | Captain | Commander | Galaxy Legend |
|---|---|---|---|---|
| Multiplication | 130 | 250 | 380 | 520 |
| Percentages (both modes) | 110 | 210 | 320 | 440 |

The lowest rank is 🔰 Cadet.

### Heroes
| Hero | Power | Type | Effect |
|---|---|---|---|
| 🤖 Turbo Bot | TURBO | self | Next 5 correct answers score ×2 |
| 🦉 Wise Owl | EASY | self | Next 4 questions are easy (factors ≤ 5, or level 1 in percentages) |
| 🛡️ Guardian | SHIELD | self | Blocks the next attack **or** saves your streak once |
| 🧙 Ice Wizard | FREEZE | attack | Leader can't answer for 3 s |
| 👽 Fog Alien | FOG | attack | Leader's question blurred for 3 s |
| 🦹 Sneaky Thief | STEAL | attack | Takes up to 40 points from the leader |

Attacks always target the **highest-scoring opponent**, which keeps games close.

---

## 5. Learning design (shared by all modes)

1. **Difficulty ramps up gently** as the player gets answers right.
2. **Missed facts come back:** each mistake is stored as a "spec", and there is a 30% chance per question of asking it again in exactly the same form.
3. **No immediate repeats:** the same fact is never asked twice in a row.
4. **Smart distractors:** wrong options come from typical mistakes, not random numbers.
5. **Kind, worked feedback:** after a mistake the correct answer is always highlighted and explained.
6. **Mastery tracking per fact:** `p.hist[key] = {r, w}` drives the mastery chart and the "Practice:" list.

### 5.1 Multiplication
- The max factor starts at 5 and grows by 1 every 3 correct answers up to 12. After factor 8, 45% of questions force one factor between 6 and 12. ×1 is mostly avoided.
- After 6 correct answers there is a 15% chance of a 3-number question (2–5 each, product ≤ 100).
- Distractors: neighbour facts (6×8 and 7×7 for 6×7), ±1/±2/±10, answer ± a factor, and digit swap (42 → 24).

### 5.2 Percentages: levels
**Level 1 → 2 after 4 correct answers, level 3 after 8.** About 60% of questions come from the newest level and 40% revise easier levels.

**Fractions → % (`FRACS`, 32 facts)**
| Level | Fractions | Idea |
|---|---|---|
| 1: benchmarks | 1/2, 1/4, 3/4, 1/5, 2/5, 3/5, 4/5, 1/10, 3/10, 7/10, 9/10 | facts to know by heart |
| 2: scale to /100 | 1/20, 3/20, 7/20, 9/20, 11/20, 17/20, 1/25, 3/25, 7/25, 1/50, 13/50, 1/100 | multiply top and bottom to make /100 |
| 3: tricky | 1/8, 3/8, 5/8, 7/8, 1/3, 2/3, 5/4, 3/2, 6/5 | decimals, thirds, over 100% |

From level 2, 30% of questions show an **equivalent fraction** (6/8, 9/12, 15/20). They still count as the simplest form for mastery.

**% of an amount (`PCTS`, 23 facts, each with a mental-maths shortcut)**
| Level | Percentages (shortcut) |
|---|---|
| 1 | 50% (½), 10% (÷10), 25% (¼), 100% (all of it) |
| 2 | 20% (÷5), 5% (10% ÷ 2), 75% (¾), 1% (÷100), 30/40/60/70/80% (n × 10%), 90% (100% − 10%) |
| 3 | 15% (10% + 5%), 35% (25% + 10%), 45% (50% − 5%), 12.5% (÷8), 2% (2 × 1%), 65% (50% + 15%), 95% (100% − 5%), 150% (100% + 50%), 200% (× 2) |

Amounts are always chosen so the **answer is a whole number**. The amount is a multiple of `1000 / gcd(pct×10, 1000)`: ×4 for 25%, ×20 for 15%, ×8 for 12.5%. The maximum is 100 / 200 / 400 by level. Amounts under 10 are avoided, and 100 is mostly avoided because it gives the answer away.

### 5.3 Percentages: distractors (misconceptions)
- **Fractions → %** (for ¾): **34%** (digits glued together), 30% / 40% (numerator or denominator × 10), 25% (complement), neighbour fractions (50%, 60%, 100%), half/double, ±10%, and rounding for decimals and thirds (12% / 13%, 33% / 30%).
- **% of an amount** (for 25% of 80 = 20): **60** (the part left over), **100** (adding instead), 200 / 2 (decimal slip), 15/20/30/35% of 80 (neighbour percentages), half/double, and **25** (the percent itself).
- These were checked with 36,000 generated questions across all modes and 0 errors: 4 unique, positive options, always including the correct answer.

---

## 6. Controls

| Player | Answer keys | Power key |
|---|---|---|
| P1 | `1 2 3 4` | `5` |
| P2 | `7 8 9 0` | `-` |
| P3 | `Z X C V` | `A` |
| P4 | `B N M ,` | `.` |

`Esc` = pause, `Enter` = start / play again. The game auto-pauses when the window loses focus.

---

## 7. Persistence (`localStorage` keys)

| Key | Content |
|---|---|
| `ttrr` | settings: `{mode: "mul"\|"frac"\|"amt", pctMode, players, length, sound, names[4], chars[4]}` |
| `ttrr_leaderboard` | `{id, n, i, s, p, l, m: mode, t}`. The best 50 are kept per mode + length. Entries without `m` (from before percentages existed) count as multiplication. |
| `best_{N}p_{len}` / `best_{mode}_{N}p_{len}` | personal best (multiplication keeps the old key name) |

Old settings and scores from the times-table-only version still work.

---

## 8. Code map

| Section in `index.html` | What it does |
|---|---|
| `CONFIG` | colors, key sets, round lengths, heroes (`CHARS`) |
| `SETTINGS` | load/save `ttrr` |
| **`CONTENT`** | `MODES` (name, tab label, step text, rank thresholds), `SUBJECTS`, and the **question builders** for each mode: `mulSpec/buildMul`, `fracSpec/buildFrac`, `amtSpec/buildAmt`, plus `FRACS`, `PCTS`, `pickOpts`, `factBank` |
| AUDIO, STARS, KEYBOARDS | generic |
| `buildSetup()` | subject / percentages mode / players / length / names / heroes |
| `GEN` + `makeQuestion(p)` | dispatches to the current mode's builders. Handles missed-retry and no-repeat. |
| `fit()` / `fitPanel()` | shrinks question and answer text to fit |
| `buildArena`, `answer`, `activate`, timer, flow | generic game engine |
| LEADERBOARD | per mode + length |
| RESULTS | scoreboard, `showMastery()` (heatmap for `mul`, fact wall + `layoutChips()` for the others), `titleFor()` |

### The mode "plug-in" contract

Every mode is a pair of functions registered in `GEN`:

```js
// spec(lvl, p) → a small object that identifies one question; must include a stable `key`
fracSpec(lvl, p)  →  {key:"3/4", n:3, d:4, k:2}        // k = shown as 6/8

// build(spec) → the question shown to the player
buildFrac(spec)   →  {
  key:     "3/4",                      // fact ID (mastery, no-repeat, retry)
  spec,                                // stored in p.missed for an exact retry
  html:    "<span class=frac>…</span> = ?",   // question box content
  ans:     "75%",                      // correct answer (string or number)
  opts:    ["34%","75%","25%","60%"],  // exactly 4, unique, shuffled, containing ans
  explain: "6/8 = 3/4 = 75%"           // shown after a wrong answer
}
```

Helpers the builders can use:
- `levelOf(p)`: 1–3 (Owl forces 1)
- `pickLevel(bank, lvl, levelIndex)`: picks a fact, weighted toward the newest level
- `pickOpts(ans, candidates, fallback)`: builds the 4 options

---

## 9. How to add a new subject or mode

1. **Add the mode to `MODES`:** `name`, `tab` (leaderboard label), `big` + `sub` (button text), `step` (how-to-play step 1), `ranks` (4 points-per-minute thresholds).
2. **Write `xxxSpec(lvl, p)` and `buildXxx(spec)`** following the contract above, and register them in `GEN`.
   - For list-type material (vocabulary, capitals) use a bank like `FRACS`/`PCTS`: `[question, answer, level, …wrongAnswers]`.
3. **Show it on the setup screen:**
   - **New mode in an existing subject:** add its id to that subject's list (e.g. `PCT_MODES`).
   - **New subject:** add it to `SUBJECTS`. If it has several modes, give it a sub-mode row like percentages does (`#pmodeGroup`), with a remembered `settings.xxxMode`.
4. **Mastery:** add the mode to `factBank(mode)` to get the fact wall for free, or add a custom branch in `showMastery()` like the multiplication heatmap.
5. **Owl power:** make sure `p.fx.easy > 0` (or `levelOf`) gives easy questions.
6. **Test:** generate thousands of questions in Node (check 4 unique options, the answer included, and the answer correct), then play 1 and 4 players on a laptop and a phone.

### Mastery visual ideas
| Material | Visual |
|---|---|
| Addition / subtraction 0–20 | N×N heatmap like multiplication |
| Division | 12×12 grid of divisor × quotient |
| Decimals ↔ %, vocabulary, capitals | fact wall via `factBank` |

---

## 10. Build history

1. Core times-table race (setup → countdown → play → results)
2. Heroes & powers
3. Owl easy questions, 3-number multiplication
4. Results fit-to-screen + local leaderboard
5. Results card layout (scoreboard | mastery | leaderboard)
6. Setup fits one screen
7. Per-player on-screen keyboards (several open at once)
8. **Percentages subject** with two modes (fractions → %, % of an amount), fact-wall mastery, per-mode leaderboard, auto-fit text

## 11. Ideas for later
- **Mixed mode:** all question types in one round.
- **More percentages modes:** reverse ("20 is 25% of ?"), "what % is 12 of 48?", increase/decrease ("increase 80 by 25%"), decimals → %.
- **Teacher mode:** pick which tables or facts are included.
- **Practice mode:** no timer, drills only the "Practice:" list from the last game.
- Read questions aloud (Web Speech API), export results for teachers, and a PWA for offline use.
