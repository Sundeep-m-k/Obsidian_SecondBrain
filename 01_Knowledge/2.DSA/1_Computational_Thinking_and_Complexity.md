# Computational Thinking & Complexity

## What is it?

**Computational Thinking** is the ability to break down complex problems into smaller, manageable steps and express solutions in a way a computer (or another human) can understand and execute.

**Complexity Analysis** is the formal study of how the **time** and **space** requirements of an algorithm grow as the input size increases. It answers: "How fast will this algorithm run on a million items instead of a thousand?"

> Without complexity analysis, you optimize blindly. With it, you know which bottlenecks matter.

---

## Why This Matters (Job Interview Context)

In technical interviews:
- You'll be asked: "What's the time complexity of your solution?"
- You need to recognize when your $O(n^2)$ brute force is wrong and optimize to $O(n \log n)$
- You need to choose between time and space tradeoffs consciously

In production systems:
- A $O(n)$ algorithm on 1 billion items runs in seconds
- An $O(n^2)$ algorithm on 1 billion items may run for days

---

## Big-O Notation

**Big-O** describes the **worst-case** upper bound on how an algorithm's runtime or memory grows with input size $n$.

Formal definition: $f(n) \in O(g(n))$ if there exist constants $c > 0$ and $n_0 > 0$ such that:
$$f(n) \leq c \cdot g(n) \quad \forall n \geq n_0$$

In plain English: for large enough inputs, $f(n)$ grows no faster than $g(n)$ (up to a constant factor).

---

## Common Complexities (From Fastest to Slowest)

| Notation | Name | Intuition | Example |
|----------|------|-----------|---------|
| $O(1)$ | **Constant** | Independent of input size | Hash table lookup, array index access |
| $O(\log n)$ | **Logarithmic** | Divide in half each iteration | Binary search, balanced BST lookup |
| $O(n)$ | **Linear** | Iterate once through input | Single loop, linear scan |
| $O(n \log n)$ | **Linearithmic** | Divide + conquer; or sort-then-process | Merge sort, quick sort (average) |
| $O(n^2)$ | **Quadratic** | Nested loops | Bubble sort, naive string matching |
| $O(n^3)$ | **Cubic** | Triple nested loops | Naive matrix multiplication |
| $O(2^n)$ | **Exponential** | Doubles with each added input | Brute-force subset enumeration |
| $O(n!)$ | **Factorial** | Explodes for even small $n$ | Brute-force permutation enumeration |

**Rule of thumb for $n = 10^6$:**
- $O(\log n)$ → ~20 operations
- $O(n)$ → $10^6$ operations (instant)
- $O(n \log n)$ → $2 \times 10^7$ operations (~0.02 sec)
- $O(n^2)$ → $10^{12}$ operations (16+ minutes)
- $O(2^n)$ → Literally impossible in the universe's lifetime

---

## How to Analyze Complexity

### Method 1: Count Operations

```python
def example(arr):
    x = 5                   # 1 operation
    for i in range(len(arr)):  # n iterations
        x += arr[i]         # 1 operation per iteration
    return x                # 1 operation
```

**Time Complexity:** $1 + n \cdot 1 + 1 = n + 2 = O(n)$ (drop constants)

### Method 2: Identify the Loop Structure

| Loop Structure | Complexity |
|---|---|
| Single loop from 1 to n | $O(n)$ |
| Nested loops (both n times) | $O(n^2)$ |
| Nested loops (one halves each time) | $O(n \log n)$ |
| Two sequential loops of size n each | $O(n)$ (not $O(2n)$; constants drop) |
| Loop that halves input each iteration | $O(\log n)$ |

### Method 3: Recursion — Master Theorem

For a recurrence of the form:
$$T(n) = a \cdot T(n/b) + O(n^d)$$

Where:
- $a$ = number of recursive subproblems
- $b$ = factor by which input shrinks
- $d$ = exponent of non-recursive work

