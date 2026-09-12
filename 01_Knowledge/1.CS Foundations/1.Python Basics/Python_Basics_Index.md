# Python Basics — Complete Learning Path & Index

## Overview

This is your **complete reference library for Python fundamentals** — everything from object model to performance optimization. Use this index to navigate based on your role, current knowledge level, or specific question.

---

## Module Structure (8 Comprehensive Files)

| File | Topics | Interview Questions | Interview Critical | Best For |
|------|--------|-------------------|-------------------|----------|
| **1.1 Language Foundations** | Object model, scope, identity | 8 | 🔴 CRITICAL | Understanding Python's core philosophy |
| **1.2 Data Types** | Lists, dicts, sets, tuples | 8 | 🔴 CRITICAL | Choosing right structures, performance |
| **1.3 Functions** | Closures, decorators, lambdas | 8 | 🔴 CRITICAL | First-class functions, functional style |
| **1.4 OOP** | Classes, inheritance, MRO | 8 | 🔴 CRITICAL | Professional code organization |
| **1.5 Functional** | Comprehensions, map/filter | 6 | 🟠 IMPORTANT | Pythonic idioms, generators |
| **1.6 Error Handling** | Exceptions, debugging | 8 | 🟠 IMPORTANT | Robust production code |
| **1.7 Performance** | Big-O, profiling, optimization | 6 | 🟡 HELPFUL | Code efficiency, algorithm choice |
| **1.8 Internals** | GIL, garbage collection, bytecode | 8 | 🟡 HELPFUL | Deep understanding, threading |

---

## Reading Paths by Role

### Data Analyst

```
Priority 1 (MUST READ):
├── 1.1 Language Foundations (identity, aliasing, mutability)
├── 1.2 Data Types (lists, dicts, sets for data manipulation)
└── 1.7 Performance (when to use list vs set vs dict)

Priority 2 (SHOULD READ):
├── 1.3 Functions (working with pandas/numpy functions)
└── 1.6 Error Handling (defensive data pipeline code)

Priority 3 (NICE TO KNOW):
├── 1.5 Functional (comprehensions, generators)
└── 1.8 Internals (understanding memory limits)

Time Investment: ~6-8 hours reading + 4-6 hours practice
```

### ML/AI Engineer

```
Priority 1 (MUST READ):
├── 1.1 Language Foundations (object identity in model state)
├── 1.2 Data Types (data structure for ML pipelines)
├── 1.4 OOP (model classes, inheritance for framework design)
└── 1.7 Performance (vectorization vs loops, Big-O)

Priority 2 (SHOULD READ):
├── 1.3 Functions (decorators for training decorators)
├── 1.5 Functional (generators for data loading)
├── 1.6 Error Handling (training pipeline robustness)
└── 1.8 Internals (memory management for large models)

Time Investment: ~10-12 hours reading + 6-8 hours practice
```

### Backend/Full-stack Developer

```
Priority 1 (MUST READ):
├── 1.1 Language Foundations (object model for web frameworks)
├── 1.2 Data Types (API response building, caching)
├── 1.3 Functions (middleware, decorators)
├── 1.4 OOP (Django/Flask class-based views)
└── 1.6 Error Handling (HTTP error handling)

Priority 2 (SHOULD READ):
├── 1.8 Internals (GIL, threading implications)
├── 1.7 Performance (database query optimization, caching)
└── 1.5 Functional (async/await patterns)

Time Investment: ~10-12 hours reading + 6-8 hours practice
```

---

## Quick Navigation by Topic

### Object Model & Mutability
- **What is Python's object model?** → 1.1 "Everything is an Object"
- **Why is aliasing dangerous?** → 1.1 "Variables Are Labels" + Common Traps #2
- **What's the difference between `=` and `.copy()`?** → 1.1 Q2 + Common Traps #3
- **How do I avoid data corruption?** → Common Traps #2, #3, #10

### Data Structures
- **When should I use a set vs list?** → 1.2 "Sets" + 1.7 "Optimization"
- **Why can't I use a list as dict key?** → 1.2 "Dictionaries" + Interview Traps #9
- **What's the performance difference?** → 1.2 "Time Complexity" + 1.7 "Data Structure Comparison"
- **How do I safely copy nested data?** → 1.1 Q1 + Common Traps #3 + #10

