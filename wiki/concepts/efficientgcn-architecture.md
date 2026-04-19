---
title: EfficientGCN Architecture
type: concept
tags: [efficientgcn, graph-convolutional-network, skeleton, action-recognition, depthwise-separable, compound-scaling]
sources: [capstone-rehab-exercise-assessment]
last_updated: 2026-04-14
---

# EfficientGCN Architecture

## Definition

EfficientGCN (Song et al., 2022) is a family of skeleton-based action recognition models designed for computational efficiency. It achieves competitive accuracy with significantly fewer parameters and lower FLOPs than prior state-of-the-art GCNs by combining three techniques: **early fusion of multiple input streams**, **depthwise separable convolutions** in both spatial and temporal dimensions, and **compound scaling** to trade off speed vs. accuracy across a model family (B0–B4).

## Why It Matters

Prior GCN models (MS-G3D: 3.2M params, InfoGCN: 1.5M params) grew increasingly large and compute-intensive. EfficientGCN-B4 achieves 91.7% accuracy on NTU RGB+D 60 while being **3.15× smaller** and **3.21× faster** than MS-G3D. The B0 variant (~690K params) runs at ~15 fps on CPU-only consumer hardware — a direct enabler of home-based rehabilitation deployment without GPU requirements.

## Key Properties / Dimensions

### 1. Multiple Input Branches (MIB) — Early Fusion

Traditional two-stream GCNs (e.g., 2s-AGCN) process joint positions and bone vectors through separate GCN stacks and fuse predictions at the output (late fusion). EfficientGCN instead **concatenates all input streams along the channel dimension before the first GCN layer** (early fusion):

```
Input tensor: [joint_angles | angle_velocity | relative_angles]
Shape: (N, 9, T, V, 1)  — 3 channels per stream × 3 streams
```

This eliminates the need for duplicate GCN stacks and late fusion modules, cutting model size roughly in half compared to multi-stream architectures with separate processing paths. The trade-off is that all streams must be compatible in spatial dimension from the start.

In the capstone project adaptation, the 3 streams are:
- **Angle stream** (ch 0–2): normalized joint angles θ/π ∈ [0,1] — instantaneous angular configuration
- **Velocity stream** (ch 3–5): frame-to-frame temporal differences Δθ — movement dynamics (smooth vs jerky)
- **Relative stream** (ch 6–8): each joint's angle minus its parent's angle in the skeleton hierarchy — hierarchical kinematic chain encoding

### 2. Separable Graph Convolution (SGC)

Standard graph convolution for `C_in` input channels and `C_out` output channels requires weight matrices of size `C_in × C_out` per partition (×3 for 3-partition). EfficientGCN factorizes this:

**Step 1 — Spatial graph conv**: Aggregate features over the 3-partition neighborhood using the adjacency matrix. Operates on channels independently (depthwise) or with a lightweight spatial weight. Output channels = input channels.

**Step 2 — Pointwise conv (1×1)**: A standard 1×1 convolution mixes information across channels after spatial aggregation.

This factorization reduces parameters by roughly a factor of `V` (number of joints) compared to a fully connected graph conv — critical for keeping the model compact. The adjacency matrix is initialized from the physical skeleton topology and made learnable, allowing the model to soft-weight edges during training.

### 3. Depthwise Temporal Convolution (TCN)

Each GCN block's temporal component uses **depthwise separable 1D convolution** along the time axis:

**Depthwise conv**: A separate 1D filter of kernel size 9 is applied independently to each channel. This captures per-channel temporal patterns without mixing channels. Parameter count: `C × kernel_size` instead of `C_in × C_out × kernel_size`.

**Pointwise conv**: A 1×1 convolution then mixes channel information. Parameter count: `C_in × C_out`.

Total cost ≈ `C(K + C_out)` vs `C_in × C_out × K` for standard conv — a reduction by factor ≈ `K/C_out + 1/C_in`, roughly `9×` cheaper for typical channel counts.

