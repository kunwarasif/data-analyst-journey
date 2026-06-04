```markdown
# Day 9 + 10: Pandas Groupby and Merging

> Two days combined because together they form the heart of pandas. Groupby summarizes within a table. Merging combines across tables. Once you have both, you can answer almost any analytical question from any tabular dataset.

---

## What you can do after these two days

- Group a DataFrame by one or more columns and aggregate each group
- Apply multiple aggregations at once with `.agg()`
- Name your aggregated columns cleanly
- Iterate over groups when needed
- Understand the Split-Apply-Combine mental model
- Merge two DataFrames using a shared key
- Use all four join types: inner, left, right, outer
- Chain multiple merges to combine three or more tables
- Generate hypotheses about why numbers move
- Catch your own errors by checking results against the underlying data

---

## 1. The Split-Apply-Combine Model

This is the foundation of `groupby`. Internalize it before learning syntax.

### What `df.groupby("salesperson")` does conceptually

Pandas mentally splits your DataFrame into smaller DataFrames — one per unique value in the grouping column.

```
Original                     After groupby("salesperson")
sales | amount               Group: Rohit          Group: Amit          Group: Neha
------+-------               sales | amount        sales | amount       sales | amount
Rohit | 50000        ──►     Rohit | 50000         Amit  | 75000        Neha  | 80000
Amit  | 75000                Rohit | 60000         Amit  | 45000        Neha  | 90000
Rohit | 60000                Rohit | 55000
Neha  | 80000
Amit  | 45000
Neha  | 90000
Rohit | 55000
```

### The 3-step pattern

1. **Split** — divide rows into groups based on the grouping column
2. **Apply** — run a function (sum, mean, count, etc.) on each group
3. **Combine** — collect the results into a single output

When you write `df.groupby("salesperson")["amount"].sum()`, pandas does all three steps at once.

### See the split with `.groups`

```python
grouped = df.groupby("salesperson")
print(grouped.groups)
# {'Amit': [1, 4], 'Neha': [3, 5], 'Rohit': [0, 2, 6]}
```

Each group name maps to the row indices that belong to it. Useful for verifying what's in each group.

### Get one specific group

```python
print(grouped.get_group("Rohit"))
# Returns all 3 rows where salesperson is Rohit
```

### Iterate over groups (rare but useful)

```python
for name, group in df.groupby("salesperson"):
    print(f"--- {name} ---")
    print(group)
```

Each iteration gives you (group_name, sub_dataframe). Use this when you need custom processing per group.

---

## 2. Basic Groupby with One Aggregation

### The pattern

```python
df.groupby(<group_column>)[<value_column>].<aggregation>()
```

Read: *"Group by X. For each group, take column Y and apply Z."*

### Examples — your entire Week 1 collapses

```python
df.groupby("salesperson")["amount"].sum()    # total per person
df.groupby("salesperson")["amount"].mean()   # average per person
df.groupby("salesperson")["amount"].count()  # number of sales per person
df.groupby("salesperson")["amount"].max()    # biggest sale per person
df.groupby("salesperson")["amount"].min()    # smallest per person
df.groupby("salesperson")["amount"].std()    # spread per person
df.groupby("month")["amount"].sum()          # total per month
```

Every helper function you wrote in Week 1 is now one line. The pattern stays the same; just swap the columns and the aggregation.

### The result is a Series

```python
result = df.groupby("salesperson")["amount"].sum()

result.index    # Index(['Amit', 'Neha', 'Rohit'])  ← group names become the index
result.values   # array([120000, 170000, 165000])  ← aggregated numbers
```

**Every groupby result is a Series indexed by group labels.** Burn that in.

### Sort the groupby result

For "top N" reporting:

```python
df.groupby("salesperson")["amount"].sum().sort_values(ascending=False)
# salesperson
# Neha     170000
# Rohit    165000
# Amit     120000
```

`.sort_values()` works directly on the groupby output because it's a Series.

### Get the top group with `.idxmax()`

```python
df.groupby("salesperson")["amount"].sum().idxmax()
# 'Neha'
```

`.idxmax()` returns the *index label* of the maximum. Since the index is your group names, this gives you the top group name directly.

`.idxmin()` does the same for the smallest.

---

## 3. Multiple Aggregations with `.agg()`

When you need several statistics per group at once.

### List-based aggregation

```python
df.groupby("salesperson")["amount"].agg(["sum", "mean", "count"])
```

Result is a DataFrame:
```
                sum      mean   count