### Functions & Scope
- **What's a closure?** → 1.3 "Closures"
- **What's the late binding problem?** → 1.3 Q6 + Common Traps #5 + Interview Traps #2
- **How do mutable defaults cause bugs?** → Common Traps #1 + Interview Traps #1
- **When should I use decorators?** → 1.3 "Decorators"

### Object-Oriented Programming
- **How does inheritance work?** → 1.4 "Inheritance"
- **What's method resolution order?** → 1.4 "MRO" + Common Traps #11
- **When should I use @classmethod?** → 1.4 "Method Types"
- **How do I make immutable objects?** → 1.4 "Encapsulation & Properties"

### Performance
- **Why is my code slow?** → 1.7 "Profiling" + 1.7 "Optimization Techniques"
- **What's Big-O complexity for this operation?** → 1.7 "Complexity in Python"
- **Should I use a loop or comprehension?** → 1.5 "Comprehensions" vs 1.7 "Micro-optimizations"
- **When should I use generators?** → 1.5 "Generators" + 1.7 "Optimization Techniques"

### Debugging & Errors
- **How do I handle exceptions?** → 1.6 "Exception Hierarchy" + "try-except-finally"
- **What's the difference between ValueError and TypeError?** → 1.6 "Exception Hierarchy"
- **How do I debug this code?** → 1.6 "Debugging Techniques"
- **Should I use custom exceptions?** → 1.6 Q2

### Internals & Advanced
- **Why is multithreading slow?** → 1.8 "GIL"
- **How does garbage collection work?** → 1.8 "Garbage Collection"
- **Why are small integers weird?** → 1.8 "Memory Management" + Interview Traps #5
- **How does `is` work?** → 1.1 "Identity vs Equality" + 1.8 "How `is` Works"

---

## Interview Preparation Strategy

### Baseline Interview (1 day prep)

Read these in order:
1. Common_Traps_and_Misunderstandings_Curated_For_Interviews.md (30 min)
2. 1.1 Language Foundations (60 min)
3. 1.2 Data Types (60 min)
4. 1.4 OOP (45 min)

**Result:** Can answer 80% of basic interviews

### Strong Interview (1 week prep)

Add these:
1. 1.3 Functions (90 min)
2. 1.5 Functional (75 min)
3. 1.7 Performance (60 min)
4. Common_Traps_and_Misunderstandings_Comprehensive.md (90 min)

**Result:** Can answer 95% of intermediate interviews

### Expert Interview (2 week prep)

Add these:
1. 1.6 Error Handling (75 min)
2. 1.8 Internals (90 min)
3. Practice problems and code reviews (3+ hours)

**Result:** Can handle advanced system design + tricky coding interviews

---

## Common Interview Questions by Category

### Object Model (1.1)
- [ ] Explain Python's object model
- [ ] What's the difference between `a = b` and `a = b.copy()`?
- [ ] Why can't you use a list as dict key?
- [ ] Explain identity vs equality (`is` vs `==`)

### Data Types (1.2)
- [ ] Time complexity of list vs dict vs set operations?
- [ ] When to use each data structure?
- [ ] What's shallow copy vs deep copy?
- [ ] How do strings differ from lists?

### Functions (1.3)
- [ ] What's a mutable default argument bug?
- [ ] Explain closures and late binding
- [ ] What are decorators and how do they work?
- [ ] When to use `*args` vs `**kwargs`?

### OOP (1.4)
- [ ] Explain inheritance and super()
- [ ] What's method resolution order?
- [ ] Difference between instance and class attributes?
- [ ] When to use @classmethod vs @staticmethod?

### Performance (1.7)
- [ ] Big-O complexity of common operations?
- [ ] How to optimize slow code?
- [ ] When to use generators vs lists?
- [ ] Profile and optimize this code

### Traps (All Common Traps files)
- [ ] Find the bug in this code
- [ ] Why does this behave unexpectedly?
- [ ] How would you fix this?

---

## Practice Problems