**Temporal stride**: Blocks 3 and 5 use stride 2, progressively downsampling the temporal axis:
- Blocks 0–2: 200 frames (full resolution)
- Blocks 3–4: 100 frames (after first stride)
- Blocks 5–7: 50 frames (after second stride)

Deeper blocks operate on shorter sequences but richer feature maps — they capture longer-range temporal dependencies by seeing a compressed view of the full sequence.

### 4. Compound Scaling (B0–B4)

Inspired by EfficientNet, EfficientGCN defines a model family by simultaneously scaling width (channel dimensions) and depth (number of blocks):

| Variant | Channels | Depth multiplier | ~Params |
|---------|----------|-----------------|---------|
| B0 | [64,64,64,64,128,128,256,256] | 1× | ~690K |
| B1 | [64,64,64,128,128,256,256,256] | 1× | ~1.0M |
| B2 | [64,64,128,128,256,256,512,512] | 1× | ~2.0M |
| B4 | larger | deeper | ~3.5M |

The compound coefficient ensures width and depth scale together — neither is inflated while the other stays fixed — which EfficientNet showed produces better scaling efficiency than naive single-axis scaling.

### 5. Capstone Adaptation (B0 for Rehabilitation)

The capstone project adapts EfficientGCN-B0 for binary rehabilitation exercise classification with the following modifications:

| Modification | Standard EfficientGCN | Capstone Adaptation |
|---|---|---|
| Input channels | 9 (joint pos + bone + velocity) | 9 (angle + velocity + relative) |
| Input modality | 3D Kinect positions | Sensor-agnostic joint angles |
| Skeleton | NTU RGB+D 25 joints | Kinect v2 22 joints |
| Output | 60 or 120 action classes | 2 (correct / incorrect) |
| Task | Action recognition | Exercise quality classification |

**Full layer-by-layer architecture:**

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

### 7. Inference Efficiency

On consumer hardware (Intel Core i7, no GPU):
- MediaPipe pose estimation: ~38 ms/frame (bottleneck — 60% of total)
- Joint angle computation: ~0.5 ms/frame
- EfficientGCN inference: ~25 ms/prediction
- **Total: ~64 ms → ~15 fps**

With GPU (NVIDIA T4):
- EfficientGCN inference drops to ~4.5 ms
- **Total: ~43 ms → ~23 fps**

Comparison: MS-G3D (3.2M params) and InfoGCN (1.5M params) would significantly exceed real-time budgets on CPU — EfficientGCN-B0 is the practical choice for home deployment.

## Evidence & Examples

From [[sources/capstone-rehab-exercise-assessment]]:
- B0 achieves 96.3% / F1 0.963 on UI-PRMD — 0.6% below the more expensive EGCN ensemble (96.9%) which uses two separate GCN models.
- The model correctly handles out-of-distribution inputs: a side lunge (not in the training set) is classified as 89% incorrect.
- Training curves show the model learns generalizable patterns up to epoch ~130, even though training loss converges near zero by epoch 50 — the generalization gap is stable, not diverging.

## Contradictions & Open Questions

- The MIB early fusion assumption (that all streams can share the same GCN layers) may be suboptimal if different streams require different spatial aggregation patterns. Late fusion (separate stacks) could recover some accuracy at the cost of model size.
- Whether the compound scaling rationale transfers from image CNNs (EfficientNet) to skeleton GCNs is not rigorously proven — the optimal scaling law may differ for graph-structured data.
- BatchNorm in EfficientGCN accumulates running statistics that encode domain-specific distributions, causing the transfer learning failure observed in the capstone (63.8% on REHAB24-6). Instance normalization or layer normalization might generalize better across datasets.

## Related

- [[concepts/graph-convolutional-networks]] — the broader GCN theory and ST-GCN lineage this builds on
- [[concepts/sensor-agnostic-features]] — the input representation fed into EfficientGCN
- [[concepts/skeleton-based-action-recognition]] — field context and competing architectures
- [[sources/capstone-rehab-exercise-assessment]] — where this architecture was applied and evaluated
