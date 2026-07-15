# Day 30 Stats: Relationships — Correlation, Causation & Simpson's Paradox

*(Topic 3 of 7. Continues from Days 27-29: descriptive statistics + distributions complete.)*

## Block 1: What r Actually Measures

```
r ranges from -1 to +1
```

| r value | Meaning |
|---|---|
| +1 | perfect positive — every point on a rising line, zero scatter |
| 0 | no linear relationship — random cloud |
| -1 | perfect negative — every point on a falling line |

**r encodes TWO things at once: direction (sign) and tightness (magnitude).** Not just "is there a relationship" but "how much can I trust a prediction from it."

**Applied to the weight-vs-freight scatter:** roughly +0.7 to +0.8 — strong positive, real spread at every weight level. The spread you saw visually IS r's magnitude made visible.

---

## The Force-Test (portable causation check)

> **Imagine forcing the "cause" to change while holding everything else fixed. Would the other variable actually move?**

**Ice cream ↔ drowning (r = 0.8, real data):** Ban ice cream — does drowning stop? No. Both are driven by an external third factor: **summer/hot weather** pushes both up independently. Neither touches the other.

> **Confounder** = a variable that influences both things being correlated, creating a relationship with NO direct connection between them.

**The same test confirms real causation too:** force Olist packages to weigh less — does freight cost drop? Yes, mechanically, no third factor needed.

---

## Three Relationship Shapes (r looks identical on paper for all three — only reasoning about mechanism sorts them)

| Pattern | Example | Force-test result |
|---|---|---|
| **Confounder** | ice cream ↔ drowning | forcing one does nothing — external cause drives both, they never touch |
| **Direct/shared mechanism** | freight_value ↔ delivery_time | forcing the shared root cause (weight/distance) genuinely moves both — the connection is INSIDE the mechanism, not hiding behind it |
| **Feedback loop** | income ↔ smartphone ownership | BOTH directions real simultaneously — income enables phones AND phones enable income (job access, information) |

**Key distinction:** confounder = external bystander driving two otherwise-unconnected things. Shared direct cause = both variables downstream of the same real mechanism (not "external," structurally connected).

**Interview note:** "correlation isn't causation" alone signals nothing. Sorting a given r into one of these three buckets — and defending the sort — is what's actually being tested.

---

## Block 2: Simpson's Paradox

**Setup — two sellers' on-time rates:**

| Seller | Small items | Large items |
|---|---|---|
| A | 95% (100 orders) | 60% (900 orders) |
| B | 90% (900 orders) | 50% (100 orders) |

**Seller A wins EVERY category.** Small: A beats B. Large: A beats B.

**Combined (weighted by volume):**
- A: (0.95×100 + 0.60×900) / 1000 = 635/1000 = **63.5%**
- B: (0.90×900 + 0.50×100) / 1000 = 860/1000 = **86.0%**

**Seller B wins overall by 22.5 points** — despite losing every subgroup.

### The mechanism
Seller A did most volume (900/1000) in the HARD category (large items, low rates everywhere). Seller B did most volume (900/1000) in the EASY category (small items, high rates everywhere). The combined number mostly measures **"who sold more of the easy stuff,"** not "who delivers better."

> **Simpson's Paradox: a trend that holds in every subgroup can REVERSE when combined, if group sizes are unequal in a way correlated with the outcome.**

### Why this is the interview favorite
Nearly every "which is better" comparison (regions, product lines, campaigns, sellers) can hide a mix-shift like this. The screened-for instinct: ask **"is the sample composition even comparable?"** before trusting a blended headline number — most candidates recite "correlation ≠ causation" on command; almost none spontaneously check the mix.

### The fix — connects back to Day 21
Don't trust the blended number. Go back to **like-for-like**: small-vs-small, large-vs-large. A stratified breakdown IS a like-for-like comparison. Same discipline from the partial-year YoY trap (Day 21), different costume.

---

## One-line takeaway
> **A number that wins in every subgroup can lose overall if the subgroups aren't equally sized — always check the mix before trusting the headline.**
