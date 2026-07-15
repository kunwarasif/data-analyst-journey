# Days 21–22: DAX — Complete Notes

## Part 1: What DAX Is

DAX = Data Analysis Expressions. Formula language of Power BI.

**Core mental shift from pandas:** In pandas you control the loop. In DAX, you describe *what*, and the engine decides *how* — your formulas react to **filter context**.

| pandas | DAX |
|---|---|
| `df['Revenue'].sum()` | `SUM(Sales[Revenue])` |
| You filter manually | Slicers/visuals filter automatically |

---

## Part 2: Measures vs Calculated Columns

| | Measure | Calculated Column |
|---|---|---|
| Computed | On the fly, per filter context | Once per row, at load |
| Responds to slicers | YES | NO (fixed values) |
| Stored in memory | No | Yes |
| Use for | Almost everything | Row-level attributes (Year, Month Name, Year-Month) |

**Rule:** Default to measures. Columns only for date attributes and row-level labels.

**The error that tells you which you need:**
> "A single value for column X cannot be determined"
= you wrote column logic (needs row context) as a measure. Switch to New Column.

---

## Part 3: Core Measures (Day 21)

```dax
Total Revenue = SUM(olist_order_payments_dataset[payment_value])

Total Orders = DISTINCTCOUNT(olist_orders_dataset[order_id])

AOV = DIVIDE([Total Revenue], [Total Orders])
```

- `DISTINCTCOUNT` = unique values (pandas `.nunique()`)
- `DIVIDE(a, b)` instead of `a / b` → handles divide-by-zero gracefully (returns blank instead of error)
- Best practice: measures reference other **measures** `[Total Revenue]`, not raw columns

**Filter context:** A slicer set to 2017 makes every measure compute on 2017 rows only. No code change. This is the heart of DAX.

---

## Part 4: CALCULATE — Modify Filter Context

```dax
Revenue 2017 = CALCULATE(
    SUM(olist_order_payments_dataset[payment_value]),
    YEAR(olist_orders_dataset[order_purchase_timestamp]) = 2017
)
```

- CALCULATE **overrides** the slicer for that column: slicer says 2018, this card still shows 2017
- The filter argument REPLACES the external filter on that column

## Part 5: ALL — Remove Filters

```dax
Revenue All Years = CALCULATE([Total Revenue], ALL(olist_orders_dataset))
```

- `ALL(table)` strips every filter from that table → grand total regardless of slicers
- CALCULATE + condition = replace filter. CALCULATE + ALL = remove filter.

**The killer use case — % of total:**
```dax
% of Total Revenue = DIVIDE(
    [Total Revenue],
    CALCULATE([Total Revenue], ALL(olist_orders_dataset))
)
```
Numerator respects the row's filter; denominator ignores it. In a table by order_status, rows sum to 100%.

**Formatting rule:** NEVER multiply by 100 for percentages. Keep the value 0.5434, format with the % button (Measure tools → %). Value and display are separate things.

---

## Part 6: Time Intelligence (Day 21)

**Prerequisite — a dedicated Date Table:**
```dax
Calendar = CALENDAR(DATE(2016,1,1), DATE(2018,12,31))
```
Why: timestamp columns have gaps (days with no orders). Time functions need every day present. (pandas equivalent: `pd.date_range()` spine.)

**Add columns (calculated columns — the correct use):**
```dax
Year = YEAR(Calendar[Date])
Month Number = MONTH(Calendar[Date])
Month Name = FORMAT(Calendar[Date], "MMM")
Year-Month = FORMAT(Calendar[Date], "YYYY-MM")
```

**Relationship:** Calendar[Date] → orders[Order Date]. Filters travel one-side → many-side.

**THE BUG WE HIT:** joining Calendar[Date] (pure date) to order_purchase_timestamp (datetime with time). Times don't match → most rows fall into blank year.
**Fix:** Power Query → Add Column → Date → Date Only → join on the new clean column. NEVER destroy source columns; add derived ones.

