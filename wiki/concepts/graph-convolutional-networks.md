---
title: Graph Convolutional Networks for Skeleton-Based Action Recognition
type: concept
tags: [graph-convolutional-network, skeleton, action-recognition, deep-learning, spatial-temporal]
sources: [capstone-rehab-exercise-assessment]
last_updated: 2026-04-14
---

# Graph Convolutional Networks for Skeleton-Based Action Recognition

## Definition

A Graph Convolutional Network (GCN) is a neural network that operates directly on graph-structured data by generalizing the convolution operation from regular grids (images) to irregular graphs. In the skeleton-based action recognition context, the human skeleton is modeled as a spatial-temporal graph where **joints are nodes** and **bones are edges**, and GCN layers aggregate features from neighboring joints to learn discriminative representations of movement.

## Why It Matters

The human skeleton is inherently a graph, not a grid. Earlier approaches that fed joint coordinates into CNNs or LSTMs either ignored the spatial structure entirely (LSTMs) or forced an arbitrary grid ordering that doesn't reflect the body's topology (CNNs). GCNs are the natural fit: they respect the kinematic chain structure of the skeleton, allow the model to learn that the elbow is more closely related to the wrist than to the knee, and can capture coordinated multi-joint patterns that are critical for exercise quality assessment.

## Key Properties / Dimensions

### 1. The Skeleton as a Spatial-Temporal Graph

Given a skeleton sequence with `T` frames and `V` joints, the spatial-temporal graph `G = (V, E)` has:
- **Spatial edges**: physical bone connections within each frame (e.g., elbow–wrist, knee–hip)
- **Temporal edges**: connections between the same joint across consecutive frames

This gives a 3D tensor of shape `(C, T, V)` — channels × time × joints — as input to the GCN stack.

### 2. The Graph Convolution Operation

Standard convolution on a grid computes a weighted sum over a fixed local neighborhood. On a graph, the neighborhood of node `v` is its set of connected neighbors `N(v)`. The graph convolution at node `v` is:

```
h_v' = Σ_{u ∈ N(v)} (1/c_v) · W · h_u
```

where `W` is a learnable weight matrix, `h_u` is the feature of neighbor `u`, and `c_v` is a normalization constant. In practice, this is expressed in matrix form using the normalized adjacency matrix `D^{-1/2} A D^{-1/2}`.

### 3. ST-GCN: The Seminal Work (Yan et al., 2018)

ST-GCN introduced the spatial-temporal graph formulation to skeleton action recognition. Its key innovations:

**Spatial Partitioning (3-subset strategy)**: Rather than using a single weight matrix for all neighbors, ST-GCN partitions each joint's neighborhood into three subsets and applies separate weight matrices to each:
- **Partition 0 (self)**: the joint itself — captures local self-features
- **Partition 1 (centripetal)**: neighbors *closer* to the skeleton root (SpineBase) — information flows from extremities toward body center
- **Partition 2 (centrifugal)**: neighbors *farther* from the root — information flows from body center toward extremities

This directional partitioning allows the model to learn asymmetric relationships: what the elbow "receives" from the shoulder differs from what it "sends" to the wrist.

**Temporal Convolution**: Applied as a standard 1D convolution along the time axis with kernel size 9, after each spatial graph convolution. This factorization — spatial GC then temporal conv — became the standard building block.

**Adjacency matrix**: Initialized from the physical skeleton topology (binary: connected or not) but the learnable scale factors allow the model to soft-weight each edge.

### 4. Evolution of the Architecture

| Model | Year | Key Innovation |
|-------|------|----------------|
| ST-GCN | 2018 | Spatial-temporal graph, 3-partition convolution |
| 2s-AGCN | 2019 | Adaptive adjacency (learned end-to-end); two-stream (joints + bones) |
| MS-G3D | 2020 | Multi-scale graph conv; simultaneous spatial-temporal aggregation (not factorized) |
| CTR-GCN | 2021 | Channel-wise topology refinement — different adjacency per feature channel |
| InfoGCN | 2022 | Information-theoretic objective; maximizes mutual information between input and latent |
| EfficientGCN | 2022 | Depthwise separable convolutions; compound scaling; early input fusion |
| BlockGCN | 2024 | Topological invariance encodings; block graph conv for body-part co-occurrences |

