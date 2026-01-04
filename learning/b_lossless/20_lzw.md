# Chapter 20: LZW

## Overview

**LZW** (Lempel-Ziv-Welch) is Terry Welch's 1984 improvement to LZ78.

Key difference: Output only dictionary indices, not (index, character) pairs.

---

## The Key Innovation

### LZ78 Output

```
(0, 'A')  ← Index + Character
(1, 'B')
(2, 'C')
```

### LZW Output

```
65        ← Just index (65 = 'A')
66        ← Just index (66 = 'B')
256       ← Index of phrase "AB"
```

Pre-initialize dictionary with all single bytes.

---

## Initial Dictionary

Start with all 256 byte values:

```
0: byte 0
1: byte 1
...
65: 'A'
66: 'B'
...
255: byte 255
```

Next available index: 256

---

## The Encoding Algorithm

```
1. Initialize dictionary with all single bytes
2. current = first byte
3. For each subsequent byte:
   a. extended = current + byte
   b. If extended in dictionary:
      - current = extended
   c. Else:
      - Output index of current
      - Add extended to dictionary
      - current = byte
4. Output index of final current
```

---

## Step-by-Step Example

### Input

```
"ABABABA"
```

### Initial Dictionary

```
..., 65:'A', 66:'B', ...
Next index: 256
```

### Trace

```
current = 'A'

Byte 'B':
  extended = "AB"
  Not in dict
  → Output 65 ('A')
  → Add dict[256] = "AB"
  → current = 'B'

Byte 'A':
  extended = "BA"
  Not in dict
  → Output 66 ('B')
  → Add dict[257] = "BA"
  → current = 'A'

Byte 'B':
  extended = "AB"
  In dict (256)
  → current = "AB"

Byte 'A':
  extended = "ABA"
  Not in dict
  → Output 256 ("AB")
  → Add dict[258] = "ABA"
  → current = 'A'

Byte 'B':
  extended = "AB"
  In dict (256)
  → current = "AB"

Byte 'A':
  extended = "ABA"
  In dict (258)
  → current = "ABA"

End of input:
  → Output 258 ("ABA")
```

### Output

```
65, 66, 256, 258
(4 codes for 7 bytes)
```

---

## The Decoding Trick

### The Problem

Decoder builds dictionary from outputs.
But what if encoder outputs a code the decoder hasn't added yet?

### When This Happens

```
Input: "ABAB"

Encoder:
  Output 'A' (65), add "AB" as 256
  Output 256 ("AB"), add "BA" as 257
  ...wait, after outputting 256, encoder immediately sees "AB" again

Actually: Input "CACA"
  Output 'C', current = 'A'
  "CA" not in dict, output 'A', add "CA"=256, current='C'
  "CA" in dict (256), current="CA"
  "CAC" not in dict, output 256, add "CAC"=257, current='C'
  "CA" in dict, current="CA"
  End, output 256

Output: 67, 65, 256, 256
```

When decoder sees second 256, it hasn't added 256 yet!

### The Pattern

This happens when input is: XYX (where XY is new phrase)

```
Encoder outputs XY code immediately before decoder learns it.
```

### The Solution

If decoder receives unknown code:

```
phrase = previous_phrase + previous_phrase[0]
```

The unknown phrase is always the previous phrase plus its first character!

---

## Decoding Algorithm

```
1. Initialize dictionary with all single bytes
2. previous = decode first code
3. Output previous
4. For each subsequent code:
   a. If code in dictionary:
      - current = dictionary[code]
   b. Else (the special case):
      - current = previous + previous[0]
   c. Output current
   d. Add (previous + current[0]) to dictionary
   e. previous = current
```

---

## Decode Example

```
Input codes: 65, 66, 256, 258
Dictionary: 0-255 = bytes, next = 256

Code 65:
  current = 'A'
  Output: "A"
  previous = 'A'

Code 66:
  current = 'B'
  Output: "B"
  Add dict[256] = 'A' + 'B' = "AB"
  previous = 'B'

Code 256:
  current = dict[256] = "AB"
  Output: "AB"
  Add dict[257] = 'B' + 'A' = "BA"
  previous = "AB"

Code 258:
  258 not in dict! (only 0-257 exist)
  Special case: current = previous + previous[0] = "AB" + 'A' = "ABA"
  Output: "ABA"
  Add dict[258] = "AB" + 'A' = "ABA"
  previous = "ABA"

Result: "A" + "B" + "AB" + "ABA" = "ABABABA" ✓
```

---

## Variable-Width Codes

### Problem

With 256 initial entries, codes need 9+ bits.
But early codes are small!

### Solution: Growing Code Width

```
Codes 0-255:     8 bits (or skip, start at 9)
Codes 256-511:   9 bits
Codes 512-1023:  10 bits
Codes 1024-2047: 11 bits
...up to 12 bits typically
```

Encoder and decoder increase width at same points.

---

## Dictionary Full

### When Dictionary Reaches Maximum

Options:

1. **Freeze**: Stop adding, keep using current dictionary
2. **Reset**: Clear dictionary, start fresh
3. **Clear code**: Special code signals reset

GIF uses a clear code (typically 256 for 8-bit data).

---

## LZW in GIF

GIF uses LZW with:

```
- Initial code size based on color depth
- Clear code = 2^min_code_size
- End code = clear_code + 1
- First data code = clear_code + 2
- Maximum code size: 12 bits
- Automatic reset when full
```

### Example (8 colors = 3 bits)

```
Clear code: 8
End code: 9
Data codes start at: 10
Initial bits: 4 (minimum 4)
Maximum: 12 bits
```

---

## Patents

LZW was patented:
- Unisys held key patents
- Caused controversy (GIF format)
- Led to PNG development (patent-free)
- All LZW patents expired by 2004

Now freely usable.

---

## Pseudocode

### Encoder

```
function lzw_encode(input):
    // Initialize with single bytes
    dict = {chr(i): i for i = 0 to 255}
    next_code = 256

    current = input[0]
    output = []

    for byte in input[1:]:
        extended = current + byte

        if extended in dict:
            current = extended
        else:
            output.append(dict[current])
            dict[extended] = next_code
            next_code += 1
            current = byte

    output.append(dict[current])
    return output
```

### Decoder

```
function lzw_decode(codes):
    // Initialize with single bytes
    dict = {i: chr(i) for i = 0 to 255}
    next_code = 256

    previous = dict[codes[0]]
    output = previous

    for code in codes[1:]:
        if code in dict:
            current = dict[code]
        else:
            // Special case
            current = previous + previous[0]

        output += current
        dict[next_code] = previous + current[0]
        next_code += 1
        previous = current

    return output
```

---

## LZW vs LZ78

| Aspect | LZ78 | LZW |
|--------|------|-----|
| Output | (index, char) | index only |
| Initial dict | Empty | All single bytes |
| Special case | No | Yes (KwKwK) |
| Output size | Larger | Smaller |
| Complexity | Simpler | Slightly more complex |

---

## Key Takeaways

1. Outputs only dictionary indices
2. Pre-initializes with all single bytes
3. Special decoding case when code not yet known
4. Variable-width codes for efficiency
5. Famous for GIF format
6. Was patented, now free

---

**Practice**:

1. Encode "TOBEORNOTTOBEORTOBEORNOT"
2. Handle the special decoding case
3. Implement variable-width codes
4. Compare compression to LZ78

---

**Next Chapter**: [LZMA](./21_lzma.md)
