# Percent Rocket Race: Recap

A percentages version of *Times Table Rocket Race* (see [`../RECAP.md`](../RECAP.md) for the shared engine). It keeps the same race, heroes, scoring, results dashboard and leaderboard. Only the **material** changed: it has **two modes** instead of times tables.

Open `percentages/index.html` in a browser. It is still one file with no dependencies.

---

## 1. The two modes

| | Mode 1: **Fractions → %** | Mode 2: **% of an amount** |
|---|---|---|
| Setup button | `¾ → %` | `% of 80` |
| Question | a stacked fraction, e.g. **¾ = ?** | **25% of 80** |
| Answers | percentages: `75%`, `12.5%`, `33⅓%` | whole numbers: `20` |
| A "fact" (for mastery) | the fraction in simplest form, e.g. `3/4` | the percentage, e.g. `25%` |
| Facts in the bank | 32 | 23 |
| Feedback after a mistake | `9/12 = 3/4 = 75%`, `3/20 = 15/100 = 15%` | `15% of 60 = 9 (15% = 10% + 5%)` |

The mode is picked on the setup screen (it is saved with the other settings) and is shown in the countdown, the result chips and the leaderboard.

---

## 2. Content banks & levels

The level goes up as the player answers correctly: **level 1 → 2 after 4 correct, level 3 after 8**. About 60% of questions come from the newest level and 40% revise easier ones. The 🦉 Owl power forces level 1.

### Mode 1: fractions (`FRACS`)
| Level | Fractions | Idea |
|---|---|---|
| 1: benchmarks | 1/2, 1/4, 3/4, 1/5, 2/5, 3/5, 4/5, 1/10, 3/10, 7/10, 9/10 | facts to know by heart |
| 2: scale to /100 | 1/20, 3/20, 7/20, 9/20, 11/20, 17/20, 1/25, 3/25, 7/25, 1/50, 13/50, 1/100 | multiply top and bottom to make /100 |
| 3: tricky | 1/8, 3/8, 5/8, 7/8, 1/3, 2/3, 5/4, 3/2, 6/5 | decimals, thirds, over 100% |

**Equivalent fractions:** from level 2, 30% of questions show the fraction un-simplified (e.g. **6/8**, **9/12**, **15/20**). The fact is still counted as `3/4`, and the feedback shows the simplifying step.

### Mode 2: percentages (`PCTS`, each with a mental-maths shortcut)
| Level | Percentages (shortcut) |
|---|---|
| 1 | 50% (½), 10% (÷10), 25% (¼), 100% (all of it) |
| 2 | 20% (÷5), 5% (10% ÷ 2), 75% (¾), 1% (÷100), 30/40/60/70/80% (n × 10%), 90% (100% − 10%) |
| 3 | 15% (10% + 5%), 35% (25% + 10%), 45% (50% − 5%), 12.5% (÷8), 2% (2 × 1%), 65% (50% + 15%), 95% (100% − 5%), 150% (100% + 50%), 200% (× 2) |

**Amounts are always chosen so the answer is a whole number.** The amount is a multiple of `1000 / gcd(pct×10, 1000)` (×4 for 25%, ×20 for 15%, ×8 for 12.5%). Maximum amount: 100 (level 1), 200 (level 2), 400 (level 3). Amounts under 10 are avoided, and an amount of 100 is mostly skipped because it gives the answer away.

---

## 3. Smart wrong answers (distractors)

Wrong options are based on real misconceptions, so a wrong pick tells the teacher something:

**Mode 1 (e.g. 3/4):**
- digits glued together: **34%**. This is the classic "3/4 = 34%" mistake.
- numerator × 10 → 30%, denominator × 10 → 40%
- the complement 100 − P → 25%
- neighbour fractions (2/4, 4/4, 3/5, 3/3) → 50%, 100%, 60%
- half/double, ±10%, and for decimals or thirds the rounded values (1/8 → 12% / 13%, 1/3 → 33% / 30%)

**Mode 2 (e.g. 25% of 80 = 20):**
- the part left over: 80 − 20 = **60**
- adding instead: 80 + 20 = **100**
- decimal-place slips: **200**, **2**
- neighbour percentages: 15%, 20%, 30%, 35% of 80
- double/half, dividing by the percent, and the percent itself (**25**)

Only valid values are used: positive numbers, whole or one-decimal percentages, and whole-number amounts. All 4 options are unique and always contain the right answer. This was checked with 24,000 generated questions and 0 errors.

---

## 4. What changed from the times-table version

| Area | Change |
|---|---|
| Setup | New **Mode** picker. Step 1 text follows the mode. New title and tagline. |
| Question engine | `makeQuestion` now dispatches to `fracSpec/buildFrac` or `amtSpec/buildAmt`. Missed facts are stored as specs (`{key,n,d,k}` / `{key,pct,a}`), so a retry asks exactly the same question. |
| Display | Questions are HTML (stacked fraction). The new `fit()` shrinks question and answer text to fit any panel width (4 players, phone). It replaces the old `.tri` class. |
| Feedback | Wrong answers show the worked step (`explain`), not just the answer. |
| Mastery card | The 12×12 heatmap is replaced by a **fact wall**: one chip per fact in level order, showing the fact and its answer or shortcut (`3/4 · 75%`, `25% · ¼`). It is colored green / yellow / red / grey, and `layoutChips()` picks the grid size to fit the card. The footer reads "X of 32 (or 23) facts tried" plus a Practice list (`3/8 = 37.5%`, `15% = 10% + 5%`). |
| Leaderboard | Saved **per mode and round length**, with two rows of tabs (mode, then length). |
| Rank titles | Thresholds lowered (110 / 210 / 320 / 440 points per minute), since percentages take longer than times tables. |
| Storage keys | `prr` (settings), `prr_leaderboard`, `prr_best_{mode}_{N}p_{len}`. These are separate from the times-table app. |
| Owl power | "EASY (½, ¼, 10%…)" means level 1 of the current mode. |

Unchanged: players 1–4, keyboard zones (`1234/5`, `7890/-`, `ZXCV/A`, `BNM,/.`), on-screen name keyboards, timer, scoring, streaks, golden ×3, all six heroes, pause, sound, confetti.

---

## 5. Ideas for later
- **Mixed mode:** both question types in one round, with the mastery wall split in two.
- **Mode 3, reverse:** "20 is 25% of ?" or "what % is 12 of 48?".
- **Percent increase/decrease:** "Increase 80 by 25%" (level 3 of mode 2).
- **Decimals → %:** 0.35 = 35%. This would reuse mode 1's formatter.
- **Context questions:** money (sale price), test scores, bar models for younger learners.
