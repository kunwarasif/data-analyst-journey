# Day 28 Stats: Standard Deviation — Complete Notes

*(Continues from Day 27 stats: mean vs median, percentiles, IQR. This completes descriptive statistics — Topic 1 of 7.)*

## The Core Idea

> **Standard deviation (SD) ≈ the typical distance of a data point from the mean.**

- Small SD → data hugs the average
- Large SD → data sprawls
- The formula (average the squared distances, square-root back) is just bookkeeping; the intuition above is what matters

## The 68/95 Rule

For roughly **bell-shaped** data:
- ~68% of values fall within **1 SD** of the mean
- ~95% fall within **2 SD**

Converts SD from an abstract number into a concrete promise about where the data lives.

**Worked example (Olist delivery routes, both mean = 10 days):**
- Route A, SD = 1 → 95% window: 10 ± 2 = **8–12 days**
- Route B, SD = 6 → 95% window: 10 ± 12 = **−2 to 22 → clamp to 0–22 days**

## THE BOUNDARY INSIGHT (discovered via the clamp)

The arithmetic said −2 days; reality says deliveries can't be negative. When a 68/95 window **crashes through an impossible boundary** (negative days, negative prices), the data is telling you **it isn't actually bell-shaped**. The rule assumes symmetry; a nonsensical bound is that assumption failing in public.

Delivery times are right-skewed: most arrive quickly, a few drag for weeks. Clamping fixes the symptom; recognizing skew is the diagnosis.

## The Business Lesson — Consistency Is Invisible to the Mean

Two routes, identical means, completely different businesses:
- Route A's customer can PLAN: 8–12 days, order accordingly
- Route B's customer holds a lottery ticket: maybe 3 days, maybe 20 — can't promise gifts, can't time anything, and the 20-day cases write the angry reviews

> **Customers can plan around slow-but-predictable; they cannot plan around a lottery. The mean hid everything; the SD told the story.** (Interview-grade sentence — keep it.)

## The Pairing Principle

| Center | Spread partner | Character |
|---|---|---|
| **Mean** | **Standard deviation** | Uses every value — powerful, but **corruptible by outliers** (huge distances get SQUARED into the average) |
| **Median** | **IQR** | Position-based — **robust to** outliers |

SD inherits the mean's weakness — same fault line as Block 1.

**Precision upgrade for interviews:** IQR doesn't "ignore" outliers (they're still in the data) — it's **unaffected by** them, because P25/P75 are positions extreme values can't move. "Robust to" is the term that signals mechanism-knowledge; "ignores" sounds like deletion.

**For right-skewed data (Olist prices): trust IQR over SD.**

## The Complete Descriptive-Statistics Kit (Topic 1 of 7 — DONE)

- **Two centers:** mean, median
- **Two spreads:** SD, IQR
- **When each pair applies:** symmetric-ish data → mean + SD; skewed data → median + IQR
- **Three independent skew detectors:**
  1. Mean vs median gap (mean above → right tail)
  2. Percentile spacing (widening gaps upward → right tail)
  3. Impossible-boundary violations (95% window crossing into nonsense values → not bell-shaped)

## One-line takeaway
> **Same mean, different SD = different businesses — consistency is invisible to the average, and a 95% window that crosses into impossible values is skew confessing itself.**
