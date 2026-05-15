Here's the complete cheatsheet through Day 3. Replace your existing `methods_cheatsheet.md` with this — fully consolidated, organized, ready to grow.

---

```markdown
# Python Methods & Functions Cheatsheet

A running reference of every built-in function, method, and pattern I've used in this journey. Updated daily.

**Last updated:** End of Day 3

---

## 1. Built-in Functions

Functions that work without a dot — they're general-purpose and accept many types.

| Function | What it does | Example |
|---|---|---|
| `print(x)` | Display on screen | `print("hi")` → `hi` |
| `len(x)` | Length of list/string/dict/set | `len([1,2,3])` → `3` |
| `type(x)` | Type of x | `type(5)` → `<class 'int'>` |
| `isinstance(x, t)` | True if x is type t | `isinstance(5, int)` → `True` |
| `sum(x)` | Sum of numbers in iterable | `sum([1,2,3])` → `6` |
| `max(x)` | Largest value | `max([1,5,3])` → `5` |
| `min(x)` | Smallest value | `min([1,5,3])` → `1` |
| `sorted(x)` | Returns new sorted list (original unchanged) | `sorted([3,1,2])` → `[1,2,3]` |
| `sorted(x, key=..., reverse=...)` | Sorted with custom key and order | `sorted(d.items(), key=lambda x: x[1], reverse=True)` |
| `range(n)` | Numbers 0 to n-1 | `range(5)` → `0,1,2,3,4` |
| `range(start, stop)` | Numbers start to stop-1 | `range(1,6)` → `1,2,3,4,5` |
| `range(start, stop, step)` | With step | `range(0,10,2)` → `0,2,4,6,8` |
| `enumerate(x)` | Pairs (index, value) | `enumerate(["a","b"])` → `(0,"a"),(1,"b")` |
| `set(x)` | Convert to set / make empty set | `set([1,1,2])` → `{1,2}` |
| `int(x)` | Convert to integer | `int("5")` → `5` |
| `float(x)` | Convert to float | `float("3.5")` → `3.5` |
| `str(x)` | Convert to string | `str(5)` → `"5"` |

---

## 2. String Methods

Called with a dot: `"text".method()`. Strings are immutable — these return NEW strings, original unchanged.

| Method | What it does | Example |
|---|---|---|
| `.strip()` | Remove leading/trailing whitespace | `"  hi  ".strip()` → `"hi"` |
| `.lstrip()` | Remove leading whitespace only | `"  hi  ".lstrip()` → `"hi  "` |
| `.rstrip()` | Remove trailing whitespace only | `"  hi  ".rstrip()` → `"  hi"` |
| `.lower()` | All lowercase | `"HI".lower()` → `"hi"` |
| `.upper()` | All uppercase | `"hi".upper()` → `"HI"` |
| `.title()` | Title Case Each Word | `"hi there".title()` → `"Hi There"` |
| `.replace(a, b)` | Replace a with b | `"hi".replace("h","b")` → `"bi"` |
| `.split(sep)` | Split into list | `"a,b,c".split(",")` → `["a","b","c"]` |
| `.join(iterable)` | Join collection with separator | `", ".join(["a","b"])` → `"a, b"` |
| `.startswith(s)` | True if starts with s | `"hello".startswith("he")` → `True` |
| `.endswith(s)` | True if ends with s | `"hello".endswith("lo")` → `True` |

---

## 3. List Methods

Called with a dot. Lists are mutable — most methods modify the original list.

| Method | What it does | Returns |
|---|---|---|
| `.append(x)` | Add x to end | None (modifies list) |
| `.insert(i, x)` | Insert x at index i | None (modifies list) |
| `.remove(x)` | Remove first occurrence of x | None (modifies list) |
| `.pop()` | Remove and return last item | The removed item |
| `.pop(i)` | Remove and return item at index i | The removed item |
| `.sort()` | Sort in place | None (modifies list) |
| `.sort(reverse=True)` | Sort descending in place | None (modifies list) |
| `.reverse()` | Reverse in place | None (modifies list) |

**Important:** `.sort()` modifies the list and returns `None`. Use `sorted(list)` if you need a new sorted list while keeping the original.

---

## 4. Dictionary Methods

Called with a dot.

| Method | What it does | Example |
|---|---|---|
| `.get(k)` | Get value (None if key missing) | `d.get("x")` → `None` if no `"x"` |
| `.get(k, default)` | Get value with fallback | `d.get("x", "N/A")` → `"N/A"` if missing |
| `.keys()` | All keys (view) | `for k in d.keys()` |
| `.values()` | All values (view) | `for v in d.values()` |
| `.items()` | Key-value pairs as tuples | `for k, v in d.items()` |

**Important:** `"key" in d` checks **keys only**, not values. To check values: `"val" in d.values()`.

---

## 5. Set Methods

Called with a dot. Sets are mutable and unordered, store unique values only.

| Method | What it does | Example |
|---|---|---|
| `.add(x)` | Add one item | `s.add("Jan")` |
| `.update(iterable)` | Add many items | `s.update(["Jan","Feb"])` |
| `.remove(x)` | Remove (errors if missing) | `s.remove("Jan")` |
| `.discard(x)` | Remove (silent if missing) | `s.discard("Jan")` |

**Important:** Use `set()` to create an empty set. `{}` creates an empty dict, not a set.

---

## 6. Type Conversion Patterns

Common conversions you'll do constantly in data work.

| From | To | How | Notes |
|---|---|---|---|
| `"5"` (str) | `5` (int) | `int("5")` | Errors if not a number string |
| `"5.5"` (str) | `5.5` (float) | `float("5.5")` | |
| `"5.5"` (str) | `5` (int) | `int(float("5.5"))` | Must go through float; truncates |
| `5` (int) | `"5"` (str) | `str(5)` | Always works |
| `5` (int) | `5.0` (float) | `float(5)` | Always works |
| `[1,1,2]` (list) | `{1,2}` (set) | `set([1,1,2])` | Removes duplicates |
| `{1,2,3}` (set) | `[1,2,3]` (list) | `list({1,2,3})` | Order not guaranteed |

---

## 7. f-string Format Specifiers

Inside `f"..."`, use `:` after a variable for formatting.

### Number formatting

| Pattern | What it does | Example output |
|---|---|---|
| `:,` | Thousands separator | `455000` → `455,000` |
| `:.2f` | Float with 2 decimals | `3.14159` → `3.14` |
| `:.0f` | Float with 0 decimals (no decimal point) | `3.7` → `4` |
| `:,.2f` | Comma + 2 decimals | `1234567.89` → `1,234,567.89` |
| `:.1%` | Percentage with 1 decimal | `0.374` → `37.4%` |
| `:.2%` | Percentage with 2 decimals | `0.374` → `37.40%` |

### String/number alignment (useful for tables)

| Pattern | What it does | Example |
|---|---|---|
| `:<10` | Left-align in 10 chars | `"Rohit     "` |
| `:>10` | Right-align in 10 chars | `"     Rohit"` |
| `:^10` | Center in 10 chars | `"  Rohit   "` |

### Combining patterns

```python
f"{name:<10} ₹{amount:>10,}   ({pct:.1%})"
# "Neha       ₹   170,000   (37.4%)"
```

---

## 8. Loop Patterns

The four core patterns every analyst loop uses.

### Pattern A: Sum/Count Accumulator

```python
total = 0
for x in data:
    total = total + x       # or total += x for shortcut
