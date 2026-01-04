# Chapter 21: LZMA

## Overview

**LZMA** (Lempel-Ziv-Markov chain Algorithm) is the algorithm behind 7-Zip. Created by Igor Pavlov in 1998.

It achieves some of the best compression ratios among practical algorithms.

---

## What Makes LZMA Special

LZMA combines:

1. **LZ77-style dictionary** (large window, up to 4GB)
2. **Range coding** (arithmetic coding variant)
3. **Markov chain models** for context prediction
4. **Sophisticated match encoding**

---

## Architecture

```
Input → LZ77 Parser → Tokens → Markov Models → Range Coder → Output
```

### Token Types

1. **Literal**: A single byte
2. **Match**: (distance, length) pair
3. **Short rep**: Repeat last distance
4. **Long rep**: Repeat one of last 4 distances

---

## The State Machine

LZMA uses a state machine with ~12 states.

State depends on last few operations:
- Was last output a literal or match?
- How long was last match?
- What came before?

Each state has different probability models.

---

## Match Encoding

### Distance Encoding

Distances are encoded in slots:

```
Slot 0-3:   Distance = slot
Slot 4-5:   Distance = 2-3 bits + slot
Slot 6-13:  Distance = 2-7 bits + slot
Slot 14+:   Distance = aligned bits + slot
```

### Length Encoding

Lengths 2-9 use small model
Lengths 10-17 use medium model
Lengths 18-273 use large model

---

## Context Modeling

### Literal Context

Based on:
- Previous byte (match byte if after match)
- Position in stream
- High bits of previous byte

### Match Context

Based on:
- Current state
- Previous match distances
- Position bits

---

## Range Coder

LZMA uses range coding (Chapter 10) for entropy coding.

Each probability is stored as 11-bit value:
```
0 = 0%
2048 = 100%
```

Probabilities adapt: increase after 0, decrease after 1.

---

## Probability Update

After each bit:

```
If bit == 0:
    prob = prob + ((2048 - prob) >> 5)
Else:
    prob = prob - (prob >> 5)
```

Moves 1/32 of the way toward observed frequency.

---

## The Markov Chains

State transitions:

```
State 0-6:  Literal states
State 7-9:  Short match states
State 10-11: Long match states

Transitions depend on operation type and length.
```

Each state has its own probability tables.

---

## Match Finding

LZMA supports:

1. **Hash chains**: Fast, lower compression
2. **Binary tree**: Balanced speed/compression
3. **BT4**: Best compression, slower

### BT4 (Binary Tree with 4-byte hash)

```
1. Hash first 4 bytes
2. Insert into binary tree at that hash
3. Search tree for best match
4. Left/right based on comparison
```

Provides optimal matches at reasonable speed.

---

## LZMA vs DEFLATE

| Aspect | DEFLATE | LZMA |
|--------|---------|------|
| Window | 32 KB | Up to 4 GB |
| Entropy coder | Huffman | Range coding |
| Context model | None | Markov chains |
| Match distances | 4 recent | 4 recent |
| Compression | Good | Excellent |
| Speed | Fast | Slower |
| Memory | Low | High |

---

## LZMA2

LZMA2 is a container format over LZMA:

1. **Chunks**: Divides data into chunks
2. **Reset capability**: Can reset state between chunks
3. **Parallel friendly**: Chunks can be processed in parallel
4. **Uncompressed chunks**: Can store incompressible data

Used in .xz and 7z formats.

---

## Parameters

### Dictionary Size

```
-d=N: Dictionary size = 2^N bytes

Common values:
  -d=16: 64 KB (fast, less compression)
  -d=20: 1 MB (balanced)
  -d=24: 16 MB (better compression)
  -d=26: 64 MB (high compression)
```

### Literal Context Bits

```
-lc=N: Use N bits of previous byte for context
       0-8, default 3
```

### Literal Position Bits

```
-lp=N: Use N bits of position for context
       0-4, default 0
```

### Position Bits

```
-pb=N: Use N bits of position for match context
       0-4, default 2
```

---

## Compression Levels

| Level | Dictionary | Match Finder | Description |
|-------|------------|--------------|-------------|
| 1 | 1 MB | HC4 | Fast |
| 3 | 4 MB | HC4 | Fast |
| 5 | 16 MB | BT4 | Normal |
| 7 | 32 MB | BT4 | Maximum |
| 9 | 64 MB | BT4 | Ultra |

---

## Typical Compression Ratios

| Data Type | LZMA Ratio | gzip Ratio |
|-----------|------------|------------|
| Text | 4:1 | 2.5:1 |
| Source code | 5:1 | 3:1 |
| Binary | 3:1 | 2:1 |
| Already compressed | 1:1 | 1:1 |

LZMA typically beats gzip by 30-50%.

---

## Pseudocode Sketch

### Encoder (Simplified)

```
function lzma_encode(input):
    state = 0
    rep[4] = {0, 0, 0, 0}  // Last 4 distances
    range_encoder.init()

    pos = 0
    while pos < length(input):
        match = find_best_match(input, pos)

        if should_use_literal(match, state):
            encode_literal(input[pos], state, pos)
            state = update_state_literal(state)
            pos += 1

        else if match.distance == rep[0]:
            encode_short_rep(match.length, state, pos)
            state = update_state_rep(state, match.length)
            pos += match.length

        else if match.distance in rep[1:4]:
            encode_long_rep(match, state, pos)
            rotate_reps(match.distance)
            state = update_state_rep(state, match.length)
            pos += match.length

        else:
            encode_match(match, state, pos)
            shift_reps(match.distance)
            state = update_state_match(state, match.length)
            pos += match.length

    range_encoder.finish()
```

---

## Decompression

Decompression mirrors encoding:

```
function lzma_decode():
    state = 0
    rep[4] = {0, 0, 0, 0}
    range_decoder.init()
    output = []

    while not end:
        if decode_is_literal(state, pos):
            byte = decode_literal(state, pos)
            output.append(byte)
            state = update_state_literal(state)

        else if decode_is_rep(state, pos):
            (which_rep, length) = decode_rep(state, pos)
            copy_from_output(output, rep[which_rep], length)
            rotate_reps(which_rep)
            state = update_state_rep(state, length)

        else:
            (distance, length) = decode_match(state, pos)
            copy_from_output(output, distance, length)
            shift_reps(distance)
            state = update_state_match(state, length)

    return output
```

---

## Why LZMA Is Good

1. **Large dictionaries**: Finds matches far back
2. **Adaptive modeling**: Probabilities match data
3. **Context awareness**: Different models for different situations
4. **Optimal parsing**: Considers multiple encodings
5. **Range coding**: Fractional bits

---

## Key Takeaways

1. Combines LZ77 + Range coding + Markov models
2. Much better compression than DEFLATE
3. Slower compression, similar decompression
4. Uses large dictionaries (up to 4GB)
5. Complex state machine with context modeling
6. Used in 7z, xz formats

---

**Practice**:

1. Use 7z to compress files at different levels
2. Compare compression ratios to gzip
3. Observe memory usage during compression
4. Study the LZMA SDK source code

---

**Next Chapter**: [Modern LZ Variants](./22_modern_lz.md)
