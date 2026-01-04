# Chapter 15: Other Integer Codes

## Overview

Beyond Elias and Golomb, several other integer codes exist for specific use cases.

---

## Unary Code

The simplest code.

### Encoding

Represent n as n ones followed by a zero:

```
n=0: 0
n=1: 10
n=2: 110
n=3: 1110
n=4: 11110
```

### Properties

- Length: n + 1 bits
- Optimal for P(n) = 1/2^(n+1) (geometric with p=0.5)
- Very inefficient for large numbers
- Building block for other codes

---

## Fibonacci Coding

Uses Fibonacci representation.

### Zeckendorf's Theorem

Every positive integer has a unique representation as sum of non-consecutive Fibonacci numbers.

```
Fibonacci: 1, 2, 3, 5, 8, 13, 21, ...

n=1:  1           = F(2)         → 11
n=2:  2           = F(3)         → 011
n=3:  3           = F(4)         → 0011
n=4:  3+1         = F(4)+F(2)    → 1011
n=5:  5           = F(5)         → 00011
n=6:  5+1         = F(5)+F(2)    → 10011
n=7:  5+2         = F(5)+F(3)    → 01011
n=8:  8           = F(6)         → 000011
n=13: 13          = F(7)         → 0000011
```

### Encoding

1. Find Fibonacci representation of n
2. Write bits from least significant Fibonacci
3. Append 1 (making "11" as terminator)

The "11" terminator is unique because Zeckendorf uses non-consecutive Fibonaccis.

### Properties

- Self-delimiting (ends with "11")
- Error resilient (can resync after errors)
- Length ≈ log_φ(n) × 2 where φ = golden ratio
- About 4% worse than Elias Delta

---

## Levenshtein Coding

Universal code similar to Elias Omega.

### Encoding

```
1. If n = 0, output 0
2. Write binary of n, prepend count of digits
3. Recursively encode the count
4. Prepend 1 as start marker
```

### Example

```
n=9:
  Binary: 1001 (4 digits)
  Encode 4: 100 (3 digits)
  Encode 3: 11 (2 digits)
  Encode 2: 10 (2 digits)
  Encode 2: stop, start with 1

Result: 1 10 11 100 1001
      = 1_10_11_100_1001 = 11011100 1001
```

### Properties

- Very efficient for large numbers
- Complex implementation
- Rarely used in practice

---

## Start-Step-Stop Codes

Generalization of Golomb/Rice.

### Parameters

- Start (s): initial bits to read
- Step (t): how many bits to add each level
- Stop (f): maximum bits

### Encoding

```
For value n:
  Level 0: Use s bits if n < 2^s
  Level 1: Prefix 0, use s+t bits if n < 2^(s+t)
  Level 2: Prefix 00, use s+2t bits if n < 2^(s+2t)
  ...
  Level f: Prefix 0..0, use s+f×t bits
```

### Common Configurations

| Name | Start | Step | Stop |
|------|-------|------|------|
| Gamma | 0 | 1 | ∞ |
| VarInt | 7 | 7 | 8 |
| UTF-8 | 7 | 5/6 | 4 |

---

## VarInt (Variable-Length Integer)

Byte-aligned integer encoding.

### Encoding

Use 7 bits per byte for data, high bit as continuation flag:

```
0xxxxxxx                    (0-127)
1xxxxxxx 0xxxxxxx           (128-16383)
1xxxxxxx 1xxxxxxx 0xxxxxxx  (16384-2097151)
...
```

### Properties

- Byte-aligned (fast)
- Self-delimiting
- Used in Protocol Buffers, SQLite, Git

### LEB128

Little-Endian Base 128 — VarInt with little-endian byte order:

```
300 = 0b100101100
    = 0b0000010 0101100
    = 10101100 00000010 (LEB128)
      ^         ^
      cont      stop
```

---

## Group VarInt

Pack multiple VarInts together.

### Simple-8b

Pack integers in 8-byte blocks:

```
4 bits: selector (0-15)
60 bits: packed integers

Selector determines:
- How many integers
- Bits per integer
```

| Selector | Integers | Bits each |
|----------|----------|-----------|
| 0 | 240 | 0 (all zeros) |
| 1 | 60 | 1 |
| 2 | 30 | 2 |
| 3 | 20 | 3 |
| 4 | 15 | 4 |
| 5 | 12 | 5 |
| 6 | 10 | 6 |
| ... | ... | ... |

### StreamVByte

- Group 4 integers
- 2-bit length tags
- SIMD-friendly decode

---

## Comparison Table

| Code | Best For | Length(n) | Complexity |
|------|----------|-----------|------------|
| Unary | p=0.5 geometric | n+1 | Trivial |
| Golomb | Geometric dist | ~log(n) | Simple |
| Gamma | Small integers | 2log(n) | Simple |
| Delta | Medium integers | log(n)+2log(log(n)) | Moderate |
| Fibonacci | Error resilience | ~1.44log(n) | Moderate |
| VarInt | Byte-aligned | 8×⌈log(n)/7⌉ | Simple |

---

## Choosing a Code

### For Random Access

Use fixed-width or VarInt (byte-aligned).

### For Maximum Compression

Use Golomb/Rice with optimal parameter.

### For Unknown Distribution

Use Elias Delta (universal).

### For Error Resilience

Use Fibonacci coding.

### For Speed

Use VarInt or Group VarInt.

---

## Pseudocode: Fibonacci Encoder

```
function fibonacci_encode(n):
    // Find Fibonacci representation
    fibs = [1, 2, 3, 5, 8, 13, 21, ...]  // Generate as needed
    bits = []

    i = find_largest_fib_index(n)
    while n > 0:
        if fibs[i] <= n:
            bits.prepend(1)
            n = n - fibs[i]
        else:
            bits.prepend(0)
        i = i - 1

    // Add terminating 1
    bits.append(1)

    return bits
```

```
function fibonacci_decode():
    fibs = [1, 2, 3, 5, 8, 13, 21, ...]
    n = 0
    i = 0
    prev_bit = 0

    while true:
        bit = read_bit()
        if bit == 1:
            n = n + fibs[i]
            if prev_bit == 1:
                break  // Found "11" terminator
        prev_bit = bit
        i = i + 1

    return n
```

---

## Key Takeaways

1. Many integer codes exist for different use cases
2. Unary: simplest, good for p=0.5 geometric
3. Fibonacci: error resilient, self-synchronizing
4. VarInt: byte-aligned, fast, widely used
5. Group VarInt: SIMD-friendly batch encoding
6. Choice depends on distribution, speed needs, alignment

---

**Practice**:

1. Encode 1-20 with Fibonacci coding
2. Implement VarInt encoder/decoder
3. Compare compressed sizes for same data
4. Measure decode speed of different codes

---

**Next Chapter**: [Dictionary Coding Introduction](./16_dictionary_intro.md)
