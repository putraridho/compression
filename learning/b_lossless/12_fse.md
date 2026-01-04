# Chapter 12: Finite State Entropy (FSE)

## What is FSE?

FSE is Yann Collet's implementation of tANS (table-based ANS).

It's the entropy coder used in **Zstandard**, one of the most important modern compression algorithms.

---

## FSE vs Generic tANS

FSE adds:

1. **Optimized symbol spreading**
2. **Accurate normalization**
3. **Efficient table building**
4. **Bitstream format**

---

## Core Concepts

### State Machine

FSE uses a state machine with:
- State size: typically 2^n (e.g., 4096)
- Each state encodes "which symbol came last"
- Transitions are deterministic

### The Tables

```
decode_table[state] → (symbol, bits_to_read, new_base_state)
encode_table[symbol][sub_state] → (bits_to_output, new_state)
```

---

## Probability Normalization

### Why Normalize?

Table size must be power of 2.
Symbol frequencies must sum to table size.

### Normalization Algorithm

```
table_size = 1 << accuracy_log  // e.g., 4096

For each symbol:
    normalized[s] = (count[s] × table_size) / total

// Fix rounding errors
while sum(normalized) != table_size:
    adjust smallest error
```

### Special Cases

- **Very rare symbols**: Get at least frequency 1
- **Very common symbols**: May need reduction to fit

---

## Symbol Spreading

Distribute symbols across state table for good mixing.

### The Goal

Symbols should be spread evenly, not clustered.

```
Bad:  AAAABBCC
Good: ACABACAB
```

### FSE Spreading Algorithm

```
position = 0
step = (table_size >> 1) + (table_size >> 3) + 3
mask = table_size - 1

For each symbol s with frequency f:
    For i = 0 to f - 1:
        symbol_table[position] = s
        position = (position + step) & mask
        while position >= table_size:
            position = (position + step) & mask
```

The step size is chosen to create good distribution.

---

## Building the Decode Table

After spreading symbols:

```
For each position pos in [0, table_size):
    s = symbol_table[pos]

    // Find next occurrence of this symbol
    next_state = find_next_occurrence(s, pos)

    // Calculate bits needed
    high_bit = log2(next_state / normalized[s])
    bits_to_read = accuracy_log - high_bit

    decode_table[pos] = {
        symbol: s,
        bits: bits_to_read,
        new_state_base: derived from next_state
    }
```

---

## Decoding

```
state = initial_state  // Read from stream

while not done:
    entry = decode_table[state]
    output(entry.symbol)

    // Read bits and compute new state
    bits = read_bits(entry.bits)
    state = entry.new_state_base + bits
```

### Example Trace

```
State: 1000
decode_table[1000] = {symbol: 'A', bits: 2, base: 500}

Output: 'A'
Read 2 bits: 11 (binary) = 3
New state: 500 + 3 = 503

State: 503
decode_table[503] = {symbol: 'B', bits: 3, base: 100}

Output: 'B'
Read 3 bits: 101 = 5
New state: 100 + 5 = 105

...
```

---

## Encoding

Encoding works in reverse (ANS is LIFO).

### Forward Encoding with Buffering

```
states = []
state = initial_state

For each symbol (forward):
    // Normalize state down if too large
    while state >= threshold[symbol]:
        output_bits.prepend(state & mask)
        state >>= bits

    // Encode symbol
    state = encode_step(state, symbol)
    states.append(state)

// Write final state and bits in reverse
```

### Encode Step

```
function encode_step(state, symbol):
    freq = normalized[symbol]
    return (state / freq) × table_size + (state mod freq) + cumulative[symbol]
```

---

## The Bitstream Format

FSE uses a specific format:

```
[Header]
  - Table description (compressed)
  - Symbol count

[Data]
  - Encoded symbols (reverse order logically)
  - Final state

[Footer]
  - End marker or length
```

### Table Compression

Instead of raw frequencies:
```
Use FSE itself to compress the frequency table!
```

Or use a simpler encoding for small tables.

---

## Accuracy Log

The `accuracy_log` controls table size:

```
table_size = 1 << accuracy_log
```

| Accuracy Log | Table Size | Memory | Compression |
|--------------|------------|--------|-------------|
| 5 | 32 | Tiny | Lower |
| 8 | 256 | Small | Good |
| 10 | 1024 | Medium | Better |
| 12 | 4096 | Larger | Best |

Higher accuracy = better compression, more memory.

---

## Interleaved FSE

For better performance, use multiple states:

```
state1, state2 = initial
bit_stream = []

For i = 0 to length - 1:
    if i is even:
        encode with state1
    else:
        encode with state2
```

Benefits:
- Instruction-level parallelism
- Hide memory latency
- SIMD opportunities

---

## Comparison with Huffman

| Aspect | Huffman | FSE |
|--------|---------|-----|
| Compression | ~1 bit overhead | Near-optimal |
| Table size | O(alphabet) | O(2^accuracy) |
| Decode speed | Very fast | Very fast |
| Encode speed | Fast | Fast |
| Adaptability | Rebuild tree | Rebuild table |

FSE is slightly better compression, similar speed.

---

## Pseudocode: Complete FSE

### Normalization

```
function normalize(counts, table_log):
    table_size = 1 << table_log
    total = sum(counts)

    normalized = []
    for s in symbols:
        n = (counts[s] × table_size) / total
        normalized[s] = max(n, 1)  // At least 1

    // Adjust to sum exactly
    while sum(normalized) > table_size:
        decrease largest
    while sum(normalized) < table_size:
        increase with most error

    return normalized
```

### Build Decode Table

```
function build_decode_table(normalized, table_log):
    table_size = 1 << table_log

    // Spread symbols
    symbol_table = spread_symbols(normalized, table_size)

    // Build transition table
    decode_table = []
    symbol_next = copy(normalized)

    for pos = 0 to table_size - 1:
        s = symbol_table[pos]
        symbol_next[s] -= 1
        next = normalized[s] + symbol_next[s]

        bits = table_log - highest_bit(next)
        baseline = (next << bits) - table_size

        decode_table[pos] = {
            symbol: s,
            bits: bits,
            baseline: baseline
        }

    return decode_table
```

### Decode Loop

```
function decode(stream, table, count):
    state = read_initial_state(stream)
    output = []

    for i = 0 to count - 1:
        entry = table[state]
        output.append(entry.symbol)

        bits = read_bits(stream, entry.bits)
        state = entry.baseline + bits

    return output
```

---

## Real-World: Zstandard's FSE

Zstandard uses FSE for:

1. **Literal lengths**: How many literals before a match
2. **Match lengths**: How long is each match
3. **Offset codes**: Where to find matches

Each has its own FSE table, optimized for that data type.

---

## Key Takeaways

1. FSE is a highly optimized tANS implementation
2. Uses power-of-2 table sizes for efficiency
3. Symbol spreading ensures good state distribution
4. Decode is table lookup + bit read + addition
5. Near-optimal compression at Huffman-like speed
6. Foundation of Zstandard's entropy coding

---

**Practice**:

1. Build symbol spread table for simple alphabet
2. Construct decode table manually
3. Trace through decode of sample bitstream
4. Compare compression ratio to Huffman

---

**Next Chapter**: [Golomb and Rice Coding](./13_golomb_rice.md)
