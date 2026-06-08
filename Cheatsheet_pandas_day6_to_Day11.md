Good. Methods list now — pure reference, no commentary. Append this to your existing `methods_cheatsheet.md` as a new section, or save as a standalone `methods_cheatsheet_pandas.md` if you prefer separating Python core from pandas. Your call.

---

```markdown
# Pandas Methods Cheatsheet — Days 6 to 10

## Importing and Setup

| Code | What it does |
|---|---|
| `import pandas as pd` | Standard import, always use `pd` as alias |
| `pd.__version__` | Check your pandas version |
| `from IPython.display import display` | For prettier output in Jupyter |
| `display(df)` | Render DataFrame as a styled HTML table |

---

## Creating DataFrames and Series

| Code | What it does |
|---|---|
| `pd.Series([1, 2, 3])` | Create a Series from a list |
| `pd.Series([1, 2, 3], index=["a", "b", "c"])` | Create a Series with custom index |
| `pd.DataFrame(dict_of_lists)` | Create a DataFrame from a dict where each key is a column name |
| `pd.DataFrame(list_of_dicts)` | Create a DataFrame from a list of dicts (each dict is a row) |

---

## Loading and Saving Files

| Code | What it does |
|---|---|
| `pd.read_csv("file.csv")` | Load a CSV into a DataFrame |
| `pd.read_csv("file.csv", sep="\t")` | Load with custom separator (tab here) |
| `pd.read_csv("file.csv", header=None)` | Load when file has no header row |
| `pd.read_excel("file.xlsx")` | Load an Excel file |
| `df.to_csv("output.csv", index=False)` | Save DataFrame to CSV (skip index) |
| `df.to_excel("output.xlsx", index=False)` | Save to Excel |

---

## Inspecting a DataFrame

| Code | What it does |
|---|---|
| `df.head()` | First 5 rows |
| `df.head(10)` | First 10 rows |
| `df.tail()` | Last 5 rows |
| `df.shape` | Tuple of (rows, columns) |
| `df.info()` | Column names, dtypes, non-null counts, memory usage |
| `df.describe()` | Summary stats (count, mean, std, min, max, quartiles) for numeric columns |
| `df.describe(include='all')` | Include text columns too (different stats) |
| `df.columns` | Index object of column names |
| `list(df.columns)` | Column names as a regular Python list |
| `df.dtypes` | Type of each column |
| `df.index` | The DataFrame's index |
| `len(df)` | Number of rows |

---

## Column Selection

| Code | Returns | What it does |
|---|---|---|
| `df["amount"]` | Series | Select one column |
| `df[["name", "amount"]]` | DataFrame | Select multiple columns |
| `df[col_list]` | DataFrame | Select columns from a list variable |
| `"amount" in df.columns` | bool | Check if a column exists |

---

## Row Selection — `.loc` and `.iloc`

| Code | Selects by | What it returns |
|---|---|---|
| `df.iloc[3]` | Position | 4th row as a Series |
| `df.loc[3]` | Label | Row with index label 3 |
| `df.iloc[1:4]` | Positions | Rows 1, 2, 3 (exclusive end) |
| `df.loc[1:4]` | Labels | Rows labeled 1, 2, 3, 4 (INCLUSIVE end) |
| `df.iloc[[0, 2, 4]]` | Positions | Specific rows by position |
| `df.loc[[0, 2, 4]]` | Labels | Specific rows by label |
| `df.iloc[2, 1]` | Position | Single cell at row 2, column 1 |
| `df.loc[2, "amount"]` | Label | Single cell at label 2, column "amount" |
| `df.loc[[0, 2], ["name", "amount"]]` | Label | Subset of rows AND columns |

**Memorize:** `.loc` slicing is INCLUSIVE on both ends. `.iloc` slicing is EXCLUSIVE on the end (like Python lists).

---

## Boolean Filtering

| Code | What it does |
|---|---|
| `df[df["amount"] > 60000]` | Rows where amount > 60000 |
| `df[df["salesperson"] == "Rohit"]` | Rows where salesperson is exactly "Rohit" |
| `df[df["amount"] != 50000]` | Rows where amount is NOT 50000 |
| `df[df["month"].isin(["Jan", "Feb"])]` | Rows where month is Jan OR Feb |
| `df[df["name"].str.startswith("R")]` | Names starting with "R" |
| `df[df["name"].str.endswith("a")]` | Names ending with "a" |
| `df[df["name"].str.contains("e")]` | Names containing "e" |
| `df[df["name"].str.lower() == "rohit"]` | Case-insensitive match |

---

## Combining Conditions

| Code | Meaning |
|---|---|
| `&` | AND |
| `\|` | OR |
| `~` | NOT |

**Always wrap each condition in parentheses:**

```python
df[(df["salesperson"] == "Rohit") & (df["amount"] > 50000)]
df[(df["month"] == "Jan") | (df["month"] == "Mar")]
df[~(df["month"] == "Mar")]                   # NOT in March
df[(df["amount"] > 50000) & (df["month"] != "Jan")]
```

---

## Sorting

| Code | What it does |
|---|---|
| `df.sort_values(by="amount")` | Sort by amount ascending |
| `df.sort_values(by="amount", ascending=False)` | Sort descending |
| `df.sort_values(by=["month", "amount"])` | Sort by multiple columns (both ascending) |
| `df.sort_values(by=["month", "amount"], ascending=[True, False])` | Mixed sort order |
| `df.sort_index()` | Sort by index ascending |
| `df.sort_index(ascending=False)` | Sort by index descending |
| `df.sort_values(by="amount").head(3)` | Top 3 lowest (chain with .head()) |
| `df.sort_values(by="amount", ascending=False).head(3)` | Top 3 highest |

**Avoid `inplace=True`** — assign to a new variable instead. Pandas is moving away from inplace operations.

---

## Single-Column Aggregations

| Code | What it returns |
|---|---|
| `df["amount"].sum()` | Total |
| `df["amount"].mean()` | Average |
| `df["amount"].median()` | Middle value when sorted |
| `df["amount"].max()` | Largest |
| `df["amount"].min()` | Smallest |
| `df["amount"].count()` | Number of non-null values |
| `df["amount"].std()` | Standard deviation |
| `df["amount"].var()` | Variance |
| `df["amount"].nunique()` | Number of unique values |
| `df["amount"].quantile(0.25)` | 25th percentile (Q1) |
| `df["amount"].quantile(0.5)` | 50th percentile (median) |
| `df["amount"].quantile(0.75)` | 75th percentile (Q3) |
| `df["amount"].idxmax()` | Index label of the maximum value |
| `df["amount"].idxmin()` | Index label of the minimum value |

---

## Unique Values and Frequency

| Code | What it returns |
|---|---|
| `df["col"].unique()` | NumPy array of distinct values |
| `df["col"].nunique()` | Count of distinct values |
| `df["col"].value_counts()` | Frequency of each value, sorted descending |
| `df["col"].value_counts().sort_index()` | Sorted by value instead of frequency |
| `df["col"].value_counts(normalize=True)` | Percentages instead of counts |
| `df["col"].value_counts(dropna=False)` | Include NaN counts |

---

## Groupby — Single Aggregation

| Code | What it does |
|---|---|
| `df.groupby("col")` | Create a groupby object (split step) |
| `df.groupby("col").groups` | Dict showing which rows are in which group |
| `df.groupby("col").get_group("value")` | Return one specific group as a DataFrame |
| `len(df.groupby("col"))` | Number of groups |
| `df.groupby("col")["value"].sum()` | Sum per group |
| `df.groupby("col")["value"].mean()` | Mean per group |
| `df.groupby("col")["value"].count()` | Count per group |
| `df.groupby("col")["value"].max()` | Max per group |
| `df.groupby("col")["value"].std()` | Standard deviation per group |
| `df.groupby("col")["value"].sum().sort_values(ascending=False)` | Sorted result |
| `df.groupby("col")["value"].sum().idxmax()` | Group name with the highest sum |

---

## Groupby — Multiple Aggregations

| Code | What it does |
|---|---|
| `df.groupby("col")["value"].agg(["sum", "mean", "count"])` | Multiple aggregations as columns |
| `df.groupby("col")["value"].agg(total="sum", avg="mean", n="count")` | Named aggregations (preferred) |
| `df.groupby("col").agg(total=("value", "sum"), avg=("value", "mean"))` | Named aggregations on specific columns |

---

## Groupby — Multiple Columns

| Code | What it does |
|---|---|
| `df.groupby(["col1", "col2"])["value"].sum()` | Group by combinations, get MultiIndex result |
| `df.groupby(["col1", "col2"])["value"].sum().reset_index()` | Flatten MultiIndex back to regular DataFrame |
| `for name, group in df.groupby("col"):` | Iterate over groups |

---

## Merging / Joining

| Code | What it does |
|---|---|
| `pd.merge(left, right, on="key", how="inner")` | Function form |
| `left.merge(right, on="key", how="inner")` | Method form (preferred for chaining) |
| `left.merge(right, left_on="a", right_on="b")` | When key columns have different names |
| `left.merge(right, on=["k1", "k2"])` | Join on multiple key columns |
| `left.merge(right, how="inner")` | Only matching rows on both sides |
| `left.merge(right, how="left")` | All rows from left + matches from right |
| `left.merge(right, how="right")` | All rows from right + matches from left |
| `left.merge(right, how="outer")` | All rows from either side |

### Chaining multiple merges

```python
(
    sales_df
    .merge(employees_df, on="salesperson_id")
    .merge(products_df, on="product_id")
)
```

---

## Handling Missing Data (Preview for Day 11)

| Code | What it does |
|---|---|
| `df.isna()` | DataFrame of True/False — which cells are NaN |
| `df["col"].isna()` | Series of True/False for one column |
| `df["col"].isna().sum()` | Count of NaN in one column |
| `df.dropna()` | Drop rows with any NaN |
| `df.fillna(0)` | Replace NaN with 0 |
| `df.fillna(df["col"].mean())` | Replace NaN with mean |

---

## Type Conversion

| Code | What it does |
|---|---|
| `df["col"].astype("int")` | Convert column to int |
| `df["col"].astype("Int64")` | Convert to nullable int (allows NaN inside int) |
| `df["col"].astype("float")` | Convert to float |
| `df["col"].astype("str")` | Convert to string |
| `pd.to_numeric(df["col"], errors="coerce")` | Convert to number, bad values become NaN |
| `pd.to_datetime(df["col"])` | Convert to datetime |

---

## Adding and Removing Columns

| Code | What it does |
|---|---|
| `df["new_col"] = df["amount"] * 0.05` | Add a new column from calculation |
| `df["new_col"] = "constant"` | Add a column with the same value |
| `df.drop(columns=["col1", "col2"])` | Drop columns (returns new DataFrame) |
| `df.drop(index=[0, 3])` | Drop rows by index label |
| `df.rename(columns={"old": "new"})` | Rename specific columns |

---

## Index Manipulation

| Code | What it does |
|---|---|
| `df.reset_index()` | Move index back to a regular column (useful after groupby) |
| `df.reset_index(drop=True)` | Reset index, throw away old one |
| `df.set_index("col")` | Make "col" the new index |
| `df.set_index("col", drop=False)` | Set as index but also keep as a column |

---

## Quick "How Do I" Reference

| Goal | Code |
|---|---|
| Total of a column | `df["col"].sum()` |
| Average of a column | `df["col"].mean()` |
| Count rows in a DataFrame | `len(df)` or `df.shape[0]` |
| Count non-null in a column | `df["col"].count()` |
| Unique values in a column | `df["col"].unique()` |
| Frequency of each value | `df["col"].value_counts()` |
| Top N highest | `df.sort_values(by="col", ascending=False).head(N)` |
| Filter rows by condition | `df[df["col"] > value]` |
| Filter rows by multiple conditions | `df[(cond1) & (cond2)]` |
| Group totals | `df.groupby("col1")["col2"].sum()` |
| Group with multiple stats | `df.groupby("col1")["col2"].agg(["sum", "mean"])` |
| Combine two tables | `df1.merge(df2, on="key")` |
| Find rows with missing values | `df[df["col"].isna()]` |
| Find unmatched rows after right join | `right_join_result[right_join_result["left_col"].isna()]` |
| Top group | `df.groupby("col")["value"].sum().idxmax()` |

---

## Three Patterns You'll Use Constantly

### Pattern 1: Filter → Aggregate

```python
df[df["category"] == "Electronics"]["amount"].sum()
```

### Pattern 2: Group → Aggregate → Sort

```python
df.groupby("salesperson")["amount"].sum().sort_values(ascending=False)
```

### Pattern 3: Merge → Filter → Group → Aggregate

```python
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

