```markdown
# Day 11 + 12: Missing Data and Dates

> Two days combined because together they form the "data quality and time" layer of analyst work. Day 11 covered how to find, drop, and fill missing values with judgment. Day 12 covered date conversion, extraction, arithmetic, and resampling. Both deal with making data analysis-ready before any actual analysis happens.

---

## What you can do after these two days

- Identify missing values across a DataFrame in multiple ways
- Decide between dropping and filling based on context, not reflex
- Apply five filling strategies with reasoning behind each choice
- Convert date strings to proper datetime types
- Extract year, month, day, weekday, quarter from date columns
- Do date arithmetic (differences, durations, before/after)
- Use resample() to group data by time intervals
- Distinguish when to use groupby-by-date-part vs resample

---

## 1. Why Missing Data Matters More Than the Methods

Every real-world dataset has missing values. How you handle them directly changes your analysis:

- Drop too much → small sample, biased results, lost information
- Fill with wrong value → fake data contaminates analysis
- Ignore → totals don't add up, averages mislead
- Handle thoughtlessly → ship reports your CEO trusts that are wrong

The pandas methods are easy. The judgment about *when* to use which method is what makes someone an analyst vs a code-runner.

---

## 2. Where Missing Values Come From

Before fixing, understand the source. The cause shapes the solution.

| Source | Example | What it tells you |
|---|---|---|
| Data not collected | Customer didn't fill age field | Truly unknown — be careful with assumptions |
| Data lost in transit | CSV row corrupted during export | Could be recovered from source |
| Not applicable | "Maternity leave" column for male employees | Design issue, shouldn't be there |
| Failed calculation | Divide by zero | Calculation problem, not data problem |
| Join didn't match | Employee with no sales | One side has no partner row |

The first question in real work is always: "Why is this missing?" Not "How do I get rid of the NaN."

---

## 3. Finding Missing Values

Always run these first on any new dataset.

```python
df.isna()                      # boolean mask for whole DataFrame
df.isna().sum()                # count of NaN per column
df.isna().sum().sum()          # total NaN in whole DataFrame
(df.isna().sum() / len(df)) * 100   # percentage missing per column

df[df.isna().any(axis=1)]      # rows with at least one NaN
df[df.isna().all(axis=1)]      # rows that are completely empty
df[df.notna().all(axis=1)]     # rows with NO missing values

df.columns[df.isna().any()]    # columns that have any NaN
df.isna().sum(axis=1)          # count of NaN per row
```

### The `axis` parameter

- `axis=0` (default) → operate **down rows** = per-column result
- `axis=1` → operate **across columns** = per-row result

Memorize: 0 means go down, 1 means go across.

### Column-% vs row-% — different stories

| Question | Tells you |
|---|---|
| "What % is missing in each column?" | Whether each variable is reliable |
| "What % of rows have at least one missing?" | How much data dropna() would actually delete |
| "Are missing values clustered or spread?" | Junk-rows problem vs spread-thin problem |

Same data can show 25% column-missing but 75% row-loss if missing values are spread strategically. Always look at both views before deciding.

---

## 4. Removing Missing Values — `dropna()`

The basic syntax:

```python
df.dropna()                              # drop ANY row with any NaN
df.dropna(how="all")                     # drop only rows that are completely empty
df.dropna(subset=["age"])                # drop only if "age" is missing
df.dropna(subset=["age", "city"])        # drop if either age OR city missing
df.dropna(axis=1)                        # drop columns with NaN instead of rows
df.dropna(thresh=4)                      # keep only rows with at least 4 non-null values
```

### When to use dropna

- Missing % is small (< 5%) AND random
- Missing rows are clearly junk (test data, errors)
- A specific column is critical and missing makes the row useless
- You have plenty of data and a few drops won't hurt

### When NOT to use dropna

- Missing % is high (>10%) — you'll lose too much
- Missing values cluster in a specific subgroup — selection bias
- Small dataset where every row matters
- You haven't asked WHY the values are missing yet

### The right pattern in production

Almost always use `subset=` to drop only when a specific column matters:

```python
# Bad — drops too aggressively
df.dropna()

