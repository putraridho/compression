# Chapter 32: Context Mixing

## Overview

**Context mixing** combines predictions from multiple models.

Used in PAQ, cmix — the best compression algorithms known.

---

## The Insight

Different models excel at different patterns:

- Order-N models: Sequential patterns
- Match models: Long repeated strings
- Word models: Word-level patterns
- Sparse models: Periodic patterns

Why choose one? **Mix them all!**

---

## Basic Mixing

### Linear Mixing

```
P_mix = w₁P₁ + w₂P₂ + ... + wₙPₙ

where Σwᵢ = 1
```

Weights can be fixed or adaptive.

### Logarithmic Mixing

Work in log-odds space:

```
stretch(p) = ln(p / (1-p))

mixed_stretch = w₁×stretch(P₁) + w₂×stretch(P₂) + ...

P_mix = squash(mixed_stretch) = 1 / (1 + e^(-mixed_stretch))
```

Better behaved mathematically.

---

## PAQ Architecture

```
┌─────────────────────────────────────────────────┐
│                    Bit                          │
├─────────────────────────────────────────────────┤
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐           │
│  │Model1│ │Model2│ │Model3│ │Model4│ ... (100+)│
│  └───┬──┘ └───┬──┘ └───┬──┘ └───┬──┘           │
│      │        │        │        │               │
│      └────────┴────────┴────────┘               │
│                   │                             │
│              ┌────▼────┐                        │
│              │  Mixer  │                        │
│              └────┬────┘                        │
│                   │                             │
│              ┌────▼────┐                        │
│              │  SSE    │ (Secondary Symbol Est.)│
│              └────┬────┘                        │
│                   │                             │
│              ┌────▼────┐                        │
│              │ Arith.  │                        │
│              │ Coder   │                        │
│              └────┬────┘                        │
│                   ▼                             │
│               Output                            │
└─────────────────────────────────────────────────┘
```

---

## Model Types in PAQ

### Context Models

```
Order-1 through Order-8 byte contexts
Order-1 through Order-32 bit contexts
Word contexts
Sparse contexts (every 2nd, 3rd byte)
```

### Match Models

```
Long match finder (like LZ)
Outputs match length and expected bit
```

### Specialized Models

```
Text model (words, sentences)
Image model (2D prediction)
Audio model (linear prediction)
x86 model (for executables)
```

---

## Neural Network Mixing

PAQ8 and later use neural networks:

```
inputs: stretched probabilities from models
hidden: weighted sum with tanh activation
output: mixed probability

weights updated by gradient descent on prediction error
```

### Simple NN Mixer

```
function mix(predictions, context):
    weights = weight_table[context]

    // Forward pass
    sum = 0
    for i in 0..num_models:
        sum += weights[i] × stretch(predictions[i])

    output = squash(sum)

    // Backward pass (after seeing actual bit)
    error = actual_bit - output
    for i in 0..num_models:
        weights[i] += learning_rate × error × stretch(predictions[i])

    return output
```

---

## SSE (Secondary Symbol Estimation)

After mixing, apply table-based correction:

```
final_prob = SSE[context][quantized_mixed_prob]
```

Learns residual errors in mixing.

---

## Context for Mixing

Mixer weights depend on context:

```
// Different weights for different situations
if (in_text_region):
    use text_model_weights
else if (in_image_region):
    use image_model_weights
else:
    use general_weights
```

Context can include:
- Previous predictions
- Match length
- File type indicators
- Recent symbols

---

## Why Context Mixing Wins

### Comparison (enwik8 benchmark)

| Algorithm | bits/byte | Compress MB/s |
|-----------|-----------|---------------|
| gzip -9 | 3.0 | 20 |
| bzip2 -9 | 2.1 | 5 |
| PPMD -o8 | 1.9 | 2 |
| PAQ8 | 1.3 | 0.02 |
| cmix | 1.2 | 0.001 |

Context mixing achieves **50%+ better** than LZ methods!

---

## The Cost

### Speed

- PAQ8: ~20 KB/s compression
- cmix: ~1 KB/s compression
- 1000x slower than gzip

### Memory

- PAQ8: ~1-2 GB
- cmix: ~32 GB
- 1000x more than gzip

### Complexity

- Hundreds of models
- Thousands of lines of code
- Constant tuning required

---

## cmix

Current state-of-the-art:

1. 2000+ context models
2. LSTM neural networks
3. Multiple mixing layers
4. Preprocessing for different file types
5. Achieves 1.16 bits/byte on enwik8

---

## Implementation Sketch

```
struct ContextMixer {
    models: Vec<Model>
    mixer: NeuralNet
    sse: SSETable
}

impl ContextMixer {
    fn predict(&mut self, ctx: &Context) -> f32 {
        // Get predictions from all models
        let preds: Vec<f32> = self.models
            .iter_mut()
            .map(|m| m.predict(ctx))
            .collect();

        // Mix predictions
        let mixed = self.mixer.forward(&preds, ctx);

        // SSE refinement
        let final_prob = self.sse.refine(mixed, ctx);

        final_prob
    }

    fn update(&mut self, bit: u8) {
        // Update all models
        for model in &mut self.models {
            model.update(bit);
        }

        // Update mixer weights
        self.mixer.backward(bit);

        // Update SSE
        self.sse.update(bit);
    }
}
```

---

## Key Takeaways

1. Combine predictions from many models
2. Mix in log-odds (stretch) space
3. Use neural networks for adaptive weighting
4. Context-dependent mixer weights
5. SSE for final refinement
6. Best compression but extremely slow

---

## End of Part B

You've now learned:
- All major lossless compression techniques
- From simple RLE to state-of-art context mixing
- Trade-offs between speed and compression

**Next**: Part C covers lossy compression fundamentals.

---

**Next Part**: [Part C: Lossy Fundamentals](../c_lossy_fundamentals/README.md)
