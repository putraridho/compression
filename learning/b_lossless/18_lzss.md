# Chapter 18: LZSS

## The Problem with LZ77

Original LZ77 uses tuples (offset, length, next_char) for everything:

```
No match: (0, 0, 'X')  ← 3 values to output 1 byte!
```

This is wasteful for non-matching bytes.

---

## LZSS Improvement

Storer and Szymanski (1982) proposed LZSS:

**Use a flag bit to distinguish:**
- 0 = literal byte follows
- 1 = match reference follows

```
LZ77:  (0,0,'A')(0,0,'B')(0,0,'C')(3,3,'X')
LZSS:  0'A' 0'B' 0'C' 1(3,3) 0'X'
```

Much more compact!

---

## The Algorithm

### Encoding

```
For each position:
    Find longest match

    If match.length >= minimum_match:
        Output: 1 (flag) + offset + length
    Else:
        Output: 0 (flag) + literal_byte

    Advance position
```

### Minimum Match Length

Since references cost bits (flag + offset + length), short matches may not save space.

```
Reference cost: 1 + log₂(window) + log₂(max_length) bits
Literal cost: 1 + 8 bits

Break-even when: match_length × 8 > reference_cost
```

Typical minimum: 3-4 bytes

---

## Step-by-Step Example

### Input

```
"ABCABCABC"
```

### Parameters

Window: 8 bytes
Minimum match: 3

### Trace

```
Pos 0: No history, output literal 'A'
       Output: 0 01000001

Pos 1: No match, output literal 'B'
       Output: 0 01000010

Pos 2: No match, output literal 'C'
       Output: 0 01000011

Pos 3: Match "ABC" at offset 3, length 3
       Length ≥ 3, output match
       Output: 1 [offset=3] [length=3]

Pos 6: Match "ABC" at offset 3, length 3
       Output: 1 [offset=3] [length=3]

Done!
```

### Encoded

```
0'A' 0'B' 0'C' 1(3,3) 1(3,3)
```

Compare to LZ77:
```
(0,0,A)(0,0,B)(0,0,C)(3,2,A)(3,2,C)  ← 5 full tuples
```

LZSS is more compact.

---

## Encoding the Match

### Fixed-Width

```
Offset: log₂(window_size) bits
Length: log₂(max_length) bits

Example (window=4096, max_length=256):
Offset: 12 bits
Length: 8 bits
Match: 1 + 12 + 8 = 21 bits
```

### Variable-Width

Use shorter codes for common values:
- Small offsets (recent matches) get short codes
- Common lengths get short codes

This is what DEFLATE does.

---

## Decoding

```
function decode(bitstream):
    output = []

    while not end_of_stream:
        flag = read_bit()

        if flag == 0:
            byte = read_bits(8)
            output.append(byte)
        else:
            offset = read_offset()
            length = read_length()

            start = len(output) - offset
            for i = 0 to length - 1:
                output.append(output[start + i])

    return output
```

Note: Same overlapping copy logic as LZ77.

---

## Lazy Matching

### The Problem

Greedy matching isn't always optimal:

```
Input: "ABCDBCDE"
Position 4: Match "BCD" at offset 3, length 3

But if we skip to position 5:
Position 5: Match "BCDE" at offset 3, length 4!
```

The longer match was hiding behind a shorter one.

### Lazy Evaluation

```
1. Find match at current position
2. Check if match at position+1 is better
3. If yes: output literal, advance 1, use longer match
4. If no: use current match
```

### Cost

More comparisons, but better compression.

DEFLATE implements lazy matching.

---

## Match Finding Strategies

### Greedy

Take first match of minimum length.
Fast, decent compression.

### Lazy

Check next position for better match.
Slower, better compression.

### Optimal Parsing

Consider all possible parsings, choose best.
Very slow, best compression.

```
Dynamic programming:
  cost[i] = min cost to encode positions 0..i
  cost[i] = min(
      cost[i-1] + literal_cost,
      cost[i-len] + match_cost for all valid matches
  )
```

Used in Zopfli (optimal DEFLATE).

---

## LZSS vs LZ77

| Aspect | LZ77 | LZSS |
|--------|------|------|
| Literal encoding | Full tuple | Flag + byte |
| Match encoding | (offset, length, char) | Flag + offset + length |
| Efficiency | Wastes bits on literals | Efficient for both |
| Complexity | Simpler | Slightly more complex |

LZSS strictly dominates LZ77 in compression ratio.

---

## Pseudocode

### Encoder

```
function lzss_encode(input):
    pos = 0

    while pos < length(input):
        match = find_longest_match(input, pos)

        if match.length >= MIN_MATCH:
            output_bit(1)
            output_offset(match.offset)
            output_length(match.length)
            pos += match.length
        else:
            output_bit(0)
            output_byte(input[pos])
            pos += 1
```

### Encoder with Lazy Matching

```
function lzss_encode_lazy(input):
    pos = 0
    pending_match = null

    while pos < length(input):
        match = find_longest_match(input, pos)

        if pending_match:
            if match.length > pending_match.length + 1:
                // Better match ahead, output pending as literal
                output_bit(0)
                output_byte(input[pos - 1])
            else:
                // Use pending match
                output_bit(1)
                output_match(pending_match)
                pos += pending_match.length - 1
                pending_match = null
                continue

        if match.length >= MIN_MATCH:
            pending_match = match
        else:
            output_bit(0)
            output_byte(input[pos])

        pos += 1

    if pending_match:
        output_bit(1)
        output_match(pending_match)
```

---

## Real-World Usage

LZSS concepts are in:
- DEFLATE (gzip, ZIP, PNG)
- LZ4
- Snappy
- Many game compression formats

Usually enhanced with:
- Variable-length offset/length encoding
- Entropy coding of tokens

---

## Key Takeaways

1. LZSS uses flag bits to separate literals from matches
2. More efficient than LZ77 for non-matching data
3. Minimum match length prevents wasteful references
4. Lazy matching improves compression at cost of speed
5. Optimal parsing gives best results (slow)
6. Foundation for DEFLATE and modern compressors

---

**Practice**:

1. Encode "ABRACADABRA" with LZSS
2. Compare output size to LZ77
3. Implement lazy matching
4. Try optimal parsing on small inputs

---

**Next Chapter**: [LZ78](./19_lz78.md)
