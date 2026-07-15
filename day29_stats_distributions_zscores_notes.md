# Day 29 Stats: Distributions & Z-Scores — Complete Notes

*(Topic 2 of 7. Continues from Days 27–28: descriptive statistics complete.)*

## Block 1: Distribution Shapes

A **histogram** = the distribution made visible. Bin the value range, count rows per bin, draw bars. Height = crowd density — the picture that percentile-spacing was sketching blind.

### The four shapes

| Shape | Signature | Where it lives |
|---|---|---|
| **Normal (bell)** | symmetric, one peak, thin tails | heights, measurement errors, averages of many things |
| **Right-skewed** | peak left, long right tail | prices, incomes, delivery times, order values — MOST business data |
| **Left-skewed** | peak right, long left tail | exam scores near ceiling, age at natural death |
| **Bimodal** | **TWO peaks** | mixed populations hiding in one column |

### THE BIMODAL TRAP (today's key catch)

> **Two peaks = two different populations wearing one label.**

**Case: review_score (1–5 stars).** Plausible-but-wrong guess: "left-skewed, most give 4–5, few give low." The real shape: **bimodal.** Why — think about who bothers to respond. Satisfied customers click 5 and leave. Mildly annoyed customers usually say nothing. FURIOUS customers make the effort to click 1 — maximum protest. Result: huge peak at 5, valley through 2–4, second bump at 1.

**Why it matters:** the mean of a bimodal column (~4.1 here) describes a "compromise customer" who doesn't exist — no real group of customers rated 4.1. Correct reporting: name both populations ("~77% rate 4–5, ~13% rate 1"), not one center.

> **Rule:** Ratings and survey data are bimodal by nature — extremes self-select into responding. Assume any 1–5 star column is two-peaked until a histogram proves otherwise.

> **The deeper lesson:** "Plausible" is where checking stops and mistakes start. Left-skewed wasn't a stupid guess — it was reasonable, which is exactly what made it dangerous. When a column involves human CHOICE to respond, ask who's choosing before naming the shape.

**Only correct move for bimodal data:** split into the two populations first, THEN summarize each separately. Same principle as like-for-like comparisons — don't average across populations that aren't the same population.

### The normal curve's honest place
Raw business data is RARELY normal. The bell shows up in **measurements** (weights, durations of stable processes) and **averages of many independent things**. "Assume normal" is a claim to verify with a histogram, never a default.

---

## Block 2: Z-Scores

### The formula and what it buys
```
z = (value − mean) / SD
```
Distance from average, in SD units — one yardstick for any column, any units.

| z-score | Meaning |
|---|---|
| 0 | exactly average |
| 1 | one SD above (inside the "normal" 68% zone) |
| 2 | two SDs above (95% boundary — starting to stand out) |
| 3 | three SDs above (RARE — under 0.3% of a normal distribution) |

**The power move:** z-scores compare wildly different units on one scale. "z = 4.2" means massively unusual regardless of whether it's a price, a delivery time, or a review count.

**Connection to Day 28:** z ≈ 2 lines up with the 68/95 outer edge; z ≈ 3 is the numeric cousin of crossing the IQR fence. Two tools, same underlying question: how far is this from where the crowd lives?

### Worked example
freight_value: mean = 20, SD = 15. Item at 50.
- z = (50 − 20) / 15 = **2** → right at the 95% boundary, genuinely unusual

### THE CIRCULARITY TRAP — z-scores on skewed data

> **Z-scores assume roughly normal data.** On a right-skewed column (freight_value, proven skewed in Block 1), the mean and SD are ALREADY inflated by the same tail you're trying to detect. The z-score built from inflated inputs systematically UNDER-FLAGS the real high-end outliers. Circular problem.

**Interview-grade phrasing:** "The mean and SD used to compute this z-score are themselves inflated by the same right tail I'm trying to detect — so z-scores under-flag real outliers on skewed data. I'd use the IQR fence instead." (Naming the mechanism > just saying "don't trust it.")

> **Rule:** z-scores → roughly symmetric data. Skewed columns (most business data) → IQR fence. Same question, robust method.

### Where z-scores get used on the job
- Fraud detection — a transaction z-scored against a customer's own history
- Quality control — a measurement z-scored against a process's normal spread

---

## Skill pattern across this session (worth naming)
Three separate moments today required choosing the RIGHT TOOL FOR THE SHAPE, not just operating a tool: IQR-vs-SD pairing, the bimodal review-score catch, z-score-vs-fence. That matching instinct — done unprompted — is the actual skill this topic exists to build.

## One-line takeaway
> **Z-scores and IQR fences answer the same question through different doors — z trusts the mean and SD, IQR trusts position, and on skewed data only one of them is honest.**
