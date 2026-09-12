# Python: Common Traps and Misunderstandings — Comprehensive

This is a reference guide for 40+ subtle Python behaviors that cause bugs, fail interviews, and lead to production disasters. Each trap shows what people think, what actually happens, why it works that way, and how to avoid it.

---

## Category 1: Mutability & References

### Trap 1: Mutable Default Arguments

**What people think:**
Default arguments are created fresh each time the function is called.

**Reality:**
Default arguments are evaluated ONCE when the function is defined, not each time it's called. If the default is mutable, all calls share the same object.

**Code Example:**
```python
def append_item(item, items=[]):
    items.append(item)
    return items

result1 = append_item(1)
print(result1)  # [1]

result2 = append_item(2)
print(result2)  # [1, 2] ← UNEXPECTED! Should be [2]

result3 = append_item(3)
print(result3)  # [1, 2, 3] ← Same list reused!

# Proof:
print(append_item.__defaults__)  # ([1, 2, 3],) ← Default list object
```

**Memory Diagram:**
```
Function definition time:
[]  ← Default list object created at address 0x5000

Call 1: append_item(1) with items=[0x5000]
0x5000 → [1]

Call 2: append_item(2) with items=[0x5000] (SAME object!)
0x5000 → [1, 2]

Call 3: append_item(3) with items=[0x5000] (STILL same object!)
0x5000 → [1, 2, 3]
```

**Why this happens:**
Python evaluates function defaults at definition time (compile-time-ish). The result is stored in the function object's `__defaults__` attribute and reused for every call.

**How to avoid:**
```python
# CORRECT:
def append_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items

result1 = append_item(1)  # [1]
result2 = append_item(2)  # [2] ✓ Correct
result3 = append_item(3)  # [3]
```

**Interview angle:**
"This code has a bug. Find it and explain why it happens."
```python
def extend_list(item, mylist=[]):
    mylist.append(item)
    return mylist
```

**Real-world impact:**
This is one of Python's most infamous gotchas. It's caused countless production bugs where shared state mysteriously appears across function calls.

---

### Trap 2: List Aliasing in Data Processing

**What people think:**
`backup = data` makes a copy of the list.

**Reality:**
`backup = data` makes an alias — both point to the same list object. Changes to one affect the other.

**Code Example:**
```python
# Data processing pipeline:
original_data = [1, 2, 3, 4, 5]
backup = original_data  # Thought we made a backup!

# Processing:
original_data.append(999)
original_data[0] = -1

print(backup)  # [-1, 2, 3, 4, 5, 999] ← CORRUPTED!
print(original_data is backup)  # True (same object)
```

**Memory Diagram:**
```
original_data ──┐
                └─> [List at 0x8000: [1, 2, 3, 4, 5]]
backup ────────┘    (same object)

After modifications:
original_data ──┐
                └─> [List at 0x8000: [-1, 2, 3, 4, 5, 999]]
backup ────────┘
```

**How to avoid:**
```python
# Shallow copy (new list, but shared inner objects):
backup = original_data.copy()
# or
backup = original_data[:]
# or
backup = list(original_data)

# Deep copy (recursively copy everything):
import copy
backup = copy.deepcopy(original_data)
```

**Which to use:**
```python
data = [1, 2, 3]
backup = data.copy()  # ✓ Usually what you want

data = [1, [2, 3], 4]  # Nested lists
backup = data.copy()  # ✗ Inner list still shared!
backup = copy.deepcopy(data)  # ✓ Correct for nested
```

**Real-world example:**
```python
# ML training: accidentally sharing model configurations
default_config = {'lr': 0.001, 'epochs': 10}

model1_config = default_config  # Alias!
model1_config['lr'] = 0.01  # Changes default!

model2_config = default_config  # Gets modified config
print(model2_config['lr'])  # 0.01 (not intended!)
```

---

### Trap 3: Shallow Copy Gotcha with Nested Objects

**What people think:**
`list.copy()` creates a fully independent copy.

**Reality:**
`list.copy()` is a shallow copy. It creates a new list, but inner mutable objects are still shared.