### Level 1: Warm-up
```python
# Problem 1: Find the bug
def get_last_digit(n):
    while n >= 10:
        n = n / 10  # Bug: should be n // 10
    return n

# Problem 2: What does this print?
x = [1, 2, 3]
y = x
y.append(4)
print(x, y)  # [1, 2, 3, 4], [1, 2, 3, 4]

# Problem 3: Fix the bug
def create_list(item, items=[]):
    items.append(item)
    return items
```

### Level 2: Intermediate
```python
# Problem 4: Optimize for performance
def has_duplicate(arr):
    for i in range(len(arr)):
        for j in range(i+1, len(arr)):
            if arr[i] == arr[j]:
                return True
    return False

# Better: Use set for O(n) instead of O(n²)

# Problem 5: Explain the output
def make_adders():
    adders = []
    for i in range(3):
        adders.append(lambda x: x + i)
    return adders

# Problem 6: Design a thread-safe counter
```

### Level 3: Advanced
```python
# Problem 7: System design
# Design a cache with automatic expiration

# Problem 8: OOP design
# Implement a model class with proper inheritance

# Problem 9: Performance analysis
# Profile and optimize a data pipeline
```

---

## Key Takeaways by File

**1.1 Language Foundations:**
- Everything is an object with identity, type, and value
- Variables are labels, not boxes; multiple labels can point to same object
- Scope: LEGB (Local, Enclosing, Global, Built-in)
- Understanding this prevents 80% of Python bugs

**1.2 Data Types:**
- Choose right structure for right job (list O(n) search, set O(1))
- Mutability matters (can't use list as dict key, but tuple is fine)
- Immutables are thread-safe; mutables need locks
- Aliasing vs copying causes silent data corruption

**1.3 Functions:**
- First-class objects: pass as arguments, return, store
- Decorators elegantly modify function behavior
- Closures capture enclosing scope by reference (late binding)
- Mutable defaults are a gotcha (evaluated once)

**1.4 OOP:**
- Inheritance: code reuse, polymorphism
- MRO determines method search order
- Class vs instance attributes: one shared, one separate
- Dunder methods enable operator overloading

**1.5 Functional:**
- Comprehensions (list/dict/set) are Pythonic and fast
- Generators are lazy and memory-efficient
- map/filter/reduce vs comprehensions (latter preferred)
- Higher-order functions enable elegant abstractions

**1.6 Error Handling:**
- Exception hierarchy: catch specific exceptions first
- Context managers (`with` statement) ensure cleanup
- Custom exceptions make code clearer
- Logging for production, print for debugging

**1.7 Performance:**
- Profile first, optimize after (measure not guess)
- Big-O complexity often matters more than micro-optimizations
- Choose right data structure, then algorithm, then optimize
- Generators win on memory; lists win on clarity

**1.8 Internals:**
- GIL prevents multithreading for CPU-bound code
- Reference counting + cycle detection for garbage collection
- Hash tables for O(1) dict/set lookups
- Small integers cached (-5 to 256)

---

## Use as Reference Library

**Bookmark this index.** When you encounter a question or bug:

1. Search for keywords in this index
2. Jump to recommended file/section
3. Find exactly what you need
4. Reference the comprehensive explanation

This is your **"go directionless" solution** — whenever you need to brush up, you know exactly where to look.

---

## Download & Organize

All files should be organized as:

```
Python_Basics/
├── Python_Basics_Index.md (this file)
├── 1.1_Python_Language_Foundations.md
├── 1.2_Data_Types_and_Structures.md
├── 1.3_Functions_and_Scope.md
├── 1.4_Object_Oriented_Programming.md
├── 1.5_Functional_Programming.md
├── 1.6_Error_Handling_and_Debugging.md
├── 1.7_Performance_and_Optimization.md
├── 1.8_Python_Internals.md
├── Common_Traps_and_Misunderstandings_Comprehensive.md
└── Common_Traps_and_Misunderstandings_Curated_For_Interviews.md
```

---

## Next Steps

After Python Basics, continue with:

1. **[[2.Memory_Model_and_Data_Representation]]** — How Python stores data in memory
2. **[[3.Operating_Systems_Basics]]** — Processes, threads, file systems (for system design)
3. **[[2.DSA]]** — Data structures & algorithms for technical interviews
4. **[[8.Systems_Cloud_System_Design]]** — System design patterns

---

#category/computer-science #topic/python #topic/learning-path #context/interviews #context/job-search
