# Chapter 1: FLAC

## Overview

**FLAC** (Free Lossless Audio Codec) is the most popular lossless audio format.

Created in 2001. Typically achieves 50-60% compression.

---

## How FLAC Works

```
Audio → Blocking → Interchannel Decorrelation → LPC Prediction → Rice Coding
```

---

## Step 1: Blocking

Divide audio into frames (1024-4096 samples).

```
Frame structure:
┌──────────────────────────────────────┐
│ Frame header                         │
├──────────────────────────────────────┤
│ Subframe (channel 1)                 │
├──────────────────────────────────────┤
│ Subframe (channel 2)                 │
├──────────────────────────────────────┤
│ CRC                                  │
└──────────────────────────────────────┘
```

---

## Step 2: Interchannel Decorrelation

For stereo, choose best representation:

```
Independent: L, R
Mid-Side:    M = (L+R)/2, S = (L-R)/2
Left-Side:   L, S = L - R
Right-Side:  S = L - R, R
```

Mid-Side often compresses better for correlated channels.

---

## Step 3: Linear Prediction

Predict sample from previous samples:

```
x̂[n] = a₁x[n-1] + a₂x[n-2] + ... + aₘx[n-m]
residual[n] = x[n] - x̂[n]
```

Order typically 0-12. Higher order = better prediction, more overhead.

---

## Step 4: Rice Coding

Encode residuals with Rice codes (optimal for geometric distribution):

```
Residual → Sign + (quotient in unary) + (remainder in k bits)
```

k is chosen per block to minimize size.

---

## Subframe Types

| Type | Description | Use Case |
|------|-------------|----------|
| Constant | All samples identical | Silence |
| Verbatim | Raw samples | Noise |
| Fixed | Fixed prediction coefficients | Quick |
| LPC | Optimal coefficients | Best compression |

---

## Compression Levels

```
Level 0: Fastest, ~50% ratio
Level 5: Default, ~55% ratio
Level 8: Slowest, ~58% ratio
```

Higher levels try more prediction orders.

---

## Key Takeaways

1. FLAC = LPC prediction + Rice coding
2. Perfect reconstruction (lossless)
3. 50-60% typical compression
4. Widely supported in players
5. Good for music archival

---

**Next Chapter**: [Other Lossless Audio](./02_lossless_audio.md)