**Code Example:**
```python
original = [1, [2, 3], 4]
shallow = original.copy()

# Modifying outer list doesn't affect original:
shallow.append(999)
print(original)  # [1, [2, 3], 4] ✓ Not affected

# But modifying inner list DOES affect original:
shallow[1].append(999)
print(original)  # [1, [2, 3, 999], 4] ← Inner list shared!
```

**Memory Diagram:**
```
original ──> [List at 0x1000]
             └─> element 0: 1
             └─> element 1: [List at 0x2000: [2, 3]]
             └─> element 2: 4

shallow ──> [List at 0x3000]  ← NEW list
             └─> element 0: 1 (copied value)
             └─> element 1: [List at 0x2000: [2, 3]] ← SAME inner list!
             └─> element 2: 4

After shallow[1].append(999):
Both original[1] and shallow[1] point to 0x2000:
[2, 3, 999]
```

**When this matters:**
```python
# Data analysis: DataFrame with nested structures
data = [{'name': 'Alice', 'scores': [90, 85, 92]},
        {'name': 'Bob', 'scores': [88, 91, 87]}]

backup = data.copy()  # Shallow copy

# Modifying a student's scores:
backup[0]['scores'].append(95)

# Original is affected!
print(data[0]['scores'])  # [90, 85, 92, 95] ← Corrupted!
```

**How to fix:**
```python
import copy

# Use deep copy for nested structures:
backup = copy.deepcopy(data)
backup[0]['scores'].append(95)
print(data[0]['scores'])  # [90, 85, 92] ✓ Not affected
```

---

### Trap 4: Modifying List While Iterating

**What people think:**
You can modify a list while iterating over it.

**Reality:**
Modifying a list during iteration can skip or repeat elements (iterator state becomes inconsistent).

**Code Example:**
```python
lst = [1, 2, 3, 4, 5]

# Remove even numbers:
for item in lst:
    if item % 2 == 0:
        lst.remove(item)

print(lst)  # [1, 3, 5] ← Sometimes works, but...

# Example where it fails:
lst = [1, 2, 3, 4, 5, 6]
for item in lst:
    if item % 2 == 0:
        lst.remove(item)

print(lst)  # [1, 3, 5] ✓ Works (by luck)

# But:
lst = [1, 2, 4, 6, 8]
for item in lst:
    if item > 1:
        lst.remove(item)

print(lst)  # [1, 2] ← Unpredictable! Skipped 4.
```

**Why this happens:**
When you modify a list, the internal array shifts. The iterator's position becomes invalid, causing it to skip elements.

```
Original: [1, 2, 4, 6, 8]
Iterator at: position 1 (value 2)

Remove 2:
[1, 4, 6, 8]
Iterator at: position 1 → now points to 6 (skipped 4!)

Remove 6:
[1, 4, 8]
Iterator at: position 1 → now points to 8 (skipped 4!)
```

**How to avoid:**
```python
# Option 1: Iterate over a copy:
lst = [1, 2, 3, 4, 5]
for item in lst.copy():  # Iterate over copy, modify original
    if item % 2 == 0:
        lst.remove(item)

# Option 2: List comprehension (create new list):
lst = [x for x in lst if x % 2 != 0]

# Option 3: Iterate backward (only for removal):
for i in range(len(lst) - 1, -1, -1):
    if lst[i] % 2 == 0:
        lst.pop(i)
```

**Best practice:**
Use list comprehension or filter, don't modify during iteration.

---

## Category 2: Scope & Variable Binding

### Trap 5: Late Binding in Closures

**What people think:**
Lambdas/nested functions capture variable values at creation time.

**Reality:**
Closures capture variables by reference, not by value. The value is looked up when the function is called, not when it's defined.

**Code Example:**
```python
# Create list of functions:
functions = []
for i in range(3):
    def func():
        return i  # Captures i by reference
    functions.append(func)

# Call each function:
print([f() for f in functions])  # [2, 2, 2] ← NOT [0, 1, 2]!

# Why? When functions are called, i=2 (loop finished):
print(i)  # 2
```

**Execution trace:**
```
Loop iteration 0:
- Define func that references i
- i = 0
- Append func

Loop iteration 1:
- Define func that references i
- i = 1
- Append func

Loop iteration 2:
- Define func that references i
- i = 2
- Append func

Call functions[0]():
- func looks up i
- i = 2 (loop finished)
- Return 2

Call functions[1]():
- Same i = 2
- Return 2

Call functions[2]():
- Same i = 2
- Return 2
```

