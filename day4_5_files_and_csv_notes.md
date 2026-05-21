# Day 4 + 5: File I/O and CSV Parsing

> Two days combined because they're the same skill family: working with files on disk. Day 4 covered the foundation (reading and writing text files). Day 5 applied it to CSV data — the most common file format in analyst work.

---

## What you can do after these two days

- Open, read, and write text files using the `with open()` pattern
- Choose the right read method (`.read()` vs `.readlines()` vs line iteration) based on file size and need
- Understand the three file modes (`r`, `w`, `a`) and the dangers of each
- Manually parse a CSV file into a list of dictionaries
- Write defensive parsing code that handles three classes of bugs: bad values, bad rows, bad keys
- Build a validation function with multiple business rules
- Use Python's built-in `csv` module to skip manual parsing
- Connect a CSV file → cleaned data → `generate_report()` end-to-end

---

## 1. The `with open()` Pattern

This is **the** pattern for working with files in Python. Memorize it.

```python
with open(filename, mode) as f:
    # do something with f
    content = f.read()
```

### How it works under the hood

```
1. Python opens the file in the requested mode
2. Assigns it to the variable f (your choice of name)
3. Runs the indented block
4. Closes the file automatically when the block ends
5. Closes the file EVEN IF an error happened inside the block
```

### Why `with` matters

Without it, you'd write:
```python
f = open(filename, mode)
content = f.read()
f.close()    # easy to forget; not called if error happens between
```

Files that stay open eat memory and can cause OS-level resource issues. `with` is Python's "you can't forget to close" guarantee.

### The variable name (`f`) is your choice

You can name it anything. Convention is `f` for short blocks, `file` for longer ones.

```python
with open("data.txt", "r") as f:           # standard
with open("data.txt", "r") as file:        # more verbose
with open("data.txt", "r") as my_file:     # also fine
```

What's **not** your choice: the mode string. `"r"`, `"w"`, `"a"` are defined by Python.

---

## 2. File Modes — Critical to Understand

| Mode | Meaning | Existing file? | Position | Use case |
|---|---|---|---|---|
| `"r"` | Read | Must exist | Start | Reading data in |
| `"w"` | Write | **OVERWRITES** | Start | Creating new file or replacing |
| `"a"` | Append | Created if missing | End | Adding to existing without losing |

### The deadly `"w"` mistake

This deletes the file:
```python
with open("important.csv", "w") as f:
    # nothing written here
    pass
```

The file `important.csv` is now empty. `"w"` truncates immediately on open, before you write a single byte. **Always double-check the mode before working on a file you care about.**

### Append's accumulating behavior

You discovered this firsthand on Day 4 — you accidentally ran the same `"a"` cell three times and got three duplicate lines.

```python
# Run this twice → file has 2 lines
# Run it three times → file has 3 lines
with open("hello.txt", "a") as f:
    f.write("Hello\n")
```

This is by design. `"a"` doesn't check what's already there.

---

## 3. Reading Methods — Choose by File Size

Three ways to read. Use the right one for your situation.

### Method A: `.read()` — Whole file as one string

```python
with open("data.txt", "r") as f:
    content = f.read()
print(type(content))    # <class 'str'>
```

- Loads entire file into memory at once
- Fine for small files (< few MB)
- Memory blows up on huge files (gigabytes)
- Good when you need to process the file as a single block of text

### Method B: `.readlines()` — List of strings

```python
with open("data.txt", "r") as f:
    lines = f.readlines()
print(type(lines))      # <class 'list'>
print(lines[0])         # first line, INCLUDING the \n at the end
```

- Loads entire file, splits into a list of lines
- Each line preserves its trailing `\n`
- Same memory limitation as `.read()`
- Good when you want random access (`lines[5]`)

### Method C: Line-by-line iteration — Memory-efficient

```python
with open("data.txt", "r") as f:
    for line in f:
        process(line.strip())
```

- Reads one line at a time
- Memory stays constant regardless of file size
- The only practical option for huge files
- Most common in production code

### Hidden character trap

When reading, every line ends with `\n`. If you `print(line)`, Python adds *another* `\n`, giving you double-spacing. Always `.strip()` lines before using them.

```python
for line in f:
    print(line)              # double-spaced output
    print(line.strip())      # clean output
```

---

## 4. Writing Methods

### `.write(text)` — No automatic newline

```python
with open("output.txt", "w") as f:
    f.write("Hello")
    f.write("World")
# File contains: "HelloWorld" (one line, no separator)
```

You must include `\n` yourself for line breaks:
```python
with open("output.txt", "w") as f:
    f.write("Hello\n")
    f.write("World\n")
# File contains:
# Hello
# World
```

### Writing many lines