# Better — only drop where age is missing for age-based analysis
df_for_age_analysis = df.dropna(subset=["age"])
```

---

## 5. Filling Missing Values — `fillna()` Strategies

### Strategy 1: Fill with a constant string

```python
df["city"].fillna("Unknown")
```

Use when: missing means "don't know" and you want to mark it explicitly.
Don't use when: missing means "zero" or "not applicable."

### Strategy 2: Fill with zero

```python
df["purchase_amount"].fillna(0)
```

Use when: missing genuinely means "no purchase made."
Don't use when: missing means "we don't know" — filling with 0 corrupts sums and averages silently. Common bug: missing salaries filled with 0 → average salary plummets → wrong report to CEO.

### Strategy 3: Fill with mean or median

```python
df["age"].fillna(df["age"].mean())
df["age"].fillna(df["age"].median())   # safer with outliers
```

Use when: missing value is something but you don't know what, and a central value is your best guess.

- **Mean** when data is symmetric and you want to preserve sums
- **Median** when there are outliers or skew

### Strategy 4: Fill with the mode (most common)

```python
df["city"].fillna(df["city"].mode()[0])
```

Use when: categorical data and the most common value is a reasonable guess.
Don't use when: each category matters individually — mode-fill artificially inflates the dominant category.

The `[0]` extracts the first mode in case of ties.

### Strategy 5: Forward fill / backward fill

```python
df["last_purchase_date"].ffill()    # use the value above
df["last_purchase_date"].bfill()    # use the value below
```

Use when: time-series data where missing should be filled with adjacent known values (stock prices, sensor readings).
Don't use when: rows aren't in meaningful order.

---

## 6. fillna Returns a New Series — Always Assign Back

```python
df["age"].fillna(df["age"].mean())              # returns new Series, df unchanged
df["age"] = df["age"].fillna(df["age"].mean())  # saves the change
```

Forgetting the assignment is the #2 most common pandas bug.

### Filling multiple columns at once with a dict

```python
df.fillna({
    "name": "Unknown",
    "age": df["age"].median(),
    "city": "Unknown",
    "purchase_amount": 0,
    "last_purchase_date": "Not Recorded"
})
```

Each key is a column, each value is the fill strategy. One explicit decision per column. This is what real production code looks like — each column gets its own analyst judgment.

---

## 7. The Stage-Appropriate Handling Principle

The most important insight from Day 11.

### Stage 1: Analysis / Computation — keep NaN

- Aggregations (`mean`, `sum`) skip NaN automatically
- Filters can find missing values with `.isna()`
- You don't accidentally count "Unknown" as a real category
- Pandas math propagates NaN correctly

If you fill with "Unknown" too early, `df.groupby("city")` shows Unknown as a real city alongside Delhi, Mumbai. Your "top 5 cities" report becomes wrong. You also lose the ability to measure how much was missing.

### Stage 2: Presentation / Reporting — fill at the end

- Dashboards showing "NaN" look broken to stakeholders
- Excel exports with blanks raise questions
- Power BI shows "(Blank)" which confuses non-technical readers

### The professional pattern

```python
# Stage 1: Analysis — NaN preserved
df_analysis = df.copy()
top_cities = df_analysis.groupby("city")["sales"].sum()
churn_rate = df_analysis["last_purchase"].isna().mean()

# Stage 2: Reporting — fill only at the end
df_display = df_analysis.fillna({"city": "Unknown", "name": "Unknown"})
df_display.to_excel("report.xlsx")
```

This separation is what distinguishes a junior analyst from a senior one. Junior fills at the top of the script and corrupts everything downstream. Senior preserves NaN during analysis and fills only at the display layer.

---

## 8. The Missing Data Decision Framework

When you see NaN, walk through these four questions:

```
1. WHY is it missing?
   ├── Truly unknown (refused, never collected)
   ├── Not applicable
   ├── Data pipeline issue (join failed, system error)
   └── Genuinely zero (no purchase made)

2. WHAT % is missing?
   ├── < 5% → dropna may be acceptable
   ├── 5-30% → fill or use carefully
   └── > 30% → seriously question if column is usable

3. IS the missingness biased?
   ├── Older customers refuse age → mean-fill underestimates
   ├── High-value customers skip surveys → analysis skews low-value
   └── Always check if missingness correlates with the metric you care about

4. WHAT stage am I in?
   ├── Analysis → keep NaN
   ├── Reporting → fill with "Unknown"
   └── Modeling → impute with thoughtful strategy