**How to avoid:**
```python
# Capture by value using default argument:
functions = []
for i in range(3):
    def func(i=i):  # i=i captures current value of i
        return i
    functions.append(func)

print([f() for f in functions])  # [0, 1, 2] ✓ Correct!

# Or use lambda with default:
functions = [lambda i=i: i for i in range(3)]
print([f() for f in functions])  # [0, 1, 2]
```

**Real-world example:**
```python
# Buttons in GUI:
buttons = []
for i in range(3):
    # WRONG:
    button = Button(text=f"Button {i}", 
                    command=lambda: print(i))
    buttons.append(button)

# All buttons print 2 when clicked!

# CORRECT:
buttons = []
for i in range(3):
    button = Button(text=f"Button {i}", 
                    command=lambda i=i: print(i))
    buttons.append(button)
# Each button prints its own number
```

---

### Trap 6: Global/Nonlocal Scope Confusion

**What people think:**
You can modify a global or enclosing variable without declaring it.

**Reality:**
If you assign to a variable, Python assumes it's local. You must use `global` or `nonlocal` to modify outer variables.

**Code Example:**
```python
x = 10

def func():
    x = x + 1  # UnboundLocalError!

func()

# Why error? Python sees "x =" on left side, so x is local.
# But then it tries to read x before it's defined (right side).
```

**Execution trace:**
```
def func():
    x = x + 1  # Python parses this as: x is local variable
               # But first we try to read local x (not assigned yet)
               # UnboundLocalError: local variable 'x' referenced before assignment
```

**How to fix:**
```python
x = 10

def func():
    global x  # Declare: use global x, not local x
    x = x + 1

func()
print(x)  # 11 ✓ Correct

# Same issue with nested functions:
def outer():
    y = 10
    
    def inner():
        y = y + 1  # UnboundLocalError!
    
    inner()

# Fix with nonlocal:
def outer():
    y = 10
    
    def inner():
        nonlocal y  # Modify enclosing y
        y = y + 1
    
    inner()
    print(y)  # 11

outer()
```

---

## Category 3: String and Number Behavior

### Trap 7: String is Immutable but Appears to Modify

**What people think:**
Strings can be modified like lists.

**Reality:**
Strings are immutable. Operations that appear to "modify" a string actually create a new string.

**Code Example:**
```python
s = "hello"
s[0] = 'H'  # TypeError: 'str' object does not support item assignment

# To "change" a string:
s = 'H' + s[1:]  # Creates NEW string

# Other operations also create new strings:
s = "hello"
s = s.upper()  # Returns new string, original unchanged
s = s.replace('l', 'x')  # Returns new string

# Common mistake:
s = "hello"
s.upper()  # Returns "HELLO" but doesn't modify s
print(s)  # "hello" (unchanged!)

# Correct:
s = s.upper()  # Must reassign
print(s)  # "HELLO"
```

**Why this matters:**
```python
# Performance: string concatenation in loop
# SLOW - O(n²):
s = ""
for word in words:
    s = s + word  # Creates new string each time!

# FAST - O(n):
s = "".join(words)  # Single operation
```

---

### Trap 8: Integer Caching Anomaly

**What people think:**
All integers are unique objects (or all are the same).

**Reality:**
Python caches integers -5 to 256. Beyond that, identity is inconsistent.

**Code Example:**
```python
# Small integers are cached:
a = 5
b = 5
print(a is b)  # True (same cached object)

# Larger integers are not cached:
a = 257
b = 257
print(a is b)  # False! (different objects)

# But equality works:
print(a == b)  # True (same value)

# Even weirder:
print((257) is (257))  # False
print(257 + 1 - 1 is 257)  # May be True or False (depends on context)

# Proof:
print(id(5))    # Same ID every time
print(id(257))  # Different IDs
```

