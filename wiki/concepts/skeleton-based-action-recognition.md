---
title: Skeleton-Based Action Recognition
type: concept
tags: [skeleton, action-recognition, deep-learning, pose-estimation, graph-convolutional-network, rehabilitation]
sources: [capstone-rehab-exercise-assessment]
last_updated: 2026-04-14
---

# Skeleton-Based Action Recognition

## Definition

Skeleton-based action recognition is the task of classifying human movements from sequences of body joint positions (skeleton data) over time. Rather than operating on raw pixels, the input is a temporal sequence of sparse joint coordinates — typically 15–33 joints per frame — derived from depth sensors, motion capture, or pose estimation models. This compact representation discards appearance and focuses purely on body kinematics.

## Why It Matters

Skeleton data is a powerful abstraction for movement analysis: it is view-invariant (a squat looks the same whether filmed from the front or side), appearance-invariant (clothing, body shape don't affect joint positions), compact (25 joints vs millions of pixels), and directly interpretable by clinicians and biomechanists. For rehabilitation exercise assessment specifically, this alignment with clinical reasoning — where therapists think in terms of joint angles and kinematic chains — makes skeleton-based methods the natural fit.

## Key Properties / Dimensions

### 1. Field Overview and Scope

The field addresses two related but distinct tasks:
- **Action recognition**: *What* movement is being performed? (e.g., "walking", "waving", "sitting down") — large inter-class variation, typically easy for modern models
- **Action quality assessment (AQA)**: *How well* is a specific movement executed? (e.g., "is this squat correct?") — subtle intra-class variation, much harder and clinically more relevant

Most benchmark datasets and architectures target the first task. The capstone project addresses the second, leveraging architectures from the first as backbone.

### 2. Input Representations

| Representation | Description | Properties |
|---|---|---|
| Raw joint positions | 3D/2D coordinates of each joint | Simple, sensor-dependent, scale/translation variant |
| Bone vectors | Joint_i − Joint_j for each bone edge | More stable than positions, encodes relative geometry |
| Joint angles | arccos(dot product) at each joint | Sensor-agnostic, scale/translation/coordinate invariant |
| Velocity | Frame-to-frame differences | Encodes dynamics; distinguishes smooth vs jerky motion |

### 3. Pre-GCN Approaches

**RNN/LSTM methods** (pre-2018): Feed joint coordinates as a flat vector at each timestep into an LSTM. Captures temporal dynamics but ignores the spatial structure of the skeleton entirely — the model cannot distinguish the wrist from the hip based on topology.

**CNN methods**: Reshape joint sequences into a pseudo-image and apply 2D CNNs. Allows feature reuse across time, but the grid ordering is arbitrary and doesn't reflect body topology.

**Key limitation shared by both**: They treat the skeleton as an unstructured set of coordinates, losing the kinematic chain structure that is central to movement analysis.

### 4. The GCN Revolution (2018–present)

ST-GCN (Yan et al., 2018) reformulated the problem: model the skeleton as a spatial-temporal graph where joints are nodes and bones are edges, then apply graph convolutions. This respects kinematic structure — the model can learn that the elbow is informationally related to the shoulder and wrist, not to the knee.

See [[concepts/graph-convolutional-networks]] for the full GCN lineage and technical details.

### 5. Benchmark Datasets

| Dataset | Subjects | Classes | Sensor | Scale |
|---|---|---|---|---|
| NTU RGB+D 60 | 40 | 60 | Kinect v2 | 56,880 sequences |
| NTU RGB+D 120 | 106 | 120 | Kinect v2 | 114,480 sequences |
| Kinetics-Skeleton | diverse | 400 | OpenPose (video) | ~300K sequences |
| UI-PRMD | 10 | 10 exercises | Kinect v2 + Vicon | ~2,000 sequences |
| KIMORE | 78 | 5 exercises | Kinect v2 | ~1,200 sequences |

NTU RGB+D is the standard benchmark for general action recognition. UI-PRMD is the primary rehabilitation benchmark — two orders of magnitude smaller, which dominates all architectural choices in the rehab domain.

### 6. Rehabilitation-Specific Challenges

**Subtle intra-class variation**: Distinguishing a correct squat from an incorrect one requires detecting small angular differences (5–10°) at specific joints. Standard action recognition models trained on cross-class variation may not develop this sensitivity.

**Data scarcity**: Rehab datasets have hundreds of sequences vs tens of thousands for general action datasets. This forces simpler models (fewer params), stronger regularization, and data augmentation.

**Label quality**: Correct/incorrect labels in datasets like UI-PRMD are assigned by healthy subjects *simulating* incorrect form, not by real patients. Real patient errors may be more varied, subtle, and idiosyncratic.

**Cross-sensor deployment**: Training data comes from clinical Kinect sensors; deployment happens on consumer webcams. The domain gap is catastrophic for raw positions, manageable for sensor-agnostic representations.

### 7. Accuracy on UI-PRMD (Historical Comparison)

| Method | Accuracy |
|---|---|
| Raw 3D positions (baseline) | 85.3% |
| Tuned 3D positions | 92.3% |
| EGCN ensemble (two-model) | 96.9% |
| **EfficientGCN-B0 (angle features)** | **96.3%** |
| Kinect Euler angles (not sensor-agnostic) | 98.0% |

The gap between sensor-agnostic angles (96.3%) and the sensor-specific upper bound (98.0%) is the literal cost of making the model deployable via webcam.

## Evidence & Examples

From [[sources/capstone-rehab-exercise-assessment]]:
- The field is data-limited, not architecture-limited: accuracy on UI-PRMD scales from 72% (400 sequences) to 96.3% (2,000 sequences) — adding more data would help more than a better model.
- EfficientGCN-B0 is competitive with the EGCN ensemble (96.9%) while being a single model — the ensemble gains are marginal and don't justify the 2× compute cost for deployment.

## Contradictions & Open Questions

- Transformer-based architectures (PoseFormer, SkateFormer) claim state-of-the-art on NTU RGB+D but haven't been systematically evaluated on small rehab datasets where their quadratic attention complexity may be a disadvantage.
- Whether subject-independent accuracy — testing on held-out subjects, not held-out sequences from the same subjects — is substantially lower than the reported 96.3% is not yet rigorously established (post-hoc analysis suggests 90–96%).
- The rehabilitation AQA community has no agreed-upon benchmark: UI-PRMD, KIMORE, REHAB24, and IRDS all exist with different exercise sets, subject populations, and labeling protocols, making cross-paper comparison unreliable.

## Related

- [[concepts/graph-convolutional-networks]] — the dominant model family for this task
- [[concepts/efficientgcn-architecture]] — the specific architecture used in the capstone
- [[concepts/sensor-agnostic-features]] — the input representation enabling cross-sensor deployment
- [[concepts/pose-estimation]] — how skeleton joint positions are obtained
- [[concepts/domain-adaptation]] — bridging the train/inference sensor gap
- [[entities/ui-prmd-dataset]] — primary rehabilitation benchmark dataset
