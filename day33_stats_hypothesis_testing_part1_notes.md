# Day 33 Stats: Hypothesis Testing — Foundations (Part 1 of 2)

*(Topic 6 of 7, Blocks 1-2. Continues from Days 27-32: descriptive stats, distributions, relationships, uncertainty, probability essentials all complete. This topic runs 2 days — continues Day 34.)*

## Block 1: The Actual Question Behind "Is This Significant?"

**Scenario:** Old checkout converts 10%. New checkout, tested on a sample, converts 11%. Real improvement, or luck?

**This is Day 31's Block 1 question in a business costume** — "850K vs 920K revenue, real growth or wobble?" and "10% vs 11% conversion, real or luck?" are the identical question. Core instinct already owned: any measured difference could be real, or could be the natural wobble you'd see even if nothing changed.

### The framework
> **Null hypothesis (H₀):** the boring default — "nothing changed, the gap is just noise."
> **Alternative hypothesis (H₁):** the exciting claim — "the effect is real."

**You never directly prove H₁.** The test works backwards: assume H₀ is true, then ask "if nothing had actually changed, how surprising would a gap this big be, purely from random luck?"

> **p-value = if H₀ were true, the probability of seeing a gap this big or bigger, purely by chance.**

Small p (conventionally < 0.05) → this would be rare if nothing changed → suspicious → reject H₀, call it significant.
Large p → this happens by chance often enough → can't reject H₀ → stay skeptical.

### THE MOST COMMON MISREADING (critical)
p = 0.03 does **NOT** mean "97% chance the new page is really better."
It means: "if the new page were secretly no different, you'd see a gap this big or bigger about 3% of the time by pure chance."

The p-value describes the DATA, assuming the boring hypothesis — not the probability that the exciting hypothesis is true. Almost every real-world misuse of statistics traces back to blurring this exact line.

### Sample size and p-values — connects directly to Day 31
Same observed gap (10% vs 11%), two sample sizes:
- **500 per group:** the gap might ride on a handful of individual random outcomes. Flip 3-4 people's decisions by chance → gap could vanish or flip. Wide wobble, plausibly noise.
- **50,000 per group:** the same gap is built from tens of thousands of individual coin-flips of luck, and per the Law of Large Numbers (Day 31), that randomness has increasingly canceled out. For the SAME-sized gap to survive averaging across 50,000 people, something structural is very likely holding it up.

> **Interview-ready phrasing:** "With the same observed gap, a larger sample makes a chance explanation less plausible, because random noise increasingly cancels out as sample size grows — so the same-sized gap becomes harder to attribute to luck. That's what a smaller p-value is capturing." (Name the mechanism, not just "bigger sample = more convincing.")

---

## Block 2: Two Ways to Be Wrong

Every test ends in one of two verdicts (reject H₀ / fail to reject H₀), and each can fail in two distinct directions:

> **False Positive (Type I error):** reject H₀ when it was actually true. Declare "the new page works!" when it doesn't — got a lucky sample. Cost: roll out a change that does nothing (or is quietly worse), waste engineering time.

> **False Negative (Type II error):** fail to reject H₀ when H₁ was actually true. The new page genuinely IS better, but the sample was too small/noisy to prove it — conclude "no significant difference," discard a real improvement.

**These are the same two failure modes from the whole stats phase, now sitting side by side as a trade-off:** false positive = trusting noise as signal (Day 31). False negative = a real effect buried under noise that didn't get to cancel out (flip side of Law of Large Numbers).

### The trade-off (can't minimize both at once with fixed sample size)
Stricter threshold (p < 0.01 instead of 0.05) → fewer false positives, BUT more false negatives (harder to detect real effects).
Looser threshold → catches more real effects, BUT lets more lucky-noise results slip through as false "wins."
**Only fix for both simultaneously: more data.** This is why real A/B tests run on tens of thousands of users when affordable.

### Which error is worse? NOT universal — reasoned fresh per context

**Medical screening (serious disease):** False negative is far worse. Missing real illness can cost a life; a false alarm just costs an inconvenient re-test.

**Business A/B test (e.g., checkout redesign):**
- False positive = ship a redesign that's actually no better (or slightly worse) because a lucky sample fooled you → wasted engineering effort + possible silent ongoing revenue drip (nobody's watching because everyone believes it's a win)
- False negative = a genuinely better redesign gets shelved because the test was underpowered → real revenue left on the table indefinitely, and the idea may never get retested ("we already tried that")
- **Genuinely more debatable than the medical case** — depends on cost of engineering time vs. rarity of good ideas. This is exactly why companies set significance thresholds (0.05 vs 0.01 vs 0.10) as a deliberate judgment call, not a universal rule.

> **Common mistake to avoid:** false positive ≠ "any bad outcome" (like a broken payment button). False positive specifically means the DATA fooled you into believing a real effect existed when it didn't. A catastrophic bug is a different problem entirely — not what A/B testing error types describe.

> **The real skill:** false-positive-vs-false-negative severity is NOT symmetric and NOT the same across contexts. Which mistake is worse depends entirely on which one costs more to undo — reasoned out fresh every time, never assumed.

---

## One-line takeaway (Day 33, Topic 6 continues Day 34)
> **A p-value tells you how surprising your data would be if nothing had really changed — not the probability that your exciting hypothesis is true. And false positives and false negatives aren't equally dangerous; which one to fear depends entirely on what a wrong call costs to undo.**
