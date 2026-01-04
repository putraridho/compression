COMPRESSION SYLLABUS - COMPLETE
================================
Every Technique from Beginner to Master

================================================================================
PART A: FOUNDATIONS
================================================================================

A1. THEORY
----------
- What is compression?
- Lossless vs Lossy
- Compression ratio, space savings
- Entropy (Shannon entropy)
- Redundancy types (spatial, temporal, statistical)
- Shannon's source coding theorem
- Kolmogorov complexity
- Rate-distortion theory

================================================================================
PART B: LOSSLESS TECHNIQUES
================================================================================

B1. BASIC ENCODING
------------------
[ ] Run-Length Encoding (RLE)
    - Byte-level RLE
    - Bit-level RLE
    - PackBits (Mac RLE variant)

[ ] Delta Encoding
    - Simple delta
    - XOR delta
    - Differential coding

[ ] Bit Packing
    - Fixed-width packing
    - Variable-width integers (varint)
    - Simple-8b, Simple-16

[ ] Null Suppression
    - Zero-byte elimination
    - Sparse data encoding

B2. STATISTICAL / ENTROPY CODING
--------------------------------
[ ] Shannon Coding
    - Original entropy coding (historical)

[ ] Shannon-Fano Coding
    - Top-down probability split

[ ] Huffman Coding
    - Standard Huffman
    - Canonical Huffman
    - Adaptive Huffman (FGK, Vitter)
    - Length-limited Huffman

[ ] Arithmetic Coding
    - Binary arithmetic coding
    - Multi-symbol arithmetic
    - Adaptive arithmetic coding
    - Q-coder, QM-coder, MQ-coder

[ ] Range Coding
    - Arithmetic coding variant
    - Carryless range coder

[ ] Asymmetric Numeral Systems (ANS)
    - rANS (range variant)
    - tANS (table variant)
    - Streaming ANS
    - Interleaved ANS

[ ] Finite State Entropy (FSE)
    - tANS implementation
    - Used in Zstandard

[ ] Golomb Coding
    - Golomb-Rice coding
    - Optimal for geometric distribution

[ ] Elias Codes
    - Elias Gamma
    - Elias Delta
    - Elias Omega

[ ] Exponential-Golomb (Exp-Golomb)
    - Used in H.264/AVC

[ ] Unary Coding
    - Simple prefix code

[ ] Fibonacci Coding
    - Universal code

[ ] Tunstall Coding
    - Fixed-length output codes

[ ] Levenshtein Coding
    - Universal integer code

B3. DICTIONARY METHODS
----------------------
[ ] LZ77 (Lempel-Ziv 1977)
    - Sliding window
    - Offset-length-literal triples

[ ] LZSS (Lempel-Ziv-Storer-Szymanski)
    - Flag bits for literals
    - More efficient than LZ77

[ ] LZ78 (Lempel-Ziv 1978)
    - Explicit dictionary

[ ] LZW (Lempel-Ziv-Welch)
    - Implicit dictionary growth
    - Used in GIF, TIFF, PDF

[ ] LZMA (Lempel-Ziv-Markov chain)
    - Range coder backend
    - Used in 7z, xz

[ ] LZMA2
    - Parallel-friendly LZMA
    - Reset capability

[ ] LZ4
    - Speed optimized
    - LZ4 HC (high compression variant)

[ ] LZO
    - Very fast decompression
    - miniLZO

[ ] LZF
    - BSD licensed fast compressor

[ ] LZFSE (Apple)
    - FSE + LZ

[ ] LZHAM
    - LZMA alternative

[ ] LZRW family
    - LZRW1, LZRW2, LZRW3, LZRW3-A

[ ] LZX
    - Used in CAB, CHM

[ ] LZJB
    - Used in ZFS

[ ] Snappy
    - Google's fast compressor

[ ] QuickLZ
    - Speed-focused

[ ] Density
    - Dual-pass compression

[ ] Lizard (LZ5)
    - LZ4 + optimal parsing

B4. TRANSFORMS
--------------
[ ] Burrows-Wheeler Transform (BWT)
    - Block sorting
    - Suffix array construction
    - Inverse BWT

[ ] Move-to-Front Transform (MTF)
    - Locality improvement
    - Used after BWT

[ ] Run-Length Transform
    - Post-BWT run-length

[ ] Distance Coding
    - BWT post-processing

[ ] Sorted Rank Transform (SRT)
    - Alternative to MTF

[ ] Bijective BWT
    - No end-of-block marker

B5. PREPROCESSORS / FILTERS
---------------------------
[ ] Delta Filter
    - Byte-wise differences

