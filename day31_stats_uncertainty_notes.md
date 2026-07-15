# Day 31 Stats: Uncertainty — Samples, Bias & the Law of Large Numbers

*(Topic 4 of 7. Continues from Days 27-30: descriptive stats, distributions, relationships all complete.)*

## Block 1: Samples, Populations, and "Is This Real?"

Every number computed from real-world data is a **sample** — one particular slice of reality, not reality itself. Rerun the same month with slightly different customers/choices and you would NOT get the exact same number. Natural wobble is baked into any real measurement.

> **Core question of this topic: is a difference a real signal, or just the wobble you'd expect anyway?**

**Same lesson as Day 21's partial-year YoY trap, harder disguise.** The partial year was visibly broken (3 months, obviously incomplete). Here, both numbers being compared look completely normal and legitimate — nothing LOOKS wrong, which is what makes this version more dangerous.

**Connects to Standard Deviation (Day 28):** if typical month-to-month wobble (SD) is 30K, a 70K jump is 2+ SDs — looks like real signal. If typical wobble is 100K, a 70K difference is SMALLER than the noise itself — unremarkable, could flip next month for no reason.

---

## The Sample-Size Question: Why More Data Builds Trust

**Claim A:** 15% lift, tested on 40 customers.
**Claim B:** 15% lift, tested on 40,000 customers.
Same percentage — but B deserves far more trust.

**The mechanism (not just "bigger is better"):** each customer's outcome is a small random draw, noisy on its own. On 40 people, the 15% lift might ride on ~6 people's results — flip 2-3 outcomes by pure chance and the lift could vanish or reverse. Small samples let individual randomness dominate the average.

On 40,000, the same random flips still happen at the individual level — but now they increasingly **cancel out against each other** (one random "no" balanced by another random "yes" across thousands of pairs). Noise cancels; real underlying effect keeps accumulating in one direction.

> **Law of Large Numbers: as sample size grows, the sample average converges toward the true underlying value, because random noise increasingly cancels out.** Small samples let noise dominate; large samples let signal survive.

**Interview-ready phrasing:** "A 15% lift on 40 people has a wide margin of error — a handful of different individual outcomes could easily erase it. On 40,000, random ups and downs largely cancel out, so I'd trust that the 15% reflects something real rather than noise." (Name the mechanism, not just the verdict.)

---

## Block 2: Sampling Bias — The Trap Size Can't Fix

**Setup:** Olist emails a satisfaction survey to 50,000 customers. 5,000 respond (solid N by Block 1 logic). Average score: 4.3/5.

**The trap:** who actually clicks "take this survey"? Not a random slice — disproportionately people with something to say.

### THE CONNECTION TO DAY 29 (same mechanism, new costume)
This is the **exact same selection mechanism as the bimodal review-score trap** — happy customers respond AND furious customers respond (to complain); neutral/mildly-satisfied customers skip it. Extremes self-select into responding — this isn't a one-off quirk of star ratings, it's a **general law about anything requiring voluntary effort to respond.**

### Why MORE data makes it WORSE, not better
Block 1's noise was random — coin-flips of luck canceling out symmetrically. Sampling bias's noise is **not random — it's a one-directional pull**. Every respondent, from the 1st to the 50,000th, is selected by the SAME biased mechanism. More responses doesn't average the pull away — it produces a larger, more confident-LOOKING sample of the exact same skew.

> **Sampling/response bias: when the mechanism selecting your sample is systematically connected to what you're measuring, more data cannot fix it — it only makes the wrong answer more precisely wrong.**

**The critical fork in this whole topic:**
| Problem | Cause | Fixed by more data? |
|---|---|---|
| Block 1: small N | random noise | YES |
| Block 2: sampling bias | systematic selection | NO — needs different collection method (e.g., random sampling of ALL customers, follow up regardless of volunteering) |

**Real-world takeaway:** "we surveyed our users and they love us" is one of the most quietly misleading sentences in business. Behavioral data (what people actually do) sidesteps the selection problem that voluntary surveys can't escape.

### Sibling concept: Survivorship Bias
Studying only the things that "survived" a filter, ignoring what got filtered out. **Classic case:** WWII engineers studied returning bomber planes, saw bullet holes clustered on wings, nearly reinforced the wings — until someone noted planes hit in the ENGINE never made it back to be studied. The missing data IS the answer.

---

## One-line takeaway
> **More data fixes random noise but not systematic bias — a bigger biased sample just measures the wrong number more precisely, which is why "who's missing from this data" matters more than "how much data do I have."**
