# Chapter 4: JPEG

## Overview

**JPEG** (Joint Photographic Experts Group) is the most used lossy image format.

Created in 1992. Excellent for photographs.

---

## How JPEG Works

```
RGB → YCbCr → Subsampling → 8×8 DCT → Quantization → Entropy Coding
```

---

## Step 1: Color Conversion

```
Y  = 0.299R + 0.587G + 0.114B  (luminance)
Cb = 0.564(B - Y)               (blue chrominance)
Cr = 0.713(R - Y)               (red chrominance)
```

---

## Step 2: Chroma Subsampling

Reduce chrominance resolution:

```
4:4:4 - Full resolution (no subsampling)
4:2:2 - Half horizontal
4:2:0 - Quarter (half each dimension) ← Most common
```

---

## Step 3: Block DCT

Divide into 8×8 blocks, apply DCT:

```
┌────────┬────────┐
│ 8×8    │ 8×8    │
│ block  │ block  │
├────────┼────────┤
│ 8×8    │ 8×8    │
│ block  │ block  │
└────────┴────────┘

Each block → 64 DCT coefficients
```

---

## Step 4: Quantization

Divide by quantization matrix, round:

```
Quantized[i][j] = round(DCT[i][j] / Q[i][j])

Q matrix has larger values for high frequencies
High frequencies → more zeros
```

### Quality Setting

```
Quality 100: Near-lossless (large file)
Quality 85:  High quality (balanced)
Quality 75:  Good quality (smaller)
Quality 50:  Medium quality
Quality 25:  Low quality (tiny file)
```

---

## Step 5: Entropy Coding

### Zigzag Scan

Read coefficients in zigzag order:

```
→ ↘ → ↘ → ...
↙   ↙
```

Groups zeros together.

### DC Coefficients

Encode difference from previous block:
```
DC_diff = DC_current - DC_previous
Huffman encode (category, bits)
```

### AC Coefficients

Run-length + Huffman:
```
(run_of_zeros, value_category)
```

---

## JPEG Artifacts

### Blocking

Visible 8×8 block boundaries at low quality.

### Ringing

Halos around sharp edges.

### Color Bleeding

Chrominance bleeds across edges.

---

## JPEG Modes

| Mode | Description |
|------|-------------|
| Baseline | Sequential, 8-bit, Huffman |
| Progressive | Multiple passes (coarse → fine) |
| Lossless | No DCT, prediction only (rare) |
| Arithmetic | Better compression (patent issues) |

---

## File Structure

```
FFD8 - Start of Image (SOI)
FFE0 - APP0 (JFIF marker)
FFDB - Define Quantization Table(s)
FFC0 - Start of Frame (image dimensions)
FFC4 - Define Huffman Table(s)
FFDA - Start of Scan (compressed data)
FFD9 - End of Image (EOI)
```

---

## Key Takeaways

1. JPEG = YCbCr + DCT + Quantization + Huffman
2. Quality setting controls quantization
3. 4:2:0 subsampling is standard
4. Best for photographs
5. Poor for graphics (use PNG)
6. Blocking artifacts at low quality

---

**Next Chapter**: [JPEG 2000](./05_jpeg2000.md)
