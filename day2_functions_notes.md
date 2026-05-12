
# Day 2 Notes: Sets, Functions, Input Validation

## What you can do after today

- Use sets to collect unique values
- Define functions with `def` and call them
- Return values from functions (and know why `return` ≠ `print`)
- Use default parameter values
- Validate inputs with `isinstance` and `try/except`
- Wrap analyst logic in reusable functions

---

## 1. Sets — for uniqueness

A **set** is a collection that automatically removes duplicates.

### Creating sets

```python
empty_set = set()                  # empty set (CANNOT use {} — that's an empty dict)
months = {"Jan", "Feb", "Mar"}     # set with values
```

**Gotcha:** `{}` creates an empty *dict*, not an empty set. For an empty set, always use `set()`.

### Adding to a set

```python
months = set()
months.add("Jan")
months.add("Feb")
months.add("Jan")    # ignored — already there
print(months)        # {'Jan', 'Feb'}
```

Use `.add()` for one item. Use `.update()` to add many at once.

### Sets are unordered

```python
unique_months = {"Mar", "Jan", "Feb"}
print(unique_months)   # might print {'Jan', 'Mar', 'Feb'} or any order
```

Don't rely on set order. If you need order, convert to a list with `sorted()`.

### When to use sets

- Removing duplicates from a list: `set(my_list)`
- Fast membership checks: `"item" in my_set` (much faster than a list for large data)
- Collecting unique values during a loop

### Patterns from today

**Pattern: Collect unique values from list of dicts**
```python
unique_months = set()
for sale in sales_data:
    unique_months.add(sale["month"])
```

**Same thing as a one-liner (set comprehension):**
```python
unique_months = set(sale["month"] for sale in sales_data)
```

---

## 2. Filter pattern — pick items meeting a condition

When you want a subset that satisfies some rule. Same accumulator pattern, with an `if` inside.

```python
big_sellers = set()
for sale in sales_data:
    if sale["amount"] > 60000:
        big_sellers.add(sale["salesperson"])
```

Read in plain English: "loop through each sale, and if its amount is above 60000, add the salesperson's name to my set."

**Note on `>` vs `>=`:**
- `>` means **strictly greater than** — 60000 is NOT > 60000
- `>=` means greater than or equal to — 60000 IS >= 60000

Tiny difference, huge impact in real reports. Read requirements carefully.

---

## 3. Functions — the most important concept today

Functions let you **define logic once and reuse it many times**.

### Anatomy

```python
def function_name(parameter1, parameter2):
    # do something with the parameters
    result = parameter1 + parameter2
    return result
```

Four parts:
- `def` — keyword that defines a function
- `function_name` — name you choose (use snake_case: lowercase with underscores)
- `(parameters)` — inputs the function takes (zero, one, or many)
- `return` — the value handed back to whoever called the function

### Calling a function

```python
answer = function_name(5, 3)   # answer = 8
```

Parentheses matter:
- `function_name` (no parens) → reference to the function, doesn't run it
- `function_name()` (with parens) → actually runs the function

### Default parameter values

```python
def calculate_commission(amount, rate=0.05):
    return amount * rate

calculate_commission(50000)         # uses default rate, returns 2500
calculate_commission(50000, 0.10)   # overrides rate, returns 5000
```

When you call with fewer arguments, defaults fill in. Useful when most calls use a common value.

### `return` vs `print` — the most common beginner confusion

```python
# Version A: uses print
def double_print(x):
    print(x * 2)

result = double_print(7)   # screen shows: 14
print(result)              # screen shows: None
```

```python
# Version B: uses return
def double_return(x):
    return x * 2

result = double_return(7)   # screen shows nothing
print(result)               # screen shows: 14
```

**The difference:**
- `print()` displays on screen, function returns `None` (nothing useful)
- `return` hands a value back to the caller, who decides what to do with it (store it, print it, pass to another function, ignore it)

**Rule of thumb:** Inside a function, **always use `return`**, not `print`. Let the caller decide whether to display the result.

The only exception: when the *whole point* of the function is to display something (like a function called `show_report` that prints formatted output by design).

---

## 4. Why functions matter — reusability

