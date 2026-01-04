# Chapter 1: What is Compression?

## Definition

**Compression** is the process of representing data using fewer bits than the original representation.

Think of compression like summarizing a book. Instead of carrying a 500-page novel, you carry a 10-page summary. The summary captures the essential information in less space.

---

## Why Compress?

1. **Storage**: Store more files in the same disk space
2. **Transmission**: Send data faster over networks
3. **Cost**: Less storage and bandwidth means lower costs
4. **Performance**: Smaller data loads faster

---

## Two Types of Compression

### Lossless Compression

The original data can be **perfectly reconstructed** from the compressed data.

```
Original → Compress → Decompress → Identical to Original
```

**Use when**: Every bit matters
- Source code
- Documents
- Executables
- Medical images
- Financial data

**Examples**: ZIP, PNG, FLAC, gzip

### Lossy Compression

Some information is **permanently discarded**. You cannot recover the original exactly.

```
Original → Compress → Decompress → Similar to Original (not identical)
```

**Use when**: Human perception matters more than exact data
- Photos (JPEG)
- Music (MP3)
- Video (H.264)
- Voice calls

**Key insight**: Lossy compression exploits the fact that humans cannot perceive every detail. We discard what you won't notice.

---

## Measuring Compression

### Compression Ratio

How many times smaller is the compressed data?

```
Compression Ratio = Original Size / Compressed Size
```

**Example**:
- Original: 1000 bytes
- Compressed: 250 bytes
- Ratio: 1000 / 250 = **4:1** (or "4x compression")

Higher ratio = better compression.

### Space Savings

What percentage of space did we save?

```
Space Savings = 1 - (Compressed Size / Original Size)
```

**Example**:
- Original: 1000 bytes
- Compressed: 250 bytes
- Savings: 1 - (250/1000) = 0.75 = **75% savings**

### Bitrate (for media)

Bits used per unit of content.

- **Images**: bits per pixel (bpp)
- **Audio**: bits per second (kbps)
- **Video**: bits per second (Mbps)

---

## The Fundamental Trade-off

Compression always involves trade-offs:

| Factor | Trade-off |
|--------|-----------|
| **Compression ratio** vs **Speed** | Better compression takes more time |
| **Compression ratio** vs **Quality** | (Lossy) Higher compression = lower quality |
| **Compression speed** vs **Decompression speed** | Some algorithms compress slowly but decompress fast |
| **Memory usage** vs **Compression ratio** | Better compression often needs more RAM |

**No free lunch**: You cannot have maximum compression, maximum speed, and minimum memory simultaneously.

---

## What Makes Data Compressible?

Data compresses well when it has **patterns** or **redundancy**.

### Highly Compressible
- Text (many repeated words/letters)
- Images with solid colors
- Databases with repeated values
- Source code (keywords repeat)

### Poorly Compressible
- Random data (no patterns)
- Already compressed data (ZIP, JPEG)
- Encrypted data (looks random by design)

### The Pigeonhole Principle

**You cannot losslessly compress all possible files.**

Proof: If you have 256 possible 1-byte files, and compression always made files smaller, you'd need to fit 256 different values into fewer than 256 slots. Impossible.

This means:
- Some files will get larger when "compressed"
- Random data cannot be compressed
- Already compressed data cannot be compressed further

---

## Compression Pipeline

Most compression systems follow this pattern:

```
┌─────────────────────────────────────────────────────────────────────┐
│                    COMPRESSION                                      │
├─────────────────────────────────────────────────────────────────────┤
│  Input → [Transform] → [Model] → [Entropy Coder] → Output           │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                   DECOMPRESSION                                     │
├─────────────────────────────────────────────────────────────────────┤
│  Input → [Entropy Decoder] → [Model] → [Inverse Transform] → Output │
└─────────────────────────────────────────────────────────────────────┘
```

**Transform**: Reorganize data to expose patterns (e.g., BWT, DCT)
**Model**: Predict what comes next based on context
**Entropy Coder**: Assign short codes to common symbols

You'll learn each of these components in detail.

---

## Quick Intuition Check

Before moving on, make sure you understand:

1. **Lossless** = perfect reconstruction; **Lossy** = approximate
2. Compression ratio = original size / compressed size
3. Random data cannot be compressed
4. All compression exploits **redundancy** or **patterns**
5. There are always trade-offs (speed vs ratio vs memory)

---

## Key Terms

| Term | Definition |
|------|------------|
| **Compression** | Encoding data in fewer bits |
| **Decompression** | Reconstructing original data from compressed form |
| **Lossless** | Perfect reconstruction possible |
| **Lossy** | Some information permanently lost |
| **Compression ratio** | Original size / Compressed size |
| **Codec** | Coder-decoder pair (compression + decompression algorithm) |
| **Bitrate** | Bits used per unit of content |

---

**Next Chapter**: [Information Theory](./02_information_theory.md) - The mathematical foundation of compression
