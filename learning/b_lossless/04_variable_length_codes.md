# Chapter 4: Variable-Length Codes Introduction

## The Key Insight

Fixed-length codes waste bits on common symbols.

```
ASCII: Every character uses 8 bits
- 'e' (most common): 8 bits
- 'z' (rare): 8 bits

But 'e' appears 180× more often than 'z'!
```

**Solution**: Give common symbols short codes, rare symbols long codes.

---

## Variable-Length Codes

Different symbols get different code lengths.

```
Instead of:
  a = 00000001 (8 bits)
  b = 00000010 (8 bits)
  c = 00000011 (8 bits)

Use:
  a = 0        (1 bit)  — most common
  b = 10       (2 bits) — medium
  c = 11       (3 bits) — rare
```

---

## The Prefix Property

### The Decoding Problem

How do you know where one code ends and the next begins?

```
Stream: 010110...

If codes are:
  a = 0
  b = 01
  c = 10

Is "01" decoded as:
  - "ab" (0, then 1 which is invalid)?
  - "b" (01)?
```

### Prefix Codes (Prefix-Free Codes)

No code is a prefix of another code.

```
Valid prefix code:
  a = 0
  b = 10
  c = 11

"0" can only be 'a' (nothing starts with 0 except a)
"10" can only be 'b' (nothing starts with 10 except b)
"11" can only be 'c'
```

### Decoding Prefix Codes

Read bits until you match a code. Unambiguous!

```
Stream: 01011

Read 0 → matches 'a', output 'a'
Read 1 → no match
Read 0 → "10" matches 'b', output 'b'
Read 1 → no match
Read 1 → "11" matches 'c', output 'c'

Result: "abc"
```

---

## Prefix Codes as Trees

Every prefix code corresponds to a binary tree.

```
Code:
  a = 0
  b = 10
  c = 110
  d = 111

Tree:
        (root)
        /    \
       a      (1)
             /   \
            b    (11)
                /    \
               c      d
```

- Left edge = 0
- Right edge = 1
- Symbols at leaves only
- Code = path from root to leaf

---

## Average Code Length

The efficiency of a variable-length code.

```
L_avg = Σ p(i) × l(i)
```

Where:
- p(i) = probability of symbol i
- l(i) = code length for symbol i

### Example

```
Symbols: a, b, c, d
Probabilities: 0.5, 0.25, 0.125, 0.125

Fixed length (2 bits each):
L_avg = 0.5×2 + 0.25×2 + 0.125×2 + 0.125×2 = 2 bits

Variable length:
  a = 0    (1 bit)
  b = 10   (2 bits)
  c = 110  (3 bits)
  d = 111  (3 bits)

L_avg = 0.5×1 + 0.25×2 + 0.125×3 + 0.125×3 = 1.75 bits
```

Variable-length code is 12.5% more efficient!

---

## Optimal Codes

For a given probability distribution, what's the best code?

### Shannon's Bound

```
H ≤ L_avg < H + 1
```

Where H is entropy. You can get within 1 bit of entropy.

### Optimal Code Properties

For probabilities p₁ ≥ p₂ ≥ ... ≥ pₙ:

1. More probable symbols have shorter or equal codes
2. Two least probable symbols have same length
3. Two least probable symbols differ only in last bit

Huffman coding achieves the optimal prefix code.

---

## Kraft Inequality

A prefix code with lengths l₁, l₂, ..., lₙ exists if and only if:

```
Σ 2^(-lᵢ) ≤ 1
```

### Example: Valid Code

Lengths: 1, 2, 3, 3

```
2^(-1) + 2^(-2) + 2^(-3) + 2^(-3)
= 0.5 + 0.25 + 0.125 + 0.125
= 1.0 ✓
```

A prefix code with these lengths exists.

### Example: Invalid Code

Lengths: 1, 1, 2

```
2^(-1) + 2^(-1) + 2^(-2)
= 0.5 + 0.5 + 0.25
= 1.25 > 1 ✗
```

No prefix code with these lengths can exist.

---

## Types of Variable-Length Codes

### 1. Huffman Codes

Optimal for symbol-by-symbol coding.
- Build tree bottom-up from probabilities
- Covered in next chapter

### 2. Shannon-Fano Codes

Near-optimal, simpler to understand.
- Build tree top-down by splitting probabilities
- Historical importance

### 3. Arithmetic Codes

Encode entire messages, not symbol-by-symbol.
- Can achieve fractional bits per symbol
- Approaches entropy more closely

### 4. Universal Codes

Work without knowing probabilities.
- Elias codes, Golomb codes
- Good for specific distributions

---

## Adaptive vs Static Codes

### Static Codes

Probabilities determined before encoding.
- Need two passes or known statistics
- Same code for entire message
- Simpler decoding

### Adaptive Codes

Probabilities updated during encoding.
- Single pass
- Code changes as you go
- Encoder and decoder must stay synchronized

---

## Code Efficiency

### Redundancy

How far from optimal is the code?

```
Redundancy = L_avg - H
```

| Code Type | Redundancy |
|-----------|------------|
| Huffman | < 1 bit/symbol |
| Arithmetic | < 0.01 bits/symbol |
| Optimal | 0 (theoretical) |

### When Variable-Length Codes Help Most

- Highly skewed distributions (some symbols very common)
- Large alphabets
- When compression matters more than speed

### When Fixed-Length Codes Are Fine

- Uniform distribution
- Need random access
- Speed critical, compression not

---

## Implementation Considerations

### Encoding

```
For each symbol:
    Look up code in table
    Output code bits
```

Fast with lookup table.

### Decoding

Two approaches:

**1. Bit-by-bit tree traversal**
```
Start at root
While not at leaf:
    Read one bit
    Go left (0) or right (1)
Output symbol at leaf
```

**2. Lookup table**
```
Read fixed number of bits
Look up in table
Table entry: symbol, code length
Consume only code_length bits
```

Table lookup is faster but uses more memory.

---

## Key Takeaways

1. Variable-length codes assign short codes to common symbols
2. Prefix codes ensure unambiguous decoding
3. Can be represented as binary trees
4. Kraft inequality determines valid length combinations
5. Optimal codes achieve close to entropy
6. Different codes suit different situations

---

## Coming Up

| Chapter | Topic |
|---------|-------|
| 5 | Shannon-Fano Coding |
| 6 | Huffman Coding |
| 7 | Canonical Huffman |
| 8 | Adaptive Huffman |
| 9 | Arithmetic Coding |
| 10 | Range Coding |
| 11 | ANS |

---

**Next Chapter**: [Shannon-Fano Coding](./05_shannon_fano.md)
