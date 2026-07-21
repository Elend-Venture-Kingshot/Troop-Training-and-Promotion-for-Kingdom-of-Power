# Kingshot &mdash; KoP Pre-Event Training Optimizer

A single-file, no-dependency HTML tool that answers one question:

> **Before the Kingdom of Power (KoP) event, which troop tier should I train — and how many points will promoting it during the event actually get me?**

Open `kop_training_optimizer.html` in any browser. Nothing to install, no server, no build step — it's a self-contained page (HTML + CSS + vanilla JS).

---

## The problem it solves

During KoP you earn points by training/promoting troops **using speedups**. Promoting an already-trained lower-tier troop up to your top tier (usually T10) costs far fewer speedups per point than training a T10 troop from scratch, because promotion time is much shorter than full training time for the same points gain.

So the standard strategy is:

1. **Before** the event, spend a fixed number of days training a *lower* troop tier (no speedups needed — just normal queue time).
2. **During** the event, spend your speedups promoting that stockpile up to T10, earning KoP points per troop promoted.

The catch: which tier to pre-train depends on how many speedups you actually have.

- Train a **very low tier** (e.g. T1): you can train *huge* numbers of troops before the event, but promoting all of them to T10 needs way more speedups than most people have — most of the stockpile goes unpromoted and earns nothing.
- Train a **very high tier** (e.g. T9): promoting is cheap per troop, but you can only train a small stockpile in the same number of days — you'll likely have leftover speedups with nothing left to promote.
- Somewhere in between is a **sweet spot** tier that uses your speedup budget almost exactly, with little waste in either direction.

This tool finds that sweet spot for your specific numbers, two different ways (see below).

---

## Inputs (shared by both methods)

| Field | Meaning |
|---|---|
| **Time before the event (days)** | How many days you have to pre-train troops. |
| **Training speed bonus (before event)** | Your in-game "Troop Training Speed" stat, as a percentage (e.g. `150` for +150%). |
| **Noble Advisor active** | Checkbox. Adds +50% training speed, active only during the event. |
| **Kingdom King Skill active** | Checkbox. Adds +30% training speed, active only during the event. |
| **Available speedups for the event (days)** | Your total speedup budget, converted to days, that you're willing to spend during KoP. |

**Event training speed** is computed automatically as:

```
event speed % = (before-event speed %) + 50% (if Noble Advisor) + 30% (if King Skill) + 25% (fixed event bonus)
multiplier used in the math = 1 + (that %) / 100
```

The pre-event multiplier uses the same `1 + pct/100` formula on the before-event speed alone (no event-only bonuses apply before the event starts).

All game constants (per-tier training time, promotion time, and KoP points per troop trained/promoted) are hardcoded near the top of the `<script>` block, in arrays indexed T1&ndash;T10. Edit those arrays directly if Kingshot changes the numbers — no other code needs to change.

There are also **3 parallel training queues** assumed throughout (matches in-game barracks), and every troop/promotion count is rounded down to whole troops.

---

## Method 1 — Single Tier

The straightforward approach: train **one** tier for the entire pre-event window, then promote as many as your speedup budget covers.

For each candidate tier X (T1&ndash;T10), the tool computes:

1. **Troops trainable**: `floor(pre-event days × seconds/day ÷ time-to-train-1-troop × 3 queues)`
2. **Full promotion requirement**: the total speedup-days needed to promote *all* of those troops to T10.
3. **Points**, in two cases:
   - **Enough speedups** (budget ≥ requirement): every troop gets promoted (troops × promotion points), and any *leftover* speedup budget is spent training brand-new T10 troops directly (at event speed), earning additional points at the T10 training rate.
   - **Not enough speedups** (budget < requirement): only as many troops as the budget allows get promoted; the points scale down accordingly, and the rest of the stockpile is wasted.

The tier with the highest resulting points wins. **Tie-break rule:** if a higher tier lands within 0.5% of the best score, the tool recommends the higher tier instead — so you get more use out of those troops even before the event starts, at essentially no cost in points.

The results panel shows the recommended tier plus a full breakdown table and bar chart across all 10 tiers, so you can see exactly why a given tier won (or how close the runner-up was).

---

## Method 2 — T9 + Secondary Tier Split

A more advanced strategy: what if you don't commit all your pre-event days to one tier?

T9 is the *cheapest* tier to promote per troop (shortest promotion time), but you can only train a small batch of T9s in a given number of days. The idea: spend a **few days training T9 first** — so a small, cheap-to-promote, ready-to-use batch exists early — then switch all 3 queues to whichever *other* tier best uses the remaining days and remaining speedups.

You control the split with **"Days allocated to T9"**. The tool then:

1. Trains T9 for that many days, and gives it **first claim** on your entire speedup budget (fully promoting the T9 batch if the budget allows).
2. Whatever speedup budget is left over goes to the best secondary tier (any tier except T9), trained for the remaining days — using the exact same full/partial/leftover-to-T10 logic as Method 1.
3. Reports the combined points from both slices, and compares that total against Method 1's best single-tier result for the **same total speedup budget**, showing the point difference (gain or loss) directly.

This lets you check, for your actual numbers, whether splitting time between two tiers ever beats committing to one. (Expectation going in: splitting usually costs a small number of points, because it shrinks the secondary tier's stockpile — but if Method 1's pick would otherwise waste speedups on plain T10 training, redirecting a few days to T9 first can occasionally help. The comparison card tells you which side of that line you're on.)

---

## Outputs

Both methods report, at minimum:

- **Troops trained before the event** (the "check the math" number)
- **Troops actually promoted** given your speedup budget
- **KoP points gained**

Method 1 additionally shows a full 10-row breakdown table and bar chart; Method 2 shows a two-slice table (T9 slice + secondary slice) and a delta card versus Method 1.

---

## Notes / assumptions baked into the model

- Only **KoP (Kingdom of Power)** point values are used — the underlying data arrays also have HoG/Strongest Governor/Alliance Brawl equivalents in the source spreadsheet, but this tool ignores those columns.
- **3 parallel training queues**, both before and during the event.
- All troop and promotion counts are **rounded down** (`floor`) to whole troops — no fractional troops.
- Leftover speedup budget beyond what's needed for full promotion is always spent training **T10 troops directly at event speed**, never left idle.
- The 0.5% tie-break margin (`TIE_EPS` in the code) can be adjusted if you want a stronger or weaker pull toward higher tiers.

---

## File

- `kop_training_optimizer.html` — the whole tool. Just open it in a browser; no build tools, no dependencies beyond a Google Fonts CDN link for styling (works fine offline too, it'll just fall back to system fonts).