**YoY measures:**
```dax
Revenue LY = CALCULATE([Total Revenue], SAMEPERIODLASTYEAR(Calendar[Date]))

YoY Growth % = DIVIDE([Total Revenue] - [Revenue LY], [Revenue LY])
```

**The partial-year lesson:** 2016 = partial start (3 months) → 2017 YoY showed 12,264%. Mathematically true, misleading. 2018 = partial end (cuts ~Oct) → growth understated.
> Before trusting any time trend, check that first AND last periods are complete. Compare like-for-like (Jan–Oct vs Jan–Oct).

---

## Part 7: SUMX — Iterators (Day 22)

`SUM` takes ONE column. It cannot do `SUM(price * quantity)`.

```dax
Gross Item Revenue = SUMX(
    olist_order_items_dataset,
    olist_order_items_dataset[price] * 1.05
)
```

- SUMX(table, expression): walks row by row, computes expression per row, sums results
- pandas twin: `(df['price'] * 1.05).sum()`

**When SUM vs SUMX matters:**
| Expression | Same result? |
|---|---|
| price + freight | SUM(a)+SUM(b) = SUMX(t, a+b) — SAME (addition distributes) |
| price × quantity | SUM(a)×SUM(b) ≠ SUMX(t, a×b) — SUMX is the ONLY correct way |

> Row-wise multiplication REQUIRES an iterator. Locked rule.

---

## Part 8: VAR / RETURN (Day 22)

```dax
YoY Growth % =
VAR RevenueLY = CALCULATE([Total Revenue], SAMEPERIODLASTYEAR(Calendar[Date]))
RETURN
    DIVIDE([Total Revenue] - RevenueLY, RevenueLY)
```

Three wins: readability (named steps), performance (computed once, not twice), debuggability (RETURN a variable alone to inspect it).

**Analyst caution learned here:** Max − Avg shows spread EXISTS, not WHY. Could be outliers OR wide even spread — indistinguishable from one number. Verify with median/percentiles/histogram before claiming "outliers."

---

## Part 9: FILTER inside CALCULATE (Day 22)

Simple conditions work directly:
```dax
CALCULATE([Total Revenue], olist_order_items_dataset[price] > 500)
```

But conditions referencing an AGGREGATE need explicit FILTER:
```dax
Above Avg Revenue = CALCULATE(
    [Total Revenue],
    FILTER(
        olist_order_items_dataset,
        olist_order_items_dataset[price] > AVERAGE(olist_order_items_dataset[price])
    )
)
```

> Rule: column vs VALUE → plain CALCULATE. Column vs AGGREGATE/measure → wrap in FILTER.

**Business finding from this block:** items priced >500 = 2.98M of 13.59M = **21.94% of revenue from premium items** (Pareto pattern). Quantify the hunch — don't just claim it.

---

## Part 10: Bridge Tables (Day 22 detour — data modeling)

**Problem:** one entity, TWO valid IDs (alias problem). Orders may reference either. Power BI relationships are one-column-to-one-column; no OR joins.

**Solution:** unpivot all ID columns into a bridge table mapping every alias → one master:

| any_id | master_id |
|---|---|
| A100 | M1 |
| B100 | M1 |

Relationships: Orders[customer_id] → Bridge[any_id] → Customer[master_id]

Build in Power Query: select ID columns → Unpivot (pandas `melt` twin) → rename.

> When keys are messy, fix them in Power Query (data prep), not in the relationship layer.

**Simple fallback-ID variant (one column sometimes null):**
```
Unified_ID = if [customer_id] <> null then [customer_id] else [alternative_customer_id]
```
(pandas: `df['a'].fillna(df['b'])`)

---

## One-line takeaways
- Day 21: Before trusting any time trend, check that first and last periods are complete — a partial year lies through percentages.
- Day 22: FILTER lets CALCULATE handle aggregate-based conditions — and always quantify the hunch.