```python
lines = ["Apple", "Banana", "Cherry"]
with open("fruits.txt", "w") as f:
    for fruit in lines:
        f.write(f"{fruit}\n")
```

---

## 5. Special Characters

These are character sequences with special meanings inside strings.

| Sequence | Meaning |
|---|---|
| `\n` | Newline (line break) |
| `\t` | Tab |
| `\\` | Backslash (literal) |
| `\"` | Double quote inside a double-quoted string |
| `\'` | Single quote inside a single-quoted string |

### Raw strings — opt out of escape sequences

```python
path = "C:\new_folder"        # WRONG: \n becomes newline
path = "C:\\new_folder"        # OK: escaped backslash
path = r"C:\new_folder"        # CLEAN: raw string, ignores \
```

You'll see `r"..."` constantly in pandas (regex patterns).

---

## 6. CSV Parsing — Three Increasing Levels of Sophistication

### Level 1: Manual parsing with `.split()`

This is what you built on Day 5. It's not how you'll do it in real work — but it teaches you what's happening underneath.

```python
with open("sales_data.csv", "r") as f:
    raw = f.read()

lines = raw.split("\n")                  # split into list of lines
header = lines[0]                        # first line is header
data_rows = lines[1:]                    # rest are data

column_names = header.split(",")         # ['salesperson', 'amount', 'month']

result = []
for row in data_rows:
    values = row.split(",")
    record = {}
    for i in range(len(column_names)):
        record[column_names[i]] = values[i]
    result.append(record)
```

### Code flow visualization

```
File on disk:
┌────────────────────────────┐
│ salesperson,amount,month   │  ← header
│ Rohit,50000,Jan            │  ← row 1
│ Amit,75000,Jan             │  ← row 2
│ ...                        │
└────────────────────────────┘

         f.read()
            ↓
"salesperson,amount,month\nRohit,50000,Jan\n..."   (one big string)

         .split("\n")
            ↓
['salesperson,amount,month', 'Rohit,50000,Jan', ...]   (list of strings)

         lines[0]               lines[1:]
            ↓                       ↓
'salesperson,amount,month'    ['Rohit,50000,Jan', ...]
            
         .split(",")           [for each: .split(",")]
            ↓                       ↓
['salesperson','amount',     [['Rohit','50000','Jan'],
 'month']                     ['Amit','75000','Jan'], ...]

         pair them up by index
                ↓
[{'salesperson': 'Rohit', 'amount': '50000', 'month': 'Jan'}, ...]
```

### What this teaches you

Manual parsing forces you to confront:

1. **Numbers come in as strings.** `'50000'` is text, not 50000. You must convert.
2. **`.append()` goes at the level where the data is "finished."** Inside outer loop, outside inner loop.
3. **Indexing matters.** `header = lines[0]` and `data_rows = lines[1:]` is the standard split.

### Level 2: Type conversion at parse time

```python
result = []
for row in data_rows:
    values = row.split(",")
    record = {
        "salesperson": values[0],
        "amount": int(values[1]),       # convert here
        "month": values[2],
    }
    result.append(record)
```

**Critical principle: clean data at the boundary, not at every use site.** Once data enters your pipeline, it should be in the right type. Don't convert each time you use it.

### Level 3: Defensive parsing (handles dirty data)

Real CSV files have three classes of bugs. You learned to handle all three.

**Class A — Bad values (Day 5 morning)**
- Whitespace inside cells: `"  Rohit  "` instead of `"Rohit"`
- Non-numeric values where numbers expected: `"not_a_number"`

Fix: `.strip()` and `safe_int()`

```python
def safe_int(value):
    try:
        return int(value)
    except (ValueError, TypeError):
        return None
```

**Class B — Bad rows (Day 5 morning)**
- Empty lines (especially the trailing blank from final `\n`)
- Rows with wrong number of fields
- Whitespace-only rows

Fix: pre-checks with `continue`

```python
for row in data_rows:
    row = row.strip()
    if not row:                       # skip empty lines
        continue
    values = row.split(",")
    if len(values) != len(column_names):    # skip malformed
        errors.append(row)
        continue
    # ... build record ...
```

**Class C — Bad keys (this morning)**
- Whitespace in column headers: `" amount "` instead of `"amount"`
- This breaks `row["amount"]` with `KeyError`

Fix: strip keys when building each record

```python
clean_row = {k.strip(): v.strip() if isinstance(v, str) else v 
             for k, v in row.items()}
```

This is a **dict comprehension with a conditional** — strip both keys and values, but only call `.strip()` on values that are strings (not numbers that were already converted).

---

## 7. The `csv` Module — Python's Built-in Parser

Three lines replace 30 lines of manual parsing.

```python
import csv

with open("sales_data.csv", "r") as f:
    reader = csv.DictReader(f)
    csv_data = list(reader)
```

