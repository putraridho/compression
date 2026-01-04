# Chapter 30: Context Modeling Introduction

## What is Context Modeling?

**Context modeling** uses surrounding data to predict the next symbol.

Better prediction → lower entropy → better compression.

---

## The Core Idea

```
Without context:
  P(e) = 12.7% (overall English frequency)

With context "th":
  P(e) = 70%+ (much more predictable!)
```

Context provides information about what comes next.

---

## Order of Context

### Order-0

No context. Only symbol frequencies.

```
P(symbol) = count(symbol) / total_symbols
```

### Order-1

One previous symbol as context.

```
P(symbol | prev_symbol)
```

256 different probability tables.

### Order-2

Two previous symbols.

```
P(symbol | prev_2_symbols)
```

65,536 tables.

### Order-N

N previous symbols.

Tables grow exponentially: 256^N

---

## Context Example

### Order-0

```
Text: "the cat sat"

P(t) = 3/10 = 30%
P(e) = 1/10 = 10%
P(h) = 1/10 = 10%
...
```

### Order-1

```
After 't':
  P(h) = 2/3 = 67%
  P(' ') = 1/3 = 33%

After ' ':
  P(c) = 1/3
  P(s) = 1/3
  P(t) = 1/3
```

More predictable!

---

## Entropy Reduction

Context reduces entropy:

```
Order-0 English: ~4.5 bits/char
Order-1 English: ~3.5 bits/char
Order-2 English: ~3.0 bits/char
Order-3 English: ~2.5 bits/char
Order-8 English: ~1.5 bits/char
```

Higher order = lower entropy = better compression.

---

## The Space Problem

### Exponential Growth

| Order | Contexts | Memory (8 bytes each) |
|-------|----------|----------------------|
| 0 | 1 | 8 B |
| 1 | 256 | 2 KB |
| 2 | 65,536 | 512 KB |
| 3 | 16,777,216 | 128 MB |
| 4 | 4,294,967,296 | 32 GB |

Order-4 already impractical!

### Solutions

1. **Escape to lower order**: Fall back when context unseen
2. **Pruning**: Remove rare contexts
3. **Hashing**: Use hash tables instead of full tables
4. **Blending**: Mix predictions from multiple orders

---

## Escape Mechanism

When symbol hasn't been seen in current context:

1. Output escape symbol
2. Try shorter context
3. Repeat until found or order-0

```
Context "xyz", symbol 'a':
  "xyz" → 'a' not seen → escape
  "yz"  → 'a' not seen → escape
  "z"   → 'a' not seen → escape
  ""    → 'a' seen → encode 'a'
```

---

## Probability Estimation

### Problem

If we've seen context "th" → 'e' 5 times and 'a' 1 time:
- P(e | th) = 5/6?
- What about unseen symbols?

### Solutions

**Add-1 smoothing (Laplace)**:
```
P(s | ctx) = (count(s, ctx) + 1) / (total(ctx) + alphabet_size)
```

**PPM exclusion**:
Exclude symbols seen in higher orders when escaping down.

---

## Blending Models

Instead of escape, blend all orders:

```
P(symbol) = w₀ × P₀(symbol) + w₁ × P₁(symbol) + ... + wₙ × Pₙ(symbol)
```

Weights adapted based on prediction success.

Used in PAQ and context mixing.

---

## Finite Context Models

### Markov Models

Order-N context = N-th order Markov model.

```
State = last N symbols
Transition = probability of next symbol
```

### Implementation

```
struct ContextModel {
    order: int
    table: HashMap<Context, ProbabilityTable>
}

function predict(context):
    if context in table:
        return table[context].probabilities
    else:
        return predict(context[1:])  // Shorter context
```

---

## Context Types

### Byte Context

Previous N bytes.

### Bit Context

Previous N bits (for binary data).

### Word Context

Previous N words (for text).

### Sparse Context

Non-consecutive positions (e.g., every other byte).

### Mixed Context

Combine different context types.

---

## Applications

| Algorithm | Context Model |
|-----------|---------------|
| PPM | Order-N with escape |
| PAQ | Context mixing |
| LZMA | Markov chain + position bits |
| Brotli | Context maps |
| CABAC | Context-adaptive binary |

---

## Key Takeaways

1. Context = previous symbols that predict next
2. Higher order = better prediction = better compression
3. Memory grows exponentially with order
4. Escape mechanism for unseen contexts
5. Blending combines multiple orders
6. Foundation for best compression algorithms

---

**Next Chapter**: [PPM (Prediction by Partial Matching)](./31_ppm.md)
