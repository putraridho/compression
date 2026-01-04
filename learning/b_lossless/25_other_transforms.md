# Chapter 25: Other Transforms

## Overview

Beyond BWT and MTF, several other transforms prepare data for compression.

---

## Run-Length Transform

Convert runs into (symbol, count) pairs.

### Basic Form

```
Input:  "aaabbbcc"
Output: [('a',3), ('b',3), ('c',2)]
```

### Zero-Run (RUNA/RUNB)

bzip2's encoding for zero runs:

```
Run of n zeros encoded as binary using RUNA/RUNB:
  1 zero:  RUNA
  2 zeros: RUNB
  3 zeros: RUNA RUNA
  4 zeros: RUNB RUNA
  ...

Binary: n = (1 + b₀) + (1 + b₁)×2 + (1 + b₂)×4 + ...
where b = RUNA→0, RUNB→1
```

Efficient for the many zeros after MTF.

---

## Distance Transform

Convert symbol to distance from last occurrence.

### Algorithm

```
For each symbol:
    Output: distance since last occurrence (or special value if first)
    Update: record position
```

### Example

```
Input:  "abracadabra"
Output: [∞, ∞, ∞, 3, ∞, 2, ∞, 4, 5, 5, 2]
         a  b  r  a  c  a  d  a  b  r  a
```

Small distances for repeated characters.

---

## Sorted Rank Transform

Alternative to MTF for BWT output.

### Idea

For position i in BWT output:
- Output = rank of character among all characters that could appear here

Works with suffix array structure.

### Advantage

Can be computed without explicit MTF list manipulation.

---

## Delta Transform

Store differences between consecutive values.

### Variants

```
First-order delta:
  y[i] = x[i] - x[i-1]

Second-order delta:
  y[i] = x[i] - 2×x[i-1] + x[i-2]

XOR delta:
  y[i] = x[i] XOR x[i-1]
```

### Best For

- Audio samples
- Time series
- Sequential data

---

## Interleave Transform

Rearrange bytes to group similar positions.

### Example: 16-bit Audio

```
Original (little-endian):
  [L₀, H₀, L₁, H₁, L₂, H₂, ...]

Interleaved:
  [L₀, L₁, L₂, ...] [H₀, H₁, H₂, ...]
```

Low bytes are similar, high bytes are similar → better compression.

### PNG Filters

PNG uses prediction then stores deltas:
```
None:    Store pixel as-is
Sub:     Store: pixel - left
Up:      Store: pixel - above
Average: Store: pixel - avg(left, above)
Paeth:   Store: pixel - paeth_predictor(left, above, upper_left)
```

---

## Context Transform

Reorganize data based on context.

### Record Reordering

For structured data (CSV, database):

```
Original (row-major):
  [name₁, age₁, city₁], [name₂, age₂, city₂], ...

Transformed (column-major):
  [name₁, name₂, ...], [age₁, age₂, ...], [city₁, city₂, ...]
```

Similar values group together.

---

## E8/E9 Transform

For x86 executables.

### The Problem

x86 CALL and JMP use relative offsets:

```
E8 xx xx xx xx  (CALL relative)
E9 xx xx xx xx  (JMP relative)
```

Same function called from different places → different bytes.

### The Transform

Convert relative addresses to absolute:

```
For each E8/E9:
    offset = read next 4 bytes
    absolute = current_position + offset
    write absolute address
```

Now same target → same bytes → better compression.

7-Zip uses this (BCJ filter).

---

## Predictor Transforms

General pattern:

```
1. Predict value
2. Store: actual - predicted
3. Residuals compress better than raw data
```

### Linear Prediction

```
predicted[i] = a₁×x[i-1] + a₂×x[i-2] + ... + aₙ×x[i-n]
residual[i] = x[i] - predicted[i]
```

Used in FLAC, PNG, video codecs.

### Coefficients

- Fixed coefficients (e.g., [1, 0, 0, 0])
- Adaptive coefficients (update based on error)
- Optimal coefficients (compute per block)

---

## Burrows-Wheeler-Scott Transform

A variant of BWT with different properties.

### Bijective BWT

Doesn't need end marker:
- Use bijective string sorting
- Slightly different but similar compression

---

## Transform Chains

Transforms can be chained:

```
bzip2:   Input → BWT → MTF → RLE → Huffman
PNG:     Input → Filter → DEFLATE
7-zip:   Input → E8/E9 → LZMA
FLAC:    Input → Interleave → LPC → Rice
```

Each transform targets different redundancy.

---

## Choosing Transforms

| Data Type | Recommended Transforms |
|-----------|----------------------|
| Text | BWT → MTF |
| Executables | E8/E9 |
| Audio | Interleave → Delta/LPC |
| Images | 2D Prediction |
| Structured | Column reorder |
| General | LZ77 (implicit transform) |

---

## Key Takeaways

1. Transforms prepare data for better compression
2. BWT groups similar contexts
3. MTF converts recency to small indices
4. Delta/prediction reduce variance
5. E8/E9 fixes relative addresses in executables
6. Interleave groups similar byte positions
7. Chain transforms for best results

---

**Next Chapter**: [DEFLATE](./26_deflate.md)