```

The framework matters more than the methods. In any interview or real-work decision, walk through these four. The right answer changes based on context.

---

## 9. The Bigger Picture on Missing Data

- Missing data is rarely random. It's usually correlated with something meaningful.
- Dropping missing rows often introduces **selection bias** — analysis based on the subset that *did* respond, which differs systematically from those who didn't.
- Imputation strategies make implicit assumptions. Median-fill assumes "missing values are in the middle of the distribution." Mode-fill assumes "missing values match the majority." Each assumption can be wrong.
- The best analysts can defend every imputation choice. Beginners call `.fillna(0)` reflexively and don't realize they corrupted the analysis.

---

## 10. Why Dates Deserve Their Own Day

Date handling is the #1 source of bugs in analyst work. Not because dates are conceptually hard — but because they show up in every dataset and there are a dozen ways to misuse them.

The fundamental rule: **dates in CSVs are strings. Convert them to datetime immediately or your analysis is gimped.**

Without conversion:
- Can't do date arithmetic
- Can't extract year/month/day
- Sort works only by alphabetical accident (only with YYYY-MM-DD format)
- Date comparisons happen as string comparisons (fragile)

With conversion: an entire class of analyst questions becomes answerable.

---

## 11. Converting with `pd.to_datetime()`

### Basic

```python
df["sale_date"] = pd.to_datetime(df["sale_date"])
```

After this, `df["sale_date"].dtype` becomes `datetime64[ns]`.

### With explicit format

For non-standard formats, specify exactly:

```python
pd.to_datetime("15/01/2024", format="%d/%m/%Y")    # Indian DD/MM/YYYY
pd.to_datetime("01/15/2024", format="%m/%d/%Y")    # US MM/DD/YYYY
pd.to_datetime("15-Jan-2024", format="%d-%b-%Y")   # verbose
```

### Format codes you'll use most

| Code | Meaning | Example |
|---|---|---|
| `%Y` | 4-digit year | 2024 |
| `%y` | 2-digit year | 24 |
| `%m` | Month number | 01 |
| `%b` | Month abbreviation | Jan |
| `%B` | Month full name | January |
| `%d` | Day | 15 |
| `%H` | Hour 24-format | 13 |
| `%I` | Hour 12-format | 01 |
| `%M` | Minutes | 30 |
| `%S` | Seconds | 45 |

### Handling errors safely

```python
pd.to_datetime(messy_dates, errors="coerce")
```

Bad values become NaT (Not a Time — the date version of NaN). Always use this in production. The default behavior of crashing on bad input is rarely what you want.

---

## 12. Extracting Date Parts — the `.dt` Accessor

Once a column is datetime type, `.dt` unlocks date-specific methods. Same pattern as `.str` for strings.

```python
df["sale_date"].dt.year             # 2024
df["sale_date"].dt.month            # 1-12
df["sale_date"].dt.month_name()     # "January"
df["sale_date"].dt.day              # 1-31
df["sale_date"].dt.weekday          # 0=Monday, 6=Sunday
df["sale_date"].dt.day_name()       # "Monday"
df["sale_date"].dt.quarter          # 1-4
df["sale_date"].dt.dayofyear        # 1-365
```

### Time components (when data has timestamps)

```python
df["sale_date"].dt.hour
df["sale_date"].dt.minute
df["sale_date"].dt.second
```

### Convenient derived columns

```python
df["is_weekend"] = df["sale_date"].dt.weekday >= 5
df["week_number"] = df["sale_date"].dt.isocalendar().week
```

### Why `.dt` matters

Every "what time pattern" question becomes one groupby once you've extracted the right part:

```python
df.groupby(df["sale_date"].dt.month_name())["amount"].sum()   # sales by month name
df.groupby(df["sale_date"].dt.quarter)["amount"].sum()         # sales by quarter
df.groupby(df["sale_date"].dt.day_name())["amount"].sum()      # weekday effect
```

Without `.dt`, every query needs custom date parsing logic. With it, time analysis is one method call away.

---

## 13. Date Arithmetic

### Calculating "days between" or "time since"

```python
latest = df["sale_date"].max()
df["days_ago"] = (latest - df["sale_date"]).dt.days
```

Two pieces:
1. Subtracting two dates returns a **Timedelta** (a duration)
2. `.dt.days` extracts the day count as a clean integer

Without `.dt.days`, the output is `"10 days 00:00:00"` — ugly in reports.

### Common date arithmetic patterns

```python
today = pd.Timestamp("2024-12-31")

df["days_since_sale"] = (today - df["sale_date"]).dt.days

# Filter by recency
df[df["days_since_sale"] <= 90]                # last 90 days
df[df["days_since_sale"] > 180]                # older than 6 months

