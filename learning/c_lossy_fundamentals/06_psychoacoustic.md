# Chapter 6: Psychoacoustic Modeling

## What is Psychoacoustics?

**Psychoacoustics** studies how humans perceive sound.

Audio codecs exploit this: discard sounds we can't hear.

---

## Key Phenomena

### 1. Absolute Hearing Threshold

We can't hear very quiet sounds at any frequency.

```
Sensitivity (dB SPL)
    │
 80 │                         ╱
    │                       ╱
 60 │                     ╱
    │      ╲            ╱
 40 │        ╲        ╱
    │          ╲    ╱
 20 │            ╲╱
    │              Most sensitive: 2-5 kHz
  0 ├──────┬──────┬──────┬──────┬──────>
    20Hz  200Hz  2kHz  10kHz  20kHz
```

### 2. Frequency Masking

Loud sounds at one frequency mask nearby frequencies.

```
Loud tone at 1kHz:
    Cannot hear quiet tones at 800Hz-1.2kHz
```

### 3. Temporal Masking

Sounds are masked just before and after loud sounds.

```
Pre-masking: 5-20ms before loud sound
Post-masking: 50-200ms after loud sound
```

---

## Critical Bands

The ear groups frequencies into ~25 critical bands (Bark scale).

```
Band 1:  20-100 Hz (80 Hz wide)
Band 2:  100-200 Hz (100 Hz wide)
...
Band 24: 13500-20000 Hz (wide)
```

Masking only occurs within the same critical band.

---

## The Masking Model

### Step 1: Frequency Analysis

Convert time domain to frequency using FFT/MDCT.

### Step 2: Identify Maskers

Find tonal (narrow) and noise-like (broadband) components.

### Step 3: Calculate Masking Thresholds

For each frequency, compute:
```
Threshold = max(absolute_threshold, sum_of_masking_from_neighbors)
```

### Step 4: Determine Inaudible Components

Anything below threshold can be removed!

---

## MP3's Psychoacoustic Model

```
Audio → FFT → Find peaks → Calculate masking → Allocate bits
                              ↓
                    ┌─────────────────────┐
                    │ Masking threshold   │
                    │ varies by frequency │
                    └─────────────────────┘
```

### Bit Allocation

```
Frequencies below threshold: Zero bits
Frequencies just above: Few bits
Frequencies well above: More bits
```

---

## Spreading Function

Masking spreads across frequencies:

```
     Masking Level
          │
    ████  │    ╱╲
    ████  │   ╱  ╲
    ████  │  ╱    ╲
    ████  │ ╱      ╲
    ████──┼╱────────╲───────────
          │         ↑
      Masker    Spread to neighbors
```

Steeper slope toward low frequencies.
Gentler slope toward high frequencies.

---

## Pre-Echo Problem

### The Issue

```
Time →
        ─────┐
             │
             │ ← Attack (loud sound starts)
             │
             └─────

With MDCT transform:
  Quantization noise spreads across block
  Noise appears BEFORE the attack
  Audible "pre-echo"
```

### Solutions

1. **Shorter blocks**: Less spreading (MP3 uses short blocks for attacks)
2. **Temporal noise shaping**: Shape noise in time domain
3. **Block switching**: Dynamically choose block size

---

## Coding Efficiency

Without psychoacoustic model:
```
CD quality: 1411 kbps
Transparent: Not possible at low rates
```

With psychoacoustic model:
```
MP3 320 kbps: Transparent for most
MP3 192 kbps: Good quality
MP3 128 kbps: Acceptable quality
```

Psychoacoustics enables 10x+ compression!

---

## Summary of Exploited Phenomena

| Phenomenon | Exploitation |
|------------|--------------|
| Absolute threshold | Remove inaudible frequencies |
| Frequency masking | Remove masked components |
| Temporal masking | Remove pre/post masked sounds |
| Critical bands | Reduce precision within band |
| Pre-echo | Short blocks for transients |

---

## Key Takeaways

1. Human hearing has limitations
2. Masking hides quiet sounds near loud ones
3. Critical bands define masking regions
4. Psychoacoustic model guides bit allocation
5. Pre-echo requires special handling
6. Enables transparent compression at ~10:1

---

**Next Chapter**: [Psychovisual Modeling](./07_psychovisual.md)
