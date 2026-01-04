# Chapter 23: Burrows-Wheeler Transform

## What is BWT?

The **Burrows-Wheeler Transform** (1994) is a reversible transformation that rearranges text to make it more compressible.

It's not compression itself—it's a preprocessing step that groups similar characters together.

---

## The Magic

BWT transforms this:

```
Input:  "banana"
Output: "annb$aa"
```

Notice: the 'a's and 'n's are grouped! This helps RLE and entropy coding.

---

## How It Works

### Step 1: Form All Rotations

For input "banana$" ($ = end marker):

```
banana$
anana$b
nana$ba
ana$ban
na$bana
a$banan
$banana
```

### Step 2: Sort Rotations Alphabetically

```
$banana
a$banan
ana$ban
anana$b
banana$
na$bana
nana$ba
```

### Step 3: Take Last Column

```
$banana  → a
a$banan  → n
ana$ban  → n
anana$b  → b
banana$  → $
na$bana  → a
nana$ba  → a
```

**BWT output: "annb$aa"**

Also remember the row where original appears (row 4, 0-indexed).

---

## Why Characters Group

Sorted rotations put similar contexts together.

```
Rotations starting with "an":
  ana$ban  (ends with 'n')
  anana$b  (ends with 'b')

But "an" often follows same letters in text.
```

Characters preceding similar substrings cluster in the last column.

---

## The Inverse Transform

You can perfectly recover the original from just the last column!

### The Key Insight

```
First column = sorted(last column)
Each character in last column precedes corresponding character in first column.
```

### Inverse Algorithm

1. Sort last column to get first column
2. Build L→F mapping (transformation vector)
3. Follow the chain from $ marker

### Example

```
Last column:  a n n b $ a a
Sort:         $ a a a b n n

Position mapping:
  L[0]='a' comes before F[0]='$', so L[0]→F[1or2or3]
  ...build full mapping using stable sort...

Start at $ in F, follow L→F links:
  $ → L[4] = $, F index where this $ came from
  ...trace back original string
```

---

## Pseudocode

### Forward Transform

```
function bwt_encode(input):
    input = input + '$'  // End marker
    n = length(input)

    // Generate all rotations
    rotations = []
    for i = 0 to n - 1:
        rotations.append(input[i:] + input[:i])

    // Sort rotations
    sorted_rotations = sort(rotations)

    // Extract last column
    last_column = ""
    original_index = -1
    for i = 0 to n - 1:
        last_column += sorted_rotations[i][-1]
        if sorted_rotations[i] == input:
            original_index = i

    return (last_column, original_index)
```

### Inverse Transform

```
function bwt_decode(last_column, original_index):
    n = length(last_column)

    // Get first column by sorting
    first_column = sort(last_column)

    // Build transformation vector
    // T[i] = position in L that corresponds to F[i]'s predecessor

    // Count occurrences for stable mapping
    count = {}
    for char in first_column:
        count[char] = count.get(char, 0) + 1

    // Build L→F mapping
    T = []
    char_seen = {}
    for i = 0 to n - 1:
        char = last_column[i]
        char_seen[char] = char_seen.get(char, 0)

        // Find this occurrence in first column
        T[i] = find_nth_occurrence(first_column, char, char_seen[char])
        char_seen[char] += 1

    // Reconstruct
    output = ""
    current = original_index
    for i = 0 to n - 1:
        output = last_column[current] + output
        current = T[current]

    return output[1:]  // Remove $ marker
```

---

## Efficient Implementation

### Suffix Array Approach

Don't store all rotations—use suffix arrays:

```
1. Build suffix array of input
2. BWT[i] = input[SA[i] - 1]  // Character before each suffix

Time: O(n) with SA-IS algorithm
Space: O(n)
```

### In-Place Inverse

```
function fast_inverse(L, idx):
    n = length(L)
    F = sort(L)

    // Build rank array
    rank = array[n]
    count = array[256] = {0}

    for i = 0 to n - 1:
        rank[i] = count[L[i]]
        count[L[i]] += 1

    // Build cumulative count
    total = 0
    for c = 0 to 255:
        temp = count[c]
        count[c] = total
        total += temp

    // LF mapping: L[i] → F[count[L[i]] + rank[i]]
    output = ""
    current = idx
    for i = 0 to n - 1:
        output = L[current] + output
        current = count[L[current]] + rank[current]

    return output
```

---

## BWT in Compression Pipeline

BWT alone doesn't compress. Full pipeline:

```
Input → BWT → MTF → RLE → Entropy Coding → Output
```

Each step:
1. **BWT**: Groups similar characters
2. **MTF**: Converts to small integers
3. **RLE**: Compresses runs
4. **Entropy**: Huffman/arithmetic on result

This is how **bzip2** works.

---

## Example Full Transform

```
Input: "mississippi"

BWT: "ipssm$pทssii" (with index)

MTF of BWT output:
  i → 0 (first i)
  p → 1 (p not seen)
  s → 2 (s not seen)
  s → 0 (s just seen)
  m → 3 ...

Result: [0, 1, 2, 0, 3, ...]

Many zeros! Great for entropy coding.
```

---

## Properties of BWT

### Good

1. Reversible (lossless)
2. Groups similar contexts
3. Works well with MTF + RLE
4. O(n) time with good implementation

### Limitations

1. Requires entire input in memory
2. Not streamable (need full block)
3. Adds some expansion (index storage)
4. Can increase size for random data

---

## BWT vs LZ

| Aspect | BWT | LZ77 |
|--------|-----|------|
| Approach | Transform | Dictionary |
| Streaming | No (needs full block) | Yes |
| Memory | O(n) | O(window) |
| Compression | Excellent for text | Good for everything |
| Speed | Slower | Faster |

BWT excels for text; LZ77 is more general.

---

## Block Size

bzip2 uses 100KB - 900KB blocks.

Larger blocks:
- Better compression (more context)
- More memory required
- Longer latency

Smaller blocks:
- Less memory
- Lower latency
- Slightly worse compression

---

## Key Takeaways

1. BWT reorders text to group similar characters
2. Completely reversible transformation
3. Sort all rotations, take last column
4. Combined with MTF + RLE + Huffman = bzip2
5. O(n) with suffix array implementation
6. Excellent for text, not streamable

---

**Practice**:

1. Hand-trace BWT on "abracadabra"
2. Implement forward and inverse transforms
3. Verify reversibility
4. Add MTF and observe the output distribution

---

**Next Chapter**: [Move-to-Front Transform](./24_mtf.md)
