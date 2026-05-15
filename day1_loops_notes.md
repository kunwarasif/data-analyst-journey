
# Day 1 Revision Notes: Loops ( Foundation)

## The core idea of a loop

A loop says: **"do this same thing, once for each item in a collection."**

```python
for item in collection:
    do_something_with(item)
```

That's the whole pattern. The collection can be a list, a string, a dictionary, a range of numbers — almost anything. Python just goes through it one item at a time and lets you work with each one.

The variable name (`item`, `sale`, `name`, whatever you choose) is a **temporary label** for "the current thing in this iteration."

---

## 1. Looping through a LIST

A list gives you each **value**, one at a time:

```python
sales = [50000, 75000, 60000]

for amount in sales:
    print(amount)
```

Output:
```
50000
75000
60000
```

**Mental model:** Imagine a stack of cards. Python picks up the top card, hands it to you as `amount`. You do something with it. Python picks up the next card. Hands it to you. Repeats until cards are gone.

You can name the variable anything:
```python
for x in sales:          # same thing
for value in sales:      # same thing
for sale_amount in sales:  # same thing, more descriptive
```

Use **descriptive names** in real code. `for amount in sales` is much clearer than `for x in sales`.

---

## 2. Looping through a STRING

A string is treated like a list of characters:

```python
name = "Rohit"

for letter in name:
    print(letter)
```

Output:
```
R
o
h
i
t
```

Most of the time you won't loop through strings character by character — but knowing it works helps you understand that **strings are iterable** like lists.

---

## 3. Looping through a RANGE (numbers)

`range()` generates a sequence of numbers. Useful when you want to repeat something N times or work with positions.

```python
for i in range(5):
    print(i)
```
Output:
```
0
1
2
3
4
```

Notice: `range(5)` gives **0 to 4**, not 1 to 5. The end is exclusive.

```python
for i in range(1, 6):       # start at 1, stop before 6
    print(i)
# 1, 2, 3, 4, 5

for i in range(0, 10, 2):   # start 0, stop before 10, step by 2
    print(i)
# 0, 2, 4, 6, 8
```

**When you'd use it:** "Repeat this action 10 times" or "loop through positions 0 to 9 of a list."

---

## 4. Looping through a LIST OF DICTIONARIES (your Sales Tracker case)

This is what tripped you up. Let's nail it.

```python
sales_data = [
    {"salesperson": "Rohit", "amount": 50000},
    {"salesperson": "Amit", "amount": 75000},
    {"salesperson": "Neha", "amount": 60000},
]

for sale in sales_data:
    print(sale)
```

Output:
```
{'salesperson': 'Rohit', 'amount': 50000}
{'salesperson': 'Amit', 'amount': 75000}
{'salesperson': 'Neha', 'amount': 60000}
```

**What's `sale` in each iteration?** It's the **whole dictionary** for that row.

To get individual values from inside the dict, use the key:
```python
for sale in sales_data:
    print(sale["salesperson"])    # gets the name
    print(sale["amount"])          # gets the number
```

**Mental model:** Each row is like a small record/form. The loop hands you one form at a time. You then look at specific fields on that form (`sale["salesperson"]`, `sale["amount"]`).

---

## 5. Looping through a DICTIONARY directly

Different from a list — dicts have keys and values, so you have **three ways** to loop:

```python
totals = {"Rohit": 165000, "Amit": 120000, "Neha": 170000}
```

**Way 1: Loop through KEYS** (default behavior)
```python
for name in totals:
    print(name)
```
Output:
```
Rohit
Amit
Neha
```

This is the default — when you loop through a dict, you get keys.

**Way 2: Loop through VALUES**
```python
for amount in totals.values():
    print(amount)
```
Output:
```
165000
120000
170000
```

**Way 3: Loop through BOTH (keys and values together)** — most useful
```python
for name, amount in totals.items():
    print(name, amount)
```
Output:
```
Rohit 165000
Amit 120000
Neha 170000
```

`.items()` gives you pairs. You unpack each pair into two variables.

---

## 6. The two patterns inside loops

This is where loops become useful. Almost every analyst loop uses one of these two patterns:

### Pattern A: Accumulator (build something as you go)

You start with an empty container, then add to it during the loop.

**Sum accumulator** — building up a number:
```python
total = 0
for amount in [50000, 75000, 60000]:
    total = total + amount
print(total)   # 185000
```

**Dict accumulator** — building up counts/totals per category:
```python
totals = {}
for sale in sales_data:
    name = sale["salesperson"]
    if name in totals:
        totals[name] = totals[name] + sale["amount"]
    else:
        totals[name] = sale["amount"]
```

**List accumulator** — building up a new list:
```python
big_sales = []
for amount in [50000, 75000, 60000, 100000]:
    if amount > 70000:
        big_sales.append(amount)
print(big_sales)   # [75000, 100000]
```

### Pattern B: Tracker (find the best/biggest/smallest)

You keep track of the "best so far" and update when you find something better.

```python
top_amount = 0
for amount in [50000, 75000, 60000]:
    if amount > top_amount:
        top_amount = amount
print(top_amount)   # 75000
```

Same pattern works for finding the minimum (use `<` instead of `>` and start with a very high number, or use `float("inf")`).

---

## 7. The shortcut symbols (good to know)

These two lines do the same thing:
```python
total = total + amount
total += amount        # shortcut, "increment"
```

Other shortcuts:
```python
count += 1        # same as: count = count + 1
total -= 100      # same as: total = total - 100
multiplier *= 2   # same as: multiplier = multiplier * 2
```

Use whichever feels clearer. `+=` is more common.

---

## 8. The mental model to lock in forever

When you see a loop, train yourself to ask:

1. **What am I looping through?** (list of numbers? list of dicts? a dictionary?)
2. **What is the loop variable each iteration?** (a number? a dict? a string? a tuple?)
3. **What am I trying to build or find?** (a total? a new list? the maximum?)
4. **What container do I need before the loop starts?** (a 0? an empty list? an empty dict?)

If you ask those four questions every time, you'll never be lost in a loop again.

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

---
