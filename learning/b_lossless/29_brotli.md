# Chapter 29: Brotli

## Overview

**Brotli** is Google's 2013 compression algorithm, optimized for web content.

Named after Swiss pastry. Designed for HTML, CSS, JavaScript.

---

## Key Features

- Static dictionary of common web strings
- Better than gzip for web content
- Supported by all major browsers
- 11 compression levels (0-11)

---

## Architecture

```
Input → LZ77 with Static Dict → Context Modeling → ANS → Output
```

### Unique Features

1. Static dictionary (120KB)
2. Context-dependent coding
3. Distance cache (last 4 + 16 static)
4. ANS entropy coding

---

## Static Dictionary

### What It Contains

~120KB of common strings:

```
" the ", "tion", "ment", "ness"
"class=\"", "script", "function"
"http://", "https://", ".com"
"Content-Type", "application"
...
```

13,000+ entries in multiple lengths.

### How It Helps

Without dictionary:
```
"Content-Type" → 12 literal bytes
```

With dictionary:
```
"Content-Type" → 1 dictionary reference (~2 bytes)
```

80% smaller for common strings!

---

## Context Modeling

### Literal Context

Based on previous 1-2 bytes:

```
After space: likely lowercase letter
After digit: likely digit or punctuation
After '<': likely letter (HTML tag)
```

Different probability models for each context.

### Distance Context

Based on copy length:

```
Short copies: likely small distances
Long copies: may use any distance
```

---

## Distance Cache

### Recent Distances

Last 4 used distances are cached:
- Distance 0: Use last distance
- Distance 1: Use second-to-last
- Distance 2: Use third-to-last
- Distance 3: Use fourth-to-last

### Static Distances

16 additional static distances:
- Last distance ± small offsets
- Common patterns

Covers most repeated patterns efficiently.

---

## Metablock Structure

```
Metablock header:
  - Metablock length
  - Is uncompressed?
  - Block type counts

Block type definitions:
  - Literal block types
  - Insert/copy block types
  - Distance block types

Context maps:
  - Literal context map
  - Distance context map

Huffman/ANS tables

Compressed data
```

---

## Compression Levels

| Level | Use Case | Ratio vs gzip |
|-------|----------|---------------|
| 0 | No compression | - |
| 1-3 | Fast, streaming | Similar |
| 4-6 | Balanced | 10-15% better |
| 7-9 | Good compression | 15-20% better |
| 10-11 | Maximum | 20-25% better |

Level 11 is very slow but maximizes compression.

---

## Web Optimization

### Why Brotli for Web?

1. **Static dictionary**: HTML/CSS/JS terms pre-indexed
2. **Better ratio**: Smaller files = faster loads
3. **Worth the CPU**: Compress once, serve many times
4. **Browser support**: Universal in modern browsers

### HTTP Usage

```http
Accept-Encoding: br, gzip, deflate
Content-Encoding: br
```

### Typical Savings

| Content | gzip | Brotli | Savings |
|---------|------|--------|---------|
| HTML | 70KB | 60KB | 14% |
| CSS | 50KB | 42KB | 16% |
| JavaScript | 200KB | 165KB | 17% |

---

## Command Line

```bash
# Compress
brotli file.txt           # Creates file.txt.br

# Decompress
brotli -d file.txt.br

# Compression levels
brotli -q 1 file.txt      # Fast
brotli -q 11 file.txt     # Maximum

# Keep original
brotli -k file.txt

# Window size
brotli -w 24 file.txt     # 16MB window
```

---

## Comparison

### vs gzip

| Aspect | Brotli-5 | gzip-6 |
|--------|----------|--------|
| Ratio | 15% better | Baseline |
| Compress | Slower | Faster |
| Decompress | Similar | Similar |
| Web content | Much better | Good |

### vs Zstandard

| Aspect | Brotli | Zstandard |
|--------|--------|-----------|
| Web content | Excellent | Very good |
| General data | Good | Excellent |
| Speed | Moderate | Fast |
| Dictionary | Built-in | Train yourself |

---

## Format Details

### Stream Header

```
Window size: 1-24 (log scale)
```

### Metablock Types

1. **Empty**: No data
2. **Uncompressed**: Raw bytes
3. **Compressed**: LZ77 + Huffman

### Command Structure

```
Insert-and-copy command:
  - Insert length (# of literals)
  - Copy length (# to copy)
  - Distance (where to copy from)
```

---

## Pseudocode

### Encoder

```
function brotli_compress(input, quality):
    output = []

    // Write window size
    output.append(encode_window_size())

    for metablock in split_metablocks(input):
        if should_store_raw(metablock, quality):
            output.append(uncompressed_metablock(metablock))
        else:
            // LZ77 parse with dictionary
            commands = lz_parse_with_dict(metablock, quality)

            // Build context maps
            lit_ctx = build_literal_context_map(commands)
            dist_ctx = build_distance_context_map(commands)

            // Compress
            output.append(compressed_metablock(commands, lit_ctx, dist_ctx))

    return output
```

### Using Static Dictionary

```
function find_dict_match(input, position):
    // Try dictionary matches
    for length in [4, 5, 6, ..., 24]:
        substring = input[position:position+length]

        if substring in STATIC_DICTIONARY:
            dict_id = STATIC_DICTIONARY[substring]
            return DictMatch(dict_id, length)

    return null
```

---

## Dictionary Internals

### Dictionary Structure

```
Total size: 122,784 bytes
Number of words: ~13,000

Word lengths: 4 to 24 bytes
Organized by length

Transformations: 121 different
  - Identity
  - Uppercase first
  - Uppercase all
  - Prefix with space
  - Suffix with various strings
  - Combinations
```

### Transformations

Dictionary word + transformation:

```
Word: "script"
Transform 0: "script"
Transform 1: " script"
Transform 2: "Script"
Transform 3: " Script"
...
```

121 transforms × 13,000 words = huge effective dictionary!

---

## Key Takeaways

1. Optimized for web content (HTML, CSS, JS)
2. 120KB static dictionary of common strings
3. Context-dependent probability models
4. 15-25% better than gzip for web
5. Supported by all modern browsers
6. Use level 4-6 for dynamic, 11 for static content

---

**Practice**:

1. Compare Brotli to gzip on web files
2. Check which dictionary words appear in your HTML
3. Try different quality levels
4. Measure real-world page load differences

---

**Next Chapter**: [Context Modeling Introduction](./30_context_intro.md)
