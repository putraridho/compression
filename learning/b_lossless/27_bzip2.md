# Chapter 27: bzip2

## Overview

**bzip2** is Julian Seward's high-compression algorithm from 1996.

Uses BWT + MTF + RLE + Huffman for excellent text compression.

---

## Architecture

```
Input → Block → RLE1 → BWT → MTF → RLE2 → Huffman → Output
```

### Pipeline Stages

1. **RLE1**: Initial run-length encoding
2. **BWT**: Burrows-Wheeler Transform
3. **MTF**: Move-to-Front
4. **RLE2**: Zero run-length encoding
5. **Huffman**: Multiple Huffman tables

---

## Block-Based

bzip2 processes in blocks:

- Block sizes: 100KB to 900KB
- Configurable with -1 to -9 flags
- Each block compressed independently
- Larger blocks = better compression

---

## Stage 1: Initial RLE

Before BWT, compress runs of 4+ identical bytes:

```
Input:  "aaaaaaaabbbb"
Output: "aaaa\x04bbbb" (4 a's, then count=4, then b's)
```

Prevents BWT pathological cases.

---

## Stage 2: BWT

Burrows-Wheeler Transform:

1. Form all rotations of block
2. Sort rotations
3. Take last column
4. Remember original row position

Groups similar characters together.

---

## Stage 3: MTF

Move-to-Front Transform:

- Recently seen symbols → small indices
- After BWT, creates many 0s and small values

---

## Stage 4: Zero Run Encoding

Special encoding for zero runs (very common after MTF):

### RUNA/RUNB Encoding

```
Number of zeros encoded in bijective base-2:
  1 zero:  RUNA
  2 zeros: RUNB
  3 zeros: RUNA RUNA
  4 zeros: RUNB RUNA
  5 zeros: RUNA RUNB
  ...
```

Formula: zeros = sum of (RUNA→1, RUNB→2) × 2^position

Very efficient for long zero runs.

---

## Stage 5: Huffman Coding

bzip2 uses multiple Huffman tables:

### Table Selection

- 2-6 different Huffman tables per block
- Block divided into 50-symbol groups
- Each group selects best table
- Table selector is also Huffman coded

### Why Multiple Tables?

Different parts of file have different statistics:
- Code section vs data section
- Different text styles
- Tables adapt to local patterns

---

## File Format

### File Header

```
Magic: "BZ" (0x42 0x5A)
Version: 'h' (0x68) for bzip2
Block size: '1'-'9' (100KB - 900KB)
```

### Block Header

```
Magic: 0x314159265359 (pi digits!)
CRC: 32 bits
Randomized: 1 bit (rarely used)
Original pointer: 24 bits (BWT index)
```

### Block Data

```
Symbol map: which symbols used
Number of trees: 3 bits
Number of selectors: 15 bits
Selector list: MTF encoded
Huffman tables: delta encoded lengths
Compressed data
```

### Stream Footer

```
Magic: 0x177245385090
CRC: 32 bits (all blocks combined)
```

---

## Compression Performance

### Typical Ratios

| Content | bzip2 | gzip | Improvement |
|---------|-------|------|-------------|
| English text | 3.5x | 2.5x | 40% better |
| Source code | 5.0x | 3.5x | 43% better |
| Binaries | 2.5x | 2.0x | 25% better |

### Speed

- Compression: ~3-5 MB/s (slower than gzip)
- Decompression: ~15-20 MB/s (slower than gzip)
- Memory: ~2.5 × block size

---

## Command Line

```bash
# Compress
bzip2 file.txt          # Creates file.txt.bz2

# Decompress
bzip2 -d file.txt.bz2   # Creates file.txt
bunzip2 file.txt.bz2    # Same as above

# Keep original
bzip2 -k file.txt

# Best compression
bzip2 -9 file.txt       # 900KB blocks

# Fastest
bzip2 -1 file.txt       # 100KB blocks
```

---

## Pseudocode

### Compress Block

```
function compress_block(input):
    // Stage 1: Initial RLE
    data = initial_rle(input)

    // Stage 2: BWT
    (bwt_data, index) = burrows_wheeler_transform(data)

    // Stage 3: MTF
    mtf_data = move_to_front(bwt_data)

    // Stage 4: Zero run encoding
    rle_data = zero_run_encode(mtf_data)

    // Stage 5: Huffman
    select_huffman_tables(rle_data)
    output_huffman_encoded(rle_data)

    return (compressed_data, index)
```

### Decompress Block

```
function decompress_block(input):
    // Read Huffman tables and selectors
    tables = read_huffman_tables()
    selectors = read_selectors()

    // Decode Huffman
    mtf_data = huffman_decode(input, tables, selectors)

    // Inverse zero run encoding
    rle_data = zero_run_decode(mtf_data)

    // Inverse MTF
    bwt_data = inverse_mtf(rle_data)

    // Inverse BWT
    data = inverse_bwt(bwt_data, original_index)

    // Inverse initial RLE
    output = inverse_initial_rle(data)

    return output
```

---

## bzip2 vs gzip

| Aspect | bzip2 | gzip |
|--------|-------|------|
| Algorithm | BWT + Huffman | LZ77 + Huffman |
| Compression | Better | Good |
| Speed | Slower | Faster |
| Memory | Higher | Lower |
| Streaming | Block-based | Stream |
| Decompression | Moderate | Fast |

### When to Use bzip2

- Maximum compression matters
- Archival storage
- Text-heavy content
- Time is not critical

### When to Use gzip

- Speed matters
- Memory constrained
- Streaming needed
- General purpose

---

## Parallelization

bzip2 is block-based → parallelizable:

### pbzip2

- Parallel bzip2 implementation
- Each block compressed on separate thread
- Near-linear speedup
- Compatible output

### lbzip2

- Another parallel implementation
- Focus on decompression speed
- Better load balancing

---

## Key Takeaways

1. BWT + MTF + RLE + Huffman pipeline
2. Excellent compression, especially for text
3. Block-based processing (100-900KB)
4. Slower than gzip but better ratio
5. Multiple Huffman tables per block
6. Parallelizable due to block independence

---

**Practice**:

1. Trace full pipeline on small input
2. Implement each stage separately
3. Compare compression to gzip on various files
4. Study pbzip2 for parallelization

---

**Next Chapter**: [Zstandard](./28_zstandard.md)
