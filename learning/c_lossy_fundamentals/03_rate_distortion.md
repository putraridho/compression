# Chapter 3: Rate-Distortion Theory

## The Fundamental Trade-off

You cannot have both:
- Minimum file size
- Perfect quality

Rate-distortion theory quantifies this trade-off.

---

## Basic Definitions

### Rate (R)

Bits per symbol (or bits per second).

```
R = total_bits / num_symbols
```

### Distortion (D)

Measure of quality loss.

```
D = average(distance(original, reconstructed))

Common measures:
  MSE: Mean Squared Error
  MAE: Mean Absolute Error
```

---

## Rate-Distortion Function

R(D) = minimum rate to achieve distortion ≤ D.

```
         Rate R
           │
           │
         ▲ │╲
         │ │ ╲
         │ │  ╲
         │ │   ╲
         │ │    ╲
         │ │     ╲──────────
         │ │
         │ │
         └─┴──────────────────▶ Distortion D
           0    D_target
```

Properties:
- R(0) = ∞ (zero distortion needs infinite bits)
- R(∞) = 0 (infinite distortion needs zero bits)
- R(D) is monotonically decreasing
- R(D) is convex

---

## Gaussian Source Example

For Gaussian source with variance σ²:

```
R(D) = (1/2) × log₂(σ² / D)  for D ≤ σ²
R(D) = 0                      for D > σ²
```

### Interpretation

- Halving distortion costs 0.5 bits/sample
- Each additional 6 dB of quality costs ~1 bit/sample
- Zero distortion requires infinite rate

---

## Practical Rate-Distortion

### The Lagrangian

Optimize quality for given rate using Lagrangian:

```
J = D + λ × R

Minimize J to find best trade-off.
```

### Choosing λ

- High λ: Prioritize low rate (more loss)
- Low λ: Prioritize low distortion (less loss)
- λ is related to quantization step

---

## R-D Optimization in Codecs

### Mode Decision

For each block, choose best coding mode:

```
modes = [skip, intra_4x4, intra_8x8, inter_16x16, ...]

for mode in modes:
    distortion = compute_distortion(original, reconstruct(mode))
    rate = estimate_bits(mode)
    cost = distortion + λ × rate

    if cost < best_cost:
        best_mode = mode
```

### Transform Decision

Choose best transform size:
- 4×4, 8×8, 16×16, 32×32?
- Each has different R-D characteristics

---

## Operational Rate-Distortion

Real codecs don't achieve theoretical R(D).

```
         Rate
           │
     ○     │     Theoretical R(D)
           │ ╲
         ● │  ╲    ● Actual codec points
           │   ╲
         ● │    ╲────────
           │     ●
         ● │
           │
           └──────────────────▶ Distortion
```

### Gap Causes

1. Suboptimal quantization
2. Transform inefficiency
3. Finite precision
4. Complexity constraints

---

## Rate Control

### CBR (Constant Bit Rate)

```
Target: R bits per frame
For each frame:
    Adjust QP to hit rate target
```

Problem: Quality varies with content.

### VBR (Variable Bit Rate)

```
Target: Average R bits per frame
Allow temporary deviation
Allocate more bits to complex frames
```

Better quality distribution.

### Two-Pass Encoding

```
Pass 1: Analyze complexity of each frame
Pass 2: Allocate bits optimally
```

Best quality for target size.

---

## Quality Metrics

### Objective (Computable)

```
MSE = Σ(x - y)² / n

PSNR = 10 × log₁₀(MAX² / MSE)

SSIM = structural_similarity(x, y)
```

### Subjective (Human)

```
MOS: Mean Opinion Score (1-5)
DMOS: Differential MOS
Paired comparison tests
```

### Perceptual (Best of Both)

```
VMAF: Machine-learned from human ratings
SSIM: Correlates with perception
```

---

## R-D Curves Comparison

```
         PSNR (dB)
           │
        50 │         ●──────● Codec A
           │       /
        45 │      ●
           │    /  ●──────● Codec B
        40 │   ●  /
           │   │/
        35 │   ●
           │  /
        30 │ ●
           │
           └────────────────────▶ Bitrate (Mbps)
              1    2    3    4

Codec A: Better quality at same bitrate
         (curve is above and to the left)
```

---

## BD-Rate (Bjontegaard Delta)

Measure codec efficiency difference:

```
BD-Rate = % bitrate difference for same quality

Example:
  Codec A needs 100 Mbps for 40 dB
  Codec B needs 80 Mbps for 40 dB
  BD-Rate improvement = -20%
```

Standard way to compare codecs.

---

## Key Takeaways

1. Rate and distortion trade off fundamentally
2. R(D) is the theoretical minimum rate
3. Lagrangian (D + λR) enables optimization
4. Real codecs approach but don't reach R(D)
5. Rate control adapts quantization to hit targets
6. BD-Rate measures codec efficiency

---

**Next Chapter**: [Discrete Cosine Transform (DCT)](./04_dct.md)
