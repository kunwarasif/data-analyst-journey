# Day 27: SQL — Complete Reference Notes

**Context:** SQL curriculum was compressed on Day 27 via placement test (4/4 — HAVING, LEFT JOIN + COUNT trap, subqueries, window functions). Prior knowledge from LeetCode + W3Schools practice. This file is the full topic-organized reference for revision. Remaining SQL work: the Olist SQL portfolio project (applying all of this to real multi-table analysis).

---

## Topic 1: The Query Skeleton

```sql
SELECT   column(s)          -- what to see
FROM     table              -- where from
WHERE    condition          -- which rows (BEFORE grouping)
GROUP BY column(s)          -- how to bucket
HAVING   condition          -- which buckets (AFTER grouping)
ORDER BY column [ASC|DESC]  -- how to sort
LIMIT    n                  -- how many rows
```

**Written order is fixed.** But EXECUTION order is different — and this explains most SQL confusion:

```
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

Consequences of execution order:
- WHERE runs before SELECT → WHERE can't see column aliases
- ORDER BY runs after SELECT → ORDER BY CAN use aliases
- WHERE runs before GROUP BY → WHERE cannot filter on aggregates (that's HAVING's job)

**Defaults:** ORDER BY is ascending unless you write DESC. Forgetting DESC = "why is my top 10 showing the bottom 10?"

**pandas mapping:**
| SQL | pandas |
|---|---|
| SELECT cols | `df[['a','b']]` |
| WHERE | boolean mask `df[df['price'] > 500]` |
| GROUP BY | `.groupby()` |
| ORDER BY | `.sort_values()` |
| JOIN | `pd.merge()` |
| LIMIT | `.head(n)` |

---

## Topic 2: Aggregation & GROUP BY

**The five aggregates:** COUNT, SUM, AVG, MIN, MAX.

```sql
SELECT product_id, SUM(price) AS revenue, COUNT(order_id) AS total_orders
FROM order_items
GROUP BY product_id
ORDER BY revenue DESC;
```

**THE GROUP BY RULE (non-negotiable):** every column in SELECT must be either (a) inside an aggregate, or (b) listed in GROUP BY. Why: each result row = one bucket; a bare column with many values per bucket has no single answer → error.

**COUNT variants — interview favorite:**
| Form | Counts |
|---|---|
| `COUNT(*)` | rows (including all-NULL rows) |
| `COUNT(column)` | non-NULL values in that column |
| `COUNT(DISTINCT column)` | unique non-NULL values |

**WHERE vs HAVING:**
```sql
SELECT product_id, SUM(price) AS total_revenue
FROM order_items
WHERE price > 0                    -- filters ROWS before bucketing
GROUP BY product_id
HAVING SUM(price) > 10000          -- filters BUCKETS after aggregating
ORDER BY total_revenue DESC;
```
- WHERE = row filter (no aggregates allowed)
- HAVING = group filter (aggregates allowed)
- Portable habit: repeat the aggregate in HAVING (`HAVING SUM(price) > 10000`) — many dialects reject the alias there

---

## Topic 3: JOINs

```sql
SELECT c.customer_id, COUNT(o.order_id) AS order_count
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id;
```

| JOIN | Keeps |
|---|---|
| INNER | only matches on both sides |
| LEFT | ALL left rows + matches (NULLs where no match) |
| RIGHT | ALL right rows + matches (rarely used; rewrite as LEFT) |
| FULL OUTER | everything from both sides |

**When INNER silently lies:** "orders per customer" with INNER JOIN drops zero-order customers entirely — they vanish from the result. LEFT JOIN keeps them with NULLs.

**THE COUNT TRAP (placement test Q2):** counting per customer after a LEFT JOIN:
- `COUNT(*)` counts the ROW → zero-order customer shows **1** (wrong!)
- `COUNT(o.order_id)` counts non-NULL values → zero-order customer shows **0** (correct)

> The full causal chain: LEFT JOIN creates a row for the unmatched customer → o.order_id is NULL in it → COUNT(column) skips NULLs → 0. COUNT(*) counts the row's existence → 1.

**Table aliases** (`customers c`) — standard practice, keeps multi-table queries readable. Prefix columns (`c.customer_id`) to avoid ambiguity.

---

## Topic 4: Subqueries

**Scalar subquery** (returns one value, used for comparison):
```sql
SELECT product_id, price
FROM order_items
WHERE price > (SELECT AVG(price) FROM order_items)
ORDER BY price DESC;
```
Inner query runs first → produces one number → outer WHERE compares against it. This is how you filter rows against an aggregate (same need that FILTER solved in DAX).

**IN subquery** (returns a list):
```sql
SELECT * FROM orders
WHERE customer_id IN (SELECT customer_id FROM customers WHERE customer_state = 'SP');
```

---

## Topic 5: CTEs (WITH clauses)

Named temporary result sets — the readable alternative to nested subqueries. How real analysts structure complex queries:

```sql
WITH product_revenue AS (
    SELECT p.product_category_name, oi.product_id, SUM(oi.price) AS revenue
    FROM products p
    JOIN order_items oi ON p.product_id = oi.product_id
    GROUP BY p.product_category_name, oi.product_id
)
SELECT * FROM product_revenue WHERE revenue > 5000;
```

- Chain multiple CTEs with commas (see Topic 6 for a two-CTE pipeline)
- Each CTE reads top-to-bottom like pipeline steps — DAX VAR/RETURN is the same idea
- CTE vs subquery: same power, CTEs win on readability and reuse within the query

---

## Topic 6: Window Functions

Aggregates collapse rows; **window functions compute across rows WITHOUT collapsing** — every row keeps its identity and gains a computed value.

```sql
ROW_NUMBER() OVER (PARTITION BY category ORDER BY revenue DESC)
```
- PARTITION BY = restart the numbering per group (like GROUP BY, but no collapse)
- ORDER BY inside OVER = the ranking order

**The ranking family — tie behavior (placement test Q4):**
| Function | Ties get | Sequence with a tie at 2nd |
|---|---|---|
| ROW_NUMBER | arbitrary distinct numbers | 1, 2, 3 (one tied row loses arbitrarily) |
| RANK | same rank, next number skips | 1, 2, 2, 4 |
| DENSE_RANK | same rank, no skip | 1, 2, 2, 3 |

**Which to use is a REQUIREMENTS question:** "top 2, any 2" → ROW_NUMBER. "All products tied for 2nd should appear" → RANK. That's analyst thinking, not syntax.

**The top-N-per-group pattern (canonical, placement test Q4):**
```sql
WITH product_revenue AS (
    SELECT p.product_category_name, oi.product_id, SUM(oi.price) AS revenue
    FROM products p
    JOIN order_items oi ON p.product_id = oi.product_id
    GROUP BY p.product_category_name, oi.product_id
),
ranked AS (
    SELECT product_category_name, product_id, revenue,
           ROW_NUMBER() OVER (PARTITION BY product_category_name ORDER BY revenue DESC) AS rn
    FROM product_revenue
)
SELECT product_category_name AS category, product_id, revenue
FROM ranked
WHERE rn <= 2
ORDER BY category, rn;
```
Why the outer filter: you can't put a window function in its own query's WHERE (execution order — WHERE runs before the window computes). Wrap it, then filter.

**Other common window uses:** running totals `SUM(x) OVER (ORDER BY date)`, prior-row comparison `LAG(x) OVER (ORDER BY date)` for month-over-month.

---

## Topic 7: Practical Extras

**NULL logic:**
- `= NULL` never works → use `IS NULL` / `IS NOT NULL`
- NULLs are skipped by aggregates (AVG ignores them — can silently shift your average)
- `COALESCE(a, b)` = first non-null (pandas `fillna` twin; Power Query if-null-then)

**CASE WHEN** (SQL's if/else — the DAX conditional twin):
```sql
SELECT price,
    CASE WHEN price > 500 THEN 'premium'
         WHEN price > 100 THEN 'mid'
         ELSE 'budget' END AS price_tier
