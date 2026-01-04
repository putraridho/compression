# Chapter 1: PNG

## Overview

**PNG** (Portable Network Graphics) is the most popular lossless image format.

Created in 1996 to replace GIF (due to LZW patents).

---

## How PNG Works

```
Pixels → Filtering → DEFLATE → PNG file
```

1. **Filtering**: Predict each pixel, store difference
2. **DEFLATE**: LZ77 + Huffman compression

---

## Color Types

| Type | Description | Channels |
|------|-------------|----------|
| 0 | Grayscale | 1 |
| 2 | RGB | 3 |
| 3 | Indexed (palette) | 1 |
| 4 | Grayscale + Alpha | 2 |
| 6 | RGBA | 4 |

Bit depths: 1, 2, 4, 8, or 16 bits per channel.

---

## PNG Filters

Each scanline can use a different filter:

### Filter Types

```
0 - None:    Store pixel as-is
1 - Sub:     X - A (left neighbor)
2 - Up:      X - B (above neighbor)
3 - Average: X - (A + B) / 2
4 - Paeth:   X - Paeth(A, B, C)

    C | B
    -----
    A | X
```

### Paeth Predictor

```
p = A + B - C
pa = |p - A|
pb = |p - B|
pc = |p - C|

if pa ≤ pb and pa ≤ pc: return A
else if pb ≤ pc: return B
else: return C
```

---

## File Structure

```
PNG Signature: 89 50 4E 47 0D 0A 1A 0A

Chunks:
┌──────────────────────────────────────┐
│ IHDR - Image header (required first) │
├──────────────────────────────────────┤
│ PLTE - Palette (if indexed)          │
├──────────────────────────────────────┤
│ IDAT - Image data (one or more)      │
├──────────────────────────────────────┤
│ IEND - End marker (required last)    │
└──────────────────────────────────────┘
```

Each chunk: Length (4) + Type (4) + Data + CRC (4)

---

## IHDR Chunk

```
Width:             4 bytes
Height:            4 bytes
Bit depth:         1 byte
Color type:        1 byte
Compression:       1 byte (always 0 = DEFLATE)
Filter method:     1 byte (always 0)
Interlace:         1 byte (0 = none, 1 = Adam7)
```

---

## Adam7 Interlacing

Progressive loading in 7 passes:

```
Pass 1: 1 6 4 6 2 6 4 6
Pass 2: 7 7 7 7 7 7 7 7
Pass 3: 5 6 5 6 5 6 5 6
Pass 4: 7 7 7 7 7 7 7 7
Pass 5: 3 6 4 6 3 6 4 6
Pass 6: 7 7 7 7 7 7 7 7
Pass 7: 5 6 5 6 5 6 5 6
```

Early passes: coarse image
Later passes: fill in details

---

## Optimization Tips

1. **Choose right color type**: Grayscale < RGB < RGBA
2. **Reduce colors**: Use indexed for ≤256 colors
3. **Filter selection**: Different filters for different content
4. **Tools**: pngcrush, optipng, zopflipng

---

## Key Takeaways

1. PNG = Filters + DEFLATE
2. Five filter types for prediction
3. Lossless—perfect quality
4. Good for graphics, screenshots, diagrams
5. Poor for photographs (use JPEG)

---

**Next Chapter**: [GIF and LZW](./02_gif.md)
