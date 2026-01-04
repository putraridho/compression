# Chapter 7: Psychovisual Modeling

## What is Psychovisual Modeling?

**Psychovisual** models exploit limitations of human vision.

Video/image codecs remove what we can't see.

---

## Key Visual Phenomena

### 1. Contrast Sensitivity

We're most sensitive to medium spatial frequencies:

```
Sensitivity
    │         ╱╲
    │        ╱  ╲
    │       ╱    ╲
    │      ╱      ╲
    │     ╱        ╲
    │────╱──────────╲────
    │   low    mid    high
    └──────────────────────> Spatial Frequency
```

Low frequency: Gradual changes (can miss slight variations)
High frequency: Fine details (poor sensitivity)

### 2. Luminance vs Chrominance

We see brightness details better than color details.

```
Resolution needed:
  Luminance (Y):  Full
  Chrominance (U,V): 1/4 to 1/2
```

This is why JPEG uses 4:2:0 or 4:2:2 subsampling.

### 3. Masking

Texture and edges mask compression artifacts.

```
Flat areas: Errors very visible (allocate more bits)
Textured areas: Errors hidden (allocate fewer bits)
```

---

## Color Subsampling

### YCbCr Color Space

```
RGB → Y (luminance) + Cb,Cr (chrominance)
```

### Subsampling Formats

```
4:4:4 - Full resolution for all
        ████ ████ ████

4:2:2 - Half horizontal chroma
        ████ ██░░ ██░░

4:2:0 - Quarter chroma (half each dimension)
        ████ █░░░ █░░░
```

4:2:0 is standard for video — 50% size reduction!

---

## JPEG Quantization Matrix

Exploits contrast sensitivity:

```
Quantization Matrix (lower = keep more detail):

16  11  10  16  24   40   51   61
12  12  14  19  26   58   60   55
14  13  16  24  40   57   69   56
14  17  22  29  51   87   80   62
18  22  37  56  68  109  103   77
24  35  55  64  81  104  113   92
49  64  78  87 103  121  120  101
72  92  95  98 112  100  103   99

Top-left (DC, low freq): Fine quantization (sensitive)
Bottom-right (high freq): Coarse quantization (insensitive)
```

---

## Visual Masking

### Texture Masking

Complex textures hide errors:
```
Grass, foliage, fabric → Compress heavily
Sky, walls → Compress carefully
```

### Edge Masking

Errors near edges are hidden:
```
Sharp edges → Tolerate more error
Smooth gradients → Need precision
```

### Temporal Masking

Fast motion hides errors:
```
Static scene → High quality needed
Fast action → Lower quality acceptable
```

---

## Perceptual Quality Metrics

### SSIM (Structural Similarity)

Measures structural similarity, not pixel differences:

```
SSIM = (2μₓμᵧ + C₁)(2σₓᵧ + C₂)
       ────────────────────────────
       (μₓ² + μᵧ² + C₁)(σₓ² + σᵧ² + C₂)

Considers:
  - Luminance
  - Contrast
  - Structure
```

### VMAF (Video Multimethod Assessment Fusion)

Machine learning model trained on human ratings:
```
Netflix's quality metric
Combines multiple measurements
Correlates with subjective quality
```

---

## Adaptive Quantization

### Spatial Adaptation

Flat regions: Finer quantization
Textured regions: Coarser quantization

### Temporal Adaptation

Scene changes: I-frame (high quality reference)
Smooth motion: P/B-frames (predict from references)

---

## Film Grain

### The Problem

Film grain is random → Expensive to encode exactly.

### Solutions

1. **Denoise**: Remove grain, encode clean, re-add grain
2. **Film grain synthesis**: Encode grain parameters, synthesize at decode
3. **AV1/H.264**: Built-in film grain synthesis

---

## Key Takeaways

1. Vision has frequency-dependent sensitivity
2. Chrominance resolution can be reduced (4:2:0)
3. Texture and motion mask errors
4. Quantization matrices exploit sensitivity
5. SSIM/VMAF measure perceptual quality
6. Adaptive quantization optimizes bit allocation

---

**Next Chapter**: [Predictive Coding](./08_predictive_coding.md)
