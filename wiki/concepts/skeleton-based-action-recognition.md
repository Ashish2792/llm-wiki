---
title: Skeleton-Based Action Recognition
type: concept
tags: [skeleton, action-recognition, deep-learning, pose-estimation, graph-convolutional-network, rehabilitation]
sources: [capstone-rehab-exercise-assessment]
last_updated: 2026-04-20
---

# Skeleton-Based Action Recognition

## Definition

Skeleton-based action recognition is the task of classifying human movements from sequences of body joint positions (skeleton data) over time. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]] Rather than operating on raw pixels, the input is a temporal sequence of sparse joint coordinates — typically 15–33 joints per frame — derived from depth sensors, motion capture, or pose estimation models. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]] This compact representation discards appearance and focuses purely on body kinematics.

## Why It Matters

Skeleton data is particularly well-suited for movement analysis: it is compact (25 joints vs millions of pixels), directly interpretable by clinicians and biomechanists, and aligns naturally with how physiotherapists reason about movement quality — in terms of joint angles and kinematic chains. {inferred} [[sources/capstone-rehab-exercise-assessment#background]] For rehabilitation exercise assessment, this clinical alignment makes skeleton-based methods the natural fit for automated quality classification. {inferred} [[sources/capstone-rehab-exercise-assessment#background]]

## Key Properties / Dimensions

### 1. Field Overview and Scope

The field addresses two related tasks: {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]]

- **Action recognition**: *What* movement is being performed? (e.g., "walking", "waving", "sitting down") — large inter-class variation, addressed by most benchmark architectures
- **Action quality assessment (AQA)**: *How well* is a specific movement executed? (e.g., "is this squat correct?") — subtle intra-class variation, clinically more relevant for rehabilitation

Most benchmark datasets and architectures target the first task. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]] The capstone project addresses the second, leveraging architectures from the first as backbone. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]]

### 2. Pre-GCN Approaches

**RNN/LSTM methods** (pre-2018): Feed joint coordinates as a flat vector at each timestep into an LSTM. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]] Captures temporal dynamics but ignores the spatial structure of the skeleton — the model cannot distinguish the wrist from the hip based on topology. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]] Du et al. (2015) proposed a hierarchical RNN dividing the skeleton into five body parts. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]]

**CNN methods**: Reshape joint sequences into a pseudo-image and apply 2D CNNs. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]] Allows feature reuse across time, but the grid ordering is arbitrary and doesn't reflect body topology. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]]

### 3. The GCN Revolution (2018–present)

