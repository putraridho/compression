# Chapter 6: Huffman Coding

## Overview

**Huffman coding** is the optimal prefix code for symbol-by-symbol encoding. Developed by David Huffman in 1952 as a term paper (when his professor challenged students to find a better method than Shannon-Fano).

---

## The Key Insight

Build the tree **bottom-up** instead of top-down.

- Start with individual symbols as leaves
- Repeatedly combine the two lowest-probability nodes
- Result: optimal prefix code

---

## The Algorithm

### Building the Huffman Tree

1. Create a leaf node for each symbol with its probability
2. Put all nodes in a priority queue (min-heap by probability)
3. While more than one node in queue:
   - Remove two nodes with smallest probabilities
   - Create new internal node with these as children
   - New node's probability = sum of children's probabilities
   - Insert new node back into queue
4. Remaining node is the root

### Assigning Codes

- Left edge = 0
- Right edge = 1
- Code = path from root to leaf

---

## Step-by-Step Example

### Given

| Symbol | Probability |
|--------|-------------|
| A | 0.40 |
| B | 0.20 |
| C | 0.15 |
| D | 0.15 |
| E | 0.10 |

### Step 1: Initialize

Priority queue (sorted by probability):
```
[E:0.10, C:0.15, D:0.15, B:0.20, A:0.40]
```

### Step 2: Combine E and C

Remove E (0.10) and C (0.15), combine into node (0.25):
```
    [0.25]
    /    \
   E      C
```

Queue: `[D:0.15, B:0.20, [EC]:0.25, A:0.40]`

### Step 3: Combine D and B

Remove D (0.15) and B (0.20), combine into node (0.35):
```
    [0.35]
    /    \
   D      B
```

Queue: `[[EC]:0.25, [DB]:0.35, A:0.40]`

### Step 4: Combine [EC] and [DB]

Remove [EC] (0.25) and [DB] (0.35), combine into node (0.60):
```
        [0.60]
       /      \
   [0.25]    [0.35]
   /    \    /    \
  E      C  D      B
```

Queue: `[A:0.40, [ECDB]:0.60]`

### Step 5: Combine A and [ECDB]

Remove A (0.40) and [ECDB] (0.60), combine into root (1.00):
```
            [1.00]
           /      \
          A      [0.60]
                /      \
            [0.25]    [0.35]
            /    \    /    \
           E      C  D      B
```

Queue: `[[root]:1.00]` — Done!

### Step 6: Assign Codes

Traverse from root, 0=left, 1=right:

| Symbol | Path | Code |
|--------|------|------|
| A | left | 0 |
| E | right→left→left | 100 |
| C | right→left→right | 101 |
| D | right→right→left | 110 |
| B | right→right→right | 111 |

### Final Huffman Codes

| Symbol | Probability | Code | Length |
|--------|-------------|------|--------|
| A | 0.40 | 0 | 1 |
| B | 0.20 | 111 | 3 |
| C | 0.15 | 101 | 3 |
| D | 0.15 | 110 | 3 |
| E | 0.10 | 100 | 3 |

---

## Efficiency Calculation

### Average Code Length

```
L_avg = 0.40×1 + 0.20×3 + 0.15×3 + 0.15×3 + 0.10×3
      = 0.40 + 0.60 + 0.45 + 0.45 + 0.30
      = 2.20 bits/symbol
```

### Entropy

```
H = -[0.40×log₂(0.40) + 0.20×log₂(0.20) + 0.15×log₂(0.15)
     + 0.15×log₂(0.15) + 0.10×log₂(0.10)]
  ≈ 2.15 bits/symbol
```

### Redundancy

```
L_avg - H = 2.20 - 2.15 = 0.05 bits/symbol
```

Huffman is within 1 bit of optimal (guaranteed by theory).

---

## Why Huffman Is Optimal

### Proof Sketch

1. Optimal code has two longest codes of equal length
2. These longest codes belong to two least probable symbols
3. These two codes differ only in last bit (siblings in tree)
4. We can combine them without loss of optimality
5. This is exactly what Huffman does!

By induction, Huffman produces optimal code.

---

## Pseudocode

### Building the Tree

```
function build_huffman_tree(symbols, probabilities):
    // Create leaf nodes
    nodes = []
    for i = 0 to length(symbols) - 1:
        nodes.append(Node(symbol=symbols[i], prob=probabilities[i]))

    // Build tree using priority queue
    heap = MinHeap(nodes, key=probability)

    while heap.size() > 1:
        left = heap.pop()
        right = heap.pop()
        parent = Node(
            symbol=null,
            prob=left.prob + right.prob,
            left=left,
            right=right
        )
        heap.push(parent)

    return heap.pop()  // Root of tree
```

