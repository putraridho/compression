# Chapter 3: Bit Packing

## Overview

**Bit packing** uses the minimum number of bits to represent each value, eliminating wasted bits.

```
Standard byte storage:  [00000005, 00000003, 00000007, 00000002]
                        ^^^^^^^   ^^^^^^^   ^^^^^^^   ^^^^^^^
                        wasted    wasted    wasted    wasted

Bit packed (3 bits each): [101][011][111][010] = 10101111 1010xxxx
```

If your values only need 3 bits, why use 8?

---

## The Core Idea

### Fixed-Width Integers Are Wasteful

```
Values: [5, 3, 7, 2, 1, 6, 4, 0]
Max value: 7 (needs 3 bits)

Standard storage: 8 values × 8 bits = 64 bits
Bit packed:       8 values × 3 bits = 24 bits

Savings: 62.5%
```

### Calculate Bits Needed

```
bits_needed = ⌈log₂(max_value + 1)⌉
```

| Max Value | Bits Needed |
|-----------|-------------|
| 1 | 1 |
| 3 | 2 |
| 7 | 3 |
| 15 | 4 |
| 255 | 8 |
| 65535 | 16 |

---

## Bit Packing Methods

### 1. Uniform Bit Packing

All values use the same number of bits.

```
Find max value in block
Calculate bits needed
Pack all values with that width
Store the bit width in header
```

**Example**:
```
Values: [5, 3, 7, 2]
Max: 7 → 3 bits

Header: bit_width = 3
Data: 101 011 111 010 (packed into bytes)
```

### 2. Frame of Reference (FOR)

Subtract minimum value first, then pack.

```
Values: [1000, 1003, 1007, 1002]
Min: 1000

Adjusted: [0, 3, 7, 2]
Max adjusted: 7 → 3 bits

Header: min = 1000, bit_width = 3
Data: 000 011 111 010
```

Much smaller than storing full values!

### 3. Patched Frame of Reference (PFOR)

Handle outliers separately.

```
Values: [3, 5, 2, 1000, 4, 7]

Without patching: max=1000 → 10 bits each
With patching:
  - Normal values (3,5,2,4,7): max=7 → 3 bits
  - Exception: 1000 stored separately

Header: bit_width = 3, has_exceptions = true
Data: 011 101 010 [exc] 100 111
Exceptions: [index=3, value=1000]
```

---

## Packing Into Bytes

### Left-to-Right Packing

Values pack from most significant bit.

```
Values: [5, 3, 7] with 3 bits each
Binary: 101, 011, 111

Byte 0: [101][011][11] = 10101111
Byte 1: [1]xxxxxxx     = 10000000 (padded)
```

### Right-to-Left Packing

Values pack from least significant bit.

```
Values: [5, 3, 7] with 3 bits each
Binary: 101, 011, 111

Byte 0: xx[111][011] = 00111011
Byte 1: xxxxx[101]   = 00000101
```

### Cross-Byte Values

When a value spans two bytes:

```
Value 5 (101) starting at bit position 6:
Byte 0: xxxxxx[10] = 00000010
Byte 1: [1]xxxxxxx = 10000000
```

---

## Variable-Width Integers (VarInt)

Different values use different numbers of bytes.

### Format

Use 7 bits per byte for data, 1 bit as continuation flag.

```
If high bit = 1: more bytes follow
If high bit = 0: this is the last byte

Value 300 (binary: 100101100):
  Split into 7-bit groups: 0000010 0101100

  Byte 1: 1_0101100 (continuation=1, data=0101100)
  Byte 2: 0_0000010 (continuation=0, data=0000010)
```

### VarInt Encoding

```
Value → Bytes needed:
0-127       → 1 byte
128-16383   → 2 bytes
16384-2M    → 3 bytes
...
```

Used in Protocol Buffers, SQLite, Git.

### LEB128 (Little Endian Base 128)

```
Encode:
1. Take 7 bits from value
2. If more bits remain, set high bit to 1
3. Repeat until done

Decode:
1. Read byte, extract 7 data bits
2. If high bit set, read more and shift
3. Combine all parts
```

