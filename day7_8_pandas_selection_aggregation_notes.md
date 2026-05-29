
# Day 7 + 8: Pandas Selection, Filtering, Sorting, and Aggregation

> Two days combined because they form one cohesive skill: pulling data out of a DataFrame and summarizing it. Day 7 covered selection and filtering (getting the right rows and columns). Day 8 covered sorting and aggregation (ordering and summarizing). Together: the foundation of all pandas analysis.

---

## What you can do after these two days

- Select columns from a DataFrame (single column, multiple columns, dynamic lists)
- Select rows by position with `.iloc` or by label with `.loc`
- Filter rows using boolean masks
- Combine multiple filter conditions with `&`, `|`, `~`
- Use `.isin()` and `.str.contains()` for category and text matching
- Sort data by one or multiple columns, ascending or descending
- Compute totals, averages, max, min, count, std, median, quartiles
- Find unique values, count distinct values, count frequencies
- Chain operations: filter → sort → aggregate

---

## 1. Column Selection — The Bracket Rules

### Single column → Series

```python
df["amount"]
```

Returns a one-dimensional Series. Use when you need just one column for math, filtering, or plotting.

### Multiple columns → DataFrame

```python
df[["salesperson", "amount"]]
```

Returns a smaller DataFrame. Note the double brackets — outer `[]` is pandas indexing, inner `[ ]` is a Python list of column names.

### Dynamic column selection — using a list variable

```python
columns_i_want = ["salesperson", "amount"]
df[columns_i_want]
```

Most production code looks like this — build a list dynamically (from config, user input, or another step), then pass it to filter columns.

### The critical mental shift

> **Single thing → Series. List of things → DataFrame.**
> The brackets define the *shape* of what comes back, not the *count* of items.

Even `df[["amount"]]` (single column wrapped in a list) returns a DataFrame, not a Series. The list says "give me a 2D structure."

### Do NOT use `df.amount`

Yes, it works. But:
- Breaks on column names with spaces
- Breaks when column name matches a pandas method
- Reads cleaner but introduces bugs

Always use `df["column_name"]`. Pick the safe pattern.

### Inspecting columns

```python
df.columns                # Index object of column names
list(df.columns)          # Python list of column names
"amount" in df.columns    # check if a column exists
```

---

## 2. Row Selection — `.loc` vs `.iloc`

### The two indexing systems

Pandas tracks **both**:
- **Position**: 0, 1, 2, 3... (always present, regardless of index labels)
- **Label**: whatever you set the index to (defaults to 0, 1, 2... but can be names, dates, anything)

| Tool | Selects by | Mnemonic |
|---|---|---|
| `.iloc[]` | Position (integer) | **i**nteger location |
| `.loc[]` | Label | **l**abel location |

When index is default (0, 1, 2...), both look identical. They diverge when you set a custom index.

### Examples on default-indexed DataFrame

```python
df.iloc[3]              # 4th row (position 3)
df.loc[3]               # row with index label 3 — same result here

df.iloc[3, 1]           # row 3, column 1 (position-based)
df.loc[3, "amount"]     # row labeled 3, column named "amount"
```

### Slicing — THE famous gotcha

```python
df.iloc[1:4]    # positions 1, 2, 3 → 3 rows (exclusive end, like Python lists)
df.loc[1:4]     # labels 1, 2, 3, 4 → 4 rows (INCLUSIVE end — pandas quirk)
```

**`.loc` slicing is inclusive on both ends.** This is the #1 surprise in pandas. Memorize this difference. When debugging "why am I getting one extra row," check if you used `.loc` for slicing.

### Selecting multiple rows with lists

```python
df.iloc[[0, 2, 4]]                          # rows at positions 0, 2, 4
df.loc[[0, 2, 4], ["salesperson", "amount"]] # specific rows AND columns
```

### Default rule

> **Use `.loc` for production code.** Positions change if you sort or filter, but labels stay with their rows. Label-based access is more stable.

Reserve `.iloc` for cases where you genuinely need "give me the first N rows by position" — like `df.iloc[:10]` for a preview.

---

## 3. Boolean Filtering — The Most Important Pattern in Pandas

### The mental model

1. Build a **boolean mask** — a Series of True/False, one per row
2. Use the mask inside `df[...]` to filter

