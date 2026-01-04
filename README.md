# Compression Learning Project

A comprehensive guide to learn data compression from beginner to master level.

## Overview

This project contains learning materials covering all compression techniques, from basic concepts to state-of-the-art algorithms.

## Structure

```
├── learning/           # Complete learning materials (book format)
│   ├── a_foundations/  # Theory: entropy, information, redundancy
│   ├── b_lossless/     # Lossless: RLE, Huffman, LZ77, DEFLATE, etc.
│   ├── c_lossy_fundamentals/  # Lossy: DCT, wavelets, quantization
│   ├── d_image/        # Image: PNG, JPEG, WebP, AVIF
│   ├── e_audio/        # Audio: FLAC, MP3, AAC, Opus
│   ├── f_video/        # Video: H.264, H.265, AV1
│   ├── g_specialized/  # 3D, time series, databases
│   └── h_exercises/    # Implementation projects
├── sylabus             # Complete syllabus with progress tracking
└── src/                # Rust implementations (practice)
```

## Getting Started

1. Read the [syllabus](./sylabus) for the complete learning path
2. Start with [Part A: Foundations](./learning/a_foundations/README.md)
3. Implement algorithms as you learn in `src/`

## Topics Covered

### Lossless Compression
- Run-Length Encoding (RLE)
- Huffman, Arithmetic, ANS coding
- LZ77, LZ78, LZW, LZMA
- DEFLATE, bzip2, Zstandard, Brotli
- Context modeling, PPM

### Lossy Compression
- Quantization and rate-distortion
- DCT and Wavelet transforms
- Psychoacoustic/psychovisual models
- JPEG, MP3, H.264/H.265

## Building

```bash
cargo build
cargo run
```

## Resources

- [Data Compression Explained](http://mattmahoney.net/dc/dce.html) - Matt Mahoney
- [RFC 1951](https://tools.ietf.org/html/rfc1951) - DEFLATE specification
- [RFC 8478](https://tools.ietf.org/html/rfc8478) - Zstandard specification

## License

Educational use.
