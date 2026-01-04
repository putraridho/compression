# Chapter 22: Modern LZ Variants

## The Modern Landscape

Several modern LZ algorithms optimize for different goals:

| Algorithm | Optimizes For |
|-----------|---------------|
| LZ4 | Decompression speed |
| Snappy | Balanced speed |
| Zstandard | Compression + speed |
| Brotli | Web compression |

---

## LZ4

### Philosophy

"Fastest decompression while maintaining reasonable compression."

### Key Features

- Decompression: 400+ MB/s per core
- Compression: 400+ MB/s (LZ4), 40 MB/s (LZ4-HC)
- Fixed-size tokens for predictable performance
- Simple format, easy to implement

### Token Format

```
[Token] [Literals] [Match Offset] [Extra Length]

Token byte:
  High 4 bits: Literal length (0-15, 15 = more bytes follow)
  Low 4 bits:  Match length (4-19, 15 = more bytes follow)
```

### Why So Fast

1. **No entropy coding**: Raw bytes + simple tokens
2. **Fixed minimum match**: 4 bytes
3. **Simple format**: Predictable branches
4. **Copy optimization**: memcpy-friendly

---

## Snappy

### Philosophy

Google's "fast enough compression with low latency."

### Key Features

- ~250 MB/s compression
- ~500 MB/s decompression
- Lower compression ratio than gzip
- Used in BigTable, Protocol Buffers, etc.

### Format

```
Literals: [tag][length][data...]
Copies:   [tag][offset][length]

Tag encodes type and initial length bits.
```

### Use Cases

- Database storage
- RPC compression
- Cache serialization
- Anywhere latency matters more than size

---

## Zstandard (zstd)

### Philosophy

"Compress like LZMA, speed like LZ4."

### Key Features

- Compression levels 1-22
- Level 1: Faster than LZ4, better ratio
- Level 19+: Approaches LZMA ratios
- Streaming support
- Dictionary support

### Architecture

```
Input → LZ77 Parser → Sequences → FSE Encoding → Output
```

### Sequences

Three streams:
1. **Literals**: Raw bytes (may be Huffman compressed)
2. **Offsets**: Match distances (FSE encoded)
3. **Match lengths**: (FSE encoded)
4. **Literal lengths**: (FSE encoded)

### Compression Levels

| Level | Speed | Ratio | Use Case |
|-------|-------|-------|----------|
| 1-3 | Very fast | Good | Real-time |
| 4-9 | Fast | Better | Default |
| 10-15 | Medium | High | Storage |
| 16-22 | Slow | Very high | Archival |

---

## Brotli

### Philosophy

Optimized for web content (HTML, CSS, JS).

### Key Features

- Better than gzip for web
- Static dictionary of common web strings
- Supported by all browsers
- Levels 0-11

### Static Dictionary

Contains common strings:

```
" the ", "tion", "with", "ment",
"class=", "script", "http://", ...
```

~120KB of common patterns. References use less bits than literals.

### Context Modeling

Different probability models for:
- After space (likely lowercase letter)
- After digit (likely digit)
- In URL (likely path characters)

---

## Comparison

### Compression Ratio (enwik9)

| Algorithm | Ratio | Compress MB/s | Decompress MB/s |
|-----------|-------|---------------|-----------------|
| LZ4 | 2.0x | 780 | 4970 |
| Snappy | 1.8x | 580 | 2100 |
| zstd -1 | 2.8x | 500 | 1700 |
| zstd -3 | 2.9x | 350 | 1700 |
| gzip -6 | 3.1x | 35 | 400 |
| zstd -19 | 3.4x | 5 | 1500 |
| brotli -11 | 3.5x | 1 | 450 |

### When to Use What

| Need | Use |
|------|-----|
| Maximum speed | LZ4 |
| Low latency | Snappy |
| General purpose | Zstandard |
| Web content | Brotli |
| Maximum compression | LZMA/zstd-22 |

---

## LZ4 Details

### Frame Format

```
Magic (4 bytes): 0x184D2204
Frame Descriptor: flags, block size
[Blocks...]
Checksum (optional)
```

### Block Format

```
Block Size (4 bytes)
[Sequences...]
End marker
```

### Sequence

```
Token: llll mmmm
  llll = literal length (0-15)
  mmmm = match length - 4 (0-15)

If llll == 15: additional length bytes follow
Literals (llll bytes)
Offset (2 bytes, little-endian)
If mmmm == 15: additional length bytes follow
```

---

## Zstandard Details

### Frame Format

```
Magic: 0xFD2FB528
Frame Header: window size, content size, dictionary ID
[Blocks...]
Checksum
```

### Block Types

1. **Raw**: Uncompressed literals
2. **RLE**: Run-length encoded
3. **Compressed**: LZ77 + FSE

### Dictionary Mode

Pre-shared dictionary for small files:

```
Train dictionary:
  zstd --train files/* -o dict

Compress with dictionary:
  zstd -D dict input

Decompress:
  zstd -D dict -d input.zst
```

Great for small similar files (JSON, logs).

---

## Implementation Tips

### For Speed (LZ4-style)

1. Use hash table for match finding
2. Minimum match = 4 bytes
3. Skip hash updates in low-entropy regions
4. Use memcpy for literal runs
5. Unroll copy loops

### For Ratio (zstd-style)

1. Try multiple match lengths
2. Use optimal parsing
3. Compress literals separately
4. Multiple probability tables
5. Block-level decisions

---

## Code Example: LZ4-like Encoder

```
function fast_lz_encode(input):
    hash_table = {}  // 4-byte hash → position
    output = []
    pos = 0
    anchor = 0  // Start of unencoded literals

    while pos < length(input) - 4:
        hash = hash4(input[pos:pos+4])

        if hash in hash_table:
            match_pos = hash_table[hash]
            match_len = count_match(input, match_pos, pos)

            if match_len >= 4:
                // Output literals
                emit_literals(output, input[anchor:pos])

                // Output match
                emit_match(output, pos - match_pos, match_len)

                pos += match_len
                anchor = pos
                continue

        hash_table[hash] = pos
        pos += 1

    // Final literals
    emit_literals(output, input[anchor:])
    return output
```

---

## Key Takeaways

1. Modern LZ variants optimize for different goals
2. LZ4: Maximum decompression speed
3. Snappy: Low-latency, balanced
4. Zstandard: Best all-around, many levels
5. Brotli: Web-optimized with static dictionary
6. All use LZ77 core with different entropy coding

---

**Practice**:

1. Benchmark LZ4 vs Snappy vs zstd on your data
2. Implement a simple LZ4-like encoder
3. Try zstd dictionary mode on similar files
4. Compare Brotli vs gzip on web content

---

**Next Chapter**: [Burrows-Wheeler Transform](./23_bwt.md)
