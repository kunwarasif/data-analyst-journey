

# Day 3 Notes: Sales Report Generator — 

## What you can do after today

- Write functions that call other functions (function composition)
- Format numbers professionally with f-string specifiers
- Align text into clean columns
- Sort dictionaries by value using lambda
- Build a multi-section report function that combines every pattern from Week 1
- Read Python error messages and debug independently
- Reuse computed values instead of recomputing (DRY principle)

---

## 1. Function composition — small functions calling other functions

The big idea: **don't write monolithic code; build small reusable functions and stack them.**

Example from today:
```python
def get_average_sale(data):
    return get_total_sales(data) / get_transaction_count(data)
```

This one-line function calls two other functions you already wrote. No new logic — just composition.

Real analyst codebases are 80% small functions composing into bigger ones. Examples you'd write later:
```python
def get_profit_margin(data):
    return (get_total_revenue(data) - get_total_cost(data)) / get_total_revenue(data)

def get_quarterly_growth(data):
    return get_quarter_total(data, 'Q2') / get_quarter_total(data, 'Q1') - 1
```

Same pattern, different domain.

---

## 2. f-string formatting — make numbers look professional

f-strings can include **format specifiers** after a colon `:` to control display.

### The four format specifiers you'll use 90% of the time

| Pattern | What it does | Example |
|---|---|---|
| `:,` | Thousands separator | `1000000` → `1,000,000` |
| `:.2f` | Float with 2 decimals | `3.14159` → `3.14` |
| `:,.2f` | Both | `1234567.89` → `1,234,567.89` |
| `:.1%` | Percentage with 1 decimal | `0.374` → `37.4%` |

### String alignment for tables

| Pattern | What it does | Example |
|---|---|---|
| `:<10` | Left-align in 10 chars | `'Rohit     '` |
| `:>10` | Right-align in 10 chars | `'     Rohit'` |
| `:^10` | Center in 10 chars | `'  Rohit   '` |

Combining them produces clean table output:
```python
print(f"{name:<10} ₹{amount:>10,}   ({pct:.1%})")
# Neha       ₹   170,000   (37.4%)
```

### General syntax (don't memorize, look up when needed)

```
{value:[align][width][,][.precision][type]}
```

Memorize the four common patterns above. The rest you can look up.

---

## 3. Dictionary sorting with lambda

To sort a dictionary by value (highest first):

```python
sorted_dict = sorted(my_dict.items(), key=lambda x: x[1], reverse=True)
```

### Breaking this down

- `my_dict.items()` → produces a list of tuples like `[('Rohit', 165000), ('Amit', 120000), ('Neha', 170000)]`
- `sorted(...)` → sorts that list
- `key=lambda x: x[1]` → for each tuple `x`, use its second element (the value) as the sort key
- `reverse=True` → highest first instead of lowest first

### The mental model

```
.items()  →  a list of tuples
              │
              ├── ('Rohit', 165000)    ← one tuple
              ├── ('Amit',  120000)    ← another tuple
              └── ('Neha',  170000)    ← another tuple

lambda x: x[1]
       │     │
       │     └── return the second element of the tuple
       └── x is one tuple at a time
```

### Variants you'll use

```python
# Sort by value, ascending
sorted(d.items(), key=lambda x: x[1])

# Sort by key (the dict's natural order)
sorted(d.items(), key=lambda x: x[0])

# Sort a list of dicts by one field
sorted(sales_data, key=lambda sale: sale["amount"], reverse=True)
```

The last one is especially common — sorting records by a field.

### Why `lambda x: x[1]` and not just `[1]`?

`sorted()` needs a **function** as its key — something it can call on each item. `lambda x: x[1]` is a tiny one-line function that says "given `x`, return `x[1]`." Python calls this function on every item during sorting.

You'll see `lambda` constantly in pandas (`.apply()`, `.map()`, `sort_values(key=...)`). Lock the pattern in now.

---

## 4. The DRY principle — Don't Repeat Yourself

**Every piece of work happens in exactly one place.**

### Bad — recomputing the same thing 3 times

```python
print(f"Total: ₹{get_total_sales(data):,}")
for name, amount in person_totals.items():
    pct = amount / get_total_sales(data)        # recomputes total here
    print(f"{name}: {pct:.1%}")
print(f"Avg: ₹{get_total_sales(data) / get_transaction_count(data):,.0f}")  # again here
```

`get_total_sales` runs 3+ times. Wasteful with small data, dangerous with large data.

### Good — compute once, reuse

```python
total = get_total_sales(data)
count = get_transaction_count(data)
average = total / count

print(f"Total: ₹{total:,}")
for name, amount in person_totals.items():
    pct = amount / total
    print(f"{name}: {pct:.1%}")
print(f"Avg: ₹{average:,.0f}")
```

Computed once, used many times. Faster, cleaner, easier to debug.

### The general rule

**If you use a value more than once, store it in a variable.** Always. Senior developers do this even when it adds an "unnecessary" line. The clarity and option to debug are worth it.

This principle appears in:
- Pandas: compute a column once with `df['profit'] = df['revenue'] - df['cost']`, then reuse
- SQL: use CTEs to compute a subquery once and reference multiple times
- Power BI: use measures instead of recalculating per-chart
- Spreadsheets: cell references instead of retyping formulas

Same idea everywhere.

---

## 5. String joining — `.join()`

To merge a collection of strings into one string:

```python
", ".join(["Amit", "Neha"])      # "Amit, Neha"
" | ".join(["a", "b", "c"])      # "a | b | c"
"".join(["a", "b", "c"])         # "abc" (joined with nothing)
```

The string you call `.join()` on is the **separator**. The argument is the **collection of strings**.

