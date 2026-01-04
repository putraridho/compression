# Chapter 26: DEFLATE

## Overview

**DEFLATE** is the most widely used compression algorithm. Created by Phil Katz for PKZIP in 1993.

Used in: ZIP, gzip, PNG, HTTP compression, countless other formats.

---

## Architecture

```
Input → LZ77 → Huffman → Output
```

DEFLATE = LZ77 (dictionary coding) + Huffman (entropy coding)

---

## The Three Block Types

DEFLATE compresses in blocks. Each block has a type:

### Type 0: Stored (No Compression)

```
Bit: 0 (not last) or 1 (last block)
Type: 00
Length: 16 bits
NLength: 16 bits (one's complement of length)
Data: raw bytes
```

Used when compression would expand data.

### Type 1: Fixed Huffman

```
Bit: 0/1
Type: 01
Data: LZ77 tokens with predefined Huffman codes
```

Codes are fixed, no need to transmit table.

### Type 2: Dynamic Huffman

```
Bit: 0/1
Type: 10
Header: Code length tables
Data: LZ77 tokens with custom Huffman codes
```

Custom codes optimized for this block.

---

## LZ77 in DEFLATE

### Tokens

Two token types:
1. **Literal**: Single byte (0-255)
2. **Match**: (length, distance) pair

### Encoding

```
Literals: 0-255
End of block: 256
Length codes: 257-285 (lengths 3-258)
Distance codes: 0-29 (distances 1-32768)
```

### Length Encoding

| Code | Length | Extra Bits |
|------|--------|------------|
| 257 | 3 | 0 |
| 258 | 4 | 0 |
| 259 | 5 | 0 |
| 260 | 6 | 0 |
| 261 | 7 | 0 |
| 262 | 8 | 0 |
| 263 | 9 | 0 |
| 264 | 10 | 0 |
| 265 | 11-12 | 1 |
| 266 | 13-14 | 1 |
| ... | ... | ... |
| 285 | 258 | 0 |

### Distance Encoding

| Code | Distance | Extra Bits |
|------|----------|------------|
| 0 | 1 | 0 |
| 1 | 2 | 0 |
| 2 | 3 | 0 |
| 3 | 4 | 0 |
| 4 | 5-6 | 1 |
| 5 | 7-8 | 1 |
| ... | ... | ... |
| 29 | 24577-32768 | 13 |

---

## Fixed Huffman Codes

### Literal/Length

```
0-143:   8 bits (00110000 - 10111111)
144-255: 9 bits (110010000 - 111111111)
256-279: 7 bits (0000000 - 0010111)
280-287: 8 bits (11000000 - 11000111)
```

### Distance

All distance codes use 5 bits (0-29).

---

## Dynamic Huffman

### Header Structure

```
HLIT:  5 bits (# of literal/length codes - 257)
HDIST: 5 bits (# of distance codes - 1)
HCLEN: 4 bits (# of code length codes - 4)

Code length codes (3 bits each, in weird order):
  16, 17, 18, 0, 8, 7, 9, 6, 10, 5, 11, 4, 12, 3, 13, 2, 14, 1, 15

Literal/length code lengths (Huffman coded)
Distance code lengths (Huffman coded)
```

### Code Length Symbols

| Symbol | Meaning |
|--------|---------|
| 0-15 | Actual code length |
| 16 | Repeat previous 3-6 times (2 extra bits) |
| 17 | Repeat 0, 3-10 times (3 extra bits) |
| 18 | Repeat 0, 11-138 times (7 extra bits) |

---

## Compression Algorithm

### Basic Approach

```
function deflate(input):
    blocks = divide_into_blocks(input)

    for block in blocks:
        // Try different strategies
        stored_size = size_if_stored(block)
        fixed_size = size_with_fixed_huffman(block)
        dynamic_size = size_with_dynamic_huffman(block)

        // Choose best
        use smallest option

        // LZ77 parse
        tokens = lz77_parse(block)

        // Output
        write_block(tokens)
```

### LZ77 Parsing Options

1. **Greedy**: Take longest match at each position
2. **Lazy**: Check if better match at next position
3. **Optimal**: Dynamic programming for best parse

### Match Finding

DEFLATE uses hash chains:

```
1. Hash first 3 bytes at position
2. Look up hash → list of positions
3. Compare each to find longest match
4. Add current position to hash chain
```

---

## Decompression

```
function inflate(input):
    output = []

    while not end_of_stream:
        is_last = read_bit()
        block_type = read_bits(2)

        if block_type == 0:
            copy_stored_block(output)
        else if block_type == 1:
            decode_with_fixed_huffman(output)
        else if block_type == 2:
            read_dynamic_tables()
            decode_with_tables(output)

    return output
```

---

## Compression Levels

Most implementations offer levels:

| Level | Strategy | Speed | Ratio |
|-------|----------|-------|-------|
| 1 | Fast, short chains | Fastest | Lowest |
| 5 | Balanced | Medium | Medium |
| 6 | Default | Good | Good |
| 9 | Max chains, lazy | Slowest | Highest |

### zlib Levels

```
0: Store only
1: Fast
2-8: Balanced
9: Best compression
```

---

## Window Size

DEFLATE uses 32KB sliding window.

Maximum match distance: 32768 bytes
Maximum match length: 258 bytes

---

## Bit Ordering

DEFLATE uses LSB-first bit packing:

```
First bit goes in position 0 (LSB)
Next bit in position 1
...
After 8 bits, start new byte
```

This is opposite to many other formats!

---

## Format Wrappers

### Raw DEFLATE

Just the compressed blocks.

### zlib Format

```
CMF: 1 byte (compression method, window size)
FLG: 1 byte (flags, check bits)
[DICTID: 4 bytes if FLG.FDICT set]
[Compressed data]
ADLER32: 4 bytes (checksum)
```

### gzip Format

```
Magic: 1f 8b
Method: 08 (deflate)
Flags: 1 byte
Modification time: 4 bytes
Extra flags: 1 byte
OS: 1 byte
[Optional fields based on flags]
[Compressed data]
CRC32: 4 bytes
Original size: 4 bytes
```

---

## Optimizations

### Zopfli

Google's optimal DEFLATE encoder:
- Exhaustive search for best parse
- 3-8% smaller than zlib -9
- 100x slower compression
- Standard DEFLATE, fast decompression

### libdeflate

Fast DEFLATE implementation:
- 2x faster compression than zlib
- 3x faster decompression
- Modern CPU optimizations

---

## Key Takeaways

1. DEFLATE = LZ77 + Huffman
2. Three block types: stored, fixed, dynamic
3. 32KB window, 258 byte max match
4. Most widely used compression algorithm
5. Used in ZIP, gzip, PNG
6. Balance of compression and speed

---

**Practice**:

1. Parse a gzip file header by hand
2. Implement fixed Huffman decoding
3. Build dynamic Huffman tables
4. Write a complete DEFLATE decompressor

---

**Next Chapter**: [bzip2](./27_bzip2.md)
