# Chapter 2: Quantization

## What is Quantization?

**Quantization** maps a range of values to a smaller set of discrete values.

This is where data loss occurs in lossy compression.

---

## Simple Example

```
Input values:  0.0, 0.3, 0.5, 0.7, 1.0, 1.2, 1.8, 2.0
Quantize to integers:
Output values: 0, 0, 0, 1, 1, 1, 2, 2

Information lost! 0.3 and 0.5 both become 0.
```

---

## Scalar Quantization

Quantize each value independently.

### Uniform Quantization

Equal-sized bins:

```
Quantizer: Q(x) = round(x / step) × step

Example with step = 10:
  Q(7) = round(7/10) × 10 = 0
  Q(13) = round(13/10) × 10 = 10
  Q(27) = round(27/10) × 10 = 30
```

### Non-Uniform Quantization

Variable-sized bins:

```
μ-law (audio):
  More precision near zero
  Less precision for loud sounds

Log quantization:
  Bin size proportional to value
```

---

## Quantization Step (Q)

The step size determines quality:

```
Larger step = more loss = smaller file
Smaller step = less loss = larger file

Example:
  Step 1: Output = 0,1,2,3,...255 (lossless for 8-bit)
  Step 2: Output = 0,2,4,6,...254 (128 levels)
  Step 4: Output = 0,4,8,12,...252 (64 levels)
  Step 16: Output = 0,16,32,...240 (16 levels)
```

---

## Quantization in JPEG

### Quantization Matrix

JPEG uses different steps for different frequencies:

```
Luminance Quantization Matrix (quality ~50):
16  11  10  16  24   40   51   61
12  12  14  19  26   58   60   55
14  13  16  24  40   57   69   56
14  17  22  29  51   87   80   62
18  22  37  56  68  109  103   77
24  35  55  64  81  104  113   92
49  64  78  87 103  121  120  101
72  92  95  98 112  100  103   99

Low frequencies (top-left): small steps (more precision)
High frequencies (bottom-right): large steps (less precision)
```

### Applying Quantization

```
Quantized[i][j] = round(DCT[i][j] / Q_matrix[i][j])
```

### Dequantization

```
Reconstructed[i][j] = Quantized[i][j] × Q_matrix[i][j]
```

---

## Dead-Zone Quantization

Values near zero map to exactly zero:

```
        Output
           |
           |      ●
           |     /
      ─────●────●─────── Input
          /     │
         ●      │
           │    │
           │    │
        ◄──┴──┴─►
        dead zone
```

Benefits:
- Many coefficients become zero
- Zeros compress very efficiently
- Used in most video codecs

---

## Quantization Error

### Error Distribution

```
Error = Original - Reconstructed

For uniform quantization:
  Max error = step / 2
  RMS error = step / √12 ≈ step / 3.46
```

### Signal-to-Quantization-Noise Ratio (SQNR)

```
SQNR = 6.02 × bits + 1.76 dB

8 bits: SQNR ≈ 50 dB
16 bits: SQNR ≈ 98 dB
```

Each bit adds ~6 dB of quality.

---

## Adaptive Quantization

### Spatially Adaptive

Different regions get different quantization:
- Flat areas: Coarse quantization (errors visible)
- Detailed areas: Fine quantization
- Edges: Preserve sharpness

### Perceptually Adaptive

Based on human perception:
- More bits where we're sensitive
- Fewer bits where we won't notice

---

## Rate Control

### Goal

Achieve target bitrate while maximizing quality.

### Methods

**Fixed QP (Quantization Parameter)**:
```
Constant quality, variable bitrate
```

**CBR (Constant Bit Rate)**:
```
Adjust QP to maintain bitrate
Easy scenes: high QP
Hard scenes: low QP (hit budget)
```

**VBR (Variable Bit Rate)**:
```
Target average, allow variation
Better quality distribution
```

**CRF (Constant Rate Factor)**:
```
Constant perceived quality
Smart allocation across time
```

---

## Quantization Parameter (QP)

H.264/H.265 use QP 0-51:

```
QP 0: Nearly lossless
QP 18: Very high quality
QP 23: High quality (CRF default)
QP 28: Medium quality
QP 35: Low quality
QP 51: Minimum quality
```

Each QP increase of 6 roughly doubles file size.

---

## Dithering

Add noise before quantization to reduce banding:

```
Without dither: Visible steps (banding)
With dither: Noise masks steps

output = quantize(input + random_noise)
```

Used in:
- Image/video downsampling
- Audio bit depth reduction
- Color reduction

---

## Pseudocode

### Uniform Scalar Quantizer

```
function quantize(value, step):
    return round(value / step)

function dequantize(index, step):
    return index × step
```

### Dead-Zone Quantizer

```
function quantize_deadzone(value, step, deadzone):
    if abs(value) < deadzone:
        return 0
    else if value > 0:
        return floor((value - deadzone) / step) + 1
    else:
        return -floor((-value - deadzone) / step) - 1
```

---

## Key Takeaways

1. Quantization = mapping to fewer discrete values
2. Step size controls quality/size trade-off
3. Non-uniform quantization can match perception
4. Dead-zone makes many values exactly zero
5. Quantization matrices vary precision by frequency
6. Rate control adjusts quantization to hit bitrate targets

---

**Next Chapter**: [Rate-Distortion Theory](./03_rate_distortion.md)