### Step by step

```python
# Step 1: build the mask
mask = df["amount"] > 60000
print(mask)
# 0    False
# 1     True
# 2    False
# ...

# Step 2: use it to filter
big_sales = df[mask]
```

### Combined into one line

```python
big_sales = df[df["amount"] > 60000]
```

Read as: *"Give me the rows of df where the amount column is greater than 60000."*

### Common operators

```python
df["amount"] > 60000       # greater than
df["amount"] >= 60000      # greater than or equal
df["amount"] < 60000       # less than
df["amount"] <= 60000      # less than or equal
df["amount"] == 60000      # equals (DOUBLE EQUALS — single = is assignment)
df["amount"] != 60000      # not equals
```

### Filtering by category — `.isin()`

```python
df[df["month"].isin(["Jan", "Feb"])]
```

Pandas equivalent of `if x in [list]`. Use when filtering by multiple discrete values.

### Filtering text — `.str` accessor

```python
df[df["salesperson"].str.startswith("R")]    # starts with R
df[df["salesperson"].str.endswith("a")]      # ends with a
df[df["salesperson"].str.contains("e")]      # contains 'e'
df[df["salesperson"].str.lower() == "neha"]  # case-insensitive match
```

The `.str` accessor unlocks all string methods on a text column. You'll use this constantly for cleaning real-world data.

---

## 4. Combining Multiple Conditions

### The three operators

| Python | Pandas Series |
|---|---|
| `and` | `&` |
| `or` | `\|` |
| `not` | `~` |

Pandas needs special operators because regular Python `and`/`or`/`not` don't work on Series.

### CRITICAL: Wrap each condition in parentheses

```python
df[(df["salesperson"] == "Rohit") & (df["amount"] > 50000)]
```

Without parentheses, Python's operator precedence breaks this. The `&`, `|`, `~` operators have *higher* priority than comparison operators like `==`, `>`, `<`. So:

```python
# WRONG — Python tries to do: 50000 & df["salesperson"]
df[df["amount"] > 50000 & df["salesperson"] == "Rohit"]

# RIGHT
df[(df["amount"] > 50000) & (df["salesperson"] == "Rohit")]
```

**Make it a habit:** every condition gets wrapped in `()`. Always.

### Multi-condition examples

```python
# AND — Rohit's sales in January
df[(df["salesperson"] == "Rohit") & (df["month"] == "Jan")]

# OR — sales in January or March
df[(df["month"] == "Jan") | (df["month"] == "Mar")]

# Cleaner OR with isin()
df[df["month"].isin(["Jan", "Mar"])]

# NOT — sales NOT in March
df[~(df["month"] == "Mar")]

# Same thing, simpler
df[df["month"] != "Mar"]

# Three-way combo
df[
    (df["salesperson"] == "Rohit") &
    (df["month"] == "Jan") &
    (df["amount"] > 40000)
]
```

### Style tip

For 3+ conditions, spread across lines with the boolean operator at the start of each line. Easier to read, easier to comment, easier to debug.

---

## 5. Sorting

### Sort by a column

```python
df.sort_values(by="amount")                   # ascending (default)
df.sort_values(by="amount", ascending=False)  # descending
```

Returns a new DataFrame. Original is unchanged.

### Sort by multiple columns

```python
df.sort_values(
    by=["month", "amount"],
    ascending=[True, False]
)
```

First sorts by month ascending, then within each month sorts by amount descending.

### Sort by index

```python
df.sort_index()                # restore original order
df.sort_index(ascending=False) # reverse index order
```

Most useful when index is dates or named labels.

### Avoid `inplace=True`

```python
# Don't do this — being phased out in modern pandas
df.sort_values(by="amount", inplace=True)

# Do this instead
df_sorted = df.sort_values(by="amount")
```

Assigning to a new variable is clearer, easier to debug, and the pandas team is moving away from `inplace`.

### Chaining — top 3 highest sales

```python
df.sort_values(by="amount", ascending=False).head(3)
```

Sort the DataFrame, then take the first 3 rows. This is the pandas idiom — chain operations to express analysis in one line.

---

## 6. Single-Column Aggregations

Each method takes a Series and returns one number.

