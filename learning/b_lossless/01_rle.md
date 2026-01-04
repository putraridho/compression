# Chapter 1: Run-Length Encoding (RLE)

## Overview

**Run-Length Encoding** is the simplest compression algorithm. It replaces consecutive repeated values with a count and the value.

```
Input:  AAAAAABBBCCCCCCCC
Output: 6A3B8C
```

---

## How It Works

### Compression

1. Read the input sequentially
2. Count consecutive identical symbols
3. Output: (count, symbol) pairs

### Decompression

1. Read (count, symbol) pairs
2. Repeat each symbol count times
3. Output the expanded data

---

## Example Walkthrough

### Compression

```
Input: WWWWWWWWWWWWBWWWWWWWWWWWWBBBWWWWWWWWWWWWWWWWWWWWWWWWBWWWWWWWWWWWWWW

Step by step:
- 12 W's → 12W
- 1 B    → 1B
- 12 W's → 12W
- 3 B's  → 3B
- 24 W's → 24W
- 1 B    → 1B
- 14 W's → 14W

Output: 12W1B12W3B24W1B14W
```

Original: 67 characters
Compressed: 18 characters
Ratio: 3.7x

### Decompression

```
Input: 12W1B12W3B24W1B14W

Step by step:
- 12W → WWWWWWWWWWWW
- 1B  → B
- 12W → WWWWWWWWWWWW
- 3B  → BBB
- 24W → WWWWWWWWWWWWWWWWWWWWWWWW
- 1B  → B
- 14W → WWWWWWWWWWWWWW

Output: WWWWWWWWWWWWBWWWWWWWWWWWWBBBWWWWWWWWWWWWWWWWWWWWWWWWBWWWWWWWWWWWWWW
```

---

## RLE Variants

### Byte-Level RLE

Operates on bytes (0-255) instead of characters.

```
Input bytes:  [0, 0, 0, 0, 255, 255, 128]
Output:       [(4, 0), (2, 255), (1, 128)]
```

### Bit-Level RLE

Operates on individual bits. Only stores run lengths since bits alternate.

```
Input bits:  0000000011111100000
             ^^^^^^^^ (8 zeros)
                     ^^^^^^ (6 ones)
                           ^^^^^ (5 zeros)
Output:      [8, 6, 5]  (with starting bit = 0)
```

Used in fax machines (mostly white pixels with some black).

### PackBits

Apple's variant with escape mechanism:

- Positive count n: copy next (n+1) bytes literally
- Negative count -n: repeat next byte (n+1) times
- -128: no operation (padding)

```
[3, A, B, C, D]     → ABCD (literal run of 4)
[-3, A]             → AAAA (repeat A, 4 times)
```

---

## When RLE Works Well

### Good Cases

1. **Images with solid colors**: Logos, icons, simple graphics
2. **Binary images**: Black and white documents
3. **Screen captures**: Large uniform regions
4. **Sparse data**: Mostly zeros with occasional values

### Bad Cases

1. **Photographs**: No long runs of identical pixels
2. **Compressed data**: Already random-looking
3. **Alternating patterns**: ABABAB becomes worse (1A1B1A1B1A1B)
4. **Random data**: No runs to exploit

---

## The Expansion Problem

RLE can make data **larger**.

```
Input:  ABCDEFGH
Output: 1A1B1C1D1E1F1G1H
```

Original: 8 bytes
"Compressed": 16 bytes (2x larger!)

### Solutions

1. **Only compress if it helps**: Check output size
2. **Use escape codes**: Special marker for literal data
3. **Minimum run length**: Only encode runs of 3+ symbols

---

## Format Considerations

### Count Encoding

How do you encode the count?

**Fixed width**:
- 1 byte count: max 255 repeats
- 2 byte count: max 65535 repeats
- Pro: Simple
- Con: Wastes space for small counts

**Variable width**:
- Use fewer bits for small counts
- Pro: More efficient
- Con: More complex

### Handling Counts > Max

If run is longer than max count:

```
500 A's with 1-byte count (max 255):
Output: 255A245A
```

---

## RLE + Other Techniques

RLE is often used as a preprocessing step:

### BWT + RLE
Burrows-Wheeler Transform clusters similar bytes, then RLE compresses the runs.

### Image Prediction + RLE
Predict each pixel, store differences. Many differences are zero, so RLE compresses well.

### Huffman + RLE
First RLE, then Huffman encode the (count, symbol) pairs.

---

## Pseudocode

### Compression

```
function compress(input):
    output = []
    i = 0

    while i < length(input):
        symbol = input[i]
        count = 1

        // Count consecutive identical symbols
        while i + count < length(input) AND input[i + count] == symbol:
            count = count + 1
            if count == 255:  // Max count
                break

        output.append(count)
        output.append(symbol)
        i = i + count

    return output
```

### Decompression

```
function decompress(input):
    output = []
    i = 0

    while i < length(input):
        count = input[i]
        symbol = input[i + 1]

        for j = 0 to count - 1:
            output.append(symbol)

        i = i + 2

    return output
```

---

## Real-World Uses

| Application | Notes |
|-------------|-------|
| PCX image format | One of earliest uses |
| BMP (RLE mode) | Optional RLE compression |
| TIFF (PackBits) | Common in scanning |
| Fax (T.4/T.6) | Bit-level RLE |
| TGA images | Optional RLE |
| PDF | For some image data |

---

## Implementation Tips

1. **Handle empty input**: Return empty output
2. **Handle single-byte input**: Return 1, byte
3. **Consider byte order**: Big-endian vs little-endian for counts
4. **Test edge cases**: Max runs, alternating bytes, all same byte
5. **Add header**: Store original size or use end marker

---

## Complexity Analysis

### Time Complexity
- Compression: O(n) — single pass
- Decompression: O(n) — single pass

### Space Complexity
- O(n) for output in worst case
- O(1) additional space

---

## Key Takeaways

1. RLE replaces runs of identical symbols with (count, symbol)
2. Works best on data with long runs (images, sparse data)
3. Can expand data if no runs exist
4. Often used as preprocessing for other algorithms
5. Very fast and simple to implement

---

**Practice**: Implement RLE compression and decompression. Test on:
1. A string of all 'A's
2. Alternating 'AB' pattern
3. Random text
4. A simple image (BMP with solid colors)

---

**Next Chapter**: [Delta Encoding](./02_delta_encoding.md)
