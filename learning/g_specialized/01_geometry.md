# Chapter 1: 3D and Geometry Compression

## Overview

3D data (meshes, point clouds) requires specialized compression.

---

## Mesh Components

```
Vertices: 3D coordinates (x, y, z)
Faces: Triangles referencing vertices
Attributes: Normals, colors, UVs
```

---

## Draco (Google)

Modern geometry compressor for WebGL/glTF.

### Techniques

1. **Vertex prediction**: Predict from neighbors, encode residual
2. **Parallelogram prediction**: Use mesh topology
3. **Quantization**: Reduce coordinate precision
4. **Entropy coding**: ANS for final compression

### Compression

```
Typical ratios: 10-20x
Speed: Fast encode/decode
```

---

## Point Cloud Compression

For LiDAR data, 3D scans:

### Octree Coding

```
Divide space recursively:
         ┌───┬───┐
        ╱   ╱   ╱│
       ├───┼───┤ │
      ╱   ╱   ╱│ │
     └───┴───┘ │ │
               │ ╱
               │╱
               └

Encode occupancy at each level
```

### Prediction

Predict point positions from neighbors.

---

## Key Takeaways

1. 3D data has spatial and connectivity structure
2. Prediction exploits mesh topology
3. Quantization reduces precision
4. 10-20x compression typical
5. Draco is industry standard for web

---

**Next Chapter**: [Time Series Compression](./02_timeseries.md)