Common uses:
- Print a list as comma-separated: `", ".join(my_set)`
- Build a CSV row: `",".join(row_values)`
- Build a file path: `"/".join(path_parts)`

**Important:** `.join()` works only on strings. If your collection has numbers, convert first:
```python
", ".join([str(x) for x in [1, 2, 3]])    # "1, 2, 3"
```

---

## 6. Tuple unpacking — bonus uses

You've used it three times now:

```python
# In for loops with dict.items()
for name, amount in totals.items():
    ...

# In for loops with enumerate
for index, value in enumerate(my_list):
    ...

# Assigning function return values
top_name, top_amount = get_top_performer(data)
```

Same mechanism in all three: Python sees a tuple of N items being assigned to N variables, and splits them automatically.

Works with any tuple, anywhere:
```python
first, second, third = (10, 20, 30)
a, b = "ab"                          # works on strings too — characters split
x, y = (5, 10)
```

---

## 7. Debugging — reading Python error messages

You hit two errors today. Let's lock in how to read them.

### Error 1: `TypeError: list indices must be integers or slices, not str`

What Python is saying: *"You're trying to access a list using a string. Lists are indexed by numbers."*

Lesson: when you see this, check whether the variable is actually a list or a dict.

### Error 2: `TypeError: 'tuple' object is not callable`

What Python is saying: *"You put `()` after something, which means 'call this as a function.' But this isn't a function."*

Lesson: `()` is for function calls only. Tuples, lists, dicts, numbers can't be called.

### The 30-second debugging routine

1. Read the **last line** of the error first — that's the actual problem in English
2. Find the line marked with `---->` — that's where it broke
3. Ask: what type is each variable on that line?
4. If type doesn't match what the operation expects → there's your bug

Try this routine on every error before asking for help. 90% of beginner errors get caught in 30 seconds.

---

## 8. Daily report structure — what makes output "professional"

Today you built a report with these elements:

1. **Dividers** at top and bottom (visual boundaries)
2. **Title** centered or padded
3. **Summary section** with key metrics
4. **Detail sections** with section headers (e.g. `--- Sales by Person ---`)
5. **Aligned columns** so numbers line up
6. **Formatted numbers** with thousand separators
7. **Percentages where relevant**
8. **Sort order** — biggest/most important first
9. **Blank lines between sections** for breathing room

This applies to **every report you'll ever build** — text reports, pandas tables, Power BI dashboards. The aesthetics matter because they signal "I respect the reader's time."

---

## 9. Docstrings — documenting your functions

Add a docstring as the first line inside every function:

```python
def generate_report(data):
    """
    Generate a formatted sales report from a list of sale records.
    
    Args:
        data: list of dicts with keys 'salesperson', 'amount', 'month'
    
    Prints:
        Multi-section formatted report to console.
    """
    # ... rest of function
```

### Why docstrings matter

- **Future-you forgets things.** In 3 weeks you'll look at this code and wonder what it does. The docstring tells you.
- **Auto-documentation tools** (Sphinx, pdoc) generate beautiful docs from docstrings.
- **Help in Jupyter:** typing `generate_report?` displays the docstring.
- **Real codebases require this.** Senior engineers reject pull requests without docstrings.

Habit to build: every function you write gets a docstring. Even short ones.

---

## 10. The big picture — what you've actually learned

This week you went from "I know loops conceptually" to writing a 60-line function that:
- Takes a real data structure as input
- Computes 4 different aggregations
- Sorts and formats output
- Filters data by condition
- Composes 6 helper functions into one report

You used **every fundamental Python concept** in one cohesive piece of code:
- Lists, dictionaries, sets
- Loops (for, with .items(), with enumerate)
- Conditionals (if/else)
- Functions with parameters and returns
- f-strings with format specifiers
- Tuple unpacking
- Sorting with lambda
- String methods (.join())
- Function composition
- The DRY principle

**This is what real analyst code looks like.** Not flashy. Not magic. Just small clean pieces stacked together. The pandas code you'll write in Week 2 follows the same philosophy — just with more powerful primitives.

---

## Methods cheatsheet additions (add to your master cheatsheet)

### New built-ins
| Function | What it does | Example |
|---|---|---|
| `sorted(iterable, key=..., reverse=...)` | Returns new sorted list | `sorted([3,1,2])` → `[1,2,3]` |

### New string methods
| Method | What it does | Example |
|---|---|---|
| `.join(iterable)` | Join with separator | `", ".join(["a","b"])` → `"a, b"` |

### New f-string format specifiers
| Pattern | What it does | Example output |
|---|---|---|
| `:,` | Thousands separator | `455,000` |
| `:.2f` | 2 decimal places | `455000.00` |
| `:,.2f` | Both | `455,000.00` |
| `:.1%` | Percentage | `37.4%` |
| `:<10` | Left-align width 10 | `Rohit     ` |
| `:>10` | Right-align width 10 | `     Rohit` |
| `:^10` | Center width 10 | `  Rohit   ` |

### New concept: lambda
| Pattern | What it does | Example |
|---|---|---|
| `lambda x: expr` | One-line anonymous function | `lambda x: x[1]` |
| Used with `sorted(...)` | Custom sort key | `sorted(d.items(), key=lambda x: x[1])` |

---

## Reflection for Day 3

- Function composition is the single most powerful pattern I learned this week. Small functions calling other small functions = clean, debuggable, reusable code.
- Errors aren't failure — they're feedback. The two errors I debugged today taught me more than 10 minutes of reading would have.
- Peeking at past code while writing new code is normal. Re-watching tutorials isn't. The first builds skill; the second builds illusion.
- I can now read a Python error message and find the bug in under a minute. That's a real skill, not a memorization trick.

---

