# Chapter 9: Vector Quantization

## What is Vector Quantization?

**Vector quantization (VQ)** encodes groups of values together instead of individually.

```
Scalar: 8 values × 8 bits each = 64 bits
Vector: 8-value block → 12-bit index = 12 bits
```

---

## The Codebook

A **codebook** contains representative vectors:

```
Codebook (256 entries, 8 values each):
  0: [102, 98, 105, 100, 99, 103, 101, 97]
  1: [55, 60, 58, 62, 57, 59, 61, 56]
  2: [200, 198, 205, 195, 202, 199, 201, 203]
  ...
  255: [128, 130, 127, 129, 131, 126, 128, 132]
```

### Encoding

Find closest codebook entry:
```
Input vector: [101, 99, 104, 99, 100, 102, 100, 98]
Best match: Entry 0
Output: 0 (8 bits instead of 64!)
```

### Decoding

Look up index in codebook:
```
Input: 0
Output: [102, 98, 105, 100, 99, 103, 101, 97]
```

---

## Training the Codebook

### LBG Algorithm (Linde-Buzo-Gray)

1. Start with single centroid (mean of all vectors)
2. Split: double centroids by adding perturbation
3. Iterate: assign vectors to nearest centroid, update centroids
4. Repeat split/iterate until desired codebook size

```
function lbg_train(vectors, codebook_size):
    codebook = [mean(vectors)]

    while len(codebook) < codebook_size:
        // Split
        codebook = split_centroids(codebook)

        // Iterate until convergence
        while not converged:
            assign vectors to nearest centroids
            update centroids to mean of assigned vectors
```

---

## VQ in Compression

### Image Compression

Divide image into 4×4 blocks:
```
┌────┬────┬────┬────┐
│ B1 │ B2 │ B3 │ B4 │
├────┼────┼────┼────┤
│ B5 │ B6 │ B7 │ B8 │  Each block → codebook index
└────┴────┴────┴────┘
```

### Audio Compression

Encode sample vectors:
```
160 samples → codebook index
Used in CELP speech coders
```

---

## Structured VQ

### Product VQ

Split vector into parts, quantize separately:
```
Vector [a,b,c,d,e,f,g,h]
  Part 1 [a,b,c,d] → index₁
  Part 2 [e,f,g,h] → index₂

Codebook size: 256 + 256 = 512 entries (not 256×256)
Still captures correlation within parts
```

### Tree-Structured VQ

Hierarchical search:
```
Level 1: Coarse (2 centroids)
Level 2: Medium (4 centroids)
Level 3: Fine (8 centroids)

Search: O(log N) instead of O(N)
```

---

## Advantages

1. Exploits correlation between values
2. Very fast decoding (just lookup)
3. Fixed output size

## Disadvantages

1. Codebook must be stored/transmitted
2. Training required for good codebook
3. Slow encoding (search)
4. Quality limited by codebook size

---

## Applications

| Application | Use of VQ |
|-------------|-----------|
| Image (older) | Texture compression |
| Speech (CELP) | Excitation codebook |
| Game textures | S3TC/DXT compression |
| Audio (SBC) | Bluetooth audio |

---

## Key Takeaways

1. VQ encodes vectors as single codebook indices
2. Exploits correlation within vectors
3. LBG algorithm trains codebook from data
4. Fast decode, slow encode
5. Used in speech coding and texture compression

---

## End of Part C

You now understand the fundamentals of lossy compression:
- Quantization and rate-distortion
- Transform coding (DCT, wavelets)
- Perceptual modeling
- Prediction
- Vector quantization

**Next**: Part D covers specific image compression formats.

---

**Next Part**: [Part D: Image Compression](../d_image/README.md)
