# Chapter 5: Shannon-Fano Coding

## Overview

**Shannon-Fano coding** is one of the first entropy coding methods, developed by Claude Shannon and Robert Fano independently in 1948-1949.

It builds a variable-length prefix code by recursively dividing symbols into groups of roughly equal probability.

---

## The Algorithm

### Top-Down Approach

1. List symbols in order of decreasing probability
2. Divide list into two parts with nearly equal total probability
3. Assign '0' to first part, '1' to second part
4. Recursively apply to each part until single symbols remain

---

## Step-by-Step Example

### Given Symbols and Probabilities

| Symbol | Probability |
|--------|-------------|
| A | 0.35 |
| B | 0.20 |
| C | 0.18 |
| D | 0.12 |
| E | 0.10 |
| F | 0.05 |

### Step 1: Sort by Probability

Already sorted (descending).

### Step 2: First Split

Find split point where probabilities are most balanced:

```
Option 1: {A} vs {B,C,D,E,F}
  Left:  0.35
  Right: 0.65
  Difference: 0.30

Option 2: {A,B} vs {C,D,E,F}
  Left:  0.55
  Right: 0.45
  Difference: 0.10  ← Best split

Option 3: {A,B,C} vs {D,E,F}
  Left:  0.73
  Right: 0.27
  Difference: 0.46
```

**Best split**: {A,B} with '0', {C,D,E,F} with '1'

### Step 3: Recursive Splits

**Left subtree {A,B}**:
- A = 0.35
- B = 0.20
- Split: {A} vs {B}
- A gets '0', B gets '1'

**Right subtree {C,D,E,F}**:
```
{C,D} vs {E,F}:
  Left:  0.30
  Right: 0.15

{C} vs {D,E,F}:
  Left:  0.18
  Right: 0.27
  Difference: 0.09  ← Better!
```

**Wait, reconsider**:
```
{C,D} vs {E,F}:
  Difference: 0.15

{C} vs {D,E,F}:
  Difference: 0.09  ← Best
```

Actually:
```
C = 0.18
D + E + F = 0.27
Difference = 0.09

C,D = 0.30
E,F = 0.15
Difference = 0.15
```

Best: {C} gets '0', {D,E,F} gets '1'

**Continue for {D,E,F}**:
```
{D} vs {E,F}:
  Left:  0.12
  Right: 0.15
  Difference: 0.03  ← Use this

{D,E} vs {F}:
  Left:  0.22
  Right: 0.05
  Difference: 0.17
```

D gets '0', {E,F} gets '1'

**Finally {E,F}**:
- E gets '0', F gets '1'

### Final Codes

| Symbol | Probability | Code | Length |
|--------|-------------|------|--------|
| A | 0.35 | 00 | 2 |
| B | 0.20 | 01 | 2 |
| C | 0.18 | 10 | 2 |
| D | 0.12 | 110 | 3 |
| E | 0.10 | 1110 | 4 |
| F | 0.05 | 1111 | 4 |

### Tree Representation

```
            (root)
           /      \
         (0)      (1)
        /   \    /   \
       A     B  C    (11)
                    /    \
                   D     (111)
                        /    \
                       E      F
```

---

## Calculating Efficiency

### Average Code Length

```
L_avg = Σ p(i) × l(i)

L_avg = 0.35×2 + 0.20×2 + 0.18×2 + 0.12×3 + 0.10×4 + 0.05×4
      = 0.70 + 0.40 + 0.36 + 0.36 + 0.40 + 0.20
      = 2.42 bits/symbol
```

### Entropy

```
H = -Σ p(i) × log₂(p(i))

H = -[0.35×log₂(0.35) + 0.20×log₂(0.20) + 0.18×log₂(0.18)
     + 0.12×log₂(0.12) + 0.10×log₂(0.10) + 0.05×log₂(0.05)]

H ≈ 2.37 bits/symbol
```

### Redundancy

```
Redundancy = L_avg - H = 2.42 - 2.37 = 0.05 bits/symbol
```

Very close to optimal!

---

## Pseudocode

```
function shannon_fano(symbols, probabilities):
    // Sort by probability descending
    sorted = sort_by_probability(symbols, probabilities)

    // Build codes recursively
    codes = {}
    build_codes(sorted, "", codes)
    return codes

function build_codes(symbols, prefix, codes):
    if length(symbols) == 1:
        codes[symbols[0]] = prefix
        return

    if length(symbols) == 2:
        codes[symbols[0]] = prefix + "0"
        codes[symbols[1]] = prefix + "1"
        return

    // Find best split point
    split = find_best_split(symbols)

    // Recurse on each half
    build_codes(symbols[0:split], prefix + "0", codes)
    build_codes(symbols[split:], prefix + "1", codes)

function find_best_split(symbols):
    total = sum of all probabilities
    running_sum = 0
    best_split = 1
    best_diff = infinity

    for i = 1 to length(symbols) - 1:
        running_sum += probability(symbols[i-1])
        left_sum = running_sum
        right_sum = total - running_sum
        diff = abs(left_sum - right_sum)

        if diff < best_diff:
            best_diff = diff
            best_split = i

    return best_split
```

---

## Shannon-Fano vs Huffman

### Shannon-Fano

- Top-down construction
- Split by probability sums
- Not always optimal
- Simpler to understand

### Huffman

- Bottom-up construction
- Combine smallest probabilities
- Always optimal for symbol-by-symbol
- Slightly more complex

### Example of Non-Optimality

| Symbol | Prob | Shannon-Fano | Huffman |
|--------|------|--------------|---------|
| A | 0.4 | 00 | 0 |
| B | 0.2 | 01 | 10 |
| C | 0.2 | 10 | 110 |
| D | 0.1 | 110 | 1110 |
| E | 0.1 | 111 | 1111 |

Shannon-Fano average: 2.2 bits
Huffman average: 2.1 bits

Huffman wins!

---

## Historical Significance

### Shannon's Contribution

- Proved entropy is the limit
- Showed optimal codes exist
- Described method but didn't optimize

### Fano's Contribution

- Independent discovery
- Practical implementation
- Assigned to Huffman as homework (leading to Huffman coding!)

### Legacy

- First practical entropy coding
- Led directly to Huffman's improvement
- Foundation for understanding compression

---

## When to Use Shannon-Fano

### Advantages

- Simple to implement and understand
- Fast encoding
- Good educational tool

### Disadvantages

- Not always optimal
- Huffman is equally simple and always optimal
- Superseded by Huffman in practice

### In Practice

Shannon-Fano is rarely used today. Huffman coding (next chapter) is preferred because:
1. It's always optimal
2. It's equally simple to implement
3. Tools and libraries use Huffman

---

## Key Takeaways

1. Shannon-Fano uses top-down recursive splitting
2. Splits symbols to balance probability sums
3. Produces valid prefix codes
4. Near-optimal but not always optimal
5. Historical importance: led to Huffman coding
6. Good for learning, Huffman for practice

---

**Practice**:

1. Build Shannon-Fano codes for: A(0.5), B(0.25), C(0.125), D(0.125)
2. Compare your result to the optimal code
3. Find an example where Shannon-Fano is suboptimal
4. Implement the algorithm

---

**Next Chapter**: [Huffman Coding](./06_huffman.md)