[ ] BCJ Filter (Branch Conversion)
    - x86 (E8/E9)
    - ARM, ARM64, PowerPC, SPARC, IA-64

[ ] Executable Preprocessor
    - E8E9 transform

[ ] Text Preprocessor
    - Word substitution
    - Dictionary-based

[ ] Structured Data Filter
    - Record reordering

B6. HYBRID ALGORITHMS
---------------------
[ ] DEFLATE
    - LZ77 + Huffman
    - Lazy matching
    - Block types

[ ] DEFLATE64
    - Extended DEFLATE

[ ] Zopfli
    - Optimal DEFLATE parsing
    - Brute-force search

[ ] ZLIB
    - DEFLATE with wrapper

[ ] gzip
    - DEFLATE with file wrapper

[ ] Zstandard (zstd)
    - LZ77 + FSE
    - Dictionary support
    - Streaming

[ ] Brotli
    - LZ77 + context modeling
    - Static dictionary
    - Distance cache

[ ] bzip2
    - BWT + MTF + RLE + Huffman

[ ] bzip3
    - Modern bzip2 replacement

[ ] LZIP
    - LZMA with CRC

[ ] XZ
    - LZMA2 container

B7. CONTEXT MODELING
--------------------
[ ] Prediction by Partial Matching (PPM)
    - PPMa, PPMc, PPMd, PPMz
    - PPMD+
    - Order-n modeling

[ ] Context Tree Weighting (CTW)
    - Optimal Bayesian mixing

[ ] Dynamic Markov Compression (DMC)
    - Finite state machine

[ ] Context Mixing
    - Linear mixing
    - Neural network mixing
    - Logistic mixing

[ ] PAQ Family
    - PAQ1 through PAQ8
    - ZPAQ
    - cmix (state-of-art)
    - Neural networks in compression

B8. ARCHIVER ALGORITHMS
-----------------------
[ ] RAR compression
    - Proprietary algorithm

[ ] 7z (p7zip)
    - LZMA, LZMA2, PPMd

[ ] ZPAQ
    - Context mixing + JIT

[ ] Oodle Family (game industry)
    - Kraken
    - Mermaid
    - Selkie
    - Leviathan

[ ] BSC (Block Sorting Compressor)
    - QLFC + BWT

[ ] CSC
    - Context-based similarity

================================================================================
PART C: LOSSY FUNDAMENTALS
================================================================================

C1. QUANTIZATION
----------------
[ ] Scalar Quantization
    - Uniform quantization
    - Non-uniform quantization
    - Lloyd-Max quantizer

[ ] Vector Quantization (VQ)
    - Codebook generation
    - LBG algorithm
    - Tree-structured VQ

[ ] Dead-zone Quantization
    - Used in video codecs

[ ] Adaptive Quantization
    - Perceptual adaptation

C2. PREDICTIVE CODING
---------------------
[ ] DPCM (Differential PCM)
    - Simple prediction

[ ] ADPCM (Adaptive DPCM)
    - Variable step size

[ ] Linear Predictive Coding (LPC)
    - Used in speech

C3. TRANSFORM CODING
--------------------
[ ] Discrete Cosine Transform (DCT)
    - 1D and 2D DCT
    - Fast DCT algorithms

[ ] Modified DCT (MDCT)
    - Overlapping windows
    - Used in audio

[ ] Discrete Fourier Transform (DFT/FFT)
    - Frequency domain

[ ] Discrete Wavelet Transform (DWT)
    - Multi-resolution
    - Haar wavelet
    - Daubechies wavelets
    - Biorthogonal wavelets

[ ] Karhunen-Loeve Transform (KLT)
    - Optimal decorrelation
    - PCA-based

[ ] Discrete Sine Transform (DST)
    - Used in video

[ ] Hadamard Transform
    - Used in video codecs

C4. PERCEPTUAL CODING
---------------------
[ ] Psychoacoustic Model
    - Absolute threshold
    - Masking curves
    - Critical bands

[ ] Psychovisual Model
    - Contrast sensitivity
    - Spatial masking
    - Color perception (YCbCr)

================================================================================
PART D: IMAGE COMPRESSION
================================================================================

D1. LOSSLESS IMAGE
------------------
[ ] PNG
    - Filters (None, Sub, Up, Average, Paeth)
    - DEFLATE compression
    - Adam7 interlacing

[ ] GIF
    - LZW compression
    - Color palette

[ ] JPEG-LS
    - LOCO-I algorithm
    - Near-lossless mode