```

### Pattern B: Group Accumulator (dict)

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

### Pattern D: Tracker (find biggest/smallest)

```python
best_amount = 0
best_name = ""
for item in data:
    if item["amount"] > best_amount:
        best_amount = item["amount"]
        best_name = item["name"]
```

These four patterns solve 80% of analyst problems with plain Python. They map directly to pandas:
- Pattern A → `df['col'].sum()`
- Pattern B → `df.groupby('category')['value'].sum()`
- Pattern C → `df[df['amount'] > 60000]['name']`
- Pattern D → `df.loc[df['amount'].idxmax()]`

---

## 9. Function Patterns

### Defining a function

```python
def function_name(parameter1, parameter2):
    """Short docstring describing what it does."""
    # logic
    return result
```

### Default parameter values

```python
def calculate_commission(amount, rate=0.05):
    return amount * rate

calculate_commission(50000)         # uses default rate
calculate_commission(50000, 0.10)   # overrides
```

### Input validation — strict (raise error)

```python
def double_strict(number):
    if not isinstance(number, (int, float)):
        raise TypeError(f"Expected number, got {type(number).__name__}")
    return number * 2
```

### Input validation — safe (return None)

```python
def double_safe(number):
    try:
        return float(number) * 2
    except (ValueError, TypeError):
        return None
