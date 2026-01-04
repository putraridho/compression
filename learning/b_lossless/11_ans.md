# Chapter 11: Asymmetric Numeral Systems (ANS)

## The Revolution

ANS, invented by Jarek Duda in 2009, combines:

- **Compression of arithmetic coding**
- **Speed of Huffman coding**

It's now used in Zstandard, LZFSE, and many modern compressors.

---

## The Core Insight

### Numeral Systems Encode Information

Consider base-10 numbers:
```
352 = 3×100 + 5×10 + 2×1
```

Each digit encodes log₂(10) ≈ 3.32 bits.

### What if Digits Had Different Weights?

Instead of equal-weight digits, what if:
- Some digits are "cheaper" (appear more often)
- Some digits are "expensive" (appear rarely)

This is exactly what ANS does!

---

## ANS Basic Concept

### State Machine

ANS encodes a message as a single integer state x.

- Start with initial state x₀
- For each symbol, update state
- Final state x encodes entire message

### Encoding Function

```
x' = encode(x, symbol)
```

The function interleaves bits from x with bits representing the symbol.

### Decoding Function

```
(x, symbol) = decode(x')
```

Extracts the last symbol and restores previous state.

---

## uANS (Uniform ANS)

Simplest version: all symbols equally likely.

### Example: Binary Alphabet

Symbols: {0, 1} with P = 0.5 each

```
Encode 0: x' = 2x      (multiply by 2)
Encode 1: x' = 2x + 1  (multiply by 2, add 1)

Decode: symbol = x' mod 2
        x = x' / 2
```

This is just binary representation!

### Example: 4-Symbol Alphabet

Symbols: {A, B, C, D} with P = 0.25 each

```
Encode A: x' = 4x + 0
Encode B: x' = 4x + 1
Encode C: x' = 4x + 2
Encode D: x' = 4x + 3

Decode: symbol = x' mod 4
        x = x' / 4
```

Average bits/symbol = log₂(4) = 2 bits. Optimal!

---

## rANS (Range ANS)

Handles non-uniform probabilities.

### Setup

Total frequency: M (power of 2)
Symbol s has frequency fₛ (where Σfₛ = M)
Cumulative frequency: Fₛ = Σ(frequencies before s)

### Encoding

```
x' = ((x / fₛ) × M) + (x mod fₛ) + Fₛ
```

Or equivalently:
```
x' = ((x / fₛ) << log₂(M)) + (x mod fₛ) + Fₛ
```

### Decoding

```
slot = x' mod M         // Called "alias" or "slot"
s = symbol_for_slot(slot)
x = fₛ × (x' >> log₂(M)) + slot - Fₛ
```

### Example

```
Alphabet: {A, B}
M = 4
Frequencies: A=3, B=1
Cumulative: A=0, B=3

Encoding 'A' from x=5:
x' = ((5 / 3) × 4) + (5 mod 3) + 0
   = (1 × 4) + 2 + 0
   = 6

Decoding x'=6:
slot = 6 mod 4 = 2
symbol = A (slot 2 is in A's range [0,3))
x = 3 × (6 >> 2) + 2 - 0 = 3×1 + 2 = 5 ✓
```

---

## tANS (Table ANS)

Uses precomputed tables for maximum speed.

### The Symbol Table

Build table of size M:
```
For each position i in [0, M):
    If i falls in symbol s's range:
        table[i] = s
```

### State Table

For each (symbol, state_bits) pair, precompute next state.

```
encode_table[state][symbol] = new_state
decode_table[state] = (symbol, prev_state)
```

### Encoding with Tables

```
While x ≥ max_state:
    output_bits(x)
    x = x >> bits

x = encode_table[x][symbol]
```

### Decoding with Tables

```
(symbol, x) = decode_table[x]

While x < min_state:
    x = (x << bits) | read_bits()
```

---

## State Renormalization

Keep state in valid range [L, 2L).

### Encoding Normalization

