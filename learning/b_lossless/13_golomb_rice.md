# Chapter 13: Golomb and Rice Coding

## Overview

Golomb and Rice codes are optimal for **geometric distributions** — where small values are much more common than large values.

Perfect for:
- Prediction residuals (errors cluster around 0)
- Run lengths
- Small integers with long tail

---

## Geometric Distribution

Probability decreases geometrically:

```
P(n) = (1-p) × p^n  for n = 0, 1, 2, ...
```

- P(0) is highest
- P(1) < P(0)
- P(2) < P(1)
- ... (exponentially decreasing)

**Example**: Prediction errors in images, audio residuals, inter-frame differences.

---

## Golomb Coding

### The Idea

Divide number n by parameter m:
```
q = n / m   (quotient)
r = n mod m (remainder)
```

Encode:
- q in unary
- r in binary (or truncated binary)

### Encoding Steps

For value n with parameter m:

1. Compute q = floor(n / m)
2. Compute r = n mod m
3. Output q ones followed by a zero: `111...110`
4. Output r in binary (using log₂(m) or ceil(log₂(m)) bits)

### Example: Golomb with m=5

Encode n=13:
```
q = 13 / 5 = 2
r = 13 mod 5 = 3

Quotient (unary): 110 (two 1s, then 0)
Remainder: 3 in ~2.3 bits

But m=5 isn't power of 2, need truncated binary.
```

---

## Truncated Binary for Remainder

When m is not a power of 2:

```
k = floor(log₂(m))
threshold = 2^(k+1) - m

If r < threshold:
    Use k bits
Else:
    Use k+1 bits for (r + threshold)
```

### Example: m=5

```
k = floor(log₂(5)) = 2
threshold = 2³ - 5 = 3

r=0: 0 < 3, use 2 bits: 00
r=1: 1 < 3, use 2 bits: 01
r=2: 2 < 3, use 2 bits: 10
r=3: 3 ≥ 3, use 3 bits: (3+3)=6 = 110
r=4: 4 ≥ 3, use 3 bits: (4+3)=7 = 111
```

---

## Rice Coding

**Rice coding** is Golomb coding where m is a power of 2:

```
m = 2^k
```

### Why This is Simpler

```
q = n >> k        (right shift)
r = n & (m-1)     (bit mask)
```

Remainder is just the low k bits — no truncated binary needed!

### Rice Encoding

1. Output q in unary (q ones, then zero)
2. Output low k bits of n directly

### Example: Rice with k=3 (m=8)

Encode n=19:
```
q = 19 >> 3 = 2
r = 19 & 7 = 3

Output: 110 (unary for 2) + 011 (binary for 3)
Full code: 110011
```

Decode 110011:
```
Read unary until 0: 110 → q=2
Read 3 bits: 011 → r=3
n = 2×8 + 3 = 19 ✓
```

---

## Choosing the Parameter

### Optimal m for Golomb

For geometric distribution with parameter p:

```
m = ceiling(-1 / log₂(p))
```

Or approximately:

```
m ≈ 0.69 × mean
```

### Optimal k for Rice

```
k = max(0, floor(log₂(mean)))
```

### Adaptive Parameter

Estimate mean from recent symbols:

```
sum = sum of last N values
mean = sum / N
k = log₂(mean)
```

Update k periodically for best compression.

---

## Exponential-Golomb Codes

Used in H.264/AVC video codec.

### Order-0 Exp-Golomb

```
1. Write (floor(log₂(n+1))) zeros
2. Write binary representation of (n+1)
```

### Examples

| n | Binary(n+1) | Zeros needed | Code |
|---|-------------|--------------|------|
| 0 | 1 | 0 | 1 |
| 1 | 10 | 1 | 010 |
| 2 | 11 | 1 | 011 |
| 3 | 100 | 2 | 00100 |
| 4 | 101 | 2 | 00101 |
| 5 | 110 | 2 | 00110 |
| 6 | 111 | 2 | 00111 |
| 7 | 1000 | 3 | 0001000 |

### Order-k Exp-Golomb

Prefix with k bits of suffix:
```
Encode (n >> k) with order-0
Append (n & ((1<<k)-1))
```

---

## Signed Values

For signed integers, use **zigzag mapping**:

```
Signed → Unsigned:
0  →  0
-1 →  1
1  →  2
-2 →  3
2  →  4
...

Formula:
if n >= 0: unsigned = 2n
if n < 0:  unsigned = -2n - 1
```

Then apply Golomb/Rice to unsigned value.

---

## Pseudocode

### Rice Encoder

```
function rice_encode(n, k):
    q = n >> k
    r = n & ((1 << k) - 1)

    // Output unary
    for i = 0 to q - 1:
        output_bit(1)
    output_bit(0)

    // Output remainder
    output_bits(r, k)
```

### Rice Decoder

```
function rice_decode(k):
    // Read unary
    q = 0
    while read_bit() == 1:
        q += 1

    // Read remainder
    r = read_bits(k)

    return (q << k) + r
```

### Golomb Encoder

```
function golomb_encode(n, m):
    q = n / m
    r = n mod m

    // Unary quotient
    for i = 0 to q - 1:
        output_bit(1)
    output_bit(0)

    // Truncated binary remainder
    k = floor(log2(m))
    threshold = (1 << (k+1)) - m

    if r < threshold:
        output_bits(r, k)
    else:
        output_bits(r + threshold, k + 1)
```

---

## Real-World Usage

| Application | Coding Used |
|-------------|-------------|
| FLAC audio | Rice coding |
| JPEG-LS | Golomb-Rice |
| H.264/H.265 | Exp-Golomb |
| PNG | Used in filter output |
| CRAM (genomics) | Golomb coding |
| LOCO-I | Golomb |

### FLAC's Rice Coding

```
For each audio block:
1. Apply linear prediction
2. Calculate residuals
3. Find optimal Rice parameter k
4. Encode residuals with Rice-k
```

---

## Comparison with Huffman

| Aspect | Huffman | Golomb/Rice |
|--------|---------|-------------|
| Optimal for | Known distribution | Geometric distribution |
| Overhead | Need tree | Just parameter k |
| Adaptivity | Rebuild tree | Just change k |
| Infinite alphabet | No | Yes |
| Implementation | Moderate | Simple |

Golomb/Rice advantages:
- Handles infinite alphabet
- Minimal overhead (just k)
- Very simple implementation

---

## Key Takeaways

1. Optimal for geometric (exponential) distributions
2. Rice = Golomb with power-of-2 parameter
3. Quotient in unary, remainder in binary
4. Parameter k ≈ log₂(mean)
5. Adaptive k improves compression
6. Used in FLAC, JPEG-LS, H.264

---

**Practice**:

1. Encode [0,1,2,3,4,5] with Rice k=2
2. Find optimal k for data with mean=5
3. Implement adaptive Rice coder
4. Compare to Huffman on prediction residuals

---

**Next Chapter**: [Elias Codes](./14_elias_codes.md)
