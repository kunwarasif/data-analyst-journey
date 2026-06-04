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

---