salesperson
Amit         120000   60000.0      2
Neha         170000   85000.0      2
Rohit        165000   55000.0      3
```

### Named aggregations (preferred)

```python
df.groupby("salesperson")["amount"].agg(
    total="sum",
    average="mean",
    transactions="count"
)
```

Cleaner column names. This is the modern pandas style. Use this in production code.

### Different aggregations on different columns

When you have multiple numeric columns:

```python
df.groupby("salesperson").agg(
    total_amount=("amount", "sum"),
    avg_amount=("amount", "mean"),
    num_sales=("amount", "count")
)
```

Each named aggregation takes a tuple: `(column_to_aggregate, function_to_apply)`. Memorize this syntax — you'll use it constantly.

---

## 4. Grouping by Multiple Columns

### Multi-level grouping

```python
df.groupby(["month", "salesperson"])["amount"].sum()
```

Pandas now groups by combinations of (month, salesperson). The result has a **MultiIndex** — two levels of grouping:

```
month  salesperson
Feb    Amit           45000
       Neha           80000
       Rohit          60000
Jan    Amit           75000
       Rohit          50000
Mar    Neha           90000
       Rohit          55000
```

### When to use it

When questions cross multiple dimensions:
- "Sales by region AND product"
- "Revenue by quarter AND department"
- "Errors by service AND severity"

### Flatten the MultiIndex with `.reset_index()`

```python
df.groupby(["month", "salesperson"])["amount"].sum().reset_index()
```

Result becomes a regular DataFrame with month and salesperson as normal columns. Easier to work with for further analysis.

**`.reset_index()` is one of the most-used pandas methods.** Use it whenever you need to "promote" an index back to a normal column.

---

## 5. Merging — The Mental Model

### Why merge

Real data lives in multiple tables. Sales records track sales, employee records track employees, product records track products. **Normalized design** — each fact lives in one place. When you need to combine them, you merge using a shared key.

### Vocabulary

- **Key column** — the column that exists in both tables and matches rows. Often called join key.
- **Left table / Right table** — in `pd.merge(left, right, ...)`, left is first.
- **Join type** — how to handle rows that don't match.

### The visual

```
sales (left)              employees (right)              Merged result
sale_id sales_id amount   sales_id name  city            sale_id sales_id amount name  city
   1      101    50000       101  Rohit Delhi              1       101    50000  Rohit Delhi
   2      102    75000       102  Amit  Mumbai             2       102    75000  Amit  Mumbai
   3      101    60000       103  Neha  Bangalore          3       101    60000  Rohit Delhi
                                                          
                            JOIN ON sales_id
```

Each sale gets its employee info attached. Rohit appears 3 times because he made 3 sales. **Each row in the result represents one sale, widened with reference info.**

### The fact/dimension pattern

- **Fact table** (sales) — one row per event
- **Dimension tables** (employees, products) — one row per entity

After merging, each fact row has all its dimension info filled in. This is the foundation of every analytics database and BI tool.

---

## 6. Inner Join — The Default

### Syntax (two equivalent forms)

```python
# Function form
pd.merge(sales_df, employees_df, on="salesperson_id", how="inner")

# Method form (preferred for chaining)
sales_df.merge(employees_df, on="salesperson_id", how="inner")
```

### What inner does

**Keeps only rows where the key exists in BOTH tables.**

- Employees who sold AND are in employee records → kept
- Sales with unknown employee IDs → dropped
- Employees who didn't sell → dropped

### When key columns have different names

```python
sales_df.merge(
    employees_df,
    left_on="emp_id",
    right_on="salesperson_id",
    how="inner"
)
```

Use `left_on=` and `right_on=` when column names differ between tables. Common in real data.

---

## 7. The Four Join Types

| Join | What it keeps |
|---|---|
| `inner` | Only rows that match on BOTH sides |
| `left` | All rows from left table, plus matches from right |
| `right` | All rows from right table, plus matches from left |
| `outer` | All rows from EITHER table |

### Visual mental model

```
        LEFT (sales)        RIGHT (employees)
        ┌─────────┐         ┌─────────┐
        │  101    │ ──────► │  101    │
        │  102    │ ──────► │  102    │
        │  103    │ ──────► │  103    │
        │         │         │  104    │ (Vikram — only in right)
        └─────────┘         └─────────┘