FROM order_items;
```

**Dates (SQLite flavor — varies by dialect):**
```sql
strftime('%Y-%m', order_purchase_timestamp)   -- '2017-03' (the Year-Month twin)
```
PostgreSQL: `TO_CHAR(ts, 'YYYY-MM')` or `DATE_TRUNC('month', ts)`.

**DISTINCT:** `SELECT DISTINCT customer_state FROM customers;`

---

## The 4 Placement-Test Queries (passed cold, Day 27)

1. **HAVING:** products with revenue > 10,000, sorted — GROUP BY + HAVING SUM(price) > 10000
2. **LEFT JOIN + COUNT trap:** all customers incl. zero-order — LEFT JOIN + COUNT(o.order_id), not COUNT(*)
3. **Scalar subquery:** items above overall average price
4. **Top-2-per-category:** two chained CTEs + ROW_NUMBER OVER PARTITION BY + outer WHERE rn <= 2

---

## One-line takeaways
- The written order and the EXECUTION order of a query are different — execution order explains every "why can't I..." error.
- WHERE filters rows, HAVING filters buckets.
- After a LEFT JOIN, COUNT(column) vs COUNT(*) is the difference between correct zeros and silent wrong ones.
- ROW_NUMBER vs RANK is a requirements question, not a syntax question.
