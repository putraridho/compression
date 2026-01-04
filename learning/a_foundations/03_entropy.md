# Chapter 3: Entropy

## What is Entropy?

**Entropy** is the average information content per symbol in a source.

It tells you the **minimum average bits** needed to encode each symbol — the theoretical compression limit.

---

## Shannon Entropy Formula

For a source with alphabet {x₁, x₂, ..., xₙ} and probabilities {p₁, p₂, ..., pₙ}:

```
H(X) = -Σ pᵢ × log₂(pᵢ)
```

Or written out:

```
H(X) = -(p₁×log₂(p₁) + p₂×log₂(p₂) + ... + pₙ×log₂(pₙ))
```

**Unit**: bits per symbol

---

## Calculating Entropy: Examples

### Example 1: Fair Coin

Alphabet: {heads, tails}
Probabilities: P(H) = 0.5, P(T) = 0.5

```
H = -(0.5 × log₂(0.5) + 0.5 × log₂(0.5))
H = -(0.5 × (-1) + 0.5 × (-1))
H = -(-0.5 - 0.5)
H = 1 bit per flip
```

A fair coin has **maximum entropy** for 2 symbols.

### Example 2: Biased Coin

P(H) = 0.9, P(T) = 0.1

```
H = -(0.9 × log₂(0.9) + 0.1 × log₂(0.1))
H = -(0.9 × (-0.152) + 0.1 × (-3.322))
H = -(-0.137 - 0.332)
H = 0.469 bits per flip
```

Less than 1 bit! The predictability reduces entropy.

### Example 3: Certain Outcome

P(H) = 1.0, P(T) = 0.0

```
H = -(1.0 × log₂(1.0) + 0 × log₂(0))
H = -(1.0 × 0 + 0)
H = 0 bits
```

No uncertainty = no information = zero entropy.

(Note: By convention, 0 × log(0) = 0)

---

## Entropy Properties

### Property 1: Non-Negative

```
H(X) ≥ 0
```

Entropy is never negative. You can't have negative information.

### Property 2: Maximum Entropy

For an alphabet of size n, entropy is maximized when all symbols are equally likely:

```
H_max = log₂(n)
```

| Alphabet Size | Max Entropy |
|--------------|-------------|
| 2 (binary) | 1 bit |
| 4 | 2 bits |
| 8 | 3 bits |
| 256 (bytes) | 8 bits |

### Property 3: Minimum Entropy

Entropy is minimized (zero) when one symbol has probability 1.

### Property 4: Concavity

Small changes in probability near 0.5 affect entropy less than changes near 0 or 1.

---

## Entropy as Compression Limit

**Shannon's Source Coding Theorem**:

> The average code length for a source cannot be less than its entropy.

```
Average bits per symbol ≥ H(X)
```

This means:
- **Entropy is the theoretical floor**
- No lossless compression algorithm can beat it
- You can get arbitrarily close, but never below

### Practical Implication

If text has entropy of 4.5 bits per character:
- Minimum compressed size ≈ 4.5 bits × number of characters
- Original ASCII uses 8 bits per character
- Maximum compression ratio ≈ 8/4.5 ≈ 1.78x

---

## Entropy of Real Data

### English Text

Different models give different entropy estimates:

| Model | Entropy (bits/char) |
|-------|-------------------|
| Single letters (no context) | ~4.5 |
| Letter pairs (bigrams) | ~3.5 |
| Word-level | ~2.5 |
| Full language model | ~1.0 - 1.5 |

**Context reduces entropy.** After seeing "th", 'e' is very likely.

### Random Data

Random bytes: H = 8 bits per byte (maximum)

**Random data cannot be compressed** — it's already at maximum entropy.

### Compressed/Encrypted Data

Already-compressed or encrypted data appears random:
- Entropy ≈ 8 bits per byte
- Cannot be compressed further

---

## Joint and Conditional Entropy

### Joint Entropy

Entropy of two random variables together:

```
H(X,Y) = -Σ Σ p(x,y) × log₂(p(x,y))
```

### Conditional Entropy

Entropy of X given we know Y:

```
H(X|Y) = H(X,Y) - H(Y)
```

**Knowing Y reduces uncertainty about X** (usually).

### Chain Rule

```
H(X,Y) = H(X) + H(Y|X) = H(Y) + H(X|Y)
```

---

## Entropy Rate

For sequences (like text), we care about **entropy rate**:

```
H_rate = lim(n→∞) H(X₁, X₂, ..., Xₙ) / n
```

This is the per-symbol entropy considering all dependencies.

For English text:
- Zeroth-order (independent letters): ~4.5 bits/char
- First-order (pairs): ~3.5 bits/char
- Higher-order (more context): approaches ~1 bit/char

**Better context modeling = lower entropy = better compression.**

---

## Cross-Entropy

When you use a model Q to encode data from distribution P:

```
H(P,Q) = -Σ p(x) × log₂(q(x))
```

Cross-entropy is always ≥ entropy:

```
H(P,Q) ≥ H(P)
```

The difference is called **KL divergence** — the "cost" of using the wrong model.

### Practical Meaning

If your compression model doesn't match the true data distribution, you waste bits. Good compression = good modeling.

---

## Redundancy

**Redundancy** measures how far a source is from maximum entropy:

```
Redundancy = H_max - H_actual
```

Or as a rate:

```
Redundancy rate = 1 - (H_actual / H_max)
```

High redundancy = high compressibility.

English text has ~75% redundancy (entropy ~2 bits vs max 8 bits per byte).

---

## Entropy Estimation

You rarely know true probabilities. You must estimate from data.

### Empirical Entropy

Count symbol frequencies and compute:

```
Ĥ = -Σ (count_i / total) × log₂(count_i / total)
```

### Limitations

- Needs enough data for accurate frequencies
- Doesn't capture dependencies (without extensions)
- Real entropy might be lower due to structure

---

## Summary

| Concept | Formula | Meaning |
|---------|---------|---------|
| Information | I(x) = -log₂(p(x)) | Bits for one event |
| Entropy | H(X) = -Σ p log₂ p | Average bits per symbol |
| Max entropy | log₂(n) | When all n symbols equally likely |
| Conditional entropy | H(X\|Y) | Uncertainty in X knowing Y |
| Cross-entropy | H(P,Q) | Cost of wrong model |

---

## Key Takeaways

1. **Entropy = average information = compression limit**
2. Maximum entropy when all symbols equally likely
3. Zero entropy when outcome is certain
4. Context reduces entropy (enables better compression)
5. Random data has maximum entropy (incompressible)
6. Good compression requires good probability modeling

---

**Next Chapter**: [Redundancy](./04_redundancy.md) - Types of redundancy and how compression exploits them
