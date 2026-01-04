# Chapter 1: Introduction to Lossy Compression

## What is Lossy Compression?

**Lossy compression** permanently discards some information to achieve smaller file sizes.

The original data cannot be perfectly reconstructed.

---

## Why Accept Data Loss?

### Human Perception is Limited

- We can't hear frequencies above 20kHz
- We can't see very fine details at distance
- We don't notice tiny color changes
- Loud sounds mask quiet sounds

### Huge Compression Potential

| Type | Lossless Ratio | Lossy Ratio |
|------|---------------|-------------|
| Image | 2x | 10-50x |
| Audio | 2x | 10-15x |
| Video | 2x | 100-1000x |

Lossy achieves 10-500x better ratios!

---

## Lossless vs Lossy

### Lossless

```
Original → Compress → Decompress → Original
                                   (identical)
```

Use when every bit matters:
- Documents, source code
- Medical images, scientific data
- Archives, backups

### Lossy

```
Original → Compress → Decompress → Approximation
                                   (similar, not identical)
```

Use when perception matters:
- Photos, videos
- Music, podcasts
- Streaming media

---

## Lossy Compression Pipeline

```
┌────────────────────────────────────────────────────────┐
│                     ENCODER                            │
├────────────────────────────────────────────────────────┤
│                                                        │
│  Input → Transform → Quantization → Entropy Coding → Output
│             │              │                           │
│             ▼              ▼                           │
│         (DCT, etc.)   (data loss here!)               │
│                                                        │
└────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────┐
│                     DECODER                            │
├────────────────────────────────────────────────────────┤
│                                                        │
│  Input → Entropy Decoding → Dequantization → Inverse Transform → Output
│                                                        │
└────────────────────────────────────────────────────────┘
```

### Key Stages

1. **Transform**: Convert to frequency/perceptual domain
2. **Quantization**: Reduce precision (the lossy step)
3. **Entropy Coding**: Compress the quantized values

---

## Types of Distortion

### Mean Squared Error (MSE)

```
MSE = (1/n) × Σ(original[i] - reconstructed[i])²
```

Average squared difference.

### Peak Signal-to-Noise Ratio (PSNR)

```
PSNR = 10 × log₁₀(MAX² / MSE) dB
```

Logarithmic scale. Higher = better.

| PSNR | Quality |
|------|---------|
| 30 dB | Poor |
| 35 dB | Fair |
| 40 dB | Good |
| 45 dB | Very good |
| 50+ dB | Excellent |

### Perceptual Metrics

PSNR doesn't match human perception. Better metrics:

- **SSIM**: Structural Similarity Index
- **VIF**: Visual Information Fidelity
- **VMAF**: Video Multi-method Assessment Fusion (Netflix)

---

## The Rate-Distortion Trade-off

```
                     Quality
                         ^
                         │
    Lossless ──────────●─┤
                      /   │
                    /     │
                  /       │
                /         │
              /           │
            /             │
          /               │
────────●─────────────────┼────────> Bitrate
     Highly lossy        │
```

- Higher bitrate = higher quality
- Lower bitrate = more distortion
- Lossless = maximum bitrate

---

## Transparent Compression

"Transparent" = perceptually lossless.

The compression introduces changes, but humans can't detect them.

### Examples

| Format | Transparent Bitrate |
|--------|-------------------|
| MP3 | ~192 kbps |
| AAC | ~128 kbps |
| JPEG | Quality 85-95 |
| H.264 | CRF 17-18 |

---

## Key Techniques

### Transform Coding

Convert spatial/temporal data to frequency domain:
- DCT (JPEG, MPEG)
- Wavelet (JPEG 2000)
- MDCT (MP3, AAC)

### Quantization

Reduce precision of transform coefficients:
- Scalar quantization
- Vector quantization

### Perceptual Modeling

Allocate bits based on human perception:
- Psychoacoustic model (audio)
- Psychovisual model (video)

### Prediction

Predict values from neighbors, encode difference:
- Intra prediction (within frame)
- Inter prediction (between frames)

---

## Format Examples

### Images

| Format | Transform | Quantization |
|--------|-----------|--------------|
| JPEG | 8×8 DCT | Scalar |
| JPEG 2000 | Wavelet | Scalar + Bit-plane |
| HEIC | DCT (HEVC) | Scalar |

### Audio

| Format | Transform | Model |
|--------|-----------|-------|
| MP3 | Filterbank + MDCT | Psychoacoustic |
| AAC | MDCT | Psychoacoustic |
| Opus | CELT | Psychoacoustic |

### Video

| Format | Spatial | Temporal |
|--------|---------|----------|
| H.264 | DCT | Motion compensation |
| H.265 | DCT/DST | Motion compensation |
| AV1 | DCT | Motion compensation |

---

## Quality Control

### Constant Quality (CQ/CRF)

Fixed quality, variable bitrate:
```
ffmpeg -crf 23 output.mp4
```

### Constant Bitrate (CBR)

Fixed bitrate, variable quality:
```
ffmpeg -b:v 2M output.mp4
```

### Variable Bitrate (VBR)

Target average with room for variation:
```
ffmpeg -b:v 2M -maxrate 3M output.mp4
```

---

## Key Takeaways

1. Lossy = permanent information loss
2. Enables 10-1000x compression
3. Exploits human perception limits
4. Transform → Quantize → Entropy code
5. Trade-off: quality vs file size
6. PSNR measures objective quality, SSIM/VMAF measure perceived quality

---

**Next Chapter**: [Quantization](./02_quantization.md)
