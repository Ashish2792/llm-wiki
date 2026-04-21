---
title: Graph Convolutional Networks for Skeleton-Based Action Recognition
type: concept
tags: [graph-convolutional-network, skeleton, action-recognition, deep-learning, spatial-temporal]
sources: [capstone-rehab-exercise-assessment]
last_updated: 2026-04-21
---

# Graph Convolutional Networks for Skeleton-Based Action Recognition

## Definition

A Graph Convolutional Network (GCN) is a neural network that operates directly on graph-structured data by generalizing the convolution operation from regular grids to irregular graphs. In the skeleton-based action recognition context, the human skeleton is modeled as a spatial-temporal graph where joints are nodes and bones are edges, and GCN layers aggregate features from neighboring joints to learn discriminative representations of movement. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]]

## Why It Matters

The human skeleton is inherently a graph, not a grid. {inferred} [[sources/capstone-rehab-exercise-assessment#background]] Pre-2018 RNN/LSTM methods treated joint coordinates as flat vectors, ignoring the spatial structure of the body entirely. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]] GCNs are the natural fit: they respect the kinematic chain structure of the skeleton, allow the model to learn that the elbow is more closely related to the wrist than to the knee, and can capture coordinated multi-joint patterns that are critical for exercise quality assessment. {inferred} [[sources/capstone-rehab-exercise-assessment#background]]

## Key Properties / Dimensions

### 1. The Skeleton as a Spatial-Temporal Graph

Given a skeleton sequence with `T` frames and `V` joints, the input to the GCN stack is a 3D tensor of shape `(C, T, V)` — channels × time × joints. {direct} [[sources/capstone-rehab-exercise-assessment#methodology]] The spatial-temporal graph has:
- **Spatial edges**: physical bone connections within each frame (e.g., elbow–wrist, knee–hip) {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]]
- **Temporal edges**: connections between the same joint across consecutive frames {inferred} [[sources/capstone-rehab-exercise-assessment#background]]

### 2. The Graph Convolution Operation

Standard convolution on a grid computes a weighted sum over a fixed local neighborhood. On a graph, the neighborhood of node `v` is its set of connected neighbors `N(v)`. The graph convolution at node `v` is defined as:

```
h_v' = Σ_{u ∈ N(v)} (1/c_v) · W · h_u
```

where `W` is a learnable weight matrix, `h_u` is the feature of neighbor `u`, and `c_v` is a normalization constant. In matrix form this is expressed using the normalized adjacency matrix `D^{-1/2} A D^{-1/2}`. {uncertain} [Standard GCN formulation — this specific derivation is not documented in current wiki sources; see Open Questions.]

### 3. ST-GCN: The Seminal Work (Yan et al., 2018)

ST-GCN introduced the spatial-temporal graph convolution formulation to skeleton-based action recognition. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]] Its key innovations:

**Spatial Partitioning (3-subset strategy)**: Rather than using a single weight matrix for all neighbors, ST-GCN partitions each joint's neighborhood into three subsets and applies separate weight matrices to each: {direct} [[sources/capstone-rehab-exercise-assessment#methodology]]
- **Partition 0 (self)**: the joint itself
- **Partition 1 (centripetal)**: neighbors closer to the skeleton root (SpineBase)
- **Partition 2 (centrifugal)**: neighbors farther from the root

This directional partitioning allows the model to learn asymmetric relationships — what the elbow "receives" from the shoulder differs from what it "sends" to the wrist. {inferred} [[sources/capstone-rehab-exercise-assessment#methodology]]

**Temporal Convolution**: Applied as a standard 1D convolution along the time axis with kernel size 9, after each spatial graph convolution. {direct} [[sources/capstone-rehab-exercise-assessment#methodology]]

**Adjacency matrix**: Initialized from the physical skeleton topology but the learnable scale factors allow the model to soft-weight each edge. {paraphrase} [[sources/capstone-rehab-exercise-assessment#methodology]]

### 4. Evolution of the Architecture

The lineage from ST-GCN through BlockGCN progressively improved accuracy but increased model size and compute. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]]

| Model | Year | Key Innovation |
|-------|------|----------------|
| ST-GCN | 2018 | Spatial-temporal graph, 3-partition convolution |
| 2s-AGCN | 2019 | Adaptive (learnable) adjacency matrix; two-stream joints + bones |
| MS-G3D | 2020 | Disentangled multi-scale GCN |
| CTR-GCN | 2021 | Channel-wise topology refinement |
| InfoGCN | 2022 | Information-theoretic training objective |
| EfficientGCN | 2022 | Depthwise separable convolutions; compound scaling; early input fusion |
| BlockGCN | 2024 | Block graph convolution for body-part co-occurrences |

{paraphrase} [[sources/capstone-rehab-exercise-assessment#background]]

**Note on detailed model descriptions**: The innovations summarized below for 2s-AGCN, MS-G3D, and CTR-GCN go beyond what the capstone source documents. They are retained here for reference but marked as unsourced.

2s-AGCN introduced a fully learnable adjacency matrix (not fixed at physical skeleton topology) and two-stream processing of joint positions and bone vectors as complementary inputs. {uncertain} [Not documented in wiki sources at this level of detail — see Open Questions.]

MS-G3D broke the spatial-then-temporal factorization assumption by aggregating over both spatial and temporal neighborhoods simultaneously at multiple scales, capturing cross-spatial-temporal dependencies. {uncertain} [Not documented in wiki sources at this level of detail.]

CTR-GCN made the adjacency matrix a function of feature channels, so different feature channels can attend to different graph topologies. {uncertain} [Not documented in wiki sources at this level of detail.]

### 5. Spatial Graph Convolution in Detail

The separable variant used in EfficientGCN factorizes the spatial GC operation: (1) aggregate over the 3-partition neighborhood using physical + learned adjacency; (2) apply a pointwise 1×1 convolution to mix channels after spatial aggregation. {paraphrase} [[sources/capstone-rehab-exercise-assessment#methodology]] This factorization reduces parameter count compared to a single full graph convolution and is the design that enables EfficientGCN to reach ~690K parameters. {inferred} [[sources/capstone-rehab-exercise-assessment#methodology]]

### 6. Temporal Modeling

Two strategies are used in the capstone adaptation: {paraphrase} [[sources/capstone-rehab-exercise-assessment#methodology]]
- **Depthwise separable temporal conv**: Each channel convolved independently along the time axis before pointwise mixing; kernel size 9. {direct} [[sources/capstone-rehab-exercise-assessment#methodology]]
- **Strided temporal conv**: Temporal downsampling by stride 2 at blocks 3 and 5 — sequence length reduced from 200 → 100 → 50 frames — allowing deeper blocks to capture longer-range dependencies at lower compute. {direct} [[sources/capstone-rehab-exercise-assessment#methodology]]

### 7. Skeleton Graph Topology

The Kinect v2 22-joint skeleton used in the capstone has 23 edges and is rooted at SpineBase. {direct} [[sources/capstone-rehab-exercise-assessment#methodology]] MediaPipe's 33 landmarks are mapped to these 22 joints, with synthetic joints (SpineBase, SpineMid, SpineShoulder) estimated by averaging nearby MediaPipe landmarks — for example, SpineBase is the average of left and right hip landmarks. {direct} [[sources/capstone-rehab-exercise-assessment#methodology]] The tree structure is what enables the centripetal/centrifugal partitioning: for any joint, its tree path to the root unambiguously defines "closer" vs "farther" neighbors. {inferred} [[sources/capstone-rehab-exercise-assessment#methodology]]

### 8. Rehabilitation vs. Action Recognition

Exercise quality assessment differs from standard action recognition in a critical way: the task is not *what* movement is happening (large inter-class variation, relatively easy) but *how well* it is executed (subtle intra-class variation). {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]] This distinction is reflected in the capstone's dataset choice: UI-PRMD provides binary correct/incorrect labels for the same exercise, not multi-class activity labels. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]] Clinical deployment requires more than binary labels — a patient needs to know *which* joints are out of position. {direct} [[sources/capstone-rehab-exercise-assessment#discussion]]

## Evidence & Examples

EfficientGCN-B0 on UI-PRMD achieves 96.3% accuracy (Macro F1: 0.963) classifying correct vs incorrect exercises — competitive with the ensemble EGCN (96.9%) at lower compute. {direct} [[sources/capstone-rehab-exercise-assessment#results]]

Feature stream ablation confirms that the multi-stream angle + velocity + relative design contributes incrementally: angle only → 93.1%; angle + velocity → 95.2%; angle + velocity + relative → 96.3%. {direct} [[sources/capstone-rehab-exercise-assessment#results]]

EfficientGCN-B0 (~690K parameters) is 4.6× smaller than MS-G3D (3.2M parameters) and 2.2× smaller than InfoGCN (1.5M parameters), while achieving competitive accuracy on UI-PRMD. {direct} [[sources/capstone-rehab-exercise-assessment#results]]

The 3-partition spatial strategy (centripetal, self, centrifugal) is retained in the capstone adaptation, exploiting the tree structure of the Kinect v2 skeleton rooted at SpineBase. {direct} [[sources/capstone-rehab-exercise-assessment#methodology]]

Temporal resolution matters: 100-frame sequences → 94.1%, 200-frame → 96.3%, 300-frame → 96.4% (+0.1% at +50% compute). 200 frames is the practical optimum. {direct} [[sources/capstone-rehab-exercise-assessment#results]]

## Contradictions & Open Questions

- The field debates whether learned (adaptive) adjacency (2s-AGCN style) always outperforms fixed topology. For small rehab datasets, learned adjacency risks overfitting the graph structure — fixed topology may actually be a useful regularizer. {inferred} [[sources/capstone-rehab-exercise-assessment#discussion]]
- The detailed architectural innovations of 2s-AGCN, MS-G3D, and CTR-GCN are not documented in current wiki sources beyond brief mentions in the capstone literature review. Ingesting those papers would be required to support the full evolution table with sourced claims.
- Transformer-based approaches (ViTPose, RTMPose) are mentioned as alternatives in the capstone literature review but not evaluated or documented in this wiki. {uncertain} [[sources/capstone-rehab-exercise-assessment#background]]
- Whether multi-scale spatial-temporal aggregation (MS-G3D) consistently helps on small datasets like UI-PRMD is unknown; the efficiency of EfficientGCN's factorized approach may be preferable under data-limited conditions. {inferred} [[sources/capstone-rehab-exercise-assessment#discussion]]
- The formal derivation of graph convolution (D^{-1/2} A D^{-1/2} normalization, neighborhood aggregation proof) is standard GCN theory not documented in current wiki sources.

## Related

- [[concepts/efficientgcn-architecture]] — the specific GCN variant used in the capstone
- [[concepts/skeleton-based-action-recognition]] — broader field context including pre-GCN approaches
- [[concepts/sensor-agnostic-features]] — the input representation fed into the GCN
- [[concepts/pose-estimation]] — how skeleton joint positions are obtained
