# Chapter 2: Information Theory

## The Birth of Compression Science

In 1948, Claude Shannon published "A Mathematical Theory of Communication" — the paper that created the field of information theory and laid the foundation for all compression algorithms.

Shannon asked: **How do we measure information?**

---

## What is Information?

Information is the **resolution of uncertainty**.

### Intuition

Consider two messages:
1. "The sun will rise tomorrow" — **Low information** (you already knew this)
2. "You won lottery" — **High information** (unexpected, surprising)

**The more surprising a message, the more information it carries.**

### Mathematical Definition

The information content of an event with probability `p` is:

```
I(x) = -log₂(p)
```

This gives us **bits** of information.

---

## Calculating Information Content

### Example 1: Coin Flip

Fair coin: P(heads) = P(tails) = 0.5

```
I(heads) = -log₂(0.5) = -(-1) = 1 bit
```

One fair coin flip gives exactly **1 bit** of information.

### Example 2: Dice Roll

Fair 6-sided die: P(any face) = 1/6

```
I(any face) = -log₂(1/6) = log₂(6) ≈ 2.58 bits
```

A dice roll gives about **2.58 bits** of information.

### Example 3: Biased Coin

Unfair coin: P(heads) = 0.9, P(tails) = 0.1

```
I(heads) = -log₂(0.9) ≈ 0.15 bits  (not surprising)
I(tails) = -log₂(0.1) ≈ 3.32 bits  (very surprising!)
```

**Rare events carry more information than common events.**

---

## Why Logarithms?

The logarithm has special properties perfect for measuring information:

### 1. Additivity

If two independent events occur, their information adds:

```
I(A and B) = I(A) + I(B)
```

Two coin flips = 2 bits. This matches our intuition.

### 2. Certain Events Have Zero Information

```
I(certain event) = -log₂(1) = 0 bits
```

If something is guaranteed to happen, learning it happened tells you nothing.

### 3. Impossible Events Have Infinite Information

```
I(impossible event) = -log₂(0) = ∞ bits
```

Learning something impossible happened would be infinitely surprising.

---

## Why Base 2?

We use log base 2 because:

1. **Computers use binary** — everything is 0s and 1s
2. **1 bit = 1 binary digit** — natural unit for digital systems
3. **Other bases work too** — base `e` gives "nats", base 10 gives "hartleys"

---

## The Communication Model

Shannon defined a general communication system:

```
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌──────────┐
│ Source  │───▶│ Encoder │───▶│ Channel │───▶│ Decoder │───▶│ Receiver │
└─────────┘    └─────────┘    └─────────┘    └─────────┘    └──────────┘
                                  │
                                  ▼
                              [Noise]
```

**Source**: Generates messages (text, images, video)
**Encoder**: Compresses and/or adds error protection
**Channel**: Transmission medium (wire, air, disk)
**Noise**: Interference that corrupts data
**Decoder**: Decompresses and/or corrects errors
**Receiver**: Gets the message

Compression is about making the encoder/decoder efficient.

---

## Source Coding vs Channel Coding

Shannon separated two problems:

### Source Coding (Compression)

Remove redundancy to represent data efficiently.

**Goal**: Minimum bits to represent the source.

### Channel Coding (Error Correction)

Add redundancy to protect against noise.

**Goal**: Reliable transmission over noisy channels.

These are opposite operations:
- Source coding **removes** redundancy
- Channel coding **adds** redundancy

This book focuses on **source coding** (compression).

---

## Symbols and Alphabets

### Alphabet

The set of possible symbols.

- Binary alphabet: {0, 1}
- English alphabet: {a, b, c, ..., z, space, ...}
- Byte alphabet: {0, 1, 2, ..., 255}

### Source

A process that generates symbols from an alphabet.

- English text source generates letters
- Image source generates pixel values
- Audio source generates sample values

### Message

A sequence of symbols from the source.

---

## Probability Distributions

To analyze compression, we need to know symbol probabilities.

### Example: English Text

| Letter | Approximate Probability |
|--------|------------------------|
| e | 12.7% |
| t | 9.1% |
| a | 8.2% |
| o | 7.5% |
| ... | ... |
| z | 0.07% |

**'e' is ~180 times more common than 'z'**

This non-uniform distribution is what makes text compressible.

### Uniform vs Non-Uniform

**Uniform distribution**: All symbols equally likely
- Example: Random bytes (each has 1/256 probability)
- **Cannot be compressed**

**Non-uniform distribution**: Some symbols more likely
- Example: English text, images, most real data
- **Can be compressed**

---

## Key Insight for Compression

If some symbols are more common:
- Give **short codes** to common symbols
- Give **long codes** to rare symbols
- On average, you use fewer bits

This is exactly what Huffman coding and other entropy coders do.

### Example

In English, 'e' appears 127 times per 1000 letters, 'z' appears 0.7 times.

If we give 'e' a 3-bit code and 'z' a 10-bit code (instead of 8 bits each), we save bits overall because 'e' appears so much more often.

---

## Shannon's Fundamental Theorems

### Source Coding Theorem (Compression Limit)

**You cannot compress data below its entropy** (on average).

Entropy (next chapter) sets the absolute minimum bits needed. No algorithm can beat this limit.

### Channel Coding Theorem (Reliable Communication)

**You can transmit reliably over a noisy channel if your rate is below channel capacity.**

This is about error correction, not compression, but shows Shannon's genius in establishing fundamental limits.

---

## Key Terms

| Term | Definition |
|------|------------|
| **Information** | Resolution of uncertainty; surprise value |
| **Bit** | Binary digit; unit of information |
| **Alphabet** | Set of possible symbols |
| **Source** | Process generating symbols |
| **Probability distribution** | Likelihood of each symbol |
| **Source coding** | Compression |
| **Channel coding** | Error correction |

---

## Summary

1. Information = surprise = -log₂(probability)
2. Rare events carry more information than common events
3. Non-uniform distributions enable compression
4. Common symbols should get short codes
5. Shannon proved there are fundamental limits to compression

---

**Next Chapter**: [Entropy](./03_entropy.md) - The average information content and compression limit
