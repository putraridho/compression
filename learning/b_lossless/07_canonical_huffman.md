# Chapter 7: Canonical Huffman Coding

## The Problem with Standard Huffman

Standard Huffman has issues:

1. **Multiple valid trees**: Same lengths, different codes
2. **Large headers**: Must transmit full tree structure
3. **Complex decoding**: Need tree traversal

---

## What is Canonical Huffman?

A standardized way to assign Huffman codes given only the code lengths.

**Key insight**: If you know all code lengths, you can reconstruct the exact codes using simple rules.

---

## Canonical Code Properties

### Rule 1: Shorter Codes Are Numerically Smaller

Codes are assigned as integers, shorter codes have smaller values.

### Rule 2: Same-Length Codes Are Consecutive

Symbols with same code length get consecutive integer codes.

### Rule 3: Alphabetical Within Same Length

Symbols with same length are ordered alphabetically (or by symbol value).

---

## Building Canonical Codes

### Given Code Lengths

From standard Huffman, we get:

| Symbol | Length |
|--------|--------|
| A | 1 |
| B | 3 |
| C | 3 |
| D | 3 |
| E | 3 |

### Step 1: Sort by Length, Then Symbol

| Symbol | Length |
|--------|--------|
| A | 1 |
| B | 3 |
| C | 3 |
| D | 3 |
| E | 3 |

(Already sorted)

### Step 2: Assign Codes

```
Start with code = 0

For each symbol (in sorted order):
    Assign current code (padded to length)
    Increment code
    If next symbol has longer length:
        code = code << (new_length - current_length)
```

### Walkthrough

```
Symbol A (length 1):
    code = 0
    Binary: 0 (1 bit)
    A = 0
    code++ → 1

Next symbol B has length 3, current length is 1:
    code = 1 << (3-1) = 1 << 2 = 4 (binary: 100)

Symbol B (length 3):
    code = 4
    Binary: 100 (3 bits)
    B = 100
    code++ → 5

Symbol C (length 3):
    code = 5
    Binary: 101 (3 bits)
    C = 101
    code++ → 6

Symbol D (length 3):
    code = 6
    Binary: 110 (3 bits)
    D = 110
    code++ → 7

Symbol E (length 3):
    code = 7
    Binary: 111 (3 bits)
    E = 111
```

### Final Canonical Codes

| Symbol | Length | Code |
|--------|--------|------|
| A | 1 | 0 |
| B | 3 | 100 |
| C | 3 | 101 |
| D | 3 | 110 |
| E | 3 | 111 |

---

## Why This Works

### Prefix Property Preserved

The algorithm guarantees no code is a prefix of another:

- Short codes are small numbers
- Before extending to longer codes, we shift left
- This creates a "gap" ensuring short codes aren't prefixes

### Example

```
A = 0 (1 bit)

Before B, we shift: 1 << 2 = 4

In binary:
A = 0       → extends to 0xx (but 0xx starts at 000, and B=100)
B = 100     → starts at 4, not in 0xx range
```

---

## Storing Canonical Codes

### Only Need Lengths!

Instead of storing full codes, just store:

```
Number of codes of length 1: 1  (symbol A)
Number of codes of length 2: 0
Number of codes of length 3: 4  (symbols B,C,D,E)
```

Plus the symbols in order.

### DEFLATE Format

DEFLATE stores:
```
[count for length 1]
[count for length 2]
...
[count for length 15]
[symbols in order]
```

Decoder rebuilds codes using canonical rules.

---

## Decoding Canonical Huffman

### Fast Table-Based Decoding

```
For each possible length L:
    first_code[L] = first code of length L
    first_symbol[L] = index of first symbol with length L

To decode:
    Read max_length bits
    For L = 1 to max_length:
        code = bits >> (max_length - L)
        if code >= first_code[L] and code < first_code[L+1]:
            index = first_symbol[L] + (code - first_code[L])
            return symbol[index], consume L bits
```

This is O(max_length) but typically max_length ≤ 15.

### Lookup Table

Even faster: precompute a table.

```
For each possible bit pattern (up to max length):
    table[pattern] = (symbol, length)

To decode:
    Read max_length bits as index
    symbol, length = table[index]
    Consume only 'length' bits
    Output symbol
```

---

## Pseudocode

### Building Canonical Codes

```
function canonical_codes(symbols, lengths):
    // Sort symbols by (length, symbol)
    sorted = sort(symbols, key=(length, symbol))

    codes = {}
    code = 0
    prev_length = 0

    for symbol in sorted:
        length = lengths[symbol]

        // Shift if length increased
        if length > prev_length:
            code = code << (length - prev_length)

        codes[symbol] = (code, length)
        code += 1
        prev_length = length

    return codes
```

### Decoding Setup

```
function build_decode_tables(lengths):
    max_length = max(lengths)

    // Count symbols per length
    count = [0] * (max_length + 1)
    for length in lengths:
        count[length] += 1

    // Calculate first code for each length
    first_code = [0] * (max_length + 2)
    code = 0
    for length = 1 to max_length:
        code = (code + count[length - 1]) << 1
        first_code[length] = code

    return first_code, sorted_symbols
```

---

## Comparison: Standard vs Canonical

| Aspect | Standard Huffman | Canonical Huffman |
|--------|-----------------|-------------------|
| Tree uniqueness | Multiple valid trees | Unique for given lengths |
| Header size | Large (full tree) | Small (just lengths) |
| Encoding speed | Same | Same |
| Decoding speed | Tree traversal | Table lookup (faster) |
| Implementation | More complex | Simpler |

---

## Real-World Usage

### DEFLATE (gzip, ZIP, PNG)

- Uses canonical Huffman
- Stores code lengths
- Max code length: 15 bits

### JPEG

- Uses canonical Huffman
- Separate tables for DC and AC coefficients

### Brotli

- Uses canonical Huffman
- Prefix codes for various symbol types

---

## Length-Limited Huffman Codes

Sometimes we need maximum length ≤ L.

### Problem

Standard Huffman might produce codes longer than desired.

### Package-Merge Algorithm

1. Creates optimal length-limited codes
2. More complex than standard Huffman
3. Guarantees max length constraint

### DEFLATE's Approach

- Maximum length: 15 bits
- If standard Huffman exceeds this, modify tree

---

## Key Takeaways

1. Canonical Huffman assigns codes using standardized rules
2. Only need code lengths to reconstruct codes
3. Much smaller headers than standard Huffman
4. Enables fast table-based decoding
5. Same optimality as standard Huffman
6. Used in all modern Huffman implementations

---

**Practice**:

1. Given lengths [2, 2, 3, 3, 3, 3], compute canonical codes
2. Build a decoding lookup table
3. Implement encoder and decoder
4. Compare header size: full tree vs lengths only

---

**Next Chapter**: [Adaptive Huffman](./08_adaptive_huffman.md)