Without functions, you'd copy-paste the same logic everywhere. With functions, you write it once and call it many times with different inputs.

```python
def get_total_sales(data):
    total = 0
    for sale in data:
        total = total + sale["amount"]
    return total

# Reuse on different datasets
get_total_sales(january_sales)
get_total_sales(february_sales)
get_total_sales(team_a_sales)
```

**This is the foundation of clean code.** Real analyst codebases are 80% small reusable functions.

### Critical naming rule

Inside a function, use a generic parameter name (`data`, `df`, `records`) — NOT the name of your specific variable (`sales_data`).

```python
# GOOD — works on any dataset
def get_total_sales(data):
    ...

# BAD — only works with one specific variable
def get_total_sales():
    total = 0
    for sale in sales_data:    # hardcoded — not reusable
        ...
```

---

## 5. Input validation — handle bad inputs gracefully

Python doesn't check input types automatically. If you pass garbage in, you get garbage out — silently.

Example: `double("hello")` returns `"hellohello"` because Python's `*` repeats strings. No error. Just weird output.

### Pattern 1: Strict (raise an error on bad input)

```python
def double_strict(number):
    if not isinstance(number, (int, float)):
        raise TypeError(f"Expected a number, got {type(number).__name__}")
    return number * 2
```

`isinstance(x, type)` checks if `x` is of that type.
`(int, float)` means "either int or float."
`raise` deliberately stops the function with an error.

**Use when:** bad input is a bug you want to catch loudly.

### Pattern 2: Safe (return None on bad input)

```python
def double_safe(number):
    try:
        return float(number) * 2
    except (ValueError, TypeError):
        return None
```

`try/except` runs the code; if there's an error, it goes to `except` instead of crashing.

**Use when:** bad input is expected (messy real-world data) and you want to skip silently rather than crash.

### Which to use as an analyst

**Pattern 2 is more common for analyst work.** You deal with messy data all the time — bad rows, missing values, wrong types. You usually want to keep processing and flag-or-skip bad rows, not crash the whole pipeline.

In pandas (Week 2), you'll see this same philosophy: `pd.to_numeric(col, errors='coerce')` converts what it can, makes the rest `NaN`, keeps going.

---

## 6. Two real functions you wrote today

### Function: Total across all sales

```python
def get_total_sales(data):
    total = 0
    for sale in data:
        total = total + sale["amount"]
    return total
```

Pattern used: **accumulator (number)**

### Function: Totals grouped by salesperson

```python
def get_sales_by_person(data):
    totals = {}
    for sale in data:
        name = sale["salesperson"]
        amount = sale["amount"]
        if name in totals:
            totals[name] = totals[name] + amount
        else:
            totals[name] = amount
    return totals
```

Pattern used: **accumulator (dict)** — also called *group-by* in analyst speak. This exact pattern, done by hand, is what pandas `groupby().sum()` does in one line. You'll see the connection in Week 2.

---

## 7. Three patterns to remember from Day 1 + Day 2

Every loop you write will likely be one of these:

### Pattern A: Sum/Count accumulator
```python
total = 0
for x in data:
    total = total + x       # or total += 1 for counting
```

### Pattern B: Group accumulator (dict)
```python
groups = {}
for item in data:
    key = item["category"]
    if key in groups:
        groups[key] = groups[key] + item["value"]
    else:
        groups[key] = item["value"]
```

### Pattern C: Filter (collect items matching a condition)
```python
matches = []                # or set() for unique
for item in data:
    if item["amount"] > 60000:
        matches.append(item["name"])
```

### Pattern D (bonus): Tracker (find biggest/smallest)
```python
best_amount = 0
best_name = ""
for item in data:
    if item["amount"] > best_amount:
        best_amount = item["amount"]
        best_name = item["name"]
```

If you internalize these four patterns, you can solve 80% of analyst problems with loops.

---

## 8. Style notes from today

- Use `return value`, not `return(value)` — parentheses aren't wrong but aren't standard
- Loop variable names should match what they contain: `for sale in sales_data`, not `for amt in sales_data`
- Use `snake_case` for function and variable names (lowercase with underscores): `get_total_sales`, not `getTotalSales` or `GetTotalSales`
- Use `data` (or another generic name) as the parameter inside a function — never hardcode a specific variable name

