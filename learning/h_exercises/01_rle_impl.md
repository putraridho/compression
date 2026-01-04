# Exercise 1: RLE Implementation

## Goal

Implement Run-Length Encoding from scratch.

---

## Specification

### Input
A byte array.

### Output
Encoded bytes: alternating (count, value) pairs.

### Constraints
- Maximum run length: 255
- Handle runs longer than 255 by splitting
- Handle empty input

---

## Algorithm

```
1. Initialize position = 0
2. While position < input length:
   a. current_byte = input[position]
   b. count = 1
   c. While next byte == current_byte AND count < 255:
      - count++
   d. Output: count, current_byte
   e. position += count
```

---

## Test Cases

### Test 1: Simple Run

```
Input:  [65, 65, 65, 65, 65]  // "AAAAA"
Output: [5, 65]               // 5 × 'A'
```

### Test 2: No Runs

```
Input:  [65, 66, 67, 68]      // "ABCD"
Output: [1, 65, 1, 66, 1, 67, 1, 68]
```

### Test 3: Mixed

```
Input:  [65, 65, 65, 66, 66, 67]  // "AAABBC"
Output: [3, 65, 2, 66, 1, 67]
```

### Test 4: Long Run

```
Input:  [65] × 300            // 300 A's
Output: [255, 65, 45, 65]     // 255 + 45 = 300
```

### Test 5: Empty

```
Input:  []
Output: []
```

---

## Pseudocode

```rust
fn rle_encode(input: &[u8]) -> Vec<u8> {
    let mut output = Vec::new();
    let mut i = 0;

    while i < input.len() {
        let byte = input[i];
        let mut count = 1u8;

        while i + count as usize < input.len()
              && input[i + count as usize] == byte
              && count < 255 {
            count += 1;
        }

        output.push(count);
        output.push(byte);
        i += count as usize;
    }

    output
}

fn rle_decode(input: &[u8]) -> Vec<u8> {
    let mut output = Vec::new();
    let mut i = 0;

    while i + 1 < input.len() {
        let count = input[i] as usize;
        let byte = input[i + 1];

        for _ in 0..count {
            output.push(byte);
        }

        i += 2;
    }

    output
}
```

---

## Verification

```rust
fn test_roundtrip(input: &[u8]) -> bool {
    let encoded = rle_encode(input);
    let decoded = rle_decode(&encoded);
    decoded == input
}
```

---

## Extensions

1. **Escape mechanism**: Only encode runs of 4+ to avoid expansion
2. **Statistics**: Track compression ratio
3. **Bit-level RLE**: For binary data
4. **PackBits variant**: Apple's format with literals

---

## What You Learn

- Basic compression logic
- Handling edge cases
- Testing methodology
- Roundtrip verification

---

**Next**: [Delta Encoding](./02_delta_impl.md)
