# Day 34 Stats: A/B Testing Process & Regression — Syllabus Complete

*(Topic 6 Block 3 + Topic 7. Closes the full 7-topic statistics syllabus, Days 27-34.)*

## Part 1: The A/B Test, End to End (Topic 6, Block 3)

### The five steps of a real test
1. State H₀ and H₁ **before** collecting data
2. Decide significance threshold (usually p < 0.05) — **before** the test runs
3. Decide sample size — **before** the test runs (the step everyone skips)
4. Run the test, collect data, **don't peek and stop early**
5. Compute the p-value once, compare to threshold, make the call

### THE P-HACKING TRAP (the step everyone skips, and why it's dangerous)

**Scenario:** checking A/B test results every day, stopping the moment p dips below 0.05, declaring a win.

**Why this fails — the full mechanism:**

A p < 0.05 threshold promises: "if H₀ is really true, there's only a 5% chance of wrongly calling this significant, **on one properly-run test, checked once.**"

A p-value on a small, still-growing sample bounces around from day to day — pure noise (same wobble idea as Day 31's revenue example). Purely by chance, with ZERO real effect underneath, it will occasionally dip below 0.05 just because that's what noisy numbers do — they wander across any line given enough looks.

**Checking daily and stopping at the first dip isn't running one test with 5% error — it's running SIX separate opportunities for noise to randomly wander under the line, and grabbing the first one that does.** The true false-positive risk compounds across every look, even though each individual look "looks like" only 5% risk.

> **Interview-ready phrasing:** "Checking a growing sample repeatedly and stopping at the first significant result inflates the true false-positive rate far above the stated 5%, because you're really running many small tests and keeping the lucky one — not one clean test on a predetermined sample."

**The fix:** decide sample size in advance based on the effect size you care about detecting, run to completion, look at the p-value exactly once.

**Connects to:** Day 33's false positive lesson (noise mistaken for signal) — p-hacking is the everyday PROCESS mistake that manufactures false positives invisibly, even when everyone involved believes they're being rigorous.

---

## Part 2: Regression (Topic 7)

### The line of best fit
```
y = mx + b
```
Applied: `freight_value = m × (product_weight_g) + b`

- **m (slope/coefficient):** for every 1-unit increase in x, y changes by m units on average. THE number a business acts on — "each extra kg costs roughly ₹X more to ship." A predicted RATE, usable going forward, not just a description of dots already collected.
- **b (intercept):** predicted y when x = 0. Often not meaningful alone (nobody ships a 0g package) — anchors the line.

### R² — how much the line actually explains
> **R² = what percentage of the variation in the outcome is explained by this line.** Ranges 0 to 1 (or 0–100%).

R² = 0.65 means 65% of variation in freight_value is explained by weight; the other 35% comes from something the model doesn't capture (distance, seller location, fragility, etc.)

**Connects directly to Day 30:** this is the formal number behind "weight is a driver, not the only driver" — the visible scatter around the trend line IS the unexplained variation. Low R² = lots of scatter = weight alone doesn't tell the story. High R² = dots hug the line = weight alone predicts well.

**PRECISION NOTE — R² is not causation:** "85% of variation is EXPLAINED/PREDICTED by weight" — not "directly tied to" or "caused by." R² measures statistical explanatory power, not mechanism. (In the weight→freight case, real causation likely DOES exist — but that's known from understanding how shipping physically works, not proven by R² alone. R² would report the same number even if weight were just a strong confounded proxy for something else.)

**A low R² is not a dead end — it's a map.** R² = 0.20 for a seller means: what other variables are driving the remaining 80% of the variance? That question IS the correct analyst move — low R² generates the next investigation, it doesn't end the analysis.

### THE EXTRAPOLATION TRAP
A regression line is fit to the range of data it was trained on (e.g., 100g–15,000g). It will mathematically keep drawing itself forever in both directions if asked, but has ZERO evidence for what happens outside that range.

> **Rule: never trust a regression line's predictions outside the range of the data that built it.** The relationship that looks linear inside the data might curve, flatten, or break entirely outside it — same caution as every other lesson this phase: don't trust a summary number beyond what the underlying data can support.

---

## STATISTICS SYLLABUS: COMPLETE (7/7 topics, Days 27-34)

1. Descriptive statistics — mean/median, skew, percentiles, IQR, standard deviation
2. Distributions — histogram shapes, the bimodal trap, z-scores
3. Relationships — correlation, force-test for causation, confounders vs. direct mechanisms vs. feedback loops, Simpson's Paradox
4. Uncertainty — Law of Large Numbers, sampling bias, survivorship bias
5. Probability essentials — conditional probability, law of total probability, base-rate fallacy
6. Hypothesis testing & A/B tests — p-values, Type I/II errors, p-hacking
7. Regression — slope, R², extrapolation trap

**Remaining before SQL portfolio project: the exit gauntlet** — mixed problems across all 7 topics, solved cold, explained aloud, no notes. This is where coverage becomes confidence.

## One-line takeaway
> **Regression gives you two numbers, not one — the slope tells you the rate to act on, R² tells you how much to trust it, and a low R² isn't a failure, it's a map to what you haven't measured yet.**