If you internalize these three patterns, you can write 80% of real analyst code.

---

## Common Gotchas

| Gotcha | Reminder |
|---|---|
| `.loc[1:4]` includes 4 | Pandas slicing is inclusive on the end for `.loc` |
| Used `and`/`or` and got an error | Use `&`/`\|` with each condition in parentheses |
| Got floats after a merge | NaN forces float — use `.astype("Int64")` to fix |
| `df.column_name` failed | Use `df["column_name"]` — safer, always works |
| Wrong "top" result | Verify against underlying data — don't trust position in output |
| `inplace=True` causing weird issues | Avoid it — assign to a new variable instead |
```

```markdown
## Missing Data

### Finding missing values

| Code | What it does |
|---|---|
| `df.isna()` | Boolean DataFrame — True wherever NaN |
| `df.notna()` | Opposite — True wherever NOT NaN |
| `df.isna().sum()` | Count of NaN per column |
| `df.isna().sum().sum()` | Total NaN in whole DataFrame |
| `(df.isna().sum() / len(df)) * 100` | Percentage missing per column |
| `df.isna().sum(axis=1)` | Count of NaN per row |
| `df[df.isna().any(axis=1)]` | Rows with at least one NaN |
| `df[df.isna().all(axis=1)]` | Rows where ALL values are NaN |
| `df[df.notna().all(axis=1)]` | Rows with NO missing values |
| `df.columns[df.isna().any()]` | Column names that have any NaN |
| `df["col"].isna().sum()` | Count of NaN in one column |

