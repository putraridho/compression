# Chapter 8: Adaptive Huffman Coding

## The Problem with Static Huffman

Static Huffman requires:

1. **Two passes**: First to count frequencies, second to encode
2. **Transmit the tree**: Decoder needs the code table
3. **Fixed statistics**: Can't adapt to changing data

---

## Adaptive (Dynamic) Huffman

Update the tree as you encode/decode.

- **Single pass**: No need to pre-scan data
- **No tree transmission**: Both sides build tree identically
- **Adapts**: Code changes as statistics change

---

## How It Works

### Core Idea

1. Start with empty (or uniform) tree
2. For each symbol:
   - Encode using current tree
   - Update tree with new symbol
3. Encoder and decoder update tree the same way

**Key**: Encoder updates AFTER encoding, decoder updates AFTER decoding. Both stay synchronized.

---

## The NYT (Not Yet Transmitted) Symbol

For new symbols not yet in tree:

1. Send NYT code
2. Send symbol in fixed-width code
3. Add symbol to tree

```
First 'A' ever:
    Send: [NYT code] + [8-bit 'A']
    Update tree: add 'A' node

Second 'A':
    Send: ['A' code from tree]
    Update tree: increment 'A' count
```

---

## The Sibling Property

A valid Huffman tree must satisfy the **sibling property**:

> Nodes can be listed in non-decreasing order of weight such that each node is adjacent to its sibling.

### What This Means

- Nodes are numbered in a specific order
- Parent's weight = sum of children's weights
- When weight increases, node might need to move

---

## FGK Algorithm (Faller-Gallager-Knuth)

### Tree Structure

Nodes have:
- Weight (frequency count)
- Number (position in ordering)
- Symbol (for leaves)

### Node Numbering

```
         8(8)
        /    \
      4(6)   4(7)
     /   \   /   \
   1(2) 3(5) 2(4) 2(3)
    A         B    C
        NYT(1)
```

Numbers in parentheses. Siblings are adjacent.

### Update Process

When encoding symbol S:

1. If S is new: output NYT code + S, add S to tree
2. Else: output S's current code
3. Increment S's weight
4. Check sibling property, swap if violated
5. Propagate weight changes up to root

---

## Step-by-Step Example

### Encoding "AABCA"

#### Initial State

```
Tree: Only NYT
      NYT(1)
      weight=0
```

#### Encode First 'A'

'A' is new:
1. Output: NYT code (empty) + 'A' (8 bits)
2. Add 'A' to tree:

```
      (2)
     /    \
   NYT(1)  A(3)
   w=0     w=1
```

#### Encode Second 'A'

'A' exists, code = 1
1. Output: 1
2. Increment A's weight: 1 → 2

```
      (2)
     /    \
   NYT(1)  A(3)
   w=0     w=2
```

#### Encode 'B'

'B' is new:
1. Output: 0 (NYT code) + 'B' (8 bits)
2. Split NYT, add 'B':

```
         (4)
        /    \
      (2)     A(5)
     /    \    w=2
  NYT(1)  B(3)
   w=0    w=1
```

Wait, need to rebalance. A(weight=2) should be higher than internal node(weight=1).

After proper update:
```
           (4)
          /    \
        A(5)   (3)
        w=2   /    \
          NYT(1)  B(2)
           w=0    w=1
```

#### Encode 'C'

'C' is new:
1. Output: 1 + 0 (path to NYT) + 'C' (8 bits)
2. Add 'C', update:

```
              (6)
             /    \
           A(7)   (5)
           w=2   /    \
               (3)    B(4)
              /   \    w=1
           NYT(1) C(2)
            w=0   w=1
```

#### Encode 'A' (third time)

'A' exists, code = 0
1. Output: 0
2. Increment A's weight: 2 → 3

```
              (6)
             /    \
           A(7)   (5)
           w=3   ...
```

### Final Encoded Output

```
'A': 00000001 (NYT empty + ASCII A)
'A': 1
'B': 0 + 00000010 (NYT=0, ASCII B)
'C': 10 + 00000011 (NYT=10, ASCII C)
'A': 0

Total: 8 + 1 + 1 + 8 + 2 + 8 + 1 = 29 bits
```

---

## Vitter's Algorithm

Improvement over FGK:

1. Better node numbering
2. Fewer swaps needed
3. Better compression

### Key Difference

- Nodes ordered by weight, then by depth
- More efficient updates

---

## Pseudocode

### Encoding

```
function encode(symbol):
    if symbol not in tree:
        output NYT code
        output symbol in fixed bits
        add_symbol(symbol)
    else:
        output code for symbol
        update_tree(symbol)

function add_symbol(symbol):
    // Replace NYT with internal node
    // NYT becomes left child
    // New symbol becomes right child with weight 1
    // Update weights up the tree
```

### Decoding

```
function decode():
    node = root
    while not node.is_leaf():
        bit = read_bit()
        if bit == 0:
            node = node.left
        else:
            node = node.right

    if node == NYT:
        symbol = read_fixed_bits(8)
        add_symbol(symbol)
    else:
        symbol = node.symbol
        update_tree(symbol)

    return symbol
```

### Update Tree

```
function update_tree(symbol):
    node = find_node(symbol)

    while node != root:
        // Find highest numbered node with same weight
        swap_candidate = find_leader(node.weight)

        if swap_candidate != node and swap_candidate != node.parent:
            swap(node, swap_candidate)

        node.weight += 1
        node = node.parent
```

---

## Advantages

1. **Single pass**: Stream processing
2. **No header**: Tree not transmitted
3. **Adapts**: Handles changing statistics
4. **Memory efficient**: Only one tree needed

---

## Disadvantages

1. **Slower**: Tree updates on every symbol
2. **Complex**: Harder to implement correctly
3. **Error propagation**: One error corrupts rest
4. **Often worse**: Static Huffman with good model often better

---

## When to Use

### Good For

- Streaming data (can't buffer all input)
- Unknown statistics
- Data with changing patterns

### Not Good For

- When you can do two passes
- When speed matters most
- When static model is accurate

---

## Real-World Usage

Adaptive Huffman is rarely used today because:

1. **Arithmetic coding** is better for adaptation
2. Static Huffman with good models works well
3. Implementation complexity

But understanding it helps grasp:
- How adaptive coding works
- Tree maintenance concepts
- Encoder-decoder synchronization

---

## Key Takeaways

1. Updates tree while encoding/decoding
2. NYT symbol handles new symbols
3. Sibling property must be maintained
4. Single pass, no header needed
5. More complex than static Huffman
6. Replaced by arithmetic coding in practice

---

**Practice**:

1. Trace FGK encoding of "ABRACADABRA"
2. Implement basic adaptive Huffman
3. Compare compression to static Huffman
4. Identify when swaps are needed

---

**Next Chapter**: [Arithmetic Coding](./09_arithmetic_coding.md)
