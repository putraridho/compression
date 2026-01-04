# Chapter 4: Redundancy

## What is Redundancy?

**Redundancy** is the portion of data that can be removed without losing information.

Compression works by identifying and removing redundancy.

---

## Types of Redundancy

### 1. Statistical Redundancy

Some symbols appear more often than others.

**Example**: In English text
- 'e' appears ~13% of the time
- 'z' appears ~0.07% of the time

If we use 8 bits for every character (like ASCII), we waste bits on common letters. Giving 'e' a shorter code exploits this redundancy.

**Exploited by**: Huffman coding, arithmetic coding, ANS

### 2. Spatial Redundancy

Nearby values are often similar.

**Example**: In images
- A blue sky has many similar blue pixels next to each other
- Skin tones vary gradually, not randomly

**Exploited by**: Delta encoding, prediction, transforms (DCT, wavelets)

### 3. Temporal Redundancy

Values close in time are often similar.

**Example**: In video
- Frame 2 is usually very similar to Frame 1
- Only small parts of the image change between frames

**Exploited by**: Motion compensation, inter-frame prediction

### 4. Spectral Redundancy

Frequency components are correlated.

**Example**: In audio
- Low frequencies (bass) change slowly
- Harmonic relationships between frequencies

**Exploited by**: Transform coding (DCT, FFT)

### 5. Psychovisual/Psychoacoustic Redundancy

Humans cannot perceive all details.

**Example**:
- We don't notice small color changes
- We can't hear very quiet sounds masked by loud ones
- We don't see high-frequency details as well

**Exploited by**: Lossy compression (JPEG, MP3)

---

## Measuring Redundancy

### Absolute Redundancy

```
R = H_max - H
```

Where:
- H_max = maximum possible entropy = log₂(n) for n symbols
- H = actual entropy of the source

### Relative Redundancy

```
r = 1 - (H / H_max) = R / H_max
```

This gives a percentage.

### Example: English Text

- Alphabet: 27 symbols (26 letters + space)
- H_max = log₂(27) ≈ 4.76 bits
- Actual entropy ≈ 1.5 bits (with context)
- Redundancy = 4.76 - 1.5 = 3.26 bits
- Relative redundancy = 3.26/4.76 ≈ 68%

English text is about **68% redundant**.

---

## How Compression Removes Redundancy

### Statistical Redundancy → Entropy Coding

| Technique | How it works |
|-----------|--------------|
| Huffman coding | Short codes for common symbols |
| Arithmetic coding | Fractional bits per symbol |
| ANS | Modern high-speed entropy coding |

### Spatial Redundancy → Prediction + Transforms

| Technique | How it works |
|-----------|--------------|
| Delta encoding | Store differences, not absolute values |
| LZ77/LZ78 | Reference previous occurrences |
| DCT (images) | Convert spatial patterns to frequencies |
| Wavelets | Multi-resolution analysis |

### Temporal Redundancy → Inter-frame Coding

| Technique | How it works |
|-----------|--------------|
| Motion vectors | Describe how blocks moved |
| P-frames | Predicted from previous frame |
| B-frames | Predicted from both directions |

### Perceptual Redundancy → Lossy Techniques

| Technique | How it works |
|-----------|--------------|
| Quantization | Reduce precision of values |
| Subsampling | Reduce resolution of color |
| Masking models | Discard inaudible/invisible details |

---

## The Compression Pipeline

Most compression systems follow this pattern:

```
Input
  │
  ▼
┌─────────────────┐
│  Decorrelation  │  ← Remove spatial/temporal redundancy
│  (Transforms)   │     (DCT, DWT, prediction, LZ)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Quantization   │  ← Remove perceptual redundancy (lossy only)
│  (Lossy step)   │     (reduce precision)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Entropy Coding  │  ← Remove statistical redundancy
│                 │     (Huffman, arithmetic, ANS)
└────────┬────────┘
         │
         ▼
     Output
```

Each stage targets a different type of redundancy.

---

## Redundancy in Different Data Types

### Text
- **High statistical redundancy**: 'e' is common, 'z' is rare
- **High sequential redundancy**: "th" is usually followed by 'e' or 'a'
- **Word patterns**: Limited vocabulary, repeated words

Typical compression: 2-4x with gzip, 3-6x with better algorithms

### Images
- **High spatial redundancy**: Neighboring pixels are similar
- **Color correlation**: R, G, B channels are correlated
- **Perceptual redundancy**: Fine details often invisible

Typical compression: 10-50x lossy (JPEG), 2-3x lossless (PNG)

### Audio
- **Temporal redundancy**: Waveform is continuous
- **Spectral redundancy**: Harmonic structure
- **Perceptual redundancy**: Masking effects

Typical compression: 10x lossy (MP3), 2x lossless (FLAC)

### Video
- **Extreme temporal redundancy**: Frames are mostly identical
- **All image redundancies**: Apply to each frame
- **Motion patterns**: Objects move predictably

Typical compression: 100-1000x possible

### Random/Encrypted Data
- **No redundancy**: By definition
- **Cannot be compressed**: Already at maximum entropy

---

## Why Can't We Compress Everything?

### The Pigeonhole Principle

If you have 256 possible 1-byte inputs:
- Any compression that makes some files smaller
- Must make other files larger
- You can't map 256 values to fewer than 256 values uniquely

### Incompressible Data

1. **Random data**: No patterns to exploit
2. **Already compressed**: ZIP, JPEG, MP3 files
3. **Encrypted data**: Designed to look random
4. **Maximally dense data**: Every bit carries information

### Practical Implication

Never compress:
- Already compressed archives
- Encrypted files
- Random test data

Doing so wastes CPU and may increase size.

---

## Self-Information and Redundancy

For a single symbol:

```
Self-information: I(x) = -log₂(p(x))
Code length in fixed encoding: log₂(n)
Redundancy for that symbol: log₂(n) - (-log₂(p(x)))
```

Common symbols have high redundancy in fixed-length codes.
Rare symbols might have negative redundancy (need more than average bits).

---

## Key Takeaways

1. **Redundancy = compressibility**
   - More redundancy → better compression possible

2. **Five types of redundancy**:
   - Statistical (symbol frequency)
   - Spatial (nearby similarity)
   - Temporal (time-based similarity)
   - Spectral (frequency correlation)
   - Perceptual (human limitations)

3. **Different techniques target different redundancy**:
   - Entropy coding → statistical
   - Transforms/prediction → spatial/temporal
   - Quantization → perceptual

4. **No redundancy = incompressible**
   - Random and encrypted data cannot be compressed

---

**Next Chapter**: [Theoretical Limits](./05_theoretical_limits.md) - Shannon's theorems and what they mean for compression
