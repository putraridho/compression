# Chapter 9: Arithmetic Coding

## The Limitation of Huffman

Huffman coding assigns integer bits per symbol.

```
If P(A) = 0.99, optimal code length = -log₂(0.99) ≈ 0.014 bits
But Huffman must use at least 1 bit.

Waste: 1 - 0.014 = 0.986 bits per symbol!
```

---

## Arithmetic Coding: The Breakthrough

Arithmetic coding encodes an **entire message** as a single number between 0 and 1.

- Uses fractional bits per symbol
- Approaches entropy arbitrarily closely
- One of the most important compression discoveries

---

## The Core Idea

### The Interval [0, 1)

Start with interval [0, 1).

Each symbol **subdivides** the current interval proportionally to its probability.

```
Alphabet: {A, B, C}
Probabilities: P(A)=0.5, P(B)=0.3, P(C)=0.2

Initial interval: [0, 1)

Subdivisions:
A: [0.0, 0.5)
B: [0.5, 0.8)
C: [0.8, 1.0)
```

### Encoding a Message

For each symbol, take its subinterval of the current interval.

**Example: Encode "BAC"**

```
Start: [0, 1)

Symbol 'B': Take B's portion [0.5, 0.8)
    Current interval: [0.5, 0.8)
    Width: 0.3

Symbol 'A': Take A's portion (first 50%)
    Low = 0.5 + 0 × 0.3 = 0.5
    High = 0.5 + 0.5 × 0.3 = 0.65
    Current interval: [0.5, 0.65)
    Width: 0.15

Symbol 'C': Take C's portion (last 20%)
    Low = 0.5 + 0.8 × 0.15 = 0.62
    High = 0.5 + 1.0 × 0.15 = 0.65
    Current interval: [0.62, 0.65)
```

**Final interval: [0.62, 0.65)**

Any number in this interval uniquely represents "BAC".

Output: 0.63 (or any value in [0.62, 0.65))

---

## Mathematical Formulation

### Cumulative Distribution

Define cumulative probabilities:

```
F(symbol) = sum of probabilities of all symbols before this one

For A, B, C with P(A)=0.5, P(B)=0.3, P(C)=0.2:
    F(A) = 0
    F(B) = 0.5
    F(C) = 0.8
```

### Update Rules

Given current interval [low, high) with width = high - low:

```
For symbol S:
    new_low = low + width × F(S)
    new_high = low + width × (F(S) + P(S))
```

---

## Decoding

Given encoded number and original probabilities:

1. Find which symbol's range contains the number
2. Rescale number to that symbol's interval
3. Repeat

**Example: Decode 0.63**

```
Probabilities: A=[0, 0.5), B=[0.5, 0.8), C=[0.8, 1.0)

Step 1: 0.63 falls in [0.5, 0.8) → 'B'
    Rescale: (0.63 - 0.5) / 0.3 = 0.433...

Step 2: 0.433 falls in [0, 0.5) → 'A'
    Rescale: (0.433 - 0) / 0.5 = 0.866...

Step 3: 0.866 falls in [0.8, 1.0) → 'C'
    Rescale: (0.866 - 0.8) / 0.2 = 0.33...

Decoded: "BAC"
```

---

## Why It's Efficient

### Bits Used

The interval width after encoding n symbols:

```
width = P(s₁) × P(s₂) × ... × P(sₙ)
```

Bits needed to specify a number in this interval:

```
bits ≈ -log₂(width) = -log₂(P(s₁)) - log₂(P(s₂)) - ... - log₂(P(sₙ))
     = I(s₁) + I(s₂) + ... + I(sₙ)
```

This is exactly the sum of information contents!

### Per-Symbol Efficiency

Average bits per symbol approaches entropy as message length increases.

```
lim(n→∞) bits/n = H
```

---

## Practical Considerations

### Precision Problem

Intervals get very small. Can't use infinite precision.

**Solution**: Fixed-precision arithmetic with renormalization.

### Renormalization

When interval fits entirely in [0, 0.5) or [0.5, 1):

1. Output the known bit (0 or 1)
2. Double the interval (zoom in)
3. Continue encoding

```
If interval is [0.2, 0.3):
    Both ends < 0.5
    Output: 0
    Rescale: [0.4, 0.6)

If interval is [0.7, 0.9):
    Both ends ≥ 0.5
    Output: 1
    Rescale: [0.4, 0.8)
```