**Why this happens:**
Caching small integers is an optimization (they're used frequently). It's an implementation detail, not guaranteed.

**How to avoid:**
Always use `==` for value comparison, `is` only for singletons (None, True, False):

```python
# WRONG:
if x is 5:
    pass

# RIGHT:
if x == 5:
    pass

# IS is only for singletons:
if x is None:
    pass

if x is True:  # Still not recommended; use bool(x) or if x:
    pass
```

---

### Trap 9: Float Precision Problems

**What people think:**
`0.1 + 0.2 == 0.3` is True.

**Reality:**
Due to binary floating-point representation, it's False.

**Code Example:**
```python
print(0.1 + 0.2)  # 0.30000000000000004 (not 0.3!)
print(0.1 + 0.2 == 0.3)  # False!

# Proof:
print(f"{0.1 + 0.2:.20f}")  # 0.30000000000000004449
print(f"{0.3:.20f}")        # 0.29999999999999998889
```

**Why this happens:**
`0.1` cannot be represented exactly in binary floating-point. It's the closest approximation, which accumulates error through operations.

**How to avoid:**
```python
# Option 1: Use math.isclose:
import math
print(math.isclose(0.1 + 0.2, 0.3))  # True

# Option 2: Use Decimal for precise arithmetic:
from decimal import Decimal
a = Decimal('0.1')
b = Decimal('0.2')
c = Decimal('0.3')
print(a + b == c)  # True!

# Option 3: Round to sensible precision:
print(round(0.1 + 0.2, 10) == 0.3)  # True
```

**Real-world impact:**
```python
# Financial calculations MUST use Decimal:
prices = [0.1, 0.2, 0.3, 0.4, 0.5]
total = sum(prices)
print(total)  # 1.5000000000000002 (not 1.5!)

# Correct:
from decimal import Decimal
prices = [Decimal(str(x)) for x in ['0.1', '0.2', '0.3', '0.4', '0.5']]
total = sum(prices)  # Decimal('1.5') ✓
```

---

## Category 4: Object Model Subtleties

### Trap 10: __init__ vs __new__

**What people think:**
`__init__` creates the object.

**Reality:**
`__new__` creates the object; `__init__` initializes it.

**Code Example:**
```python
class MyClass:
    def __new__(cls, value):
        print(f"__new__ called with {value}")
        instance = super().__new__(cls)  # Creates object
        return instance
    
    def __init__(self, value):
        print(f"__init__ called with {value}")
        self.value = value  # Initializes object

obj = MyClass(42)
# Output:
# __new__ called with 42
# __init__ called with 42

print(obj.value)  # 42
```

**Why it matters:**
```python
# Singleton pattern:
class Singleton:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

s1 = Singleton()
s2 = Singleton()
print(s1 is s2)  # True (same object)
```

---

### Trap 11: Method Resolution Order (MRO) in Multiple Inheritance

**What people think:**
Multiple inheritance is simple; methods are found in left-to-right order.

**Reality:**
Python uses C3 Linearization algorithm (complex).

**Code Example:**
```python
class A:
    def method(self):
        return "A"

class B(A):
    def method(self):
        return "B"

class C(A):
    def method(self):
        return "C"

class D(B, C):
    pass

d = D()
print(d.method())  # "B" (not "C", not "A")

# Why? Check MRO:
print(D.__mro__)
# (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)
```

**How to check MRO:**
```python
print(D.mro())  # Shows method resolution order
```

---

## Category 5: Iteration & Generators

### Trap 12: Generator is Exhausted After Iteration

**What people think:**
You can iterate over a generator multiple times.

**Reality:**
Generators are lazy and can only be iterated once. After that, they're exhausted.

**Code Example:**
```python
def my_generator():
    yield 1
    yield 2
    yield 3

gen = my_generator()

# First iteration:
print(list(gen))  # [1, 2, 3]

# Second iteration:
print(list(gen))  # [] ← Empty! (generator exhausted)

# Proof:
for item in gen:
    print(item)  # Nothing prints
```

**Why this happens:**
Generators maintain state (where they left off). After yielding all values, they're done.

**How to fix:**
```python
# Option 1: Create new generator:
gen1 = my_generator()
print(list(gen1))  # [1, 2, 3]

gen2 = my_generator()
print(list(gen2))  # [1, 2, 3]

# Option 2: Convert to list (uses memory):
data = list(my_generator())
print(data)  # [1, 2, 3]
print(data)  # [1, 2, 3] (can reuse list)

# Option 3: Use itertools.tee:
import itertools
gen1, gen2 = itertools.tee(my_generator(), 2)
print(list(gen1))  # [1, 2, 3]
print(list(gen2))  # [1, 2, 3] (independent copies)
```

---

## Category 6: Performance Traps

### Trap 13: String Concatenation in Loop is O(n²)

**Code Example:**
```python
# SLOW - O(n²):
s = ""
for word in ["apple", "banana", "cherry", ..., 1000 words]:
    s = s + word  # Creates new string each iteration

# FAST - O(n):
s = "".join(["apple", "banana", "cherry", ...])

# Performance difference:
# 1000 words: 0.1 ms vs 100 ms
# 10000 words: 1 ms vs 10 seconds
```

---

### Trap 14: List vs Generator Memory Usage

**Code Example:**
```python
# List: keeps all in memory
data = [x**2 for x in range(10_000_000)]  # Allocates huge array

# Generator: lazy, O(1) memory
data_gen = (x**2 for x in range(10_000_000))  # Minimal memory

# But generator can only be iterated once:
sum(data_gen)  # Works
sum(data_gen)  # 0 (exhausted!)
```

---

## Category 7: Interview Traps

### Trap 15: Off-by-One Errors in Slicing

**Code Example:**
```python
lst = [0, 1, 2, 3, 4]

print(lst[1:4])   # [1, 2, 3] (includes start, excludes end)
print(lst[:3])    # [0, 1, 2]
print(lst[2:])    # [2, 3, 4]
print(lst[-2:])   # [3, 4] (last 2 elements)

# String slicing:
s = "hello"
print(s[1:4])  # "ell"
print(len(s))  # 5
print(s[5])    # IndexError! (valid index: 0-4)
print(s[:len(s)])  # "hello" (safe, doesn't need len(s))
```

---

### Trap 16: None Handling in Comparisons

**Code Example:**
```python
x = None

# CORRECT:
if x is None:
    pass

# WRONG (but works):
if x == None:
    pass

# WRONG (may have surprising behavior):
if not x:  # Also True for 0, "", [], etc.
    pass

# Checking for False:
if x is False:  # Correct (though rare)
    pass

if x == False:  # Works but not idiomatic
    pass

if not x:  # Idiomatic for "falsy"
    pass
```

---

## Category 8: Data Processing Traps

### Trap 17: Modifying Dict While Iterating

**Code Example:**
```python
d = {'a': 1, 'b': 2, 'c': 3}

# WRONG - RuntimeError:
for key in d:
    if d[key] == 2:
        del d[key]  # RuntimeError: dictionary changed size during iteration

# CORRECT - iterate over copy of keys:
for key in list(d.keys()):  # or d.copy()
    if d[key] == 2:
        del d[key]

print(d)  # {'a': 1, 'c': 3}

# Or use comprehension:
d = {k: v for k, v in d.items() if v != 2}
```

---

### Trap 18: List Identity vs Equality in Comparisons

**Code Example:**
```python
lst1 = [1, 2, 3]
lst2 = [1, 2, 3]
lst3 = lst1

print(lst1 == lst2)   # True (equal content)
print(lst1 is lst2)   # False (different objects)
print(lst1 is lst3)   # True (same object)

# Dictionary key lookup uses equality, not identity:
d = {}
d[lst1] = "value"     # TypeError! Lists not hashable

d[tuple(lst1)] = "value"  # ✓ Works (tuples hashable)
```

---

## Quick Reference: How to Avoid Each Trap

| Trap | Solution |
|------|----------|
| Mutable defaults | Use `None` as default, create new inside function |
| List aliasing | Use `.copy()` or `deepcopy()` |
| Shallow copy issues | Use `deepcopy()` for nested structures |
| Modify during iteration | Iterate over copy or use comprehension |
| Closure late binding | Use default argument to capture value |
| Global/nonlocal confusion | Use `global`/`nonlocal` keyword |
| String immutability | Reassign or use `.join()` |
| Integer caching | Always use `==`, not `is` |
| Float precision | Use `math.isclose()` or `Decimal` |
| Generator exhaustion | Create new generator or convert to list |
| String concat in loop | Use `.join()` instead |
| Dict key mutability | Use tuples, frozensets, or strings only |
| Modify dict during iteration | Iterate over `list(d.keys())` |

---

#category/computer-science #topic/python #topic/traps #context/interviews #context/debugging