---

## Methods & functions cheatsheet (growing list)

Save this in a separate file `methods_cheatsheet.md` and ADD to it daily as you learn more.

### Built-in functions
| Function | What it does | Example |
|---|---|---|
| `len(x)` | Length of list/string/dict/set | `len([1,2,3])` → `3` |
| `sum(x)` | Sum of numbers in iterable | `sum([1,2,3])` → `6` |
| `max(x)` | Largest value | `max([1,5,3])` → `5` |
| `min(x)` | Smallest value | `min([1,5,3])` → `1` |
| `sorted(x)` | Returns new sorted list | `sorted([3,1,2])` → `[1,2,3]` |
| `range(n)` | Numbers 0 to n-1 | `range(5)` → `0,1,2,3,4` |
| `enumerate(x)` | Pairs (index, value) | `enumerate(["a","b"])` → `(0,"a"),(1,"b")` |
| `type(x)` | Type of x | `type(5)` → `<class 'int'>` |
| `isinstance(x, t)` | True if x is type t | `isinstance(5, int)` → `True` |
| `print(x)` | Display on screen | `print("hi")` → screen: `hi` |
| `set(x)` | Convert to set / empty set | `set([1,1,2])` → `{1,2}` |
| `float(x)` | Convert to float | `float("3.5")` → `3.5` |
| `int(x)` | Convert to int | `int("5")` → `5` |
| `str(x)` | Convert to string | `str(5)` → `"5"` |

### String methods
| Method | What it does | Example |
|---|---|---|
| `.strip()` | Remove leading/trailing whitespace | `"  hi  ".strip()` → `"hi"` |
| `.lower()` | All lowercase | `"HI".lower()` → `"hi"` |
| `.upper()` | All uppercase | `"hi".upper()` → `"HI"` |
| `.title()` | Title Case Each Word | `"hi there".title()` → `"Hi There"` |
| `.replace(a, b)` | Replace a with b | `"hi".replace("h","b")` → `"bi"` |
| `.split(sep)` | Split into list | `"a,b,c".split(",")` → `["a","b","c"]` |
| `.startswith(s)` | True if starts with s | `"hello".startswith("he")` → `True` |
| `.endswith(s)` | True if ends with s | `"hello".endswith("lo")` → `True` |

### List methods
| Method | What it does | Example |
|---|---|---|
| `.append(x)` | Add to end | `[1,2].append(3)` → `[1,2,3]` |
| `.insert(i, x)` | Insert at index | `[1,3].insert(1,2)` → `[1,2,3]` |
| `.remove(x)` | Remove first occurrence | `[1,2,1].remove(1)` → `[2,1]` |
| `.pop()` / `.pop(i)` | Remove last / at index | `[1,2,3].pop()` → returns `3`, list becomes `[1,2]` |
| `.sort()` | Sort in place | modifies original list |
| `.reverse()` | Reverse in place | modifies original list |

### Dict methods
| Method | What it does | Example |
|---|---|---|
| `.get(k)` | Get value (None if missing) | `d.get("x")` |
| `.get(k, default)` | Get value with fallback | `d.get("x", "N/A")` |
| `.keys()` | All keys | `d.keys()` |
| `.values()` | All values | `d.values()` |
| `.items()` | Key-value pairs | `for k, v in d.items()` |

### Set methods
| Method | What it does | Example |
|---|---|---|
| `.add(x)` | Add an item | `s.add("Jan")` |
| `.update(iterable)` | Add many items | `s.update(["Jan","Feb"])` |
| `.remove(x)` | Remove (errors if missing) | `s.remove("Jan")` |
| `.discard(x)` | Remove (silent if missing) | `s.discard("Jan")` |

---

## Reflection for today

- The reusability idea is the heart of programming. One function, many use cases.
- Confidence about loops will come from reps, not from re-reading theory.
- Pattern 2 (forgiving input handling) is the analyst's default — messy data needs forgiving code.
- Functions are infrastructure. Every analyst project you do in Weeks 3+ will be built from small functions.