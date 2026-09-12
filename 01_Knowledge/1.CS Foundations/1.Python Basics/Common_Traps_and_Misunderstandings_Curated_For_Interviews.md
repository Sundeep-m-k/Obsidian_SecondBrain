# Python: Common Traps and Misunderstandings — Curated for Interviews

This is a shorter, interview-focused list of the 15-20 most common Python gotchas that appear in interviews.

---

## Trap 1: Mutable Default Arguments

**The Interview Question:**
"This code has a bug. Find it and explain why."

```python
def add_user(name, users=[]):
    users.append(name)
    return users

add_user("Alice")   # ['Alice']
add_user("Bob")     # ['Alice', 'Bob'] ← Why isn't it just ['Bob']?
add_user("Charlie") # ['Alice', 'Bob', 'Charlie']
```

**Explanation:**
Default arguments are evaluated ONCE at function definition time, not each call. The same list is reused.

**How to fix:**
```python
def add_user(name, users=None):
    if users is None:
        users = []
    users.append(name)
    return users
```

---

## Trap 2: Late Binding in Closures

**The Interview Question:**
"What does this print?"

```python
functions = []
for i in range(3):
    functions.append(lambda: i)

print([f() for f in functions])  # [0, 1, 2] or [2, 2, 2]?
```

**Answer:** `[2, 2, 2]`

**Explanation:**
Closures capture variables by reference. All lambdas reference the same `i`, which is 2 after loop.

**How to fix:**
```python
functions = [lambda i=i: i for i in range(3)]
print([f() for f in functions])  # [0, 1, 2]
```

---

## Trap 3: List Aliasing

**The Interview Question:**
"What's wrong with this backup strategy?"

```python
original = [1, 2, 3]
backup = original

original.append(4)
print(backup)  # [1, 2, 3, 4] ← Backup also changed!
```

**Explanation:**
`backup = original` creates an alias (both point to same list), not a copy.

**How to fix:**
```python
backup = original.copy()  # or original[:]
```

---

## Trap 4: String Immutability

**The Interview Question:**
"Why doesn't this work?"

```python
s = "hello"
s[0] = 'H'  # TypeError!
```

**Explanation:**
Strings are immutable. You must create a new string.

**How to fix:**
```python
s = 'H' + s[1:]
```

---

## Trap 5: Integer Caching Anomaly

**The Interview Question:**
"What does this print and why is it weird?"

```python
print(256 is 256)   # True
print(257 is 257)   # False (weird!)
print(256 == 257)   # False
```

**Explanation:**
Python caches small integers (-5 to 256) for performance. Beyond that, identity is unreliable.

**Interview Lesson:**
Always use `==` for values, `is` only for singletons (None).

---

## Trap 6: Scope Confusion with Global

**The Interview Question:**
"Fix this bug."

```python
x = 10

def increment():
    x = x + 1  # UnboundLocalError!
    return x

increment()
```

**Explanation:**
Python sees `x = ...` and assumes `x` is local. But we're reading `x` before assigning.

**How to fix:**
```python
def increment():
    global x
    x = x + 1
    return x
```

---

## Trap 7: Float Precision

**The Interview Question:**
"Why doesn't equality work?"

```python
print(0.1 + 0.2 == 0.3)  # False!
print(0.1 + 0.2)  # 0.30000000000000004
```

**Explanation:**
Binary floating-point can't represent 0.1 exactly. Errors accumulate.

**How to fix:**
```python
import math
math.isclose(0.1 + 0.2, 0.3)  # True
```

---

## Trap 8: Tuple with Single Element

**The Interview Question:**
"What are these types?"

```python
a = (1)    # <class 'int'>
b = (1,)   # <class 'tuple'>
c = ()     # <class 'tuple'>
```

**Explanation:**
The comma makes a tuple. `(1)` is just 1 in parentheses.

**Interview Lesson:**
Single-element tuples need a trailing comma.

---

## Trap 9: Dictionary Key Mutability

**The Interview Question:**
"Why doesn't this work?"

```python
d = {[1, 2]: "value"}  # TypeError!
d = {(1, 2): "value"}  # Works!
```

**Explanation:**
Dict keys must be hashable (immutable). Lists are mutable, tuples aren't.

**Why it matters:**
```python
lst = [1, 2]
key = lst
# If list mutates, hash changes, breaking dict lookup
```

---

## Trap 10: Shallow Copy with Nested Structures

**The Interview Question:**
"What does this print?"

```python
original = [1, [2, 3], 4]
backup = original.copy()

backup[1].append(999)
print(original)  # [1, [2, 3, 999], 4] ← Original affected!
```

**Explanation:**
Shallow copy creates new list but inner objects are shared.

**How to fix:**
```python
import copy
backup = copy.deepcopy(original)
```

---

## Trap 11: Modifying List During Iteration

**The Interview Question:**
"Why does this skip elements?"

```python
lst = [1, 2, 3, 4, 5]
for item in lst:
    if item % 2 == 0:
        lst.remove(item)

print(lst)  # [1, 3, 5] (sometimes works by luck)
```

**Explanation:**
Modifying during iteration makes iterator position invalid.

**How to fix:**
```python
lst = [x for x in lst if x % 2 != 0]
```

---

## Trap 12: None in Comparisons

**The Interview Question:**
"Which is correct?"

```python
x = None

if x is None:      # ✓ Correct
    pass

if x == None:      # Works but not idiomatic
    pass

if not x:          # Not the same (also True for 0, "", [])
    pass
```

**Interview Lesson:**
Always use `is None`, not `== None`.

---

## Trap 13: Mutable Class Attributes

**The Interview Question:**
"Why is all classes sharing the same list?"

```python
class Dog:
    tricks = []  # Mutable class attribute
    
    def __init__(self, name):
        self.name = name
    
    def add_trick(self, trick):
        self.tricks.append(trick)

d1 = Dog("Rex")
d2 = Dog("Buddy")

d1.add_trick("sit")
print(d2.tricks)  # ['sit'] ← Shared!
```

**Explanation:**
Class attributes are shared. Instance attributes are separate.

**How to fix:**
```python
class Dog:
    def __init__(self, name):
        self.name = name
        self.tricks = []  # Instance attribute
```

---

## Trap 14: Generator Exhaustion

**The Interview Question:**
"Why is the second list empty?"

```python
gen = (x**2 for x in range(5))

print(list(gen))  # [0, 1, 4, 9, 16]
print(list(gen))  # [] ← Empty!
```

**Explanation:**
Generators are lazy and can only be iterated once.

**How to fix:**
```python
gen1 = (x**2 for x in range(5))
gen2 = (x**2 for x in range(5))
```

---

## Trap 15: `is` for Identity vs `==` for Equality

**The Interview Question:**
"What's the difference?"

```python
a = [1, 2, 3]
b = [1, 2, 3]

print(a == b)   # True (equal content)
print(a is b)   # False (different objects)
```

**Explanation:**
- `==` checks if values are the same
- `is` checks if it's literally the same object in memory

**When it matters:**
```python
if x is None:   # Correct (None is singleton)
    pass

if x == None:   # Works but less efficient
    pass
```

---

## Trap 16: Dict Key Changing After Insertion

**The Interview Question:**
"Why can't we use lists/dicts as keys?"

```python
key = [1, 2]
d = {key: "value"}
key.append(3)  # Modified the key!

print(d)  # Dict is now broken (key's hash changed)
# Trying to access might give KeyError or wrong value
```

**Lesson:**
Only immutable objects (tuple, string, frozenset) can be dict keys.

---

## Trap 17: String Concatenation in Loops is O(n²)

**The Interview Question:**
"Optimize this code."

```python
# SLOW:
s = ""
for word in words:
    s = s + word  # Creates new string each time

# FAST:
s = "".join(words)
```

**Why:** Each `+` creates a new string (old strings are immutable).

---

## Trap 18: Set vs List Performance for Membership

**The Interview Question:**
"Which is faster for 1M items?"

```python
# SLOW - O(n):
if item in big_list:
    pass

# FAST - O(1):
if item in big_set:
    pass
```

**Impact:** With 1M items, 1000x faster using set.

---

## Quick Reference: Fix Each Trap

| Trap | Solution |
|------|----------|
| Mutable defaults | Use `None` as default, create inside function |
| Late binding | Capture with default arg: `lambda i=i: ...` |
| List aliasing | Use `.copy()` or `deepcopy()` |
| String immutable | Reassign or use `.join()` |
| Integer caching | Use `==` not `is` |
| Global/nonlocal | Use `global`/`nonlocal` keywords |
| Float precision | Use `math.isclose()` or `Decimal` |
| Tuple syntax | Add trailing comma: `(1,)` |
| Dict keys | Only use hashable types (tuple, str, frozenset) |
| Shallow copy | Use `deepcopy()` for nested structures |
| Modify during iteration | Iterate over copy or use comprehension |
| None checks | Always use `is None` |
| Mutable class attrs | Move to `__init__` as instance attrs |
| Generator exhaustion | Create new generator or convert to list |
| `is` vs `==` | Use `==` for values, `is` for singletons |

---

#category/computer-science #topic/python #topic/traps #context/interviews