```

**Rule of thumb for analysts:** prefer the safe pattern. Real-world data is messy; you usually want to skip bad rows, not crash.

### Function composition

Small functions calling other functions:

```python
def get_average_sale(data):
    return get_total_sales(data) / get_transaction_count(data)
```

---

## 10. Lambda Functions

One-line anonymous functions.

```python
lambda x: x * 2                  # equivalent to: def f(x): return x * 2
lambda x: x[1]                   # returns the second element of x
lambda sale: sale["amount"]      # returns the "amount" key of a dict
```

### Most common use: custom sort key

```python
# Sort dict by value, descending
sorted(d.items(), key=lambda x: x[1], reverse=True)

# Sort list of dicts by a field
sorted(sales_data, key=lambda sale: sale["amount"], reverse=True)
```

You'll use lambda constantly in pandas. Get comfortable with it now.

---

## 11. Tuple Unpacking

Splitting a tuple into multiple variables in one line.

```python
# Basic unpacking
first, second = (10, 20)
a, b, c = "abc"                  # works on strings too

# In for loops with .items()
for name, amount in totals.items():
    ...

# In for loops with enumerate
for index, value in enumerate(my_list):
    ...

# From function returns
top_name, top_amount = get_top_performer(data)
```

If the function returns a tuple, you can unpack on assignment.

---

## 12. List/Set/Dict Comprehensions

Compact one-line versions of loops.

### List comprehension

```python
# Long form
squares = []
for x in range(10):
    squares.append(x ** 2)

# Comprehension
squares = [x ** 2 for x in range(10)]

# With condition
even_squares = [x ** 2 for x in range(10) if x % 2 == 0]
```

### Set comprehension

```python
unique_months = {sale["month"] for sale in sales_data}
```

### Dict comprehension

```python
discounted = {k: v * 0.9 for k, v in prices.items()}
```

**Pattern:** `[expression for item in iterable if condition]`

Use when the logic fits in one readable line. If complex, use a regular for loop.

---

## 13. Exception Handling

```python
try:
    result = risky_operation()
except SpecificError:
    # handle this specific error
    result = default_value
except (Error1, Error2):
    # handle multiple error types
    result = default_value
except Exception as e:
    # catch-all (use sparingly)
    print(f"Error: {e}")
finally:
    # always runs, whether error or not
    cleanup()