[ ] WebP Lossless
    - Transforms + entropy coding

[ ] FLIF
    - MANIAC entropy coding
    - Interlaced

[ ] QOI (Quite OK Image)
    - Simple, fast
    - Run, index, diff, luma

[ ] JPEG XL Lossless
    - Modular mode

[ ] BPG Lossless
    - HEVC intra

[ ] AVIF Lossless
    - AV1 intra

D2. LOSSY IMAGE
---------------
[ ] JPEG
    - Color space (YCbCr)
    - Chroma subsampling (4:2:0, 4:2:2, 4:4:4)
    - 8x8 DCT
    - Quantization tables
    - Huffman/arithmetic coding

[ ] JPEG 2000
    - DWT (wavelet)
    - EBCOT entropy coder
    - ROI coding

[ ] JPEG XR
    - Integer transforms
    - Tile-based

[ ] WebP Lossy
    - VP8 intra frame

[ ] HEIC
    - HEVC intra frame

[ ] AVIF
    - AV1 intra frame
    - Film grain synthesis

[ ] JPEG XL
    - VarDCT mode
    - Modular mode
    - Progressive decode

[ ] BPG
    - HEVC-based

================================================================================
PART E: AUDIO COMPRESSION
================================================================================

E1. LOSSLESS AUDIO
------------------
[ ] FLAC
    - LPC prediction
    - Rice coding
    - Frame structure

[ ] ALAC (Apple Lossless)
    - Similar to FLAC

[ ] WavPack
    - Hybrid lossy/lossless