INNER:  101, 102, 103       (intersection)
LEFT:   101, 102, 103       (everything from left)
RIGHT:  101, 102, 103, 104  (everything from right, Vikram included)
OUTER:  101, 102, 103, 104  (everything from both)
```

### When to use each

| Goal | Join type |
|---|---|
| "Show me sales with employee info, ignore unknowns" | inner |
| "Show me every sale, even if I don't have employee info for it" | left |
| "Show me every employee, even if they haven't sold yet" | right |
| "Audit everything — show me both unmatched sales AND unsold employees" | outer |

### Finding gaps with right join + `.isna()`

```python
result = sales_df.merge(employees_df, on="salesperson_id", how="right")
print(result[result["sale_id"].isna()])
# Shows employees who haven't sold yet
```

This is the standard pattern for finding "what's missing":
- Customers who haven't ordered
- Products that haven't sold
- Employees with no activity

---

## 8. The NaN-Forces-Float Gotcha

When a join can't find a match, pandas inserts `NaN` (Not a Number).

But there's a problem: **NaN is technically a float type.** `int64` columns can't hold NaN. So when a join introduces NaN into an integer column, **pandas converts the whole column to float**.

Result:
```
sale_id  → 1.0, 3.0, 7.0, ...      ← was int64, now float64
amount   → 50000.0, 60000.0, ...   ← same
```

### Why this matters

- Outputs look ugly in reports (`101.0` instead of `101`)
- ID lookups using these as keys will break if you compare to integers elsewhere
- Type checks fail (`isinstance(x, int)` returns False)

### Fix in production code

```python
result["sale_id"] = result["sale_id"].astype("Int64")
```

Capital `Int64` is pandas' nullable integer type — it can hold NaN AND stay an integer. The lowercase `int64` cannot.

---

## 9. Chaining Multiple Merges

Real datasets have many tables. Chain merges to combine them.

```python
full_view = (
    sales_df
    .merge(employees_df, on="salesperson_id")
    .merge(products_df, on="product_id")
)
```

Three tables become one analytical view. Each row now has:
- Sale info (sale_id, amount, month)
- Employee info (name, city, commission_rate)
- Product info (product_name, category)

This is the **single most common analyst workflow.** Start with the fact table, then merge in each dimension you need. You'll do variations of this constantly.

---

## 10. The Filter → Merge → Group → Aggregate Pattern

This is the shape of nearly every real analyst question.

```python
# "Who sold the most Electronics?"
(
    sales_df
    .merge(employees_df, on="salesperson_id")
    .merge(products_df, on="product_id")
    .query("category == 'Electronics'")
    .groupby("name")["amount"]
    .sum()
    .sort_values(ascending=False)
)
```

Read it like a sentence:

> Start with sales.
> Join in employee info.
> Join in product info.
> Filter to Electronics only.
> Group by name.
> Sum the amounts.
> Sort highest first.

**Every analyst report is some variation of this chain.** The methods change, the order changes, but the shape is always: combine → filter → group → summarize.

---

## 11. The Analyst Mindset — Most Important Section

Beyond pandas syntax, here's what Day 10 taught about thinking like an analyst.

### Code is not analysis

> *"Just running query or writing code is not analysis. Right question and finding right cause of result is the analysis."*

Running pandas is a tool. The job is **figuring out which question to ask**. A senior analyst spends maybe 20% of their time writing code. The other 80% is generating hypotheses, designing investigations, and explaining results to stakeholders.

### From observation to question

An observation says: "X happened."
A question asks: "Why did X happen, and what do we do about it?"

Most beginners stop at observations: "Sales dropped 15%."
Analysts ask: "Sales dropped 15% — is this because of fewer transactions, smaller order sizes, or losing our best salesperson?"

### Hypothesis-driven analysis

When you see a number move, generate 2-3 specific causes that could explain it. Then design how to test each one.

| Hypothesis | How to test |
|---|---|
| Campaign was stopped | Compare campaign-active vs inactive months |
| Harsh weather | Cross-reference with weather data or compare similar unaffected regions |
| Free service discontinued | Segment customers who used the service vs didn't, check churn |

**Don't just find correlations. Generate causes, then prove which one.**

### Verify your own outputs against the data

When pandas tells you "Amit sold the most Electronics," check it manually:
- Which sales were Electronics? (Look at product IDs)
- Sum each salesperson's contribution
- Confirm the answer matches

Pandas isn't wrong — but you might read the output wrong. Verification protects against confident misreads. This is the #1 source of "wrong numbers in reports sent to leadership."

### Stand your ground when you've checked the data

If a senior person (or AI teacher) tells you a number is wrong, and you've verified it from the source — **don't fold.** Show your math. Be respectful but firm. Senior people are wrong sometimes. The data is the data.

This came up today. You corrected me, you were right, the math proved it. Remember that you can do this.

---

## Reflection on Day 9 + 10

- `groupby` collapses Week 1's accumulator patterns to one line — but understanding what's underneath (Split-Apply-Combine) makes it permanent, not memorized.
- Merging makes analysis possible across multiple sources. Real analyst work is mostly this.
- NaN forces floats. Memorize this — it'll bite you in real datasets constantly.
- The job isn't running queries. The job is asking the right questions and finding root causes.
- Verify outputs against underlying data. Always.
- Stand your ground when the math is on your side.

