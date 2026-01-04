# Chapter 5: Theoretical Limits

## Shannon's Source Coding Theorem

The most important theorem in compression:

> **For a source with entropy H, you cannot encode messages using fewer than H bits per symbol on average. Conversely, you can get arbitrarily close to H bits per symbol.**

### What This Means

1. **Entropy is the floor**: No lossless algorithm can beat it
2. **You can approach it**: Good algorithms get close
3. **It's an average**: Individual symbols might use more or fewer bits

### Mathematical Statement

For a source X with entropy H(X):

```
L_avg ≥ H(X)
```

Where L_avg is the average code length.

Furthermore, there exists a code with:

```
H(X) ≤ L_avg < H(X) + 1
```

---

## Kraft Inequality

For any uniquely decodable code with codeword lengths l₁, l₂, ..., lₙ:

```
Σ 2^(-lᵢ) ≤ 1
```

### What This Means

- You can't make all codewords short
- If some symbols get short codes, others must get long codes
- This is a fundamental constraint on prefix codes

### Example

For lengths {1, 2, 2}:
```
2^(-1) + 2^(-2) + 2^(-2) = 0.5 + 0.25 + 0.25 = 1.0 ✓
```

This satisfies Kraft inequality, so a valid code exists.

For lengths {1, 1, 2}:
```
2^(-1) + 2^(-1) + 2^(-2) = 0.5 + 0.5 + 0.25 = 1.25 ✗
```

No valid code with these lengths exists.

---

## The Limits of Lossless Compression

### You Cannot Compress All Files

**Counting argument**:
- There are 2^n possible n-bit files
- If compression reduced all n-bit files to (n-1) bits or less
- That's at most 2^(n-1) + 2^(n-2) + ... + 2 + 1 = 2^n - 1 outputs
- But we have 2^n inputs
- By pigeonhole principle: some inputs must map to same output
- That means we can't decompress uniquely

**Conclusion**: Any compression algorithm must make some files larger.

### Incompressible Strings

For any length n:
- At least half of all n-bit strings cannot be compressed by even 1 bit
- Almost all strings are incompressible
- Compressible strings are the exception, not the rule

### Kolmogorov Complexity

The Kolmogorov complexity K(x) of a string x is the length of the shortest program that outputs x.

- K(x) measures the "true" information content
- K(x) is uncomputable (no algorithm can calculate it)
- For most strings: K(x) ≈ |x| (incompressible)
- For structured strings: K(x) << |x| (compressible)

---

## Rate-Distortion Theory (Lossy Limits)

For lossy compression, Shannon also established limits.

### Rate-Distortion Function R(D)

The minimum bitrate R needed to achieve distortion D:

```
R(D) = min I(X;Y)  such that  E[d(X,Y)] ≤ D
```

Where:
- I(X;Y) is mutual information
- d(X,Y) is a distortion measure (e.g., mean squared error)
- D is the maximum allowed distortion

### Key Insights

1. **You can't have zero distortion at finite rate** (for continuous sources)
2. **More bits = less distortion** (quality vs size tradeoff)
3. **There's a minimum rate for any target quality**

### Example: Gaussian Source

For a Gaussian source with variance σ²:

```
R(D) = (1/2) log₂(σ²/D)  for D ≤ σ²
R(D) = 0                  for D > σ²
```

To halve distortion, you need 0.5 more bits per sample.

---

## Practical Gaps from Theory

Real compression algorithms don't achieve theoretical limits:

### Why We Don't Hit the Limit

1. **Model mismatch**: We don't know the true distribution
2. **Finite block length**: Theory assumes infinite sequences
3. **Computational constraints**: Optimal coding may be too slow
4. **Integer bit constraint**: Can't use fractional bits directly

### How Close Do We Get?

| Algorithm | Gap from entropy |
|-----------|-----------------|
| Huffman coding | < 1 bit/symbol |
| Arithmetic coding | < 0.01 bits/symbol |
| ANS | < 0.01 bits/symbol |
| Context mixing (PAQ) | Near-optimal |

Modern entropy coders are essentially optimal. The challenge is modeling.

---

## The Modeling Problem

Entropy coding only removes statistical redundancy **given a model**.

```
Actual compression = Model quality + Coder efficiency
```

### Perfect Model

If you knew exactly P(next symbol | context):
- Entropy coder would achieve H bits/symbol
- This is the theoretical minimum

### Imperfect Model

With wrong probability estimates:
- You achieve cross-entropy H(P,Q) > H(P)
- The gap is KL divergence: D_KL(P||Q)

### Adaptive Models

Learn probabilities from data as you compress:
- Start with uniform (poor model)
- Update as patterns emerge
- Converge toward true distribution

---

## Universal Compression

**Universal** algorithms work well without knowing the source distribution.

### Asymptotic Optimality

A universal algorithm is **asymptotically optimal** if:

```
lim (n→∞) L_n / n = H
```

Where L_n is the code length for n symbols.

### Examples of Universal Compressors

1. **LZ77/LZ78**: Dictionary methods
2. **Adaptive Huffman**: Update tree as you go
3. **PPM**: Context modeling
4. **Context mixing**: Combine multiple models

These don't need to know P(x) in advance.

---

## Summary: The Three Laws of Compression

### First Law: Conservation
You cannot compress all possible inputs. Saving space on some inputs costs space on others.

### Second Law: Entropy Bound
Average compression cannot exceed the entropy limit. No algorithm beats H bits/symbol.

### Third Law: Lossy Tradeoff
For lossy compression, rate and distortion trade off. Lower rate means higher distortion.

---

## Practical Implications

| Principle | Practical meaning |
|-----------|-------------------|
| Entropy bound | Don't expect miracles; know your data's entropy |
| Incompressibility | Random/encrypted data won't compress |
| Model matters | Better prediction = better compression |
| Universal is possible | Algorithms can adapt to unknown sources |
| Rate-distortion | Choose quality level based on bitrate budget |

---

## Key Formulas

| Concept | Formula |
|---------|---------|
| Shannon entropy | H(X) = -Σ p(x) log₂ p(x) |
| Source coding bound | L_avg ≥ H(X) |
| Kraft inequality | Σ 2^(-lᵢ) ≤ 1 |
| Cross-entropy | H(P,Q) = -Σ p(x) log₂ q(x) |
| KL divergence | D_KL(P\|\|Q) = H(P,Q) - H(P) |

---

## End of Part A

You now understand:

1. What compression is and why it works
2. How information is measured (bits)
3. What entropy means and how to calculate it
4. Types of redundancy and how to exploit them
5. Theoretical limits that no algorithm can beat

You're ready for **Part B: Lossless Techniques** — where you'll learn specific algorithms to implement.

---

**Next Part**: [Part B: Lossless Techniques](../b_lossless/README.md)