[ ] APE (Monkey's Audio)
    - High compression ratio

[ ] TAK
    - Tom's Audio Kompressor

[ ] TTA
    - True Audio

[ ] Shorten
    - Historical

E2. LOSSY AUDIO
---------------
[ ] MP3 (MPEG-1 Layer III)
    - Subband filtering
    - MDCT
    - Psychoacoustic model
    - Huffman coding
    - Bit reservoir

[ ] AAC
    - Advanced Audio Coding
    - MDCT
    - TNS, PNS

[ ] HE-AAC (AAC+)
    - SBR (Spectral Band Replication)
    - PS (Parametric Stereo)

[ ] Vorbis
    - Ogg container
    - Floor/residue coding

[ ] Opus
    - Hybrid SILK + CELT
    - Low latency

[ ] AC3 (Dolby Digital)
    - Surround sound

[ ] DTS
    - Cinema audio

[ ] WMA
    - Windows Media Audio

[ ] Musepack (MPC)
    - Quality-focused

E3. SPEECH CODECS
-----------------
[ ] G.711 (PCM)
    - A-law, μ-law

[ ] G.726 (ADPCM)
    - 16-40 kbps

[ ] G.729
    - CS-ACELP
    - 8 kbps

[ ] AMR (Adaptive Multi-Rate)
    - AMR-NB, AMR-WB

[ ] Speex
    - VoIP optimized

[ ] Codec2
    - Very low bitrate

[ ] LPCNet
    - Neural vocoder

[ ] CELP, ACELP, RCELP
    - Code-excited linear prediction

================================================================================
PART F: VIDEO COMPRESSION
================================================================================

F1. VIDEO FUNDAMENTALS
----------------------
[ ] Frame Types
    - I-frame (Intra)
    - P-frame (Predicted)
    - B-frame (Bidirectional)

[ ] GOP Structure
    - Open vs Closed GOP
    - Hierarchical B-frames

[ ] Motion Estimation
    - Block matching
    - Full search, diamond, hexagon
    - Sub-pixel motion

[ ] Motion Compensation
    - Fractional-pel interpolation
    - Bi-prediction

[ ] Rate Control
    - CBR, VBR, CRF
    - QP adjustment

F2. VIDEO CODECS
----------------
[ ] MPEG-1
    - Historical standard

[ ] MPEG-2 (H.262)
    - DVD, broadcast

[ ] MPEG-4 Part 2
    - DivX, Xvid era

[ ] H.264 / AVC
    - Intra prediction (9 modes)
    - Inter prediction
    - CABAC / CAVLC
    - Deblocking filter

[ ] H.265 / HEVC
    - CTU/CU/PU/TU structure
    - 35 intra modes
    - SAO filter

[ ] VP8
    - WebM container
    - Boolean coder

[ ] VP9
    - Superblocks
    - Tile-based

[ ] AV1
    - 10-bit default
    - Film grain synthesis
    - CDEF filter

[ ] VVC / H.266
    - Latest standard
    - ALF, LMCS, BDOF

[ ] AVS (Chinese standard)
    - AVS2, AVS3

F3. VIDEO ENTROPY CODING
------------------------
[ ] CAVLC
    - Context-Adaptive VLC

[ ] CABAC
    - Context-Adaptive Binary Arithmetic

[ ] Boolean Coder (VP8/VP9)
    - Range coding variant

================================================================================
PART G: SPECIALIZED COMPRESSION
================================================================================

G1. 3D / GEOMETRY
-----------------
[ ] Draco
    - Google's 3D compressor
    - Point cloud, mesh

[ ] OpenCTM
    - Compact 3D mesh

[ ] glTF compression
    - KHR_mesh_quantization
    - EXT_meshopt_compression

G2. TIME SERIES
---------------
[ ] Gorilla (Facebook)
    - Timestamp + value compression

[ ] Delta-of-delta
    - For monotonic timestamps

[ ] Simple-8b
    - Integer packing

[ ] FOR (Frame of Reference)
    - Columnar compression

G3. DATABASE / COLUMNAR
-----------------------
[ ] Dictionary Encoding
    - Categorical columns

[ ] Run-Length Encoding
    - Sorted columns

[ ] Bit Packing
    - Integer columns

[ ] Parquet compression
    - Snappy, zstd, gzip

[ ] ORC compression
    - Zlib, snappy, LZO

G4. TEXT / DOCUMENT
-------------------
[ ] SCSU (Unicode compression)
    - Standard Compression Scheme

[ ] BOCU-1
    - Binary Ordered Unicode

[ ] Word-based compression
    - Lexical modeling

G5. NEURAL / LEARNED
--------------------
[ ] Learned Image Compression
    - Autoencoder-based
    - Hyperprior models

[ ] Neural Video Compression
    - Optical flow + learned

[ ] Compressive Autoencoders
    - End-to-end training

G6. GENOMIC / SCIENTIFIC
------------------------
[ ] DNA Compression
    - 2-bit encoding
    - Reference-based

[ ] FASTQ Compression
    - Quality score compression

[ ] NetCDF / HDF5
    - Scientific data

G7. NETWORK / PROTOCOL
----------------------
[ ] HTTP Compression
    - gzip, br, zstd

[ ] Header Compression
    - HPACK (HTTP/2)
    - QPACK (HTTP/3)

[ ] Delta Sync
    - rsync algorithm
    - zsync

================================================================================
PART H: IMPLEMENTATION ORDER
================================================================================

BEGINNER (implement these first):
1. [ ] RLE encoder/decoder
2. [ ] Bit packing
3. [ ] Delta encoding
4. [ ] Unary coding
5. [ ] Elias gamma coding

INTERMEDIATE:
6. [ ] Huffman coding (tree + canonical)
7. [ ] LZ77 sliding window
8. [ ] LZSS with flags
9. [ ] Golomb-Rice coding
10.[ ] Adaptive Huffman

ADVANCED:
11.[ ] Arithmetic coding
12.[ ] Range coding
13.[ ] LZ78 / LZW
14.[ ] BWT + inverse BWT
15.[ ] MTF transform

EXPERT:
16.[ ] ANS (rANS + tANS)
17.[ ] DEFLATE (full implementation)
18.[ ] PPM (order-1, order-2)
19.[ ] Context mixing basics
20.[ ] Mini-zstd

MASTER:
21.[ ] Full JPEG decoder
22.[ ] Simple video codec (I-frames only)
23.[ ] Audio LPC + encoding
24.[ ] Custom hybrid algorithm
25.[ ] Benchmark against standards

================================================================================
RESOURCES
================================================================================

BOOKS:
- "Data Compression Explained" - Matt Mahoney (free online)
- "Introduction to Data Compression" - Khalid Sayood
- "Managing Gigabytes" - Witten, Moffat, Bell
- "The Data Compression Book" - Mark Nelson
- "Handbook of Data Compression" - Salomon, Motta

SPECIFICATIONS:
- RFC 1951 (DEFLATE)
- RFC 7932 (Brotli)
- RFC 8478 (Zstandard)
- RFC 8878 (Zstandard framing)
- ITU-T T.81 (JPEG)
- ISO 14495-1 (JPEG-LS)

RUST CRATES:
- flate2 (DEFLATE/zlib/gzip)
- zstd
- lz4, lz4_flex
- brotli
- snap (Snappy)
- xz2, lzma
- bzip2
- weezl (LZW)

ONLINE:
- Matt Mahoney's Data Compression page
- encode.su compression forum
- Squash Compression Benchmark
- lzbench (benchmark tool)