```

**Rule of thumb:** catch specific exceptions, not bare `except:`. Hidden errors create silent bugs.

---

## 14. Common Error Messages — Decoded

| Error | What it usually means |
|---|---|
| `TypeError: list indices must be integers or slices, not str` | You used `["key"]` on a list instead of a dict |
| `TypeError: 'tuple' object is not callable` | You added `()` after a non-function variable |
| `KeyError: 'xyz'` | Dict has no key `'xyz'`. Use `.get("xyz", default)` to handle safely |
| `IndexError: list index out of range` | Tried to access index beyond list length |
| `NameError: name 'x' is not defined` | Variable `x` doesn't exist yet — typo or scope issue |
| `IndentationError` | Inconsistent indentation. Use 4 spaces, no tabs |
| `SyntaxError: invalid syntax` | Typo in code — missing colon, quote, bracket |

### The 30-second debug routine

1. Read the **last line** of the error first — that's the actual problem
2. Find the line marked with `---->` — that's where it broke
3. Ask: what *type* is each variable on that line?
4. If the type doesn't match what the operation expects → there's your bug

---

## 15. Style & Naming Conventions

- **`snake_case`** for variables and functions: `get_total_sales`, `sales_data`
- **`PascalCase`** for classes (you'll meet this later)
- **`UPPER_CASE`** for constants: `MAX_RETRIES = 5`
- Use **descriptive names**: `for sale in sales_data` (good), `for x in d` (bad)
- Use **`return`** inside functions, not `print` (let the caller decide what to do)
- **`return value`** — no parentheses (e.g. `return total`, not `return(total)`)
- One function = one job. If a function does 3 things, split it into 3 functions.
- **Always add a docstring** to functions:
  ```python
  def my_func(data):
      """Short description of what this does."""
  ```

---

## 16. The DRY Principle — Don't Repeat Yourself

**Every piece of work happens in exactly one place.**

If you compute a value (or write a piece of logic) more than once, store it in a variable or extract it into a function. Repeated work means:
- Wasted performance
- Harder debugging
- More places to update if logic changes

Good habit: when you see yourself typing the same expression twice, stop and refactor.

---

## 17. Quick Reference — "How do I...?"

| Goal | How |
|---|---|
| Sum a list of numbers | `sum(my_list)` |
| Count items in a list | `len(my_list)` |
| Get unique items from a list | `set(my_list)` |
| Sort a list ascending | `sorted(my_list)` |
| Sort a list descending | `sorted(my_list, reverse=True)` |
| Sort a list of dicts by a field | `sorted(data, key=lambda x: x["field"])` |
| Group totals by category | Loop + dict accumulator (Pattern B) |
| Find the max item by some field | Loop + tracker (Pattern D) |
| Filter items meeting a condition | Loop + filter (Pattern C), or list comprehension |
| Combine items into a string | `", ".join(items)` |
| Check if key is in dict | `"key" in my_dict` |
| Check if value is in dict | `"val" in my_dict.values()` |
| Convert string to number | `int(s)` or `float(s)` |
| Format number with commas | `f"{n:,}"` |
| Format as percentage | `f"{x:.1%}"` |
| Pad string to fixed width | `f"{s:<10}"` (left) or `f"{s:>10}"` (right) |

---

## What's still to come (will add as I learn)

- File I/O (`open`, `read`, `write`) — Day 4
- CSV parsing — Day 4
- Date handling — Day 5
- Pandas methods — Week 2
- Common pandas patterns — Week 2-3
- SQL window functions — Week 9
- Power BI / Power Query reference — Weeks 6-8
```

---

## How to use this cheatsheet

**Don't read it cover to cover.** It's a reference. Use it when:
- You're trying to remember "how do I again?" — check section 17 first
- You see a method in someone's code and want to know what it does
- You're stuck on an error — check section 14
- You want to refresh on a pattern before solving a new problem

## File I/O

### Opening files — the `with open()` pattern

```python
with open(filename, mode) as f:
    # do something with f
```

Always use `with` — it auto-closes the file.

### File modes

| Mode | Meaning | Behavior |
|---|---|---|
| `"r"` | Read | Default. File must exist. |
| `"w"` | Write | **Overwrites entire file.** Creates if doesn't exist. |
| `"a"` | Append | Adds to end. Creates if doesn't exist. |

### Reading methods

| Method | Returns | Use when |
|---|---|---|
| `f.read()` | One big string | Small file, want all content as text |
| `f.readlines()` | List of strings, one per line | Small file, want lines separately |
| `for line in f:` | One line at a time | Large file, memory-efficient |

### Writing methods

| Method | What it does |
|---|---|
| `f.write(text)` | Writes string to file. No newline added — include `\n` if you want one. |

### Special characters

| Character | Meaning |
|---|---|
| `\n` | Newline (start a new line) |
| `\t` | Tab |
| `\\` | Backslash (escaped) |
| `\"` | Double quote inside a string |



