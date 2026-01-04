# Chapter 14: Elias Codes

## Overview

**Elias codes** are universal codes for positive integers.

"Universal" means:
- Works for any distribution
- No parameters needed
- Self-delimiting (know where code ends)

Three main variants:
- Elias Gamma (γ)
- Elias Delta (δ)
- Elias Omega (ω)

---

## Elias Gamma Code

### The Idea

Encode n as:
1. Number of bits in n (in unary)
2. The value n (in binary)

### Encoding

```
1. Let k = floor(log₂(n))  // number of bits minus 1
2. Output k zeros
3. Output n in binary (k+1 bits)
```

### Examples

| n | k | Zeros | Binary | Gamma Code |
|---|---|-------|--------|------------|
| 1 | 0 | (none) | 1 | 1 |
| 2 | 1 | 0 | 10 | 010 |
| 3 | 1 | 0 | 11 | 011 |
| 4 | 2 | 00 | 100 | 00100 |
| 5 | 2 | 00 | 101 | 00101 |
| 6 | 2 | 00 | 110 | 00110 |
| 7 | 2 | 00 | 111 | 00111 |
| 8 | 3 | 000 | 1000 | 0001000 |
| 16 | 4 | 0000 | 10000 | 000010000 |

### Decoding

```
1. Count zeros until you hit a 1
   This count is k
2. Read k more bits after the 1
3. Combine to get n
```

### Code Length

For value n:
```
Length = 2 × floor(log₂(n)) + 1
```

Approximately 2 log₂(n) bits.

---

## Elias Delta Code

### The Idea

Use Gamma code to encode the length.

For large numbers, this is shorter than Gamma.

### Encoding

```
1. Let k = floor(log₂(n))
2. Encode (k+1) using Gamma code
3. Output last k bits of n (without leading 1)
```

### Examples

| n | k | Gamma(k+1) | Last k bits | Delta Code |
|---|---|-----------|-------------|------------|
| 1 | 0 | 1 | (none) | 1 |
| 2 | 1 | 010 | 0 | 0100 |
| 3 | 1 | 010 | 1 | 0101 |
| 4 | 2 | 011 | 00 | 01100 |
| 5 | 2 | 011 | 01 | 01101 |
| 6 | 2 | 011 | 10 | 01110 |
| 7 | 2 | 011 | 11 | 01111 |
| 8 | 3 | 00100 | 000 | 00100000 |
| 16 | 4 | 00101 | 0000 | 001010000 |

### Decoding

```
1. Decode Gamma to get (k+1)
2. Read k more bits
3. Prepend 1, these k+1 bits form n
```

### Code Length

For value n:
```
Length = floor(log₂(n)) + 2 × floor(log₂(floor(log₂(n)) + 1)) + 1
```

Approximately log₂(n) + 2 log₂(log₂(n)) bits.

---

## Elias Omega Code

### The Idea

Recursive: encode length of length of length...

Most efficient for very large numbers.

### Encoding

```
1. Put 0 at the end (terminator)
2. Let current = n
3. While current > 1:
   a. Prepend binary of current
   b. current = length of binary - 1
4. Output the sequence
```

### Examples

| n | Process | Omega Code |
|---|---------|------------|
| 1 | Just terminator | 0 |
| 2 | 10, len=2→1, stop | 10 0 |
| 3 | 11, len=2→1, stop | 11 0 |
| 4 | 100, len=3→2, 10, len=2→1 | 10 100 0 |
| 5 | 101, len=3→2, 10 | 10 101 0 |
| 7 | 111, len=3→2, 10 | 10 111 0 |
| 8 | 1000, len=4→3, 11, len=2→1 | 11 1000 0 |
| 16 | 10000, len=5→4, 100, len=3→2, 10 | 10 100 10000 0 |

### Decoding

```
1. current = 1
2. While next bit is 1:
   a. Read (current + 1) bits including the 1
   b. This value becomes new current
3. Return current
```

### Code Length

Approximately log₂(n) + log₂(log₂(n)) + log₂(log₂(log₂(n))) + ...

Best for very large numbers.

---

## Comparison

### Code Length

| n | Gamma | Delta | Omega |
|---|-------|-------|-------|
| 1 | 1 | 1 | 1 |
| 2 | 3 | 4 | 3 |
| 4 | 5 | 5 | 6 |
| 8 | 7 | 8 | 8 |
| 16 | 9 | 9 | 11 |
| 256 | 17 | 14 | 17 |
| 65536 | 33 | 22 | 22 |

**Gamma**: Best for small numbers
**Delta**: Best for medium/large numbers
**Omega**: Best for very large numbers

---

## When to Use Each

### Gamma

- Small positive integers
- Simple implementation
- Known small range

### Delta

- Medium to large integers
- When range is unknown
- General purpose

### Omega

- Very large integers
- Theoretical interest
- Rarely used in practice

---

## Handling Zero

Elias codes encode positive integers (n ≥ 1).

To include zero:
- Encode (n + 1) instead of n
- Or use separate zero symbol

---

## Signed Integers

Use zigzag mapping:

```
0  → 1
-1 → 2
1  → 3
-2 → 4
2  → 5
...

Formula:
  positive n → 2n + 1
  negative n → -2n
```

---

## Pseudocode

### Gamma Encoder

```
function gamma_encode(n):
    k = floor(log2(n))

    for i = 0 to k - 1:
        output_bit(0)

    for i = k downto 0:
        output_bit((n >> i) & 1)
```

### Gamma Decoder

```
function gamma_decode():
    k = 0
    while read_bit() == 0:
        k += 1

    n = 1
    for i = 0 to k - 1:
        n = (n << 1) | read_bit()

    return n
```

### Delta Encoder

```
function delta_encode(n):
    k = floor(log2(n))
    gamma_encode(k + 1)

    for i = k - 1 downto 0:
        output_bit((n >> i) & 1)
```

### Delta Decoder

```
function delta_decode():
    k = gamma_decode() - 1
    n = 1

    for i = 0 to k - 1:
        n = (n << 1) | read_bit()

    return n
```

---

## Universality

### What Makes a Code Universal?

A code is universal if for any source distribution:

```
lim(n→∞) L(n) / log₂(n) = 1
```

The code approaches optimal as numbers grow.

### Redundancy

| Code | Redundancy per symbol |
|------|----------------------|
| Gamma | ~log₂(n) |
| Delta | ~log₂(log₂(n)) |
| Omega | ~log₂*(n) (iterated log) |

---

## Real-World Usage

| Application | Code Used |
|-------------|-----------|
| Compressed integers | Gamma, Delta |
| Inverted indices | Gamma, Delta |
| Graph compression | Various Elias |
| File formats | Delta common |

### Why Not More Common?

- Bit-by-bit processing is slow
- VarInt (byte-aligned) often preferred
- Golomb/Rice better for geometric distributions

---

## Key Takeaways

1. Universal codes work without knowing distribution
2. Gamma: length in unary + value in binary
3. Delta: length encoded with Gamma
4. Omega: recursive length encoding
5. Gamma best for small, Delta for larger numbers
6. Add 1 to handle zero

---

**Practice**:

1. Encode 1-20 with Gamma code
2. Encode 1-20 with Delta code
3. Find crossover point where Delta beats Gamma
4. Implement encoder and decoder

---

**Next Chapter**: [Other Integer Codes](./15_integer_codes.md)