### What `csv.DictReader` does for you

- Reads the first row as the header automatically
- Uses headers as dictionary keys
- Returns each subsequent row as a dict
- Handles standard CSV escaping (quoted fields, commas inside quotes)
- Strips off the trailing `\n` from each row

### What it does NOT do

- **Does not convert types.** Everything is still a string until you convert.
- **Does not strip whitespace from keys or values.** Use your own cleaning step.
- **Does not skip empty rows automatically.** (Actually it does for completely blank lines, but doesn't validate row structure.)
- **Does not validate business rules.** That's your job.

### The full real-world pipeline

```python
import csv

def safe_int(value):
    try:
        return int(value)
    except (ValueError, TypeError):
        return None

with open("messy_sales.csv", "r") as f:
    reader = csv.DictReader(f)
    raw_data = []
    for row in reader:
        # Step 1: clean keys and values
        clean_row = {k.strip(): v.strip() if isinstance(v, str) else v 
                     for k, v in row.items()}
        # Step 2: convert types
        clean_row["amount"] = safe_int(clean_row["amount"])
        raw_data.append(clean_row)

# Step 3: validate
cleaned_data = [r for r in raw_data if is_valid_record(r)]
invalid_data = [r for r in raw_data if not is_valid_record(r)]
```

This is **the standard three-layer pattern** every data tool uses in some form:

```
┌──────────────────┐
│  1. PARSE        │  csv.DictReader handles structure
└──────────────────┘
         ↓
┌──────────────────┐
│  2. CONVERT      │  safe_int + strip handles types/format
└──────────────────┘
         ↓
┌──────────────────┐
│  3. VALIDATE     │  is_valid_record handles business rules
└──────────────────┘
         ↓
    cleaned_data
```

Pandas combines all three into `pd.read_csv()` with options for each. But the three layers still exist conceptually.

---

## 8. Validation Functions — The Real Analyst Skill

You wrote this today:

```python
def is_valid_record(record):
    """Check if a sales record meets all validation rules."""
    
    # Rule 1: salesperson must be non-empty
    if not record.get("salesperson") or not str(record["salesperson"]).strip():
        return False
    
    # Rule 2: amount must be a number
    if not isinstance(record.get("amount"), (int, float)):
        return False
    
    # Rule 3: amount must be positive
    if record["amount"] <= 0:
        return False
    
    # Rule 4: month must be non-empty
    if not record.get("month") or not str(record["month"]).strip():
        return False
    
    # Rule 5: month must be a valid month
    valid_months = {"Jan", "Feb", "Mar", "Apr", "May", "Jun", 
                    "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"}
    if record["month"].strip() not in valid_months:
        return False
    
    return True
```

### Five techniques to study

1. **`.get(key)` instead of `[key]`** — returns `None` if missing, doesn't crash. Safer in validation where you might receive incomplete dicts.

2. **`isinstance(x, (int, float))`** — checking against multiple types with a tuple. Cleaner than `isinstance(x, int) or isinstance(x, float)`.

3. **Sets for lookup, not lists** — `valid_months` is a `set`, not a `list`. Why? Membership checks (`x in s`) are O(1) for sets but O(n) for lists. For 12 items it doesn't matter; for thousands it does. Build the habit.

4. **One rule per `if` block** — don't combine rules with `and`. Easier to debug. Easier to add/remove rules.

5. **Early returns** — `return False` as soon as anything fails. Don't waste time checking the rest.

### Why this matters more than the report itself

Your observation today: *"For today, invalid records is real kick, not generate_report."*

This is senior-analyst thinking. A report is the *rendering layer* — anyone can build one. The validation layer is where the actual business knowledge lives.

Examples of what "validation" looks like in real companies:
- "A sale is only valid if it has a customer ID that exists in the customers table"
- "Revenue under ₹10 is treated as a test transaction and excluded"
- "Sales after midnight are attributed to the previous day's books"
- "Returns are negative amounts; positive amounts are sales"

None of these are visible in code. They live in business rules. Analysts who can articulate and codify these rules get paid much more than those who can only run queries.

---

## 9. Patterns from the Day 4-5 Work

### Pattern: Read → Process → Write

```python
# Read
with open(input_file, "r") as f:
    data = f.read()

# Process
result = some_transformation(data)

# Write
with open(output_file, "w") as f:
    f.write(result)
```

This is the shape of half of all analyst scripts. Read from a CSV, do something, write to another CSV.

### Pattern: Continue on bad data

```python
for row in rows:
    if bad_condition:
        log_error(row)
        continue
    process(row)
```

`continue` skips the rest of the current iteration and moves on. Use when "skip this and keep going" is the right behavior.

### Pattern: Keep both valid and invalid

```python
valid = [r for r in data if is_valid_record(r)]
invalid = [r for r in data if not is_valid_record(r)]
```

**Never silently drop data.** Always preserve what was rejected so you can review it later. This is the analyst's prime directive: *know what you threw away*.

### Pattern: Three-layer pipeline

```
PARSE → CONVERT → VALIDATE → USE
```

Each layer has one job. Each layer's failures are caught at that layer. This is reusable across every data tool you'll touch.

---

## 10. Debugging Skills from These Two Days

You used these techniques today. They're real skills.

### Technique 1: `repr()` to see hidden characters

```python
print(repr(line))    # shows '\n', spaces, tabs explicitly
```

Use whenever you suspect whitespace or hidden characters.

### Technique 2: Inspect first row to see actual structure

```python
first_row = next(reader)
print(list(first_row.keys()))
```

Useful when a CSV/dict isn't behaving like you expect. Always check the actual keys before assuming what they are.

### Technique 3: Print inside loops to see flow

```python
for row in rows:
    print(f"Processing: {repr(row)}")
    # ... rest of loop ...
```

Most useful for finding "which row breaks it." You used this on Day 5 to find the bad row.

### Technique 4: Read the last line of the error first

```
Cell In[50], line 13
---> 13     row["amount"] = safe_int(row["amount"])

KeyError: 'amount'
```

Last line tells you *what* went wrong. The `--->` line tells you *where*. Look at both, then think about *why*.

---

## 11. Common Errors You've Now Seen

| Error | Cause | Fix |
|---|---|---|
| `TypeError: list indices must be integers or slices, not str` | Used `["key"]` on a list | Use `.get()` on a dict instead |
| `TypeError: 'tuple' object is not callable` | Wrote `()` after a non-function variable | Remove the parentheses |
| `TypeError: unsupported operand type(s) for +: 'int' and 'str'` | Tried to add a number to a string | Convert with `int()` or `float()` |
| `IndexError: list index out of range` | Accessed index N when list has fewer items | Check `len(list)` before accessing |
| `KeyError: 'amount'` | Dict doesn't have that exact key | Check actual keys; use `.get()` |
| `ValueError: invalid literal for int() with base 10: 'abc'` | Tried `int("abc")` | Use `safe_int()` pattern |

You've now seen all six of these in practice. Most beginner errors are one of these.

---

## 12. Code Style Reminders

From your Day 5 work, three things to keep in mind:

1. **Use literal dict construction when columns are known**:
```python
record = {"salesperson": values[0], "amount": int(values[1]), "month": values[2]}
```
Not the loop version unless you don't know the columns.

2. **Match indentation purpose to logic**:
- `.append()` for completed items → outside the inner loop
- Type conversion → inside the loop, on the right field

3. **Read error messages slowly**:
- Last line first
- Then the arrow `---->`
- Then ask "what's the type of the thing on that line?"

---


---

## 13. What You Built — The Bigger Picture

If you put it all together, you now have:

```
sales_data.csv (on disk)
    ↓
csv.DictReader (parse structure)
    ↓
clean_row dict comprehension (strip keys + values)
    ↓
safe_int (convert types safely)
    ↓
is_valid_record (apply business rules)
    ↓
cleaned_data (list of validated dicts)
    ↓
generate_report (display formatted output)
```

This is **a real data pipeline.** Not a toy. Every analyst job will have you building variations of this. The only thing that changes between jobs is which library does the heavy lifting (pandas, polars, dbt, etc.) — the conceptual structure stays the same.

---

## 14. What's Next — Week 2 Preview

In Week 2, you'll discover that pandas does all of this — parsing, cleaning, validation, aggregation — in roughly **10 lines of code** instead of the 60+ you have now.

```python
import pandas as pd

df = pd.read_csv("sales_data.csv")           # parse
df["amount"] = pd.to_numeric(df["amount"], errors="coerce")   # convert
df = df.dropna()                              # remove invalid
totals = df.groupby("salesperson")["amount"].sum()    # aggregate
```

When you see this, you'll have two reactions:

1. **"That's so much shorter!"** — yes, that's the magic.
2. **"But I understand exactly what's happening under the hood."** — that's the gift of having built it manually first.

This second reaction is what separates people who use pandas as a black box from people who debug pandas confidently. Most beginners only have the first reaction. You'll have both.

---

## Reflection for Day 4-5

- Manual parsing teaches you what tools do. Tools save you from doing it again.
- "Never drop data silently" is the single most important data philosophy.
- The validation layer is where analyst judgment lives. Not in dashboards.
- Three classes of CSV bugs — values, rows, keys — each need their own defense.
- The three-layer pipeline (parse → convert → validate) applies everywhere.
- A 4-day break is fine. The plan absorbed it. Show up when you can.
```