**Memory aid for axis:** `axis=0` means "go down rows" → per-column result. `axis=1` means "go across columns" → per-row result.

### Removing missing values

| Code | What it does |
|---|---|
| `df.dropna()` | Drop any row with at least one NaN |
| `df.dropna(how="all")` | Drop only rows where ALL values are NaN |
| `df.dropna(subset=["age"])` | Drop only rows where "age" is NaN |
| `df.dropna(subset=["age", "city"])` | Drop if either age OR city missing |
| `df.dropna(axis=1)` | Drop columns containing NaN (instead of rows) |
| `df.dropna(thresh=4)` | Keep only rows with at least 4 non-null values |

**Production rule:** Almost always use `subset=` for targeted drops. Avoid blanket `df.dropna()`.

### Filling missing values

| Code | What it does |
|---|---|
| `df["col"].fillna("Unknown")` | Fill with a constant string |
| `df["col"].fillna(0)` | Fill with zero |
| `df["col"].fillna(df["col"].mean())` | Fill with mean |
| `df["col"].fillna(df["col"].median())` | Fill with median (safer with outliers) |
| `df["col"].fillna(df["col"].mode()[0])` | Fill with most common value |
| `df["col"].ffill()` | Forward fill — use value above |
| `df["col"].bfill()` | Backward fill — use value below |