**Master Theorem:**
- If $a > b^d$: $T(n) = O(n^{\log_b a})$ (recursion dominates)
- If $a = b^d$: $T(n) = O(n^d \log n)$ (balanced)
- If $a < b^d$: $T(n) = O(n^d)$ (base case dominates)

**Examples:**
- Merge sort: $T(n) = 2T(n/2) + O(n)$ → $a=2, b=2, d=1$ → $a = b^d$ → $O(n \log n)$
- Binary search: $T(n) = 1 \cdot T(n/2) + O(1)$ → $a=1, b=2, d=0$ → $a < b^d$ → $O(1)$

---

## Space Complexity

**Space Complexity** = memory used by an algorithm as a function of input size.

| Algorithm | Space | Notes |
|-----------|-------|-------|
| In-place sorting (e.g., quick sort) | $O(1)$ | Only a few variables; uses recursion stack $O(\log n)$ |
| Merge sort | $O(n)$ | Creates temporary arrays for merging |
| Hash table with n items | $O(n)$ | Stores n key-value pairs |
| Recursion depth | $O(h)$ | $h$ = height of recursion tree |

**Time-Space Tradeoff:** Often you can trade time for space:
- **Faster, more space:** Memoize (cache) intermediate results
- **Slower, less space:** Recompute as needed (don't cache)

---

## Best, Average, Worst Case

**Worst Case** ($O$ notation): Performance on the hardest possible input
- Most common for interviews; gives upper bound guarantee
- Example: Linear search on an array where target is at the end → $O(n)$

**Average Case** (rarely asked): Performance on a "typical" input
- Requires assuming a probability distribution
- Example: Linear search on average → $O(n/2) = O(n)$ (same $O$ class, different constant)

**Best Case** ($\Omega$ notation): Performance on the easiest possible input
- Rarely useful; mostly ignored in interviews
- Example: Linear search when target is first element → $\Omega(1)$

**In interviews, assume "complexity" means worst case unless stated otherwise.**

---

## Complexity Classes Cheat Sheet

### Reduce Problem to Known Complexity

When you see a problem, ask:
- **$O(1)$** — Direct lookup or fixed operation
- **$O(\log n)$** — Eliminate half the search space each step (binary search)
- **$O(n)$** — Single pass through input
- **$O(n \log n)$** — Divide & conquer or sort-then-process
- **$O(n^2)$** — Nested iteration or all pairs
- **$O(2^n)$** — All subsets; only feasible for $n \leq 20$
- **$O(n!)$** — All permutations; almost never feasible

If your solution is **slower than needed**, recognize which technique could optimize it:
- Too slow? Maybe you need **binary search**, **sorting**, **hashing**, or **memoization**

---

## Practical Optimization Techniques

| Problem | Optimization | Complexity Improvement |
|---------|--------------|------------------------|
| Finding duplicates | Use hash set instead of nested loop | $O(n^2) \to O(n)$ |
| Searching sorted data | Binary search instead of linear | $O(n) \to O(\log n)$ |
| Repeated subproblems | Memoization (cache results) | $O(2^n) \to O(n)$ (for Fibonacci) |
| Multiple passes needed | Two pointers or sliding window | $O(n^2) \to O(n)$ |

---

## Interview Tips

1. **State your complexity before coding.** "This is $O(n \log n)$ because we sort first..."
2. **Defend your complexity.** Walk through loop structure.
3. **Optimize if possible.** "Can we do better than $O(n^2)$?" → Yes, maybe $O(n \log n)$ with sorting
4. **Test on small inputs.** Make sure $O(n)$ isn't hiding an inner loop you missed.
5. **Discuss tradeoffs.** "Space-efficient solution uses $O(1)$ memory but $O(n^2)$ time. Better solution uses $O(n)$ space for $O(n)$ time."

---

## Related Notes

- [[2.DSA]] — Algorithms whose complexity you'll analyze
- [[5.Mathematics]] — Discrete math behind Big-O proofs
- [[11.Professional Skills]] — How to explain complexity in interviews

#category/computer-science #topic/complexity #math/asymptotics
