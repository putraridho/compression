# Chapter 31: PPM (Prediction by Partial Matching)

## Overview

**PPM** is one of the best text compression algorithms.

Uses context modeling with escape to lower orders.

---

## How PPM Works

1. Try to encode symbol using longest context
2. If symbol not seen in context, output escape
3. Try shorter context
4. Repeat until symbol found

---

## Example

### Text: "abracadabra"

Encoding 'c' (after "abra"):

```
Order-4 context "abra":
  Haven't seen 'c' after "abra" → ESCAPE

Order-3 context "bra":
  Haven't seen 'c' after "bra" → ESCAPE

Order-2 context "ra":
  Haven't seen 'c' after "ra" → ESCAPE

Order-1 context "a":
  Seen: 'b' once
  Haven't seen 'c' → ESCAPE

Order-0 (no context):
  Seen: a(3), b(2), r(1)
  Not seen: c → ESCAPE

Order-(-1):
  All 256 symbols equally likely
  Encode 'c' with -log₂(1/256) = 8 bits
```

---

## PPM Variants

### PPMA (Method A)

Escape probability:
```
P(escape) = distinct_symbols / (total + distinct_symbols)
```

Simple, but not optimal.

### PPMC (Method C)

Escape probability:
```
P(escape) = distinct_symbols / total
```

Symbol probability:
```
P(symbol) = (count - 1) / total  (if count > 1)
```

Better than PPMA.

### PPMD (Method D)

Most popular variant.

Uses exclusion and improved estimation:
```
P(symbol) = (2 × count - 1) / (2 × total)
```

### PPM* (Unbounded Order)

No fixed maximum order.
Uses longest matching context found.

---

## Exclusion

When escaping, exclude symbols already rejected:

```
Order-2 has: a, b, c
Order-1 has: a, b, c, d, e

When escaping from order-2:
  Don't count a, b, c in order-1 probabilities
  Only d, e are "new" symbols
```

Improves compression significantly.

---

## PPMD Algorithm

```
function encode(symbol, context):
    for order = max_order down to 0:
        ctx = context.last(order)

        if symbol in ctx.seen_symbols:
            // Encode symbol in this context
            prob = ppmd_probability(symbol, ctx)
            arithmetic_encode(symbol, prob)
            update(ctx, symbol)
            return

        else:
            // Escape to lower order
            prob = escape_probability(ctx)
            arithmetic_encode(ESCAPE, prob)

    // Order -1: all symbols equally likely
    arithmetic_encode(symbol, 1/256)
```

---

## Update Rules

After encoding each symbol:

```
function update(context, symbol):
    for order = 0 to max_order:
        ctx = context.suffix(order)
        ctx.count[symbol] += 1
        ctx.total += 1

        if symbol not in ctx.seen:
            ctx.seen.add(symbol)
            ctx.num_distinct += 1
```

---

## Memory Management

### Problem

Memory usage grows with text.

### Solutions

**Fixed memory limit**:
- When full, halve all counts
- Or restart fresh

**LRU eviction**:
- Remove least recently used contexts

**Aging**:
- Periodically reduce all counts

---

## Comparison

### PPM vs Dictionary Methods

| Aspect | PPM | LZ77/DEFLATE |
|--------|-----|--------------|
| Text compression | Excellent | Good |
| Binary data | Good | Good |
| Speed | Slow | Fast |
| Memory | High | Low |
| Implementation | Complex | Moderate |

### Compression Ratios (text)

| Algorithm | bits/char |
|-----------|-----------|
| gzip | 2.5-3.0 |
| bzip2 | 2.0-2.5 |
| PPMD | 1.8-2.2 |
| PAQ | 1.4-1.8 |

---

## PPMD Parameters

### Order

```
-o5: Order 5 (default)
-o16: Higher order (better compression, more memory)
```

### Memory

```
-m64: 64MB memory
-m256: 256MB memory
```

### Typical Settings

```
Text:    -o8 -m128
Binary:  -o4 -m64
```

---

## Implementation Tips

### Context Representation

Use trie or hash table:

```
struct Context {
    counts: [u16; 256]   // Symbol counts
    total: u32           // Total count
    num_distinct: u8     // Distinct symbols seen
}
```

### Context Lookup

Suffix tree or hash of last N bytes.

### Arithmetic Coder

PPM requires arithmetic coding (not Huffman).

---

## Key Takeaways

1. PPM uses context with escape to shorter contexts
2. PPMD is the most popular variant
3. Exclusion improves compression
4. Excellent for text (1.8-2.2 bits/char)
5. Slower than LZ methods
6. Memory grows with context order

---

**Practice**:

1. Implement order-1 PPM
2. Add escape mechanism
3. Implement exclusion
4. Compare to gzip on text

---

**Next Chapter**: [Context Mixing](./32_context_mixing.md)
