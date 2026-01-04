# Chapter 28: Zstandard

## Overview

**Zstandard (zstd)** is Facebook's 2015 compression algorithm by Yann Collet.

Goal: Compress like LZMA, decompress like LZ4.

---

## Key Features

- 22 compression levels
- Extremely fast decompression (~1.5 GB/s)
- Better ratio than gzip at similar speeds
- Dictionary support
- Streaming support

---

## Architecture

```
Input → LZ Parsing → Sequences → FSE Encoding → Output
```

### Sequences

Each match produces a sequence:
```
(literal_length, offset, match_length)
```

Three separate streams:
1. Literals (may be Huffman compressed)
2. Sequences (FSE encoded)
3. Raw offset values

---

## Compression Levels

| Level | Ratio | Compress MB/s | Decompress MB/s |
|-------|-------|---------------|-----------------|
| 1 | 2.8x | 500 | 1700 |
| 3 | 2.9x | 350 | 1700 |
| 5 | 3.0x | 190 | 1700 |
| 9 | 3.1x | 60 | 1700 |
| 15 | 3.3x | 15 | 1700 |
| 19 | 3.4x | 5 | 1700 |
| 22 | 3.5x | 2 | 1700 |

Note: Decompression speed is nearly constant!

---

## Frame Format

```
Magic Number: 0xFD2FB528 (4 bytes)
Frame Header:
  - Frame Header Descriptor (1 byte)
  - Window Descriptor (0-1 bytes)
  - Dictionary ID (0-4 bytes)
  - Frame Content Size (0-8 bytes)
  - Header Checksum (0-1 bytes)
Blocks...
Checksum (0-4 bytes)
```

---

## Block Types

### Raw Block

Uncompressed data.

### RLE Block

Single byte repeated.

### Compressed Block

```
Literals Section:
  - Literals header
  - Huffman tree (if compressed)
  - Literals stream(s)

Sequences Section:
  - Number of sequences
  - Symbol compression modes
  - FSE tables
  - Sequences bitstream
```

---

## Literals Compression

Four modes:

1. **Raw**: Store as-is
2. **RLE**: Single repeated byte
3. **Huffman**: Standard Huffman
4. **Repeat**: Reuse previous Huffman table

### Four-Stream Literals

For parallel decoding, literals split into 4 streams:
- CPU can decode streams simultaneously
- SIMD-friendly

---

## Sequences Encoding

Each sequence has:
- **Literal length**: bytes before match
- **Match length**: bytes to copy
- **Offset**: distance to match

### FSE Tables

Three FSE tables:
1. Literal length codes
2. Match length codes
3. Offset codes

### Predefined Tables

Level 1 uses predefined tables (no header needed).
Higher levels compute optimal tables.

---

## Repeat Offsets

Zstandard tracks last 3 offsets:

```
Offset code 1: Repeat offset 1
Offset code 2: Repeat offset 2
Offset code 3: Repeat offset 3
Offset code 3 (after literal_length=0): Repeat offset 1 - 1
```

Repeated offsets are very common → saves bits.

---

## Dictionary Mode

For small files, train dictionary on similar data:

```bash
# Train dictionary
zstd --train samples/* -o dict

# Compress with dictionary
zstd -D dict input

# Decompress
zstd -D dict -d input.zst
```

### How It Works

Dictionary provides:
- Initial window content
- Huffman table
- FSE tables
- Common strings

Great for JSON, logs, small records.

---

## Comparison

### vs gzip

| Aspect | zstd -3 | gzip -6 |
|--------|---------|---------|
| Ratio | 2.9x | 3.1x |
| Compress | 350 MB/s | 35 MB/s |
| Decompress | 1700 MB/s | 400 MB/s |

10x faster compression, 4x faster decompression!

### vs LZ4

| Aspect | zstd -1 | LZ4 |
|--------|---------|-----|
| Ratio | 2.8x | 2.0x |
| Compress | 500 MB/s | 780 MB/s |
| Decompress | 1700 MB/s | 4970 MB/s |

40% better compression, similar speed class.

---

## Command Line

```bash
# Compress
zstd file.txt           # Creates file.txt.zst

# Decompress
zstd -d file.txt.zst

# Compression levels
zstd -1 file.txt        # Fast
zstd -19 file.txt       # Best ratio

# Keep original
zstd -k file.txt

# Dictionary
zstd --train dir/* -o dict
zstd -D dict file.txt

# Streaming
cat file | zstd > file.zst
```

---

## Long-Range Mode

For very repetitive data:

```bash
zstd --long file.txt
zstd --long=31 file.txt  # 2GB window
```

Enables matches up to 128MB or more.

---

## Multi-Threading

Built-in parallel compression:

```bash
zstd -T0 file.txt       # Auto-detect threads
zstd -T4 file.txt       # 4 threads
```

Frames can be split and compressed in parallel.

---

## Pseudocode

### Encoder Overview

```
function zstd_compress(input, level):
    // Select parameters based on level
    params = get_level_params(level)

    frame = []
    frame.append(MAGIC_NUMBER)
    frame.append(frame_header(input.size))

    // Process in blocks
    for block in split_blocks(input):
        if block is compressible:
            sequences = lz_parse(block, params)
            literals = extract_literals(sequences)

            // Compress literals
            lit_stream = compress_literals(literals)

            // Compress sequences
            seq_stream = fse_compress_sequences(sequences)

            frame.append(compressed_block(lit_stream, seq_stream))
        else:
            frame.append(raw_block(block))

    frame.append(checksum)
    return frame
```

### Decoder Overview

```
function zstd_decompress(input):
    verify_magic(input)
    header = read_frame_header(input)

    output = []
    while not end_of_frame:
        block = read_block(input)

        if block.type == RAW:
            output.append(block.data)
        else if block.type == RLE:
            output.append(block.byte × block.count)
        else:
            literals = decompress_literals(block)
            sequences = fse_decompress_sequences(block)

            // Execute sequences
            output.append(execute_sequences(literals, sequences, output))

    verify_checksum(output)
    return output
```

---

## Why Zstandard Excels

1. **FSE**: Near-optimal entropy coding
2. **Repeat offsets**: Common patterns cheap
3. **Level flexibility**: 22 levels for any use case
4. **Dictionary**: Small file optimization
5. **Parallel-friendly**: 4-stream literals, threading
6. **Modern design**: CPU cache aware

---

## Key Takeaways

1. Best general-purpose compressor today
2. Extremely fast decompression at all levels
3. LZ77 + FSE (tANS) entropy coding
4. 22 compression levels
5. Dictionary mode for small similar files
6. Drop-in gzip replacement with better performance

---

**Practice**:

1. Benchmark zstd vs gzip on your data
2. Try different compression levels
3. Train a dictionary on similar files
4. Implement simplified sequence decoder

---

**Next Chapter**: [Brotli](./29_brotli.md)