### Filling multiple columns at once

```python
df.fillna({
    "name": "Unknown",
    "age": df["age"].median(),
    "city": "Unknown",
    "amount": 0,
    "date": "Not Recorded"
})
```

### Critical: fillna returns a new Series

```python
df["age"].fillna(df["age"].mean())              # returns new, df unchanged
df["age"] = df["age"].fillna(df["age"].mean())  # actually saves the change
```

Forgetting the assignment is the #2 most common pandas bug.

### The Missing Data Decision Framework

```
1. WHY is it missing? (truly unknown / not applicable / pipeline issue / genuinely zero)
2. WHAT % is missing? (< 5% drop ok, 5-30% fill, > 30% reconsider column)
3. IS the missingness biased? (check if missing correlates with metric of interest)
4. WHAT stage am I in? (analysis = keep NaN, reporting = fill with "Unknown")
```

The framework matters more than any specific method.

### Stage-appropriate handling

- **During analysis:** keep NaN (aggregations skip it automatically, you can measure missingness with `.isna()`)
- **For reporting/display:** fill at the end with "Unknown" or similar (dashboards can't show NaN cleanly)

---

## Dates and Time Series

### Converting strings to datetime

| Code | What it does |
|---|---|
| `pd.to_datetime(df["col"])` | Auto-detect format, convert column |
| `pd.to_datetime(df["col"], format="%d/%m/%Y")` | Explicit format (Indian DD/MM/YYYY) |
| `pd.to_datetime(df["col"], format="%m/%d/%Y")` | US format MM/DD/YYYY |
| `pd.to_datetime(df["col"], errors="coerce")` | Bad values become NaT instead of crashing |
| `pd.Timestamp("2024-01-15")` | Create a single datetime value |
| `df.dtypes` | Verify conversion — should show `datetime64[ns]` |

**Production rule:** Always use `errors="coerce"` unless you specifically want a crash on bad data.

### Common format codes

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

### The `.dt` accessor — extracting date parts

| Code | Returns |
|---|---|
| `df["date"].dt.year` | Year as integer |
| `df["date"].dt.month` | Month number 1-12 |
| `df["date"].dt.month_name()` | Month as string ("January") |
| `df["date"].dt.day` | Day of month 1-31 |
| `df["date"].dt.weekday` | Day of week 0-6 (Mon=0, Sun=6) |
| `df["date"].dt.day_name()` | Day as string ("Monday") |
| `df["date"].dt.quarter` | Quarter 1-4 |
| `df["date"].dt.dayofyear` | Day of year 1-365 |
| `df["date"].dt.hour` | Hour (for timestamped data) |
| `df["date"].dt.minute` | Minute |
| `df["date"].dt.isocalendar().week` | ISO week number |

### Date arithmetic

| Code | What it does |
|---|---|
| `df["date1"] - df["date2"]` | Returns Timedelta (duration) |
| `(df["date1"] - df["date2"]).dt.days` | Extract day count as integer |
| `df["date"] + pd.Timedelta(days=30)` | Add 30 days (fixed duration) |
| `df["date"] + pd.DateOffset(months=1)` | Add 1 calendar month (handles month length) |
| `df["date"] - pd.Timedelta(days=7)` | Subtract 7 days |
| `pd.Timestamp("2024-12-31") - df["date"]` | Days from reference to each date |

### Useful date patterns

```python
# Days since each row
today = pd.Timestamp("2024-12-31")
df["days_ago"] = (today - df["sale_date"]).dt.days

# Weekend flag
df["is_weekend"] = df["sale_date"].dt.weekday >= 5

# Date range filter
df[(df["sale_date"] >= "2024-04-01") & (df["sale_date"] <= "2024-06-30")]

# Per-group date range
df.groupby("salesperson")["sale_date"].agg(["min", "max"])
```

### Timedelta vs DateOffset

| Use | When |
|---|---|
| `pd.Timedelta(days=30)` | Exactly 30 days, regardless of month |
| `pd.DateOffset(months=1)` | "Same calendar date next month" |
| `pd.DateOffset(years=1)` | "Same date next year" (handles leap years) |

For warranty/follow-up periods: Timedelta. For "same date next month": DateOffset.

### Resampling — time-series aggregation

Requires the date column to be the DataFrame's **index**.

```python
df_ts = df.set_index("sale_date")
df_ts["amount"].resample("ME").sum()
```

| Code | Period |
|---|---|
| `resample("D")` | Daily |
| `resample("W")` | Weekly |
| `resample("ME")` | Month end (new) |
| `resample("MS")` | Month start |
| `resample("QE")` | Quarter end (new) |
| `resample("YE")` | Year end (new) |
| `resample("H")` | Hourly |

**Deprecation note:** Old code uses `M`, `Q`, `Y`. New code uses `ME`, `QE`, `YE`. Update when you see the deprecation warning.

### Multiple aggregations with resample

```python
df_ts["amount"].resample("ME").agg(["sum", "mean", "count"])
df_ts["amount"].resample("ME").agg(total="sum", avg="mean", n="count")
```

### Resample vs groupby — the critical distinction

| Approach | Behavior |
|---|---|
| `df.groupby(df["date"].dt.month_name())` | Skips months with no data; lumps same months across different years |
| `df_ts["col"].resample("ME")` | Includes empty months (shows as 0 or NaN); keeps years separate |

For business reports where missing periods matter, **use resample**. An August with no sales is a signal stakeholders need to see — groupby would hide it.

---

## Common Errors with Missing Data and Dates

| Error | Likely cause | Fix |
|---|---|---|
| `KeyError` after merge | Column doesn't exist after the join | Check `df.columns`; verify the join didn't drop it |
| `TypeError: unsupported operand type(s) for -: 'str' and 'str'` | Trying date math on strings | Convert with `pd.to_datetime()` first |
| `ValueError: time data does not match format` | Wrong format string | Match actual format in data |
| `FutureWarning: 'M' is deprecated` | Old frequency code | Use `ME` instead |
| `df` not changed after `fillna()` | Didn't assign back | `df["col"] = df["col"].fillna(...)` |
| Unexpected NaT | `errors="coerce"` did its job | Inspect with `df[df["date"].isna()]` |
| Float dtype after merge | NaN forced float conversion | Use `.astype("Int64")` for nullable integers |

---

## Quick Reference Additions

| Goal | Code |
|---|---|
| Count missing per column | `df.isna().sum()` |
| Percentage missing per column | `(df.isna().sum() / len(df)) * 100` |
| Drop rows where specific column is missing | `df.dropna(subset=["col"])` |
| Fill with median | `df["col"].fillna(df["col"].median())` |
| Fill multiple columns differently | `df.fillna({"col1": 0, "col2": "Unknown"})` |
| Convert column to datetime | `df["col"] = pd.to_datetime(df["col"], errors="coerce")` |
| Get day name from date | `df["col"].dt.day_name()` |
| Days between two dates | `(df["date1"] - df["date2"]).dt.days` |
| Group by month including empty months | `df.set_index("date").resample("ME").sum()` |
| Check date type after conversion | `df.dtypes` (should show `datetime64[ns]`) |
```

---

