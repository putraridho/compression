# Chapter 8: Predictive Coding

## The Core Idea

Predict the next value, encode only the **prediction error** (residual).

If prediction is good, residuals are small → compress better.

---

## Basic Formula

```
Prediction: p[n] = f(x[n-1], x[n-2], ...)
Residual:   e[n] = x[n] - p[n]
Encode:     e[n] (small values compress well)
Decode:     x[n] = p[n] + e[n]
```

---

## Types of Prediction

### Intra Prediction (Within Frame)

Predict from neighbors in same frame:
```
Image:
    ┌───┬───┐
    │ A │ B │
    ├───┼───┤
    │ C │ X │ ← Predict X from A, B, C
    └───┴───┘

X̂ = (A + B + C) / 3
```

### Inter Prediction (Between Frames)

Predict from previous frames:
```
Frame t-1:  ████████
            ████████
                 ↓ motion vector
Frame t:    ████████
              ████████ (shifted)
```

---

## DPCM (Differential PCM)

Simplest prediction: previous sample.

```
p[n] = x[n-1]
e[n] = x[n] - x[n-1]  (delta encoding)
```

Used in:
- Simple audio
- PNG filter mode "Sub"

---

## Linear Prediction (LPC)

Weighted sum of previous samples:

```
p[n] = a₁x[n-1] + a₂x[n-2] + ... + aₘx[n-m]
```

### Finding Coefficients

Minimize mean squared error:
```
MSE = E[(x[n] - p[n])²]
```

Use autocorrelation and Levinson-Durbin algorithm.

### Applications

- FLAC audio compression
- Speech coding (CELP, LPC-10)
- Seismic data

---

## 2D Prediction (Images)

### Simple Predictors

```
Horizontal: X̂ = A (left neighbor)
Vertical:   X̂ = B (above neighbor)
Diagonal:   X̂ = C (above-left)
DC:         X̂ = (A + B + C + D) / 4
```

### H.264 Intra Prediction

9 modes for 4×4 blocks:
```
Mode 0: Vertical
Mode 1: Horizontal
Mode 2: DC
Mode 3: Diagonal down-left
Mode 4: Diagonal down-right
Mode 5: Vertical-right
Mode 6: Horizontal-down
Mode 7: Vertical-left
Mode 8: Horizontal-up
```

Choose mode with smallest residual.

---

## Motion Compensation

### Block Matching

Find where current block appeared in reference:

```
Reference frame:
┌─────────────────────┐
│                     │
│    ┌───┐            │
│    │ A │ ← Best match here
│    └───┘            │
│                     │
└─────────────────────┘

Current frame:
┌─────────────────────┐
│                     │
│          ┌───┐      │
│          │ A │ ← Block to encode
│          └───┘      │
│                     │
└─────────────────────┘

Motion vector: (dx, dy)
Residual: Current - Predicted
```

### Sub-Pixel Motion

Quarter-pixel precision for better matching:
```
Interpolate between pixels
Motion vector: (3.25, -1.75)
```

---

## Prediction in Video Codecs

### I-Frame (Intra)

Only intra prediction (within frame).
- Largest frames
- Random access points

### P-Frame (Predicted)

Predict from previous frame(s).
- Smaller than I-frames
- Require previous frame

### B-Frame (Bidirectional)

Predict from both past and future frames.
- Smallest frames
- Require multiple references

---

## Adaptive Prediction

### Choose Best Predictor

Try all prediction modes, pick best:
```
for mode in prediction_modes:
    residual = actual - predict(mode)
    cost = distortion(residual) + λ × bits(mode)
    if cost < best_cost:
        best_mode = mode
```

### Signal Mode to Decoder

Encode mode index (e.g., 4 bits for 16 modes).

---

## Key Takeaways

1. Encode prediction error, not original values
2. Good prediction → small residuals → better compression
3. Intra: predict from same frame
4. Inter: predict from other frames (motion compensation)
5. Linear prediction for audio/speech
6. Modern codecs use many prediction modes

---

**Next Chapter**: [Vector Quantization](./09_vector_quantization.md)
