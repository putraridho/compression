# Chapter 4: Discrete Cosine Transform (DCT)

## What is DCT?

The **Discrete Cosine Transform** converts spatial data into frequency components.

It's the foundation of JPEG, MPEG, and most image/video codecs.

---

## Why Transform?

### Spatial Domain

```
Pixel values: 52, 55, 61, 66, 70, 61, 64, 73
              (8 independent values)
```

### Frequency Domain

```
DCT: 185, -18, 15, -9, 23, -9, -14, 19
     DC   low freq.   →   high freq.

Energy concentrates in few coefficients.
High frequencies often near zero.
```

Quantize high frequencies heavily → big compression!

---

## 1D DCT Formula

```
           N-1
X[k] = c[k] × Σ x[n] × cos(π(2n+1)k / 2N)
           n=0

where:
  c[0] = √(1/N)
  c[k] = √(2/N) for k > 0
```

### Inverse DCT (IDCT)

```
           N-1
x[n] = Σ c[k] × X[k] × cos(π(2n+1)k / 2N)
       k=0
```

DCT is reversible — no information lost in transform itself.

---

## 2D DCT

For images, apply DCT in both dimensions:

```
              M-1 N-1
F[u][v] = c[u]c[v] × Σ   Σ f[x][y] × cos(π(2x+1)u/2M) × cos(π(2y+1)v/2N)
              x=0 y=0
```

Or equivalently:
```
2D DCT = 1D DCT on rows, then 1D DCT on columns
```

---

## DCT Basis Functions (8×8)

Each coefficient corresponds to a basis pattern:

```
DC (0,0):    Low (1,0):   High (7,7):
████████     ████░░░░     ▓░▓░▓░▓░
████████     ████░░░░     ░▓░▓░▓░▓
████████     ████░░░░     ▓░▓░▓░▓░
████████     ████░░░░     ░▓░▓░▓░▓
████████     ░░░░████     ▓░▓░▓░▓░
████████     ░░░░████     ░▓░▓░▓░▓
████████     ░░░░████     ▓░▓░▓░▓░
████████     ░░░░████     ░▓░▓░▓░▓
(constant)   (horizontal) (checkerboard)
```

---

## Energy Compaction

Most image energy ends up in low frequencies:

```
Original 8×8 block:
140 144 147 140 140 155 179 175
144 152 140 147 140 148 167 179
152 155 136 167 163 162 152 172
168 145 156 160 152 155 136 160
162 148 156 148 140 136 147 162
147 167 140 155 155 140 136 162
136 156 123 167 162 144 140 147
148 155 136 155 152 147 147 136

DCT:
1260.25  -29.07   -43.72   ...
 -73.08   -6.53    10.22
  52.86   -7.73   -14.89
  ...                      (many near zero)
```

DC coefficient (top-left) holds average.
Most energy in top-left region.

---

## JPEG's 8×8 DCT

JPEG divides image into 8×8 blocks and applies DCT:

```
┌────────┬────────┬────────┐
│ 8×8    │ 8×8    │ 8×8    │
│ block  │ block  │ block  │
├────────┼────────┼────────┤
│ 8×8    │ 8×8    │ 8×8    │
│ block  │ block  │ block  │
└────────┴────────┴────────┘

Each block: DCT → Quantize → Entropy code
```

### Why 8×8?

- Good balance of efficiency and quality
- Larger blocks: better energy compaction
- Smaller blocks: avoid blocking artifacts
- 8×8 chosen empirically

---

## Fast DCT

Naive DCT: O(N²) multiplications

### Fast algorithms

**Loeffler**: O(N log N)
**Integer approximations**: Used in hardware

```
8-point DCT:
  Naive: 64 multiplications
  Fast: 11 multiplications + 29 additions
```

Modern CPUs have SIMD DCT instructions.

---

## Quantization After DCT

```
DCT coefficients:
305  -12   -6    2    0   -1    0    0
-23    4    1    0    0    0    0    0
-11    1    1    0    0    0    0    0
  5    0    0    0    0    0    0    0
  0    0    0    0    0    0    0    0
  0    0    0    0    0    0    0    0
  0    0    0    0    0    0    0    0
  0    0    0    0    0    0    0    0

After heavy quantization:
19    0    0    0    0    0    0    0
 0    0    0    0    0    0    0    0
 0    0    0    0    0    0    0    0
...

Most coefficients become zero!
```

---

## Blocking Artifacts

DCT processes blocks independently.

At low quality, block boundaries become visible:

```
┌────────┬────────┐
│        │        │
│  Shade │  Shade │  ← Discontinuity at boundary
│   A    │   B    │
│        │        │
├────────┼────────┤
```

### Solutions

1. **Deblocking filter**: Smooth boundaries after decode
2. **Overlapping transforms**: MDCT (audio), lapped transforms
3. **Larger blocks**: H.265 uses up to 32×32

---

## DCT vs DFT

| Aspect | DCT | DFT |
|--------|-----|-----|
| Output | Real | Complex |
| Boundary | Implicit reflection | Wrap-around |
| Energy compaction | Excellent | Good |
| Use | Images, video | Audio, signals |

DCT has better energy compaction for typical images.

---

## Pseudocode

### 1D DCT

```
function dct_1d(input[N]):
    output = array[N]

    for k = 0 to N-1:
        sum = 0
        for n = 0 to N-1:
            sum += input[n] × cos(π × (2n+1) × k / (2N))

        if k == 0:
            output[k] = sum × sqrt(1/N)
        else:
            output[k] = sum × sqrt(2/N)

    return output
```

### 2D DCT

```
function dct_2d(block[N][N]):
    // DCT on rows
    for row in block:
        row = dct_1d(row)

    // DCT on columns
    for col = 0 to N-1:
        column = [block[row][col] for row in 0..N]
        column = dct_1d(column)
        for row = 0 to N-1:
            block[row][col] = column[row]

    return block
```

---

## Key Takeaways

1. DCT converts spatial data to frequencies
2. Energy concentrates in low frequencies
3. High frequencies can be quantized heavily
4. JPEG uses 8×8 DCT blocks
5. Fast algorithms make DCT practical
6. Blocking artifacts are main quality issue

---

**Next Chapter**: [Wavelet Transform](./05_wavelet.md)
