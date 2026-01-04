# Compression Learning Guide

A comprehensive book to learn compression from beginner to master.

---

## How to Use This Guide

1. **Read in order**: Each part builds on previous knowledge
2. **Understand before moving on**: Don't skip—compression concepts compound
3. **Implement as you go**: Learning by doing is essential
4. **Use the syllabus**: Track your progress in `/sylabus`

---

## Parts

### [Part A: Foundations](./a_foundations/README.md)
Theory and mathematics of compression.
- What is compression?
- Information theory
- Entropy
- Redundancy
- Theoretical limits

### [Part B: Lossless Techniques](./b_lossless/README.md)
All lossless compression algorithms.
- Basic encoding (RLE, Delta, Bit packing)
- Entropy coding (Huffman, Arithmetic, ANS)
- Dictionary methods (LZ77, LZ78, LZW, LZMA)
- Transforms (BWT, MTF)
- Hybrid algorithms (DEFLATE, bzip2, Zstandard, Brotli)
- Context modeling (PPM, Context mixing)

### [Part C: Lossy Fundamentals](./c_lossy_fundamentals/README.md)
Foundations of lossy compression.
- Quantization
- Rate-distortion theory
- DCT and Wavelets
- Psychoacoustic/Psychovisual modeling
- Predictive coding
- Vector quantization

### [Part D: Image Compression](./d_image/README.md)
Image-specific formats.
- PNG (lossless)
- JPEG (lossy)
- Modern formats (WebP, HEIC, AVIF)

### [Part E: Audio Compression](./e_audio/README.md)
Audio-specific formats.
- FLAC (lossless)
- MP3, AAC, Opus (lossy)
- Speech codecs

### [Part F: Video Compression](./f_video/README.md)
Video-specific compression.
- Frame types (I/P/B)
- Motion compensation
- H.264, H.265, AV1

### [Part G: Specialized Compression](./g_specialized/README.md)
Domain-specific techniques.
- 3D geometry
- Time series
- Databases
- Neural compression

### [Part H: Implementation Guide](./h_exercises/README.md)
Hands-on implementation exercises.
- Beginner projects
- Intermediate challenges
- Advanced implementations

---

## Recommended Path

| Week | Focus |
|------|-------|
| 1-2 | Part A (Theory) + RLE implementation |
| 3-4 | Part B chapters 1-10 (Basic + Entropy) |
| 5-6 | Part B chapters 11-20 (Dictionary) |
| 7-8 | Part B chapters 21-32 (Advanced) |
| 9-10 | Part C (Lossy fundamentals) |
| 11-12 | Part D (Images) |
| 13-14 | Part E (Audio) |
| 15-16 | Part F (Video) |
| 17+ | Part G + H (Specialize) |

---

## Quick Reference

### Compression Ratios

| Technique | Typical Ratio |
|-----------|---------------|
| gzip | 2-3x |
| bzip2 | 3-4x |
| Zstandard | 2.5-4x |
| JPEG (quality 85) | 10-20x |
| MP3 (192 kbps) | 10x |
| H.264 (1080p) | 50-100x |

### When to Use What

| Data Type | Recommended |
|-----------|-------------|
| Text, code | Zstandard, brotli |
| Archives | Zstandard, 7z |
| Photos | JPEG, WebP |
| Graphics | PNG, WebP lossless |
| Music | FLAC (lossless), Opus (lossy) |
| Video | H.265, AV1 |

---

## Resources

### Books
- "Data Compression Explained" by Matt Mahoney
- "Introduction to Data Compression" by Khalid Sayood

### Specifications
- RFC 1951 (DEFLATE)
- RFC 7932 (Brotli)
- RFC 8478 (Zstandard)

### Tools
- Squash Benchmark
- lzbench

---

Good luck on your compression journey!