# Date range filter
df[(df["sale_date"] >= "2024-04-01") & (df["sale_date"] <= "2024-06-30")]
```

### Adding/subtracting durations

```python
df["sale_date"] + pd.Timedelta(days=30)        # 30 days later (fixed)
df["sale_date"] + pd.DateOffset(months=1)      # same date next month (calendar-aware)
df["sale_date"] - pd.Timedelta(days=7)         # 7 days earlier
```

### Timedelta vs DateOffset

- **`Timedelta`**: fixed durations (days, hours, seconds)
- **`DateOffset`**: calendar-aware (months handle different lengths correctly)

Use Timedelta for "exactly N days." Use DateOffset for "same calendar date in N months."

### Per-group date analysis

```python
span = df.groupby("salesperson")["sale_date"].agg(["min", "max"])
span["days_active"] = (span["max"] - span["min"]).dt.days
```

This gives you each salesperson's first sale, last sale, and active span in three numbers. Real analyst summaries look like this.

---

## 14. Resampling — Time-Series Aggregation

`.resample()` is groupby for time intervals.

### Mechanics

Two requirements:
1. Date column converted to datetime
2. Date column set as the DataFrame's index

```python
df_ts = df.set_index("sale_date")
df_ts["amount"].resample("ME").sum()
```

### Frequency codes (current pandas)

| Code | Period |
|---|---|
| `D` | Day |
| `W` | Week |
| `ME` | Month end |
| `MS` | Month start |
| `QE` | Quarter end |
| `YE` | Year end |
| `H` | Hour |

Older code uses `M`, `Q`, `Y` (deprecated). New code should use `ME`, `QE`, `YE`. The deprecation warnings tell you when to switch.

### Multiple aggregations work like groupby

```python
df_ts["amount"].resample("ME").agg(["sum", "mean", "count"])
```

---

## 15. Resample vs Groupby — The Critical Distinction

```python
# Groupby by month name
df.groupby(df["sale_date"].dt.month_name())["amount"].sum()
# Shows only months with sales

# Resample monthly
df_ts["amount"].resample("ME").sum()
# Shows EVERY month, including empty ones with 0
```

**Resample fills in missing periods. Groupby skips them.**

For business reporting, you usually want resample. An August with zero sales is a meaningful signal — your CEO needs to see it. Groupby would hide that gap.

### When to use each

| Approach | Question it answers |
|---|---|
| `groupby(df["date"].dt.month_name())` | "Which months tend to be biggest?" (seasonal pattern across years) |
| `resample("ME")` | "What happened in each specific month?" (time-series with gaps visible) |

Also critical: with multi-year data, groupby by month name lumps January 2024 + January 2025 together. Resample keeps them separate. The right choice depends on whether you want a seasonal view or a time-series view.

---

## 16. Common Errors with Dates

| Error | Cause | Fix |
|---|---|---|
| Wrong sort order | Column is string, not datetime | `pd.to_datetime()` first |
| `TypeError: unsupported operand type for -: 'str' and 'str'` | Trying to subtract string dates | Convert to datetime first |
| `ValueError: time data '15/01/2024' does not match format '%Y-%m-%d'` | Wrong format string | Match the actual format in data |
| `FutureWarning: 'M' is deprecated` | Using old frequency code | Use `ME` instead |
| NaT appearing unexpectedly | `errors="coerce"` converted bad rows | Inspect with `df[df["date"].isna()]` |

---

## 17. The Mental Model — Why These Two Days Matter Together

Both missing data and date handling are **pre-analysis** concerns. They happen before any "real" analysis.

Most beginner analysts skip this stage. They jump straight to groupby and aggregation. Then their reports come out wrong and they don't know why.

Senior analysts spend the first 30 minutes on any new dataset:

1. `df.isna().sum()` — what's missing
2. `df.dtypes` — what types are columns
3. `df.describe()` — what are the ranges

Only after answering these do they touch analysis. **Skipping data quality leads to confident wrong answers.**

The framework underneath both topics: ask why first, fix the data thoughtfully second, analyze third.

---

## Reflection on Days 11 + 12

- Identification of what to fill is more important than the syntax
- NaN during analysis is a feature, not a bug — preserves the "missingness" signal
- Date columns require explicit conversion immediately on load
- Resample fills missing time periods; groupby doesn't — choose based on whether gaps matter
- The deprecation warning about `M` → `ME` is a real example of why reading pandas warnings matters
- Most bugs in real analyst work come from missing data and bad date handling, not from complex aggregation logic

---

## What's coming next

Day 13 — `apply()`, `map()`, and `lambda` in pandas. Once you can clean data and handle dates, the next unlock is doing **custom logic** on each row or value. This is the bridge between standard pandas methods and arbitrary business rules.

Day 14 — reshaping with `pivot`, `melt`, `stack`, `unstack`.

Day 15 — first real Kaggle dataset project applying everything.

Day 16 — revision day before Power BI begins.
```