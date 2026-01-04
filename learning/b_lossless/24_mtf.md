# Chapter 24: Move-to-Front Transform

## What is MTF?

**Move-to-Front** converts symbols to indices based on recency.

Recently used symbols get small indices (0, 1, 2...).
Rarely used symbols get large indices.

---

## The Algorithm

Maintain a list of all symbols. For each input symbol:

1. Find its position in the list (this is the output)
2. Move it to the front of the list

---

## Example

### Input

```
"abracadabra"
```

### Initial List

```
[a, b, c, d, e, ..., r, ..., z]
(alphabetical order)
```

### Transform

```
Input: 'a'
  Position of 'a': 0
  Output: 0
  List: [a, b, c, d, ...]  (unchanged, a already at front)

Input: 'b'
  Position of 'b': 1
  Output: 1
  List: [b, a, c, d, ...]  (b moved to front)

Input: 'r'
  Position of 'r': 17 (somewhere back)
  Output: 17
  List: [r, b, a, c, ...]  (r moved to front)

Input: 'a'
  Position of 'a': 2 (a is now at position 2)
  Output: 2
  List: [a, r, b, c, ...]

Input: 'c'
  Position of 'c': 4
  Output: 4
  List: [c, a, r, b, ...]

Input: 'a'
  Position of 'a': 1
  Output: 1
  List: [a, c, r, b, ...]

Input: 'd'
  Position of 'd': 5
  Output: 5
  List: [d, a, c, r, ...]

Input: 'a'
  Position of 'a': 1
  Output: 1
  List: [a, d, c, r, ...]

Input: 'b'
  Position of 'b': 4
  Output: 4
  List: [b, a, d, c, ...]

Input: 'r'
  Position of 'r': 4
  Output: 4
  List: [r, b, a, d, ...]

Input: 'a'
  Position of 'a': 2
  Output: 2
  List: [a, r, b, d, ...]
```

### Output

```
Input:  a  b  r  a  c  a  d  a  b  r  a
Output: 0  1  17 2  4  1  5  1  4  4  2
```

---

## Why This Helps

### After BWT

BWT groups similar characters:

```
BWT output: "aaannnnbb"
```

MTF transforms this to:

```
a → 0
a → 0
a → 0
n → 1 (n moved to front)
n → 0
n → 0
n → 0
b → 2 (b is now at position 2)
b → 0
```

Output: [0, 0, 0, 1, 0, 0, 0, 2, 0]

**Many zeros!** This compresses extremely well with RLE or entropy coding.

---

## The Inverse

Maintain same list, but:
1. Index tells you which symbol to output
2. Move that symbol to front

```
function mtf_inverse(indices):
    list = [0, 1, 2, ..., 255]  // or initial alphabet
    output = []

    for index in indices:
        symbol = list[index]
        output.append(symbol)
        list.remove(symbol)
        list.insert(0, symbol)

    return output
```

---

## Pseudocode

### Forward MTF

```
function mtf_encode(input):
    list = [0, 1, 2, ..., 255]
    output = []

    for symbol in input:
        index = list.index(symbol)
        output.append(index)
        list.pop(index)
        list.insert(0, symbol)

    return output
```

### Inverse MTF

```
function mtf_decode(indices):
    list = [0, 1, 2, ..., 255]
    output = []

    for index in indices:
        symbol = list[index]
        output.append(symbol)
        list.pop(index)
        list.insert(0, symbol)

    return output
```

---

## Time Complexity

### Naive Implementation

- Find position: O(alphabet_size)
- Move to front: O(alphabet_size)
- Total: O(n × alphabet_size)

For 256 byte values: O(256n) = O(n)

### Optimized with Data Structures

Using splay trees or linked lists with hash:
- O(log n) per operation
- But constant factors often make naive faster for small alphabets

---

## MTF Variants

### Move-to-Front-2

Move symbol to position 1 instead of 0 on first access.
Move to 0 on subsequent accesses.

Reduces false promotions.

### Timestamp MTF

Track recency with timestamps instead of list manipulation.
More cache-friendly.

### Weighted MTF

Weight by frequency, not just recency.

---

## Interaction with BWT

BWT + MTF synergy:

```
Text:     "the quick brown fox"
BWT:      Groups similar contexts
MTF:      Recent characters → small numbers
Result:   Many 0s and small values
RLE:      Compress runs of 0s
Huffman:  Short codes for common values
```

This is the bzip2 pipeline.

---

## MTF Output Distribution

After BWT + MTF, typical distribution:

| Value | Frequency |
|-------|-----------|
| 0 | ~50% |
| 1 | ~15% |
| 2-3 | ~10% |
| 4-10 | ~15% |
| 11+ | ~10% |

Heavy concentration at small values!

---

## Zero-Run Encoding

bzip2 special handling:

Instead of outputting many zeros:
```
0, 0, 0, 0, 0
```

Use run-length for zeros only:
```
RUNA, RUNB encoding (binary run length)
```

Very efficient for long zero runs.

---

## Key Takeaways

1. MTF converts symbols to recency-based indices
2. Recently used symbols get index 0
3. Perfect complement to BWT (which clusters similar chars)
4. Output has many 0s and small values
5. Simple, reversible, O(n) time
6. Foundation of bzip2's excellent compression

---

**Practice**:

1. Hand-trace MTF on "mississippi"
2. Verify inverse recovers original
3. Apply to BWT output and count zeros
4. Compare entropy before/after MTF

---

**Next Chapter**: [Other Transforms](./25_other_transforms.md)
