# Day 23: Visualizations — Complete Notes

## Part 1: The Core Principle

> A chart's job isn't to show data. It's to **answer a question** so fast the viewer barely notices they asked.

Pick charts by **what the data is doing**, not by what looks nice.

---

## Part 2: The Chart-Selection Framework

| The question | Chart |
|---|---|
| **Comparison** — "which category is biggest?" | Bar / Column |
| **Trend over time** — "is it growing?" | Line |
| **Part-to-whole** — "what share of total?" | Stacked bar / Donut (2–3 slices max) |
| **Relationship** — "does X drive Y?" | Scatter |

These four cover ~90% of real dashboards.

**The pie/donut rule:** the human eye is bad at comparing angles, good at comparing lengths. Donut only for 2–3 parts of a whole. More categories → bar chart.

---

## Part 3: Line Chart — Continuous Trend (and the traps)

**Goal:** monthly revenue trend across 2016–2018.

### Trap 1 — Wrong grain
- Year level = 3 points (too coarse, no story)
- Day level = 1,096 points (noise)
- **Month = goldilocks.** Match the time grain to the question.

### Trap 2 — Seasonal profile vs continuous trend (IMPORTANT)
Putting only `Month` on the axis **aggregates all Januaries together** (Jan 2017 + Jan 2018 = one point). That answers "which month of the year is strongest?" (seasonality) — NOT "how did revenue change over time?" (trend).

**Fix:** create a Year-Month column in the Calendar table:
```dax
Year-Month = FORMAT(Calendar[Date], "YYYY-MM")
```
- Must be a **calculated column** (row-by-row), NOT a measure
- Error "A single value for column Date cannot be determined" = you tried to make it a measure

### Trap 3 — Axis sorted by measure instead of time
Symptom: X-axis labels scrambled (2017-11, 2018-04, 2016-09...), line looks like a smooth fake decline — because it's literally sorted by revenue descending.
**Fix:** chart "..." menu → Sort axis → Year-Month → Sort ascending.

### Trap 4 — Month Name sorts alphabetically
Apr, Aug, Dec... **Fix:** select Month Name column → Column tools → **Sort by Column** → Month Number.

### Reading the finished trend — edge effects
- 2016 ramp-up from zero = partial-year START (Olist barely launched)
- 2018 cliff at the end = partial-year END (data cuts ~Oct 2018), NOT a business collapse
> **The first and last data points lie.** Always ask: real, or incomplete period? Present honestly: trim or annotate partial months.

---

## Part 4: Bar Chart — Top N Ranking

**Build:** Horizontal bar (category names read better sideways). Y-axis: category. X-axis: Total Revenue.

**The Top N filter (dynamic ranking):**
Filters pane → category field → Filter type: **Top N** → Top 5 by Total Revenue.

> Why Top N beats manual selection: it's dynamic. Slicer to 2017 → the Top 5 RECALCULATES for 2017. Never hardcode.

**Sorting rule:** a ranked chart must be sorted largest-first — the reader's eye lands on the winner with zero effort.

**Olist finding:** Top 5 categories (beleza_saude ~1.25M → informatica_acessorios ~0.92M) are CLOSE to each other → diversified revenue, no single-category dependence → healthy, lower-risk base.

---

## Part 5: Scatter — Relationships

**Build:** X-axis: product_weight_g (Average). Y-axis: freight_value (Average). **Details: product_category_name** — without Details, the scatter collapses to ONE dot. Details "explodes" it into one dot per category.

**Question tested:** do heavier products cost more freight?
**Result:** clear bottom-left → top-right pattern = positive relationship. Confirmed by data, not assumed.

**Two analyst readings:**
1. **Trend but with spread** — at 3K g, freight ranges ~20–36. Weight is *a* driver, not *the* driver. Say "positively correlated with meaningful spread," never "determines."
2. **Cluster + outliers** — most categories bottom-left; a few heavy categories top-right = where freight eats margin.

---

## Part 6: Donut — Part-to-Whole (and the honesty rule)

**Build:** Legend: order_status. Values: Count of order_id.

**Finding:** delivered ≈ 97% by count AND by revenue → no skew (canceled orders aren't disproportionately high-value).

**THE HONESTY RULE:**
> A part-to-whole chart must show the ACTUAL whole. Dropping categories makes the remaining slices lie — they become shares of a subset you silently chose.

- Want "delivered vs canceled only"? → label it explicitly ("final-outcome orders only")
- Cancellation rate specifically? → don't use a donut at all; a 0.6% slice is invisible. Use a **card**: "Cancellation rate: 0.6%"

---

## One-line takeaway
> Pick the chart by the question (comparison→bar, time→line, relationship→scatter, share→donut), then read the story — especially the edges and the spread — instead of trusting the shape.
