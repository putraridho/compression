# Chapter 16: Dictionary Coding Introduction

## The Core Idea

**Dictionary coding** replaces repeated patterns with references to earlier occurrences.

```
Input:  "the cat and the dog and the bird"
                  ^^^         ^^^
                  repeated    repeated

Output: "the cat and <ref:0,4> dog <ref:8,5> bird"
```

Instead of storing "the " multiple times, store it once and reference it.

---

## Why Dictionary Coding Works

### Observation

Real data has repeated patterns:
- Text: common words, phrases
- Code: repeated identifiers, keywords
- Binary: repeated byte sequences

### The Method

1. Build a dictionary of seen patterns
2. When pattern repeats, output reference to dictionary
3. References are shorter than the pattern itself

---

## Two Approaches

### Explicit Dictionary

Build dictionary explicitly:

```
Dictionary:
  0 → "the "
  1 → "cat"
  2 → "and "

Input: "the cat and the dog and the bird"
Output: [0][1][2][0]"dog "[2][0]"bird"
```

LZ78 and LZW use this approach.

### Implicit Dictionary (Sliding Window)

Use recent input as implicit dictionary:

```
Window: [the cat and ]
New input: "the dog"

"the " found at position 0, length 4
Output: <offset=12, length=4> + "dog"
```

LZ77 uses this approach.

---

## Key Terminology

### Match

A repeated pattern found in dictionary or window.

### Literal

Data that must be output directly (no match found).

### Reference / Pointer / Token

Encoded form of a match: typically (offset, length).

### Offset (or Distance)

How far back the match is located.

### Length

How many bytes the match covers.

---

## Basic Dictionary Compression

### The Algorithm

```
position = 0
while position < input.length:
    match = find_longest_match(position)

    if match.length >= min_match:
        output_reference(match.offset, match.length)
        position += match.length
    else:
        output_literal(input[position])
        position += 1
```

### Example

```
Input: "ABCABCABC"
Position 0: No match, output 'A'
Position 1: No match, output 'B'
Position 2: No match, output 'C'
Position 3: Match at offset 3, length 3 ("ABC")
           Output: <3, 3>
Position 6: Match at offset 3, length 3 ("ABC")
           Output: <3, 3>

Output: A B C <3,3> <3,3>
```

---

## The LZ Family

```
                    LZ (Lempel-Ziv)
                   /              \
               LZ77               LZ78
        (Sliding Window)    (Explicit Dictionary)
           /     \                    |
        LZSS    LZ77 variants       LZW
          |                           |
       DEFLATE                   GIF, TIFF
          |
    gzip, PNG, ZIP
```

### LZ77 Family

- Uses sliding window as dictionary
- Reference = (distance back, length)
- Basis of DEFLATE, gzip, ZIP, PNG

### LZ78 Family

- Builds explicit dictionary
- Reference = dictionary index
- Basis of LZW (GIF, compress)

---

## Trade-offs

### Window/Dictionary Size

| Size | Pros | Cons |
|------|------|------|
| Small | Fast, low memory | Fewer matches |
| Large | More matches | Slower search, more memory |

### Minimum Match Length

| Length | Pros | Cons |
|--------|------|------|
| Short (2-3) | More matches | Overhead may exceed savings |
| Long (4+) | Efficient references | Misses short matches |

### Lazy vs Greedy Matching

**Greedy**: Take first/longest match
**Lazy**: Look ahead, might find better match

---

## Compression Workflow

```
┌─────────────┐
│   Input     │
└──────┬──────┘
       ▼
┌─────────────┐
│   Parsing   │  ← Find matches
└──────┬──────┘
       ▼
┌─────────────┐
│   Tokens    │  ← Literals + References
└──────┬──────┘
       ▼
┌─────────────────────┐
│   Entropy Coding    │  ← Huffman, ANS, etc.
└──────┬──────────────┘
       ▼
┌─────────────┐
│   Output    │
└─────────────┘
```

Dictionary coding produces tokens. Entropy coding compresses those tokens.

---

## Why Combine with Entropy Coding?

### Token Distribution is Skewed

- Some literals more common
- Some offsets more common (small distances)
- Some lengths more common

### Entropy Coding Exploits This

```
DEFLATE = LZ77 + Huffman
Zstandard = LZ77 + FSE/ANS
brotli = LZ77 + ANS + static dictionary
```

---

## Performance Considerations

### Matching is the Bottleneck

Finding matches can be slow. Solutions:

1. **Hash chains**: Hash position, chain same hashes
2. **Binary trees**: Suffix trees for fast search
3. **Hash tables**: O(1) lookup for recent patterns

### Memory-Speed Trade-off

| Approach | Speed | Memory | Compression |
|----------|-------|--------|-------------|
| Small hash | Fast | Low | Lower |
| Large hash | Medium | Medium | Medium |
| Full suffix tree | Slow | High | Best |

---

## Dictionary Coding Variants

| Algorithm | Dictionary Type | Special Features |
|-----------|-----------------|------------------|
| LZ77 | Sliding window | Original algorithm |
| LZSS | Sliding window | Flag bits for literals |
| LZ78 | Growing tree | Dictionary grows |
| LZW | Growing table | No explicit output of matched data |
| LZMA | Sliding window | Range coder, Markov chains |
| LZ4 | Sliding window | Speed optimized |
| Snappy | Sliding window | Speed optimized |
| Zstandard | Sliding window | FSE entropy coding |

---

## Key Takeaways

1. Dictionary coding replaces repeated patterns with references
2. Two approaches: sliding window (LZ77) and explicit dictionary (LZ78)
3. References are (offset, length) pairs
4. Combined with entropy coding for best compression
5. Trade-offs: window size, match length, search speed

---

**Coming Up**

| Chapter | Topic |
|---------|-------|
| 17 | LZ77 - The sliding window algorithm |
| 18 | LZSS - LZ77 with improvements |
| 19 | LZ78 - Explicit dictionary approach |
| 20 | LZW - Dictionary coding for GIF |

---

**Next Chapter**: [LZ77](./17_lz77.md)