### Underflow Handling

When interval straddles 0.5 (like [0.4, 0.6)):

1. Can't output a bit yet
2. Zoom into middle: [0.3, 0.7)
3. Keep count of pending bits
4. When finally resolved, output pending bits inverted

---

## Integer Arithmetic Implementation

Use integers instead of floats for precision.

```
Scale [0, 1) to [0, 2^32)

Low = 0
High = 0xFFFFFFFF

Update:
    range = High - Low + 1
    High = Low + range × CDF(symbol+1) / total - 1
    Low = Low + range × CDF(symbol) / total
```

---

## Pseudocode

### Encoder

```
function encode(symbols, probabilities):
    low = 0
    high = MAX_VALUE
    pending_bits = 0

    for symbol in symbols:
        range = high - low + 1
        high = low + range × cum_prob(symbol + 1) / total - 1
        low = low + range × cum_prob(symbol) / total

        while true:
            if high < HALF:
                output_bit(0)
                output_pending_bits(1)
                pending_bits = 0
            else if low >= HALF:
                output_bit(1)
                output_pending_bits(0)
                pending_bits = 0
                low -= HALF
                high -= HALF
            else if low >= QUARTER and high < THREE_QUARTER:
                pending_bits += 1
                low -= QUARTER
                high -= QUARTER
            else:
                break

            low = 2 × low
            high = 2 × high + 1

    // Flush remaining bits
    pending_bits += 1
    if low < QUARTER:
        output_bit(0)
        output_pending_bits(1)
    else:
        output_bit(1)
        output_pending_bits(0)
```

### Decoder

```
function decode(bits, probabilities, length):
    low = 0
    high = MAX_VALUE
    value = read_initial_bits()

    output = []

    for i = 1 to length:
        range = high - low + 1
        scaled = ((value - low + 1) × total - 1) / range

        // Find symbol
        symbol = find_symbol(scaled, cum_prob)
        output.append(symbol)

        // Update interval
        high = low + range × cum_prob(symbol + 1) / total - 1
        low = low + range × cum_prob(symbol) / total

        // Renormalize
        while true:
            if high < HALF:
                // Nothing
            else if low >= HALF:
                value -= HALF
                low -= HALF
                high -= HALF
            else if low >= QUARTER and high < THREE_QUARTER:
                value -= QUARTER
                low -= QUARTER
                high -= QUARTER
            else:
                break

            low = 2 × low
            high = 2 × high + 1
            value = 2 × value + read_bit()

    return output
```

---

## Comparison with Huffman

| Aspect | Huffman | Arithmetic |
|--------|---------|------------|
| Bits per symbol | Integer ≥ 1 | Fractional |
| Optimality | Optimal for integer codes | Approaches entropy |
| Speed | Faster | Slower |
| Complexity | Simpler | More complex |
| Adaptivity | Harder | Natural |
| Patents | None | Were patented (expired) |

### When Arithmetic Wins

- Highly skewed probabilities (P > 0.5 for some symbol)
- Large alphabets
- When every bit matters
- Adaptive coding needed

### When Huffman Wins

- Speed is critical
- Simple implementation needed
- Probabilities are balanced

---

## Binary Arithmetic Coding

Special case for two-symbol alphabet:

- Only two probabilities: P and 1-P
- Used in context-based coders
- Foundation for modern codecs

### Q-Coder, QM-Coder, MQ-Coder

Approximations for fast implementation:
- Avoid multiplications
- Use table lookups
- Used in JBIG, JPEG 2000

---

## Real-World Usage

| Application | Notes |
|-------------|-------|
| JPEG 2000 | MQ-coder |
| H.264/H.265 | CABAC (binary arithmetic) |
| LZMA/7z | Range coder variant |
| bzip2 | After MTF |
| PPM compressors | High compression |
| WebP | VP8 uses boolean coder |

---

## Key Takeaways

1. Encodes entire message as single number
2. Achieves fractional bits per symbol
3. Approaches entropy as message grows
4. Requires careful precision handling
5. Slower but better than Huffman for skewed distributions
6. Foundation of modern video codecs (CABAC)

---

**Practice**:

1. Hand-trace encoding "ABBA" with P(A)=0.6, P(B)=0.4
2. Decode a given number back to message
3. Implement with integer arithmetic
4. Compare compression to Huffman on skewed data

---

**Next Chapter**: [Range Coding](./10_range_coding.md)