ST-GCN (Yan et al., 2018) reformulated the problem: model the skeleton as a spatial-temporal graph where joints are nodes and bones are edges, then apply graph convolutions. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]] This respects kinematic structure — the model can learn that the elbow is informationally related to the shoulder and wrist, not to the knee. {inferred} [[sources/capstone-rehab-exercise-assessment#background]]

Key developments in the ST-GCN lineage: {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]]

- **2s-AGCN** (Shi et al., 2019): Introduced adaptive graph convolution (learned adjacency matrix) and two-stream architecture processing joint positions and bone vectors
- **MS-G3D** (Liu et al., 2020): Disentangled multi-scale spatial-temporal graph convolution; 3.2M parameters
- **CTR-GCN** (Chen et al., 2021): Channel-topology refinement
- **InfoGCN** (Chi et al., 2022): Representation learning with information bottleneck; 1.5M parameters
- **BlockGCN** (Zhou et al., 2024): Redefined topology awareness

See [[concepts/graph-convolutional-networks]] for the full GCN lineage and technical details.

### 4. Benchmark Datasets

| Dataset | Subjects | Classes | Sensor | Scale |
|---|---|---|---|---|
| NTU RGB+D 60 | 40 | 60 | Kinect v2 | 56,880 sequences |
| NTU RGB+D 120 | 106 | 120 | Kinect v2 | 114,480 sequences |
| UI-PRMD | 10 | 10 exercises | Kinect v2 + Vicon | ~2,000 sequences |
| KIMORE | 78 | 5 exercises | Kinect v2 | ~1,200 sequences |

{paraphrase} [[sources/capstone-rehab-exercise-assessment#background]]

NTU RGB+D is the standard benchmark for general action recognition. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]] UI-PRMD is the primary rehabilitation benchmark — two orders of magnitude smaller, which dominates all architectural choices in the rehabilitation domain. {inferred} [[sources/capstone-rehab-exercise-assessment#background]]

### 5. Rehabilitation-Specific Challenges

**Subtle intra-class variation**: Distinguishing correct from incorrect form requires detecting small angular differences at specific joints. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]] Standard action recognition models trained on cross-class variation may not develop this sensitivity. {inferred} [[sources/capstone-rehab-exercise-assessment#background]]

**Data scarcity**: Rehabilitation datasets have hundreds of sequences vs tens of thousands for general action datasets. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]] This forces simpler models (fewer parameters) and stronger regularization. {inferred} [[sources/capstone-rehab-exercise-assessment#discussion]] The capstone finding that accuracy scales directly with data volume (400 seqs → 72%, 1,020 seqs → 80.4%, 2,000 seqs → 96.3%) confirms the field is data-limited, not architecture-limited. {direct} [[sources/capstone-rehab-exercise-assessment#results]]

**Label quality**: Correct/incorrect labels in datasets like UI-PRMD are assigned by healthy subjects *simulating* incorrect form, not by real patients. {paraphrase} [[sources/capstone-rehab-exercise-assessment#limitations]] Real patient errors may be more varied, subtle, and idiosyncratic. {paraphrase} [[sources/capstone-rehab-exercise-assessment#limitations]]

**Cross-sensor deployment**: Training data comes from Kinect sensors; deployment happens on consumer webcams. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]] The domain gap is catastrophic for raw positions but manageable for sensor-agnostic angle representations. {direct} [[sources/capstone-rehab-exercise-assessment#results]]

**Evaluation protocol inconsistency**: Many published works use random splits that allow sequences from the same subject in both training and test sets, inflating reported accuracy. {paraphrase} [[sources/capstone-rehab-exercise-assessment#discussion]] Subject-independent evaluation is more rigorous but harder with small datasets (holding out 2 of 10 subjects removes 20% of data). {paraphrase} [[sources/capstone-rehab-exercise-assessment#discussion]]

**Per-exercise variation in difficulty**: Large sagittal plane movements (squats, shoulder flexion) yield higher accuracy because computed angles directly capture the primary movement dimension. {paraphrase} [[sources/capstone-rehab-exercise-assessment#discussion]] Out-of-plane rotation exercises (shoulder internal-external rotation) are harder because the 2D angle approximation loses depth information. {paraphrase} [[sources/capstone-rehab-exercise-assessment#discussion]]

### 6. Accuracy Comparison on UI-PRMD

| Method | Features | Accuracy | Protocol |
|---|---|---|---|
| Williams et al. (2019) | PCA + GMM | 88.5% | Per-exercise |
| Liao et al. (2020) | Spatio-temporal NN | 91.2% | Cross-val |
| Kourbane et al. (2025) | Lightweight ST-GCN | 94.8% | Subject-indep |
| EGCN ensemble (Bruce et al., 2022) | GCN ensemble (pos+orient) | 96.9% | Cross-val |
| **EfficientGCN-B0 (this work)** | **Sensor-agnostic angles** | **96.3%** | **Random split** |
| Kinect Euler angles (not deployable) | Pre-computed Euler angles | 98.0% | Random split |

{direct} [[sources/capstone-rehab-exercise-assessment#results]]

Direct comparison across studies is complicated by different evaluation protocols, data preprocessing, and split configurations. {paraphrase} [[sources/capstone-rehab-exercise-assessment#results]]

### 7. Clinical Deployment Barriers

Beyond raw classification accuracy, several factors complicate clinical use: {paraphrase} [[sources/capstone-rehab-exercise-assessment#discussion]]

- **Feedback granularity**: Binary correct/incorrect is too coarse — patients need joint-level error attribution (knee flexion insufficient vs. trunk lean excessive vs. weight asymmetry) {paraphrase} [[sources/capstone-rehab-exercise-assessment#discussion]]
- **Confidence calibration**: Neural networks with cross-entropy loss tend toward overconfident predictions; temperature scaling required before clinical use {paraphrase} [[sources/capstone-rehab-exercise-assessment#discussion]]
- **Exercise recognition**: A clinical system needs to first recognize which exercise is being performed, then assess quality {paraphrase} [[sources/capstone-rehab-exercise-assessment#discussion]]

## Evidence & Examples

EfficientGCN-B0 is competitive with the EGCN ensemble (96.9%) while being a single model, directly deployable to webcam-based inference without duplicate GCN stacks. {inferred} [[sources/capstone-rehab-exercise-assessment#results]]

Qualitative real-time validation: shoulder abduction classified correctly on 100% of frames; out-of-distribution side lunge classified as incorrect on 89% of frames. {direct} [[sources/capstone-rehab-exercise-assessment#results]]

## Contradictions & Open Questions

Whether subject-independent accuracy — testing on entirely held-out subjects — is substantially lower than the reported 96.3% is not yet rigorously established; post-hoc analysis suggests 90–96%. {paraphrase} [[sources/capstone-rehab-exercise-assessment#discussion]]

The rehabilitation AQA community has no agreed-upon benchmark: UI-PRMD, KIMORE, REHAB24, and IRDS all exist with different exercise sets, subject populations, and labeling protocols, making cross-paper comparison unreliable. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]]

Transformer-based architectures claim state-of-the-art on NTU RGB+D but a 2024 review found no transformer-based pose estimation methods had been adopted in practical rehabilitation feedback systems, partly due to lack of open-source implementations comparable to MediaPipe. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]]

## Related

- [[concepts/graph-convolutional-networks]] — the dominant model family for this task
- [[concepts/efficientgcn-architecture]] — the specific architecture used in the capstone
- [[concepts/sensor-agnostic-features]] — the input representation enabling cross-sensor deployment
- [[concepts/pose-estimation]] — how skeleton joint positions are obtained
- [[concepts/domain-adaptation]] — bridging the train/inference sensor gap
- [[entities/ui-prmd-dataset]] — primary rehabilitation benchmark dataset
- [[entities/kimore-dataset]] — secondary rehabilitation benchmark
