# Chapter 10: Range Coding

## What is Range Coding?

Range coding is a variant of arithmetic coding that:

- Produces exactly the same compression ratio
- Outputs bytes instead of bits
- Was designed to avoid arithmetic coding patents (now expired)
- Often faster in practice

---

## The Key Difference

### Arithmetic Coding
- Outputs bits as soon as determined
- Complex bit-level bookkeeping
- Underflow handling with pending bits

### Range Coding
- Outputs bytes as soon as determined
- Simpler carry propagation
- Byte-aligned operations

---

## How It Works

### The Range

Maintain two values:
- **low**: Bottom of interval
- **range**: Width of interval (not high!)

```
Instead of [low, high):
Use: low, range where range = high - low
```

### Normalization

When range gets small, output bytes and renormalize.

```
While range < THRESHOLD:
    output_byte(low >> 24)
    low = (low << 8)
    range = (range << 8)
```

---

## Encoding

### For Each Symbol

```
function encode_symbol(symbol):
    // Update interval
    range = range / total
    low = low + range × cum_freq(symbol)
    range = range × freq(symbol)

    // Normalize
    while range < THRESHOLD:
        output_byte(low >> 24)
        low = low << 8
        range = range << 8
```

### The Carry Problem

When low overflows (exceeds maximum value):

```
If low + increment > MAX:
    Carry into previously output bytes
```

**Solutions**:

1. **Carry propagation**: Go back and fix previous bytes
2. **Delayed output**: Buffer bytes until carry resolved
3. **Range reduction**: Keep range small enough to prevent overflow

---

## Decoding

### Initialize

```
Read initial bytes into 'code' variable
low = 0
range = MAX
```

### For Each Symbol

```
function decode_symbol():
    // Calculate scaled position
    scaled = (code - low) / (range / total)

    // Find symbol whose range contains scaled
    symbol = find_symbol(scaled)

    // Update interval (same as encoder)
    range = range / total
    low = low + range × cum_freq(symbol)
    range = range × freq(symbol)

    // Normalize (same as encoder, but read instead of write)
    while range < THRESHOLD:
        low = low << 8
        range = range << 8
        code = (code << 8) | read_byte()

    return symbol
```

---

## Example: Encoding "AB"

### Setup

```
Alphabet: A, B
Frequencies: A=3, B=1 (total=4)
Cumulative: A=0, B=3

32-bit range coding:
THRESHOLD = 2^24
Initial: low=0, range=2^32
```

### Encode 'A'

```
range_step = range / 4 = 2^30
low = 0 + 2^30 × 0 = 0
range = 2^30 × 3 = 3 × 2^30

Check: range = 3 × 2^30 > 2^24 ✓ (no normalization needed)
```

### Encode 'B'

```
range_step = range / 4 = 3 × 2^28
low = 0 + 3 × 2^28 × 3 = 9 × 2^28
range = 3 × 2^28 × 1 = 3 × 2^28

Check: range = 3 × 2^28 > 2^24 ✓ (no normalization needed)
```

### Flush

Output remaining bytes from low.

---

## Avoiding Division

Division is slow. Use multiplication with reciprocal:

```
Instead of: range / total
Use: (range × reciprocal) >> shift

Where reciprocal and shift are precomputed for each total.
```

This makes range coding very fast.

---

## Comparison: Arithmetic vs Range Coding

| Aspect | Arithmetic Coding | Range Coding |
|--------|-------------------|--------------|
| Output unit | Bit | Byte |
| Carry handling | Pending bits | Carry propagation |
| Implementation | More complex | Simpler |
| Speed | Slower | Faster |
| Compression | Identical | Identical |
| Byte alignment | No | Yes |

---

## Pseudocode

### Full Encoder

```
THRESHOLD = 1 << 24
TOP = 1 << 32

function range_encode(symbols, freqs):
    low = 0
    range = TOP

    for symbol in symbols:
        total = sum(freqs)
        cum = cumulative_freq(symbol)

        low = low + (range / total) × cum
        range = (range / total) × freqs[symbol]

        // Normalize
        while range < THRESHOLD:
            // Handle carry
            if (low >> 24) < 0xFF:
                output_byte(buffer)
                while underflow > 0:
                    output_byte(0xFF)
                    underflow -= 1
                buffer = low >> 24
            else if (low >> 24) > 0xFF:
                output_byte(buffer + 1)
                while underflow > 0:
                    output_byte(0x00)
                    underflow -= 1
                buffer = (low >> 24) & 0xFF
            else:
                underflow += 1

            low = (low << 8) & (TOP - 1)
            range = range << 8

    // Flush
    flush_remaining(low)
```

### Full Decoder

```
function range_decode(data, freqs, length):
    low = 0
    range = TOP
    code = read_initial_bytes(data)

    output = []

    for i = 0 to length - 1:
        total = sum(freqs)

        // Find symbol
        value = ((code - low) × total) / range
        symbol = find_symbol(value, cumulative)

        output.append(symbol)

        // Update
        cum = cumulative_freq(symbol)
        low = low + (range / total) × cum
        range = (range / total) × freqs[symbol]

        // Normalize
        while range < THRESHOLD:
            low = (low << 8) & (TOP - 1)
            range = range << 8
            code = (code << 8) | next_byte()

    return output
```

---

## Carryless Range Coding

Variant that prevents carries entirely:

```
Keep range small enough that:
low + range < TOP

Never overflows, no carry propagation needed.
```

Trade-off: Slightly worse compression (negligible in practice).

---

## Real-World Usage

| Application | Notes |
|-------------|-------|
| LZMA / 7-Zip | Core entropy coder |
| Zstandard | FSE is similar concept |
| FFmpeg | Range coder option |
| CRAM | Genomic data |
| RAR | Uses range coding |

---

## Why Range Coding is Popular

1. **Byte-aligned**: Easier to implement correctly
2. **Faster**: Byte operations, easier carry handling
3. **Same compression**: No loss compared to arithmetic
4. **Patent-free**: Was designed to avoid patents
5. **Simpler**: Less edge cases

---

## Key Takeaways

1. Produces identical compression to arithmetic coding
2. Works with bytes instead of bits
3. Simpler carry handling
4. Often faster in practice
5. Foundation of LZMA/7-Zip
6. Good balance of speed and compression

---

**Practice**:

1. Implement basic range encoder/decoder
2. Compare output to arithmetic coding
3. Benchmark speed difference
4. Handle carry propagation correctly

---

**Next Chapter**: [Asymmetric Numeral Systems (ANS)](./11_ans.md)