| Method | What it does |
|---|---|
| `.sum()` | Total |
| `.mean()` | Average |
| `.median()` | Middle value when sorted |
| `.max()` | Largest |
| `.min()` | Smallest |
| `.count()` | Number of non-null values |
| `.std()` | Standard deviation (spread) |
| `.nunique()` | Number of unique values |
| `.quantile(0.25)` | 25th percentile (Q1) |
| `.quantile(0.75)` | 75th percentile (Q3) |
| `.idxmax()` | Index of the max value |
| `.idxmin()` | Index of the min value |

### Examples

```python
df["amount"].sum()       # 455000
df["amount"].mean()      # 65000.0
df["amount"].max()       # 90000
df["amount"].min()       # 45000
df["amount"].count()     # 7
df["amount"].std()       # 16832.51 — measures spread
df["amount"].median()    # 60000 — middle value
```

### Note: pandas may return NumPy types

```python
df["amount"].sum()
# np.int64(455000)
```

This isn't a bug. Pandas is built on NumPy, so it returns NumPy's number types. They behave exactly like regular Python `int` and `float` for math and printing. You can ignore the `np.int64` wrapper — treat it like an integer.

### `.count()` vs `len(df)`

```python
len(df)                  # total rows including nulls
df["amount"].count()     # only counts non-null amounts
```

When data is clean, both return the same number. When there are nulls in a column, `.count()` excludes them. **Use `.count()` when you specifically want "how many non-null values does this column have."** Use `len(df)` when you want "how many total rows exist."

### `.std()` interpretation

Standard deviation measures how spread out the data is around the mean.

- Mean 65000, std 5000 → values clustered tightly (consistent sales)
- Mean 65000, std 30000 → values spread widely (volatile sales)

Same average, very different story. Senior analysts always look at std alongside mean — never just one.

---

## 7. Unique Values and Frequency

### Three core methods

```python
df["salesperson"].unique()        # array of distinct values
df["salesperson"].nunique()       # count of distinct values (just the number)
df["salesperson"].value_counts()  # frequency of each value
```

### `.unique()` returns a NumPy array

```python
df["salesperson"].unique()
# array(['Rohit', 'Amit', 'Neha'], dtype=object)
```

Useful for "what categories appear in this column?"

### `.value_counts()` — the heavy hitter

```python
df["salesperson"].value_counts()
# Rohit    3
# Neha     2
# Amit     2
```

Returns a Series sorted by frequency descending. **One method gives you which values exist AND how often.**

### Variants

```python
# Sort by index alphabetically instead of by frequency
df["month"].value_counts().sort_index()

# Return percentages instead of counts
df["salesperson"].value_counts(normalize=True)
# Rohit    0.428571
# Neha     0.285714
# Amit     0.285714

# Include null counts (default skips them)
df["salesperson"].value_counts(dropna=False)
```

### `.idxmax()` — find the name with the highest count

```python
df["salesperson"].value_counts().idxmax()
# 'Rohit'
```

`.idxmax()` returns the *index* of the maximum value. In a value_counts Series, the index *is* the names — so this gives you the most frequent value directly without further indexing.

---

## 8. The Filter → Sort → Aggregate Pattern

Real analyst work combines these. The pattern reads left to right like a sentence.

### Examples

```python
# Total sales by Rohit
df[df["salesperson"] == "Rohit"]["amount"].sum()

# Average sale in February
df[df["month"] == "Feb"]["amount"].mean()

# Top 3 biggest sales by Rohit
df[df["salesperson"] == "Rohit"].sort_values(by="amount", ascending=False).head(3)

# How many big sales (>60000) did each salesperson make?
df[df["amount"] > 60000]["salesperson"].value_counts()
```

### How to read these

Read each line left to right, treating each operation as a step:

```python
df[df["salesperson"] == "Rohit"]["amount"].sum()
└──┬──┘└──────────┬────────────┘└──┬──┘└──┬──┘
   │              │                │      │
   start          filter rows      pick   reduce to
   with df        where Rohit      amount sum
                                   column
```

Every pandas chain is just: **start → filter → select → aggregate**. Once you see this shape, complex pandas code stops looking intimidating.

---

## 9. Week 1 vs Week 2 — The Collapse

### Total sales

