# Chapter 2: Delta Encoding

## Overview

**Delta encoding** stores the differences between consecutive values instead of the values themselves.

```
Original: 100, 102, 105, 103, 108
Deltas:   100, +2,  +3,  -2,  +5
```

When values change slowly, the deltas are small and compress better.

---

## How It Works

### Encoding

1. Output the first value as-is (or a reference)
2. For each subsequent value, output: current - previous

### Decoding

1. Read the first value
2. For each delta, add it to the previous value

---

## Example Walkthrough

### Encoding

```
Input:  [1000, 1004, 1006, 1005, 1010, 1012]

Step by step:
- First value: 1000 (store as-is)
- 1004 - 1000 = +4
- 1006 - 1004 = +2
- 1005 - 1006 = -1
- 1010 - 1005 = +5
- 1012 - 1010 = +2

Output: [1000, 4, 2, -1, 5, 2]
```

### Why This Helps

Original values need 11 bits each (values up to 1012).
Deltas need only a few bits each (-1 to +5 fit in 4 bits signed).

### Decoding

```
Input:  [1000, 4, 2, -1, 5, 2]

Step by step:
- Start: 1000
- 1000 + 4 = 1004
- 1004 + 2 = 1006
- 1006 + (-1) = 1005
- 1005 + 5 = 1010
- 1010 + 2 = 1012

Output: [1000, 1004, 1006, 1005, 1010, 1012]
```

---

## Types of Delta Encoding

### Simple Delta

Store difference from previous value.

```
δᵢ = xᵢ - xᵢ₋₁
```

### Delta of Delta

Store difference of differences. Good for linear trends.

```
Original:    100, 110, 120, 130, 140
First delta: 100, 10, 10, 10, 10
Delta-delta: 100, 10, 0, 0, 0, 0
```

Linear sequences become zeros!

### XOR Delta

Store XOR instead of subtraction. Good for floating-point or when values share high bits.

```
δᵢ = xᵢ XOR xᵢ₋₁
```

When values are similar, most bits are zero.

---

## When Delta Encoding Works

### Excellent For

1. **Time series data**: Temperature, stock prices, sensor readings
2. **Audio samples**: Waveforms change gradually
3. **Sorted data**: Differences are always positive and small
4. **Incremental backups**: Store only what changed
5. **Video frames**: Frame differences (motion)

### Poor For

1. **Random data**: Deltas are as large as originals
2. **Uncorrelated data**: No benefit
3. **Already small values**: Nothing to gain

---

## Delta + Entropy Coding

Delta encoding doesn't compress by itself—it transforms data to be more compressible.

```
Original data: Needs many bits per value
      ↓
Delta encoding: Values cluster around zero
      ↓
Entropy coding: Small values get short codes
      ↓
Compressed output
```

Small deltas appear frequently → short codes → good compression.

---

## Predictive Coding (Generalized Delta)

Delta encoding is the simplest predictor:

```
Prediction: x̂ᵢ = xᵢ₋₁
Residual:   rᵢ = xᵢ - x̂ᵢ
```

Better predictors give smaller residuals:

### Linear Prediction

```
x̂ᵢ = a₁xᵢ₋₁ + a₂xᵢ₋₂ + ... + aₙxᵢ₋ₙ
```

Used in audio compression (FLAC, MPEG).

### 2D Prediction (Images)

```
Predict pixel from neighbors:
    A B C
    D X

Predictors:
- X = A (left)
- X = B (above)
- X = A + B - C (linear)
- X = median(A, B, C)
```

PNG uses these prediction filters.

---

## Handling Negative Deltas

Deltas can be negative, which complicates encoding.

### Signed Representation

Use signed integers directly.
- Pro: Natural
- Con: Variable-length encoding is tricky

### Zigzag Encoding

Map signed to unsigned:

```
0 → 0
-1 → 1
1 → 2
-2 → 3
2 → 4
...

Formula: zigzag(n) = (n << 1) ^ (n >> 31)  // for 32-bit
```

Now all values are non-negative, easier to encode.

### Absolute + Sign

Store absolute value and separate sign bit.

---

## Pseudocode

### Encoding

```
function delta_encode(input):
    if length(input) == 0:
        return []

    output = [input[0]]  // First value as-is

    for i = 1 to length(input) - 1:
        delta = input[i] - input[i-1]
        output.append(delta)

    return output
```

### Decoding

```
function delta_decode(input):
    if length(input) == 0:
        return []

    output = [input[0]]
    current = input[0]

    for i = 1 to length(input) - 1:
        current = current + input[i]
        output.append(current)

    return output
```

### Zigzag Encoding

```
function zigzag_encode(n):
    return (n << 1) ^ (n >> 31)

function zigzag_decode(n):
    return (n >> 1) ^ -(n & 1)
```

---

## Real-World Applications

| Application | How delta is used |
|-------------|-------------------|
| **Git** | Stores file differences (diffs) |
| **rsync** | Transfers only changed blocks |
| **Video codecs** | P-frames store differences from previous |
| **FLAC audio** | Linear prediction residuals |
| **PNG images** | Filter modes are prediction-based |
| **Time-series DB** | Gorilla, InfluxDB use delta encoding |
| **Backup systems** | Incremental backups |

---

## Delta Chains and Error Propagation

### Problem

If one delta is corrupted, all subsequent values are wrong.

```
Correct:  100, +5, +3, +2  → 100, 105, 108, 110
Corrupt:  100, +5, +9, +2  → 100, 105, 114, 116 (all wrong after)
```

### Solutions

1. **Keyframes**: Periodically store absolute values
2. **Error correction**: Add redundancy for critical deltas
3. **Checksums**: Detect corruption

Video codecs use I-frames (keyframes) periodically for this reason.

---

## Optimization: Block-Based Delta

Instead of per-value deltas, use per-block reference:

```
Block 1: Base = 1000, deltas = [0, +4, +2, +5]
Block 2: Base = 2000, deltas = [0, +1, +3, +2]
```

Benefits:
- Random access (jump to any block)
- Error containment (corruption limited to one block)
- Better parallelization

---

## Complexity Analysis

### Time Complexity
- Encoding: O(n)
- Decoding: O(n)

### Space Complexity
- O(n) for output
- O(1) additional space

---

## Key Takeaways

1. Delta encoding stores differences, not absolute values
2. Works when consecutive values are similar
3. Transform, not compression—combine with entropy coding
4. Delta-of-delta helps with linear trends
5. Zigzag encoding handles negative values efficiently
6. Used extensively in time series, audio, video, and backups

---

**Practice**: Implement delta encoding and decoding. Test on:
1. Sorted array of integers
2. Audio samples (load a WAV file)
3. Random integers (should not help)
4. Implement zigzag encoding for signed deltas

---

**Next Chapter**: [Bit Packing](./03_bit_packing.md)
