# Part H: Implementation Guide

This part guides you through implementing compression algorithms.

## Learning Path

### Beginner (Start Here)
1. [RLE Implementation](./01_rle_impl.md)
2. [Delta Encoding](./02_delta_impl.md)
3. [Bit Packing](./03_bitpack_impl.md)

### Intermediate
4. [Huffman Coding](./04_huffman_impl.md)
5. [LZ77 Implementation](./05_lz77_impl.md)
6. [LZSS with Lazy Matching](./06_lzss_impl.md)

### Advanced
7. [Arithmetic Coding](./07_arithmetic_impl.md)
8. [ANS Implementation](./08_ans_impl.md)
9. [BWT + MTF](./09_bwt_impl.md)

### Expert
10. [Mini-DEFLATE](./10_deflate_impl.md)
11. [Simple JPEG Decoder](./11_jpeg_impl.md)

---

## Implementation Tips

### General Advice

1. **Start simple**: Get a working version first
2. **Test extensively**: Edge cases matter
3. **Benchmark**: Measure compression ratio AND speed
4. **Compare**: Test against existing implementations

### Testing Strategies

```
1. Round-trip test: encode → decode → compare
2. Edge cases: empty, single byte, repeated patterns
3. Random data: should not crash, may expand
4. Real files: text, binary, images
```

### Debugging Tips

```
1. Print intermediate values
2. Visualize: bit patterns, trees, buffers
3. Compare byte-by-byte with reference
4. Use small test cases
```

---

## Project Ideas

### Level 1
- [ ] RLE compressor with statistics
- [ ] Huffman tree visualizer
- [ ] LZ77 with configurable window

### Level 2
- [ ] Full DEFLATE decompressor
- [ ] gzip/zip file parser
- [ ] Simple PNG decoder

### Level 3
- [ ] JPEG decoder with DCT
- [ ] LZ4-compatible compressor
- [ ] Brotli decompressor

### Level 4
- [ ] Full zstd implementation
- [ ] Video frame decoder (I-frames)
- [ ] Context mixing compressor

---

## Resources

### Books
- "Data Compression Explained" - Matt Mahoney
- "Introduction to Data Compression" - Khalid Sayood
- "Managing Gigabytes" - Witten, Moffat, Bell

### Specifications
- RFC 1951 (DEFLATE)
- RFC 7932 (Brotli)
- RFC 8478 (Zstandard)

### Libraries (Rust)
- `flate2` - DEFLATE/gzip/zlib
- `zstd` - Zstandard
- `lz4_flex` - LZ4
- `brotli` - Brotli

### Benchmarks
- Squash Benchmark
- lzbench
- compression.ru

---

## Recommended Order

Week 1-2:
- Read Part A (Theory)
- Implement RLE
- Implement Delta encoding

Week 3-4:
- Implement Huffman
- Implement LZ77

Week 5-6:
- Implement Arithmetic coding
- Implement BWT + MTF

Week 7-8:
- Implement mini-DEFLATE
- Study Zstandard

Week 9+:
- Deep dive into chosen specialty
- Contribute to open source projects
