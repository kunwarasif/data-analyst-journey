# Stats Day 1: Mean vs Median & Percentiles — Complete Notes

## Block 1: When the Average Lies

### The core mechanism
The **mean** is democratic but corruptible — every value votes, and one extreme value can buy the election. The **median** only cares about position — the middle stays the middle no matter how extreme the tails get. Statisticians call this being **robust to outliers**.

**Worked example:** 4, 5, 7, 100
- Mean = 116 ÷ 4 = **29** → describes nobody (nothing in the data is near 29)
- Median = (5+7)/2 = **6** → sits where the data actually lives

### THE FEWNESS CORRECTION (key mistake fixed this session)
Wrong intuition: "the high values are few, so the median ends up near the mean."
Truth: **fewness doesn't neutralize outliers — fewness IS the outlier situation.** One value out of four (the 100) split mean and median by 5×. Few extreme values drag the MEAN; the MEDIAN stays anchored with the crowd.

### The general law
> **The mean chases the tail. The median stays with the crowd.**
- Right skew (tail of high values): mean > median — prices, incomes, house values
- Left skew (tail of low values): mean < median — e.g., exam scores where most score 85–95 and a few score 10–20
- Symmetric: mean ≈ median

### Applied to Olist
- AOV (mean) = 136.68, but the price distribution is right-skewed (long expensive tail)
- → median order value sits BELOW the mean (~100 on real Olist data, ~25% under)
- **Business consequence:** planning around "typical order ≈ 137" calibrates inventory, marketing spend, and free-shipping thresholds around a number no typical customer actually spends. The typical order is ~100.

### The reverse diagnostic (interview move)
Given ONLY mean and median, you can detect skew without seeing the data:
> "Mean is 137 but median is 100 → long right tail exists → I'd want the distribution before trusting the mean."

---

## Block 2: Percentiles — Describing the Whole Distribution

### The reframe
**The median is just the 50th percentile.** General definition: the **Nth percentile** = the value below which N% of the data falls.
- P25: quarter of values below | P50: median | P75: three-quarters below | P90: the expensive end | P99: extreme tail

Mean and median are two dots. Percentiles give you the **shape**.

### The five-number summary
Min, P25, P50, P75, Max — five numbers that sketch an entire distribution. This is what a box plot draws, and the standard "describe this data" interview answer.

### IQR — the defensible outlier detector
> **IQR = P75 − P25** (the width of the middle 50%, where the crowd lives)
> **Fence rule:** outlier if below P25 − 1.5×IQR or above P75 + 1.5×IQR

Why it works: the fence is built from the MIDDLE of the data — the part outliers can't corrupt. The outliers can't move the goalposts used to catch them (same robustness logic as the median).

**Worked example (Olist-style):** P25=40, P50=75, P75=135
- IQR = 135 − 40 = 95
- Upper fence = 135 + 1.5×95 = 135 + 142.5 = **277.5**
- Item priced 350 → above 277.5 → **flagged as outlier**

**The upgrade this delivers:** Max−Avg said "spread exists, probably outliers, should confirm." IQR fence says "anything above 277.5 is an outlier by the 1.5×IQR rule." Suspicion → criterion.

**Interview habit:** show the arithmetic chain, not just the answer. "IQR is 95, so 135 + 142.5 = 277.5" earns full credit; "277.5" alone earns partial.

### Percentile spacing reveals shape (the deep insight)
P25→P50 gap = 35, but P50→P75 gap = 60 → **widening gaps going up** = data thins out as you climb = each quarter of the data occupies more price-space = **right skew announcing itself**.
- Equal-width gaps → symmetric distribution
- Widening upward → right tail
- Project the widening past P75 → you can feel P90 ≈ 250, P99 > 1,000 coming

### Honest business communication (P90 thinking)
"Average delivery: 12 days" hides everything. "Median 10, P90 = 24 days" tells the truth: most orders fine, but 1 in 10 customers waits 3+ weeks — and the P90 customer writes the angry review. The mean never shows you that person.

---

## Skew-detection toolkit (two independent instruments, same diagnosis)
1. **Mean vs median gap** — mean above median → right tail exists
2. **Percentile spacing** — widening gaps upward → right tail exists

Five percentiles let an analyst sketch a distribution blind. That's what "describing data" means.

---

## One-line takeaways
- Block 1: The mean chases the tail, the median stays with the crowd — and a gap between them is a skew detector that works without seeing the data.
- Block 2: Percentile spacing is the distribution's confession — equal gaps say symmetric, widening gaps say skewed, and the IQR fence turns "probably an outlier" into a defensible threshold.