Before encoding symbol, ensure state is small enough:
```
While x ≥ threshold[s]:
    output_bits(x)  // Output low bits
    x = x >> bits
```

### Decoding Normalization

After decoding symbol, ensure state is large enough:
```
While x < L:
    x = (x << bits) | read_bits()
```

---

## Streaming ANS

For long messages, interleave output.

### Problem

ANS is LIFO (Last In, First Out) — decode in reverse.

### Solution: Interleaved Streams

1. Encode forward, buffer states
2. Write in reverse order
3. Decode reads forward (because written in reverse)

Or: Use multiple interleaved ANS states.

---

## Comparison

| Feature | Huffman | Arithmetic | ANS |
|---------|---------|------------|-----|
| Compression | Good | Excellent | Excellent |
| Encode speed | Fast | Slow | Fast |
| Decode speed | Fast | Slow | Very fast |
| Memory | Low | Low | Tables (moderate) |
| SIMD-friendly | No | No | Yes |

ANS achieves arithmetic-level compression with Huffman-like speed!

---

## Why ANS is Fast

1. **No divisions**: Uses shifts and table lookups
2. **No carries**: Unlike arithmetic coding
3. **SIMD friendly**: Can process multiple symbols in parallel
4. **Cache efficient**: Table fits in L1/L2 cache
5. **Predictable branches**: Easy for CPU prediction

---

## FSE (Finite State Entropy)

Yann Collet's implementation of tANS, used in Zstandard.

### Key Optimizations

1. Table size is power of 2
2. Symbol spreading for good distribution
3. Accurate probability normalization
4. Bitstream interleaving

### Symbol Spreading

Distribute symbols in table to minimize state changes:

```
Instead of: AAABBCC
Spread to:  ACABACB
```

More uniform state distribution.

---

## Pseudocode

### rANS Encoder

```
L = 1 << 23        // Min state
M = 1 << 16        // Total frequency

function encode(x, freq, cum_freq):
    // Normalize
    max_x = ((L >> log₂(M)) << 32) × freq
    while x >= max_x:
        output_16bits(x)
        x = x >> 16

    // Encode
    x = ((x / freq) << log₂(M)) + (x mod freq) + cum_freq
    return x
```

### rANS Decoder

```
function decode(x):
    slot = x & (M - 1)
    s = symbol_table[slot]
    freq = frequencies[s]
    cum_freq = cumulative[s]

    x = freq × (x >> log₂(M)) + slot - cum_freq

    // Renormalize
    while x < L:
        x = (x << 16) | read_16bits()

    return (s, x)
```

---

## Real-World Usage

| Application | Variant |
|-------------|---------|
| Zstandard | FSE (tANS) |
| LZFSE (Apple) | tANS |
| Draco (Google 3D) | rANS |
| CRAM (genomics) | rANS |
| PIK/JPEG XL | ANS |

---

## Building tANS Tables

### Step 1: Normalize Frequencies

Scale to power-of-2 sum:
```
For each symbol:
    normalized[s] = round(freq[s] × M / total)

Adjust to ensure sum = M exactly.
```

### Step 2: Spread Symbols

```
For each symbol s with frequency f:
    Place s at approximately f evenly-spaced positions in table
```

### Step 3: Build State Transitions

```
For each state x in table:
    s = table[x]
    next_x[x] = ... // Precomputed transition
```

---

## Key Takeaways

1. ANS encodes message as single integer
2. Achieves arithmetic coding compression at Huffman speed
3. rANS: uses multiplication, more flexible
4. tANS: uses tables, maximum speed
5. LIFO nature requires reverse or interleaving
6. Foundation of modern compressors (Zstandard)

---

**Practice**:

1. Implement binary uANS
2. Implement rANS with 2-symbol alphabet
3. Build tANS lookup tables
4. Benchmark against Huffman and arithmetic coding

---

**Next Chapter**: [Finite State Entropy (FSE)](./12_fse.md)