**2s-AGCN** (Shi et al., 2019) introduced two important ideas: (1) the adjacency matrix is fully learnable — it isn't fixed at the physical skeleton topology — allowing the model to discover non-obvious joint relationships; (2) two-stream processing of joint positions and bone vectors (joint[i] − joint[j] for each edge) as complementary modalities.

**MS-G3D** broke the factorization assumption: rather than doing spatial GC then temporal conv separately, it aggregates over both spatial and temporal neighborhoods simultaneously at multiple scales. This captures cross-spatial-temporal dependencies that factorized methods miss.

**CTR-GCN** argues that a single adjacency matrix per layer is too coarse — different features (e.g., one channel tracking arm movement, another tracking leg movement) should attend to different graph topologies. Channel-wise topology refinement makes the adjacency matrix a function of the feature channels.

### 5. Spatial Graph Convolution in Detail

The separable variant used in EfficientGCN factorizes the operation into:
1. **Spatial GC**: Aggregate over the 3-partition neighborhood using the physical + learned adjacency. Output: same spatial dimension, potentially different channel count.
2. **Pointwise conv (1×1)**: Mix channels after spatial aggregation.

This reduces parameters from `O(C_in × C_out × |E|)` to `O(C_in × V + C_in × C_out)` approximately, enabling a much more compact model.

### 6. Temporal Modeling

Two strategies in common use:
- **Factorized temporal conv**: Standard or depthwise 1D conv along the time axis after spatial GC. Kernel size 9 is typical. EfficientGCN uses depthwise separable temporal conv — each channel convolved independently before pointwise mixing.
- **Strided temporal conv**: Temporal downsampling by stride 2 at intermediate blocks (e.g., 200 → 100 → 50 frames), allowing deeper blocks to capture longer-range temporal dependencies while reducing compute.

### 7. Skeleton Graph Topology

The Kinect v2 22-joint skeleton used in the capstone project forms a tree rooted at SpineBase. The 23 edges represent physical bone connections. This tree structure is what enables the centripetal/centrifugal partitioning: for any joint, its tree path to the root unambiguously defines "closer" vs "farther" neighbors.

### 8. Rehabilitation vs. Action Recognition

Exercise quality assessment is harder than standard action recognition in a specific way: the task is not *what* movement is happening (easy, large inter-class variation) but *how well* it is executed (subtle intra-class variation — a correct squat vs incorrect squat may differ by only a few degrees of angular displacement at the knee). GCNs are well-suited here because they can learn joint-relationship patterns that distinguish quality, and their spatial structure aligns with how physiotherapists reason about movement (kinematic chains, proximal-distal coordination).

## Evidence & Examples

From [[sources/capstone-rehab-exercise-assessment]]:
- EfficientGCN-B0 on UI-PRMD achieves 96.3% accuracy classifying correct vs incorrect exercises — competitive with the ensemble EGCN (96.9%) at lower compute.
- Feature stream ablation confirms that the angle + velocity + relative multi-stream design (inherited from the ST-GCN lineage's two-stream tradition) contributes incrementally: 93.1% → 95.2% → 96.3%.
- The 3-partition spatial strategy is retained in the capstone adaptation: centripetal, self, centrifugal — exploiting the tree structure of the Kinect v2 skeleton.

## Contradictions & Open Questions

- The field debates whether learned (adaptive) adjacency (2s-AGCN style) always outperforms fixed topology. For small rehab datasets, learned adjacency risks overfitting the graph structure — fixed topology may actually be a useful regularizer.
- Transformer-based approaches (ViT applied to joints) theoretically capture long-range dependencies better via self-attention, but quadratic complexity and lack of easy open-source implementations have prevented adoption in real-time rehabilitation systems as of 2024.
- Whether multi-scale spatial-temporal aggregation (MS-G3D) consistently helps on small datasets like UI-PRMD is unknown; the efficiency of EfficientGCN's factorized approach may be preferable under data-limited conditions.

## Related

- [[concepts/efficientgcn-architecture]] — the specific GCN variant used in the capstone
- [[concepts/skeleton-based-action-recognition]] — broader field context including pre-GCN approaches
- [[concepts/sensor-agnostic-features]] — the input representation fed into the GCN
- [[concepts/pose-estimation]] — how skeleton joint positions are obtained