### Generating Codes

```
function generate_codes(root):
    codes = {}
    traverse(root, "", codes)
    return codes

function traverse(node, prefix, codes):
    if node.is_leaf():
        codes[node.symbol] = prefix
        return

    traverse(node.left, prefix + "0", codes)
    traverse(node.right, prefix + "1", codes)
```

### Encoding

```
function encode(text, codes):
    output = ""
    for char in text:
        output += codes[char]
    return output
```

### Decoding

```
function decode(bits, root):
    output = ""
    node = root

    for bit in bits:
        if bit == '0':
            node = node.left
        else:
            node = node.right

        if node.is_leaf():
            output += node.symbol
            node = root

    return output
```

---

## Handling Ties

When two nodes have equal probability, which to pick?

### Common Approaches

1. **Arbitrary**: Pick any two
2. **Alphabetical**: Prefer earlier symbols
3. **Leaf preference**: Prefer leaf over internal node

Different choices give different (equally optimal) codes.

### Canonical Huffman

Standardizes tie-breaking for consistent codes.
(Covered in next chapter)

---

## Edge Cases

### Single Symbol

Only one symbol with P=1.0:
- Can't build meaningful tree
- Use code "0" or "1" (length 1)

### Two Symbols

```
Symbols: A(0.7), B(0.3)
Tree:
    (root)
    /    \
   A      B

Codes: A=0, B=1
```

### All Equal Probabilities

With n symbols at 1/n each:
- All codes have length ⌈log₂(n)⌉ or ⌈log₂(n)⌉-1
- Similar to fixed-length code

---

## Compression Workflow

### Encoding Process

```
1. Read input, count symbol frequencies
2. Calculate probabilities
3. Build Huffman tree
4. Generate code table
5. Write header (tree or code lengths)
6. Encode input using code table
7. Write encoded bits
```

### Decoding Process

```
1. Read header, reconstruct tree
2. Read bits
3. Traverse tree, output symbols
4. Repeat until end of data
```

---

## Storing the Tree

The decoder needs the tree. Options:

### 1. Store Full Tree

Write tree structure + symbols.
```
0 = internal node
1 = leaf (followed by symbol)

Example: 0 0 1'A' 1'B' 0 1'C' 1'D'
```

### 2. Store Code Lengths

Just store length for each symbol.
```
A:1, B:3, C:3, D:3, E:3
```
Canonical Huffman can rebuild tree from lengths.

### 3. Store Frequencies

Send symbol counts; decoder rebuilds tree.
Same algorithm on both ends.

---

## Limitations of Huffman

### Minimum 1 Bit Per Symbol

Can't use fractional bits.

Example: P(A) = 0.99, P(B) = 0.01
- Optimal: ~0.08 bits/symbol
- Huffman: 1 bit/symbol (A=0, B=1)

### Solution

**Block Huffman**: Group symbols into blocks.
```
Instead of encoding A, B:
Encode AA, AB, BA, BB

Probabilities: AA=0.98, AB=0.01, BA=0.01, BB=0.0001
Now we can assign variable lengths effectively.
```

Or use **Arithmetic coding** (Chapter 9).

---

## Complexity Analysis

### Time

| Operation | Complexity |
|-----------|------------|
| Build tree | O(n log n) |
| Generate codes | O(n) |
| Encode | O(m) for m input symbols |
| Decode | O(m) for m output symbols |

### Space

| Component | Space |
|-----------|-------|
| Tree | O(n) nodes |
| Code table | O(n) entries |
| Encoded data | Variable |

---

## Real-World Use

| Application | Notes |
|-------------|-------|
| DEFLATE (ZIP, gzip) | Huffman + LZ77 |
| JPEG | Huffman for coefficients |
| MP3 | Huffman for spectral data |
| PNG | DEFLATE = Huffman + LZ77 |
| Fax (CCITT) | Modified Huffman |
| PKZIP | Huffman |

---

## Key Takeaways

1. Huffman is **optimal** for symbol-by-symbol prefix codes
2. Build tree **bottom-up** by combining smallest probabilities
3. Common symbols get short codes (near root)
4. Guaranteed within 1 bit of entropy per symbol
5. Need to transmit tree structure to decoder
6. Minimum 1 bit per symbol (use arithmetic coding for fractional)

---

**Practice**:

1. Build Huffman tree for: A(0.5), B(0.25), C(0.125), D(0.125)
2. Encode "ABAD" with your codes
3. Implement the algorithm
4. Compare compression of text file vs random bytes

---

**Next Chapter**: [Canonical Huffman](./07_canonical_huffman.md)
