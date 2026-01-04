# Chapter 5: Wavelet Transform

## What are Wavelets?

**Wavelets** analyze signals at multiple resolutions simultaneously.

Unlike DCT (fixed blocks), wavelets capture both frequency and location.

---

## DCT vs Wavelet

### DCT

```
8×8 block → 64 frequency coefficients
            (frequency only, no location)
```

### Wavelet

```
Image → Multi-resolution pyramid
        (frequency + location at each scale)
```

---

## Basic Idea

### Multi-Resolution Analysis

```
Original    Coarse      Coarser     Coarsest
1024×1024 → 512×512  →  256×256  →  128×128
            + detail    + detail    + detail
```

At each level:
- Approximation (low frequency)
- Detail (high frequency differences)

---

## 1D Wavelet Decomposition

### Step 1: Filter

Apply low-pass and high-pass filters:

```
Signal: [a, b, c, d, e, f, g, h]

Low-pass (average):  [(a+b)/2, (c+d)/2, (e+f)/2, (g+h)/2]
High-pass (detail):  [(a-b)/2, (c-d)/2, (e-f)/2, (g-h)/2]
```

### Step 2: Downsample

Keep every other sample:

```
Low: [L0, L1, L2, L3]
High: [H0, H1, H2, H3]
```

### Step 3: Recurse

Apply to low-pass output repeatedly.

---

## Haar Wavelet Example

Simplest wavelet — just averages and differences.

```
Level 0: [56, 40, 8, 24, 48, 48, 40, 16]

Level 1:
  Averages: [48, 16, 48, 28]
  Details:  [8, -8, 0, 12]

Level 2:
  Averages: [32, 38]
  Details:  [16, 10]

Level 3:
  Averages: [35]
  Details:  [-3]

Final: [35 | -3 | 16, 10 | 8, -8, 0, 12]
        └─┬┘ └──┬──┘  └───────┬────────┘
       coarsest   coarser        finest
```

---

## 2D Wavelet Transform

Apply 1D wavelet in both directions:

```
         ┌───────┬───────┐
         │  LL   │  HL   │
         │(approx)│(horiz)│
Original ├───────┼───────┤
         │  LH   │  HH   │
         │(vert) │(diag) │
         └───────┴───────┘

LL: Low-pass both directions (approximation)
HL: High-pass horizontal, low vertical
LH: Low-pass horizontal, high vertical
HH: High-pass both (diagonal detail)
```

### Multi-Level

```
┌─────┬─────┬───────────┐
│LL3  │HL3  │           │
├─────┼─────┤   HL2     │
│LH3  │HH3  │           │
├─────┴─────┼───────────┤
│           │           │
│    LH2    │    HH2    │
│           │           │
├───────────┴───────────┤
│                       │
│         HL1           │
│                       │
├───────────────────────┤
│                       │
│         LH1           │
│                       │
├───────────────────────┤
│                       │
│         HH1           │
│                       │
└───────────────────────┘
```

---

## Common Wavelets

### Haar

```
Simplest, box-shaped
Good for piecewise constant signals
```

### Daubechies (db2, db4, etc.)

```
Compact support
Smoother than Haar
Used in JPEG 2000
```

### Biorthogonal (CDF 9/7)

```
Symmetric, near-linear phase
Best for images
Default in JPEG 2000 lossy
```

### LeGall (CDF 5/3)

```
Integer coefficients
Perfect for lossless
Used in JPEG 2000 lossless
```

---

## Advantages Over DCT

### No Blocking Artifacts

```
DCT: Block boundaries visible at low quality
Wavelet: Gradual quality degradation, no blocks
```

### Scalability

```
Embedded bitstream:
  First N bytes → low resolution preview
  More bytes → higher resolution
  All bytes → full quality
```

### Localization

```
A local change affects local coefficients only
Better for sparse signals
```

---

## JPEG 2000

Uses wavelet transform (CDF 9/7 or 5/3):

```
Image → DWT → Quantization → EBCOT → Bitstream
```

### EBCOT

Embedded Block Coding with Optimized Truncation:
- Encode bit-planes
- Truncate at any point for target size
- Progressive transmission

---

## Lifting Scheme

Efficient wavelet computation:

```
Traditional: Convolve with filters
Lifting: In-place prediction and update

Split → Predict → Update

1. Split into even/odd
2. Predict odd from even (detail)
3. Update even using odd (approximation)
```

### Advantages

- In-place computation
- Integer-to-integer transform possible
- Faster than convolution

---

## Pseudocode: Haar Wavelet

### 1D Forward

```
function haar_forward(signal):
    n = length(signal)
    while n > 1:
        for i = 0 to n/2 - 1:
            avg = (signal[2i] + signal[2i+1]) / 2
            diff = (signal[2i] - signal[2i+1]) / 2
            signal[i] = avg
            signal[n/2 + i] = diff
        n = n / 2
    return signal
```

### 1D Inverse

```
function haar_inverse(coeffs):
    n = 2
    while n <= length(coeffs):
        for i = n/2 - 1 downto 0:
            avg = coeffs[i]
            diff = coeffs[n/2 + i]
            coeffs[2i] = avg + diff
            coeffs[2i+1] = avg - diff
        n = n × 2
    return coeffs
```

---

## Wavelets vs DCT

| Aspect | DCT (JPEG) | Wavelet (JPEG 2000) |
|--------|------------|---------------------|
| Blocking | Yes | No |
| Scalability | No | Yes |
| Compression | Good | Similar or better |
| Complexity | Lower | Higher |
| Hardware | Common | Rare |

---

## Key Takeaways

1. Wavelets capture frequency AND location
2. Multi-resolution decomposition
3. No blocking artifacts
4. Enables progressive transmission
5. JPEG 2000 uses wavelets (but not widely adopted)
6. Lifting scheme for efficient implementation

---

**Next Chapter**: [Psychoacoustic Modeling](./06_psychoacoustic.md)
