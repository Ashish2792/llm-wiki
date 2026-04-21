---
title: EfficientGCN Architecture
type: concept
tags: [efficientgcn, graph-convolutional-network, skeleton, action-recognition, depthwise-separable, compound-scaling]
sources: [capstone-rehab-exercise-assessment]
last_updated: 2026-04-20
---

# EfficientGCN Architecture

## Definition

EfficientGCN (Song et al., 2022) is a family of skeleton-based action recognition models designed for computational efficiency. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]] It achieves competitive accuracy with significantly fewer parameters and lower FLOPs than prior state-of-the-art GCNs by combining three techniques: **early fusion of multiple input streams**, **depthwise separable convolutions** in both spatial and temporal dimensions, and **compound scaling** to trade off speed vs. accuracy across a model family (B0–B4). {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]]

## Why It Matters

Prior GCN models grew increasingly large: MS-G3D has 3.2M parameters and InfoGCN has 1.5M parameters. {paraphrase} [[sources/capstone-rehab-exercise-assessment#results]] EfficientGCN-B0 (~690K params) runs at ~15 fps on CPU-only consumer hardware — a direct enabler of home-based rehabilitation deployment without GPU requirements. {direct} [[sources/capstone-rehab-exercise-assessment#results]] This efficiency is a direct consequence of architectural choices (early fusion, depthwise separable convolutions) rather than accuracy sacrifice. {inferred} [[sources/capstone-rehab-exercise-assessment#methodology]]

## Key Properties / Dimensions

### 1. Multiple Input Branches (MIB) — Early Fusion

Traditional two-stream GCNs (e.g., 2s-AGCN) process joint positions and bone vectors through separate GCN stacks and fuse predictions at the output (late fusion). {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]] EfficientGCN instead **concatenates all input streams along the channel dimension before the first GCN layer** (early fusion): {paraphrase} [[sources/capstone-rehab-exercise-assessment#methodology]]

```
Input tensor: [joint_angles | angle_velocity | relative_angles]
Shape: (N, 9, T, V, 1)  — 3 channels per stream × 3 streams
```

This eliminates duplicate GCN stacks and late fusion modules, cutting model size roughly in half compared to multi-stream architectures with separate processing paths. {inferred} [[sources/capstone-rehab-exercise-assessment#methodology]] The trade-off is that all streams must be compatible in spatial dimension from the start. {inferred} [[sources/capstone-rehab-exercise-assessment#methodology]]

In the capstone project adaptation, the 3 streams are: {direct} [[sources/capstone-rehab-exercise-assessment#methodology]]

- **Angle stream** (ch 0–2): normalized joint angles θ/π ∈ [0,1] — instantaneous angular configuration
- **Velocity stream** (ch 3–5): frame-to-frame temporal differences Δθ — movement dynamics (smooth vs jerky)
- **Relative stream** (ch 6–8): each joint's angle minus its parent's angle in the skeleton hierarchy — hierarchical kinematic chain encoding

### 2. Separable Graph Convolution (SGC)

EfficientGCN factorizes spatial graph convolution into two steps: {paraphrase} [[sources/capstone-rehab-exercise-assessment#methodology]]

**Step 1 — Spatial graph conv**: Aggregate features over the 3-partition neighborhood using the adjacency matrix. The three partitions are: self (each joint with itself), centripetal (neighbors closer to SpineBase along the tree path), and centrifugal (neighbors farther from SpineBase). {direct} [[sources/capstone-rehab-exercise-assessment#methodology]]

**Step 2 — Pointwise conv (1×1)**: A standard 1×1 convolution mixes information across channels after spatial aggregation. {paraphrase} [[sources/capstone-rehab-exercise-assessment#methodology]]

The adjacency matrix is initialized from the physical Kinect v2 skeleton topology (22 joints, 23 edges) and made learnable, allowing the model to soft-weight edges during training and potentially discover non-obvious joint relationships. {paraphrase} [[sources/capstone-rehab-exercise-assessment#methodology]]

### 3. Depthwise Temporal Convolution (TCN)

Each GCN block's temporal component uses **depthwise separable 1D convolution** along the time axis: {paraphrase} [[sources/capstone-rehab-exercise-assessment#methodology]]

**Depthwise conv**: A separate 1D filter of kernel size 9 is applied independently to each channel, capturing per-channel temporal patterns. {paraphrase} [[sources/capstone-rehab-exercise-assessment#methodology]]

**Pointwise conv**: A 1×1 convolution then mixes channel information. {paraphrase} [[sources/capstone-rehab-exercise-assessment#methodology]]

**Temporal stride**: Blocks 3 and 5 use stride 2, progressively downsampling the temporal axis: {direct} [[sources/capstone-rehab-exercise-assessment#methodology]]

- Blocks 0–2: 200 frames (full resolution)
- Blocks 3–4: 100 frames (after first stride-2)
- Blocks 5–7: 50 frames (after second stride-2)

Deeper blocks operate on shorter, compressed sequences, allowing them to capture longer-range temporal dependencies. {inferred} [[sources/capstone-rehab-exercise-assessment#methodology]]

### 4. Compound Scaling (B0–B4)

EfficientGCN defines a model family by simultaneously scaling width (channel dimensions): {paraphrase} [[sources/capstone-rehab-exercise-assessment#methodology]]

| Variant | Channels | ~Params |
|---------|----------|---------|
| B0 | [64,64,64,64,128,128,256,256] | ~690K |
| B1 | [64,64,64,128,128,256,256,256] | ~1.0M |
| B2 | [64,64,128,128,256,256,512,512] | ~2.0M |
| B4 | larger | ~3.5M |

For the combined WLU+REHAB24-6 experiment (only ~1,020 training sequences), a reduced configuration [48,48,96,96,192,192,192,192] (~370K params) was used to avoid overfitting with limited data. {direct} [[sources/capstone-rehab-exercise-assessment#results]]

### 5. Capstone Adaptation (B0 for Rehabilitation)

Full layer-by-layer architecture: {direct} [[sources/capstone-rehab-exercise-assessment#methodology]]

| Component | Details | Output Shape | Params |
|---|---|---|---|
| Input | — | (N, 9, 200, 22, 1) | — |
| Stem | Conv2d(9→64) + BN + ReLU | (N, 64, 200, 22) | 640 |
| Block 0 | SGC + TCN, stride 1 | (N, 64, 200, 22) | ~17K |
| Block 1 | SGC + TCN, stride 1 | (N, 64, 200, 22) | ~17K |
| Block 2 | SGC + TCN, stride 1 | (N, 64, 200, 22) | ~17K |
| Block 3 | SGC + TCN, stride 2 | (N, 64, 100, 22) | ~17K |
| Block 4 | SGC + TCN, stride 1 | (N, 128, 100, 22) | ~50K |
| Block 5 | SGC + TCN, stride 2 | (N, 128, 50, 22) | ~50K |
| Block 6 | SGC + TCN, stride 1 | (N, 256, 50, 22) | ~165K |
| Block 7 | SGC + TCN, stride 1 | (N, 256, 50, 22) | ~165K |
| Global Pool | Avg over T, V | (N, 256) | — |
| Dropout | p = 0.5 | (N, 256) | — |
| FC | Linear(256→2) | (N, 2) | 514 |
| **Total** | | | **~690K** |

Modifications from the original EfficientGCN: input is 9-channel sensor-agnostic joint angles (not raw positions), skeleton is Kinect v2 22-joint (not NTU 25-joint), output is 2 classes (not 60 or 120). {paraphrase} [[sources/capstone-rehab-exercise-assessment#methodology]]

### 6. Training Configuration

| Parameter | Value |
|---|---|
| Optimizer | Adam |
| Learning rate | 5×10⁻⁴ |
| Weight decay | 1×10⁻⁴ (L2) |
| Batch size | 32 |
| Max epochs | 200 |
| Early stopping patience | 30 epochs (on validation F1) |
| LR scheduler | ReduceLROnPlateau (factor 0.5, patience 10) |
| Dropout rate | 0.5 (before FC) |
| Loss | Cross-Entropy |
| Best epoch | 139 (out of 200) |

{direct} [[sources/capstone-rehab-exercise-assessment#methodology]]

The jump from 85.3% (v1) to 92.3% (v3) was achieved purely through hyperparameter tuning — reducing learning rate from 10⁻³ to 5×10⁻⁴ and extending training — demonstrating GCN sensitivity to optimization configuration. {paraphrase} [[sources/capstone-rehab-exercise-assessment#results]]

### 7. Inference Efficiency

On consumer hardware (Intel Core i7, no GPU): {direct} [[sources/capstone-rehab-exercise-assessment#results]]

- MediaPipe pose estimation: ~38 ms/frame (bottleneck — 60% of total)
- Joint angle computation: ~0.5 ms/frame
- EfficientGCN inference: ~25 ms/prediction
- **Total: ~64 ms → ~15 fps**

With GPU (NVIDIA T4): {direct} [[sources/capstone-rehab-exercise-assessment#results]]

- EfficientGCN inference drops to ~4.5 ms
- **Total: ~43 ms → ~23 fps**

MediaPipe is the system bottleneck regardless of GPU availability, since it runs on CPU even when a GPU is present. {inferred} [[sources/capstone-rehab-exercise-assessment#results]] Optimizing the GCN architecture has diminishing returns compared to improving pose estimation speed for CPU-only deployment. {inferred} [[sources/capstone-rehab-exercise-assessment#discussion]]

## Evidence & Examples

B0 achieves 96.3% / F1 0.963 on UI-PRMD — 0.6% below the EGCN ensemble (96.9%) which uses two separate GCN models. {direct} [[sources/capstone-rehab-exercise-assessment#results]] The ensemble gains are marginal and don't justify the 2× compute and deployment cost. {inferred} [[sources/capstone-rehab-exercise-assessment#discussion]]

The model correctly handles out-of-distribution inputs: a side lunge (not in training set) is classified as incorrect on 89% of frames. {direct} [[sources/capstone-rehab-exercise-assessment#results]]

Training curves show the model learns generalizable patterns up to epoch ~130, even though training loss converges near zero by epoch 50. {paraphrase} [[sources/capstone-rehab-exercise-assessment#results]] The generalization gap (training accuracy ~100% vs validation ~96%) is stable and consistent with model capacity relative to dataset size. {paraphrase} [[sources/capstone-rehab-exercise-assessment#results]]

## Contradictions & Open Questions

The MIB early fusion assumption — that all streams can share the same GCN layers — may be suboptimal if different streams require different spatial aggregation patterns. {inferred} [[sources/capstone-rehab-exercise-assessment#methodology]] Late fusion (separate stacks) could recover some accuracy at the cost of model size.

BatchNorm in EfficientGCN accumulates running statistics that encode domain-specific feature distributions, causing the transfer learning failure observed in the capstone: 63.8% on REHAB24-6 vs 72% training from scratch. {direct} [[sources/capstone-rehab-exercise-assessment#results]] Instance normalization or layer normalization might enable successful cross-dataset fine-tuning. {inferred} [[sources/capstone-rehab-exercise-assessment#discussion]]

## Related

- [[concepts/graph-convolutional-networks]] — the broader GCN theory and ST-GCN lineage this builds on
- [[concepts/sensor-agnostic-features]] — the input representation fed into EfficientGCN
- [[concepts/skeleton-based-action-recognition]] — field context and competing architectures
- [[sources/capstone-rehab-exercise-assessment]] — where this architecture was applied and evaluated
