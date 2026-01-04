# Chapter 1: Video Compression Fundamentals

## Why Video Needs Special Compression

Raw video is enormous:

```
1080p @ 30fps, 24-bit color:
= 1920 × 1080 × 3 bytes × 30 fps
= 186 MB/s = 11 GB/min!
```

Compression achieves 100-1000x reduction.

---

## Frame Types

### I-Frame (Intra)

Encoded independently (like JPEG).
- Largest frames
- Random access points
- Scene changes

### P-Frame (Predicted)

Predicted from previous frame(s).
- Motion compensation
- Smaller than I-frames

### B-Frame (Bidirectional)

Predicted from past AND future.
- Smallest frames
- Best compression
- Adds latency

```
Display order: I  B  B  P  B  B  P  B  B  I
Decode order:  I  P  B  B  P  B  B  I  B  B
```

---

## GOP (Group of Pictures)

Structure of frame types:

```
GOP = I B B P B B P B B I B B P ...
      ├──────── GOP ────────┤

GOP length: 15-30 frames typical
More B-frames = better compression, more latency
```

---

## Motion Estimation

Find where blocks moved:

```
Reference frame:
┌─────────────────────┐
│    ┌───┐            │
│    │ A │            │  Block A in reference
│    └───┘            │
└─────────────────────┘

Current frame:
┌─────────────────────┐
│          ┌───┐      │
│          │ A │      │  Block moved here
│          └───┘      │
└─────────────────────┘

Motion vector: (dx, dy)
Store: MV + residual (difference)
```

### Search Algorithms

- Full search: Check all positions (slow, optimal)
- Diamond search: Pattern-based (fast, good)
- Hexagon search: 6-point pattern

---

## Motion Compensation

Apply motion vectors to predict frame:

```
Predicted[x,y] = Reference[x + mvx, y + mvy]
Residual = Current - Predicted
```

### Sub-Pixel Motion

Quarter-pixel precision:
```
Interpolate reference at fractional positions
Motion vector: (3.25, -1.75)
```

---

## Block Sizes

Modern codecs use variable blocks:

```
H.264: 16×16, 16×8, 8×16, 8×8, 4×4
H.265: 64×64 down to 4×4 (CTU → CU → TU)
AV1:   128×128 down to 4×4
```

Larger blocks: Less overhead
Smaller blocks: Better motion accuracy

---

## Intra Prediction

Predict from neighbors in same frame:

```
    A B C D E
  ┌─────────
F │ ? ? ? ?
G │ ? ? ? ?
H │ ? ? ? ?
I │ ? ? ? ?

Predict ? from A-I using various modes
```

H.264: 9 modes for 4×4
H.265: 35 modes
AV1: 65+ modes

---

## Transform and Quantization

Same as image compression:

```
Residual → DCT → Quantize → Entropy code
```

Quantization Parameter (QP) controls quality.

---

## Entropy Coding

### H.264

- CAVLC (simpler, faster)
- CABAC (better compression)

### H.265, AV1

- CABAC variants
- Context-adaptive binary coding

---

## Typical Compression Ratios

| Codec | Bitrate (1080p30) | Ratio |
|-------|-------------------|-------|
| H.264 | 5-8 Mbps | ~25x |
| H.265 | 3-5 Mbps | ~40x |
| AV1 | 2-4 Mbps | ~50x |

---

## Key Takeaways

1. I/P/B frames exploit temporal redundancy
2. Motion compensation is key
3. Variable block sizes optimize efficiency
4. Each generation is ~30-50% better
5. Trade-off: compression vs encoding time

---

**Next Chapter**: [H.264 / AVC](./02_h264.md)