---

## SIMD Bit Packing

Modern CPUs can pack/unpack 4-8 values simultaneously.

### Benefits

- 4x-8x faster than scalar code
- Critical for database query performance
- Specialized instructions (PEXT, PDEP on x86)

### Libraries

- FastPFOR
- TurboPFor
- Streamvbyte

---

## Group Varint

Pack multiple varints together for better performance.

### Simple-8b

Pack up to 240 integers in 8 bytes.

```
8-byte block format:
- 4 bits: selector (determines bit width)
- 60 bits: packed integers

Selector determines how many integers and their width:
Selector 0: 240 × 0-bit integers (all zeros)
Selector 1: 120 × 0-bit + 1 × 60-bit
...
Selector 7: 15 × 4-bit integers
...
```

### Simple-16

Similar concept with 16 bits for integers.

---

## Pseudocode

### Uniform Bit Packing

```
function pack(values, bit_width):
    output = []
    buffer = 0
    bits_in_buffer = 0

    for value in values:
        buffer = (buffer << bit_width) | value
        bits_in_buffer += bit_width

        while bits_in_buffer >= 8:
            bits_in_buffer -= 8
            byte = (buffer >> bits_in_buffer) & 0xFF
            output.append(byte)
            buffer = buffer & ((1 << bits_in_buffer) - 1)

    if bits_in_buffer > 0:
        output.append(buffer << (8 - bits_in_buffer))

    return output

function unpack(data, bit_width, count):
    values = []
    buffer = 0
    bits_in_buffer = 0
    byte_index = 0

    for i = 0 to count - 1:
        while bits_in_buffer < bit_width:
            buffer = (buffer << 8) | data[byte_index]
            byte_index += 1
            bits_in_buffer += 8

        bits_in_buffer -= bit_width
        value = (buffer >> bits_in_buffer) & ((1 << bit_width) - 1)
        values.append(value)

    return values
```

### VarInt

```
function encode_varint(value):
    output = []
    while value >= 0x80:
        output.append((value & 0x7F) | 0x80)
        value >>= 7
    output.append(value)
    return output

function decode_varint(data, offset):
    result = 0
    shift = 0
    while True:
        byte = data[offset]
        offset += 1
        result |= (byte & 0x7F) << shift
        if (byte & 0x80) == 0:
            break
        shift += 7
    return result, offset
```

---

## Real-World Applications

| Application | Technique |
|-------------|-----------|
| Protocol Buffers | VarInt |
| Apache Parquet | Bit packing, RLE |
| Apache Arrow | Dictionary + bit packing |
| SQLite | VarInt |
| Git pack files | VarInt for sizes |
| Lucene (search) | VInt, bit packing |
| Video codecs | Fixed bit packing for coefficients |

---

## Choosing a Method

| Data Characteristic | Best Method |
|--------------------|-------------|
| All small values | Uniform bit packing |
| Small range, large base | Frame of Reference |
| Mostly small, few large | Patched FOR |
| Unknown distribution | VarInt |
| Need random access | Fixed width |
| Sequential access | VarInt or packed |

---

## Complexity Analysis

### Uniform Bit Packing
- Encoding: O(n)
- Decoding: O(n)
- Random access: O(1) if bit width known

### VarInt
- Encoding: O(n)
- Decoding: O(n)
- Random access: O(n) — must scan from start

---

## Key Takeaways

1. Bit packing eliminates wasted bits in fixed-width storage
2. Uniform packing: all values same width based on max
3. FOR: subtract base to reduce range
4. VarInt: variable bytes per value (good for unknown distributions)
5. SIMD enables very fast packing/unpacking
6. Used extensively in databases, serialization, search engines

---

**Practice**: Implement:
1. Uniform bit packing (pack 8 values of 5 bits each)
2. VarInt encoding and decoding
3. Frame of Reference with bit packing
4. Benchmark against standard byte arrays

---

**Next Chapter**: [Variable-Length Codes Introduction](./04_variable_length_codes.md)