```python
# Week 1: 5 lines + a function
def get_total_sales(data):
    total = 0
    for sale in data:
        total += sale["amount"]
    return total

# Week 2: 1 line
df["amount"].sum()
```

### Filter Rohit's sales

```python
# Week 1
rohit_sales = []
for sale in sales_data:
    if sale["salesperson"] == "Rohit":
        rohit_sales.append(sale)

# Week 2
df[df["salesperson"] == "Rohit"]
```

### Per-person totals (this is where it gets wild)

```python
# Week 1: 8 lines, accumulator dict pattern
def get_sales_by_person(data):
    totals = {}
    for sale in data:
        name = sale["salesperson"]
        if name in totals:
            totals[name] += sale["amount"]
        else:
            totals[name] = sale["amount"]
    return totals

# Week 2: 1 line (preview of Day 9's groupby)
df.groupby("salesperson")["amount"].sum()
```

**The point isn't that pandas is "better."** It's that:

- Week 1 forced you to understand WHAT was happening
- Week 2 gives you tools to do it concisely

Without Week 1, pandas feels like magic. With Week 1, pandas feels like shorthand for things you already understand. That's why you built the manual version first.

---

## 10. Common Errors and How to Read Them

| Error | Likely cause | Fix |
|---|---|---|
| `KeyError: 'column_name'` | Column doesn't exist in this DataFrame | Check `df.columns` for the actual name |
| `TypeError: '<' not supported between 'str' and 'int'` | Comparing wrong types (e.g. unconverted column) | Convert with `.astype()` or fix the read |
| `ValueError: The truth value of a Series is ambiguous` | Used Python `and`/`or` instead of `&`/`\|` | Use `&`, `\|`, `~` with parentheses |
| `IndexError: indices are out-of-bounds` | `.iloc[]` position doesn't exist | Check `df.shape` for actual row count |
| `KeyError: 5` (with `.loc`) | No row with that label exists | Use `.iloc` for position, or check the index |

### The mental model for debugging

1. Read the error type (TypeError, KeyError, ValueError)
2. Look at the line where it broke
3. Ask: what's the *type* of each thing on that line?
4. Type mismatch is the answer 80% of the time

---

## 11. Style and Habits

### Always assign filtered results to a name

```python
# Less readable
df[df["amount"] > 60000][df["month"] == "Jan"]

# More readable
big_sales = df[df["amount"] > 60000]
big_jan_sales = big_sales[big_sales["month"] == "Jan"]
```

Or combine cleanly:
```python
df[(df["amount"] > 60000) & (df["month"] == "Jan")]
```

### Don't chain too aggressively

These chains are fine:
```python
df.sort_values(by="amount", ascending=False).head(3)
df[df["month"] == "Feb"]["amount"].mean()
```

This chain is too much:
```python
df[df["amount"] > 60000].sort_values(by="month").groupby("salesperson").mean().head(3).reset_index()
```

When in doubt, break into named intermediate steps. Future-you (or a teammate) will thank you.

### Always use parentheses with `&` and `|`

Habit, not optional. Even when "it works without them" in your specific case — write them anyway. Consistency prevents bugs in the case where it matters.

---

## 12. The Mental Shift That's Happening

In Week 1, you thought in terms of **rows being processed one at a time** (the for-loop model).

In Week 2, you're shifting to thinking in terms of **whole columns being operated on at once** (the vector model).

```python
# Row-thinking
for sale in sales_data:
    if sale["amount"] > 60000:
        ...

# Column-thinking
df[df["amount"] > 60000]
```

This shift is the single biggest mental change between regular Python and data analysis Python. Once it lands, NumPy and SQL become much more natural too — they all use column/vector thinking.

You're in the middle of this shift right now. It'll click more solidly when you hit `groupby()` on Day 9.

---

## Reflection for Day 7-8

- Pandas syntax is short and precise. Tiny mistakes (missing `df`, missing parentheses) cause clear errors. Read the errors.
- `.loc` slicing is inclusive on the end. Memorize this — it bites everyone.
- The filter → sort → aggregate chain reads left to right like a sentence. Read pandas like English.
- Build the boolean-mask intuition. Once filtering feels natural, half of analyst work becomes easy.
- Week 1's manual loops aren't wasted — they make pandas feel like shorthand instead of magic.


