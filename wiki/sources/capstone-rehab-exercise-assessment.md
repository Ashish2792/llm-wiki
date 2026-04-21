---
title: AI-Powered Rehabilitation Exercise Assessment System Using EfficientGCN and Webcam-Based Pose Estimation
type: source
domain: research
date_ingested: 2026-04-14
last_updated: 2026-04-20
source_file: raw/research/capstone_report.pdf
tags: [rehabilitation, graph-convolutional-network, pose-estimation, domain-adaptation, efficientgcn, mediapipe, skeleton-based-action-recognition, deep-learning]
---

# AI-Powered Rehabilitation Exercise Assessment System

## TL;DR

This B.Tech capstone project (Ashish, April 2026) builds a real-time system that classifies physical therapy exercises as correct or incorrect using only a standard webcam. The central technical contribution is a **sensor-agnostic joint angle feature representation** that eliminates the coordinate-system domain gap between Kinect depth-sensor training data (UI-PRMD) and MediaPipe RGB-based inference, achieving 96.3% accuracy and ~15 fps on consumer CPU hardware — without any webcam-domain training data.

## Background

Physical rehabilitation is critical for recovery from musculoskeletal injuries, neurological disorders, and post-surgical conditions. Home-based exercise programs (HEPs) have emerged as a practical alternative to clinic visits but lack real-time expert feedback on exercise quality, leading to compensatory movements, re-injury, and suboptimal outcomes.

Most high-quality labeled rehabilitation datasets were collected with Microsoft Kinect depth sensors: UI-PRMD (2,000 sequences, 10 exercises, 10 subjects) and KIMORE (~1,200 sequences, 5 exercises, 78 subjects). Deploying models trained on this data to webcam video requires bridging a severe domain gap: Kinect outputs 3D coordinates in millimeters (range ~[-1800, 1800] mm) while MediaPipe outputs normalized 2D coordinates (range [0.01, 0.99]).

The literature review (Chapter 2) identifies four research gaps motivating this project: (1) cross-sensor deployment — train on Kinect, deploy on webcam — is underexplored; (2) existing domain adaptation approaches either require labeled target-domain data or add complexity without guaranteed improvement; (3) evaluation protocols vary across papers, with many using subject-dependent random splits that inflate reported accuracy; (4) EfficientGCN has not been systematically evaluated for rehabilitation assessment despite its favorable efficiency-accuracy tradeoff.

Competing methods on UI-PRMD:

| Study | Method | Accuracy | Protocol |
|---|---|---|---|
| Williams et al. (2019) | PCA + GMM | 88.5% | Per-exercise |
| Liao et al. (2020) | Spatio-temporal NN | 91.2% | Cross-val |
| Kourbane et al. (2025) | Lightweight ST-GCN | 94.8% | Subject-indep |
| Bruce et al. / EGCN (2022) | GCN ensemble (pos+orient) | 96.9% | Cross-val |
| **This project** | EfficientGCN (angles) | **96.3%** | Random split |

Prior skeleton-based action recognition history: pre-2018 RNN/LSTM methods treated joints as flat vectors, ignoring spatial structure. ST-GCN (Yan et al., 2018) introduced spatial-temporal graph convolution, modeling joints as nodes and bones as edges. The lineage through 2s-AGCN (adaptive adjacency, 2019), MS-G3D (disentangled multi-scale GCN, 2020), CTR-GCN (2021), InfoGCN (2022), and BlockGCN (2024) progressively improved accuracy but increased model size and compute.

## Methodology

### System Architecture

Four-stage pipeline: RGB webcam at 30 fps → MediaPipe BlazePose (33 landmarks per frame) → sensor-agnostic joint angle feature extraction (9 channels × 22 joints × 200 frames) → EfficientGCN-B0 binary classifier (correct / incorrect).

### Joint Angle Feature Engineering

For joint j with connected neighbors a and b, the joint angle is:

```
v_a = p_a − p_j
v_b = p_b − p_j
θ_j = arccos( (v_a · v_b) / max(‖v_a‖ · ‖v_b‖, ε) ) / π   ∈ [0, 1]
```

Joints at skeleton extremities (hands, feet) with only one neighbor have angle set to zero. The formula is invariant to translation (subtraction cancels absolute position), scale (denominator cancels uniform scaling), and coordinate system — a 90° elbow flexion produces the same normalized value of 0.5 whether measured in mm (Kinect) or normalized pixels (MediaPipe).

Three complementary streams, each with 3 components (x, y, z angle decomposition), produce 9 channels total:

- **Angle stream (ch 0–2)**: Normalized joint angles θ_j / π — instantaneous angular configuration
- **Velocity stream (ch 3–5)**: Frame-to-frame differences Δθ_j = θ_j^(t) − θ_j^(t−1) — movement dynamics
- **Relative stream (ch 6–8)**: Each joint's angle minus its parent's in the skeleton hierarchy θ_j^rel = θ_j − θ_parent(j) — kinematic chain encoding

Final feature tensor shape: (C=9, T=200, V=22, M=1). Raw sequences (50–500 frames) are resampled to 200 frames via linear interpolation.

### Skeleton Graph Topology

Kinect v2 22-joint model with 23 edges, tree-rooted at SpineBase. MediaPipe's 33 landmarks are mapped to these 22 joints; synthetic joints (SpineBase, SpineMid, SpineShoulder) are estimated by averaging nearby MediaPipe landmarks (e.g., SpineBase = average of left/right hip). Key mapping entries:

| Kinect Joint | MediaPipe Landmark(s) |
|---|---|
| SpineBase | Avg(Hip_L, Hip_R) |
| Neck | Avg(Shoulder_L, Shoulder_R) |
| Head | Nose (landmark 0) |
| ShoulderLeft | Left Shoulder (11) |
| ElbowLeft | Left Elbow (13) |
| WristLeft | Left Wrist (15) |
| KneeLeft | Left Knee (25) |
| AnkleLeft | Left Ankle (27) |

### Model Architecture

EfficientGCN-B0 adapted for binary rehabilitation classification:

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

Each block uses: (1) spatial graph convolution (SGC) with 3-partition strategy — self, centripetal (toward SpineBase), centrifugal (away from SpineBase) — with learnable adjacency matrix; (2) depthwise separable temporal convolution, kernel size 9.

### Training Strategy

| Parameter | Value |
|---|---|
| Optimizer | Adam |
| Learning rate | 5×10⁻⁴ |
| Weight decay | 1×10⁻⁴ (L2) |
| Batch size | 32 |
| Max epochs | 200 |
| Early stopping patience | 30 (on validation F1) |
| LR scheduler | ReduceLROnPlateau (factor 0.5, patience 10) |
| Dropout | 0.5 (before FC) |
| Loss | Cross-Entropy |

Data split: random stratified 70/15/15 on UI-PRMD. Augmentation: temporal speed perturbation (±10%), Gaussian noise injection (σ=0.01). Spatial flipping and joint dropout deliberately avoided.

Cross-dataset transfer learning: froze blocks 0–3, trained blocks 4–7 + classification head, unfroze BatchNorm affine parameters in frozen blocks.

Hardware: NVIDIA T4 via Kaggle Notebooks; PyTorch 2.0, MediaPipe 0.10, Python 3.10.

## Results

### UI-PRMD Primary Results

Accuracy: **96.3%**, Macro F1: **0.963**

| Class | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| Incorrect | 0.96 | 0.97 | 0.96 | 150 |
| Correct | 0.97 | 0.96 | 0.96 | 150 |

Best model at epoch 139 of 200. 4 false negatives (incorrect classified as correct), 7 false positives (correct classified as incorrect) — well-distributed, not systematically biased.

### Feature Representation Evolution (UI-PRMD)

| Version | Features | Accuracy | Sensor-Agnostic? |
|---|---|---|---|
| v1 | 3D Kinect positions | 85.3% | No |
| v3 | 3D positions (tuned hyperparams: lr=5×10⁻⁴, longer training) | 92.3% | No |
| 2D v2 | Z=0 projected positions | 89.3% | Partial |
| Kinect Euler angles | Pre-computed from Kinect SDK | 98.0% | No — not reproducible from MediaPipe |
| **Final** | **Vector-computed joint angles** | **96.3%** | **Yes** |

The jump from v1 to v3 (85.3% → 92.3%) came purely from hyperparameter tuning — demonstrating GCN sensitivity to optimization. The 1.7% gap between 98.0% (Euler angles) and 96.3% (vector angles) is the literal cost of sensor-agnosticity.

### Feature Stream Ablation

| Streams | Channels | Accuracy | F1 |
|---|---|---|---|
| Angle only | 3 | 93.1% | 0.931 |
| Angle + Velocity | 6 | 95.2% | 0.952 |
| Angle + Velocity + Relative | 9 | 96.3% | 0.963 |

Velocity adds +2.1% (smooth vs jerky motion dynamics); relative adds +1.1% (hierarchical kinematic chain patterns).

### Temporal Resolution Ablation

| Sequence length | Accuracy |
|---|---|
| 100 frames | 94.1% |
| 200 frames | 96.3% |
| 300 frames | 96.4% (+0.1% at +50% compute) |

200 frames is the practical optimum.

### Domain Invariance Validation

| Feature type | Kinect | MediaPipe | Domain gap |
|---|---|---|---|
| Position range (min) | −1842 mm | 0.01 | Catastrophic — no overlap |
| Position range (max) | 1756 mm | 0.99 | Catastrophic — no overlap |
| Position mean | 12.4 | 0.48 | Catastrophic |
| **Angle range (min)** | **−1.82** | **−1.73** | **Minimal** |
| **Angle range (max)** | **1.96** | **1.92** | **Minimal** |
| **Angle mean** | **0.041** | **0.038** | **Minimal** |
| **Angle std** | **0.389** | **0.440** | **Minimal** |

### Cross-Dataset Results

REHAB24-6 (native MediaPipe, ~400 sequences, strict subject-independent split):
- Training from scratch: ~72% accuracy
- Transfer learning from UI-PRMD (frozen blocks 0–3, fine-tuned blocks 4–7): ~63.8% — **worse than scratch**

Combined WLU + REHAB24-6 (~1,020 native MediaPipe sequences), reduced model (channels=[48,48,96,96,192,192,192,192], ~370K params), 5-fold CV, label smoothing ε=0.1:
- CV mean F1: 0.811 ± 0.044; Test accuracy: 80.4%

Data volume scaling law: 400 seqs → 72%; 1,020 seqs → 80.4%; 2,000 seqs → 96.3%.

### Real-Time Webcam Results

- Shoulder abduction (in-distribution): 100% of frames classified correct
- Side lunge (out-of-distribution, not in training set): 89% of frames classified incorrect — expected and clinically appropriate

### Computational Efficiency

| Component | CPU (Intel i7) | GPU (NVIDIA T4) |
|---|---|---|
| MediaPipe pose estimation | 38 ms/frame | 38 ms/frame |
| Joint angle computation | 0.5 ms/frame | 0.5 ms/frame |
| EfficientGCN inference | 25 ms/prediction | 4.5 ms/prediction |
| **Total** | **~64 ms → ~15 fps** | **~43 ms → ~23 fps** |

MediaPipe is the bottleneck (~60% of total). EfficientGCN-B0 (~690K params) is 4.6× smaller than MS-G3D (3.2M params) and 2.2× smaller than InfoGCN (1.5M params).

## Discussion

**Sensor-agnostic features**: Joint angle features effectively bridge the Kinect→MediaPipe domain gap without any target-domain training data. The residual std difference (0.389 vs 0.440) reflects MediaPipe's higher landmark noise; this is within BatchNorm tolerance for most exercises. However, distribution tails differ — exercises at extreme range-of-motion may show occasional misclassification. The domain gap is not perfectly eliminated but practically bridged for deployment purposes.

**Data volume challenge**: The consistent accuracy-vs-data-volume relationship (72%→80.4%→96.3% as sequences increase 400→1,020→2,000) indicates the field is **data-limited, not architecture-limited**. The most impactful research direction is large-scale RGB rehabilitation dataset collection with clinical annotations, not further architecture improvements.

**Transfer learning failure — three causes**: (1) BatchNorm running statistics accumulated during UI-PRMD training encode the source-domain angle distribution; even unfrozen affine parameters (scale/shift) cannot compensate because running mean and variance remain fixed; (2) ~400 fine-tuning sequences are insufficient for ~380K trainable parameters in blocks 4–7, causing severe overfitting; (3) exercise sets differ (10 UI-PRMD vs 6 REHAB24-6 exercises), so exercise-specific patterns learned in early blocks don't transfer. For cross-dataset rehab assessment with limited target data, simpler architectures with fewer parameters are preferable.

**Evaluation protocol**: Subject-dependent random splits systematically overestimate generalization because the model exploits individual body proportions and habitual movement patterns. With only 10 UI-PRMD subjects, strict leave-subjects-out evaluation is high-variance. The reported 96.3% likely overestimates true subject-independent performance; post-hoc analysis on subjects 9/10 showed 98.5% but not under strict conditions. True subject-independent accuracy estimated at 90–96%.

**Per-exercise variation**: Large sagittal plane movements (squats, shoulder flexion) yield higher accuracy because computed angles directly capture the primary movement dimension. Out-of-plane rotation exercises (shoulder internal-external rotation) are harder because the 2D angle approximation drops depth information.

**Clinical deployment barriers**:
- Binary correct/incorrect is too coarse: a patient needs to know *which* joints are out of position
- Neural networks trained with cross-entropy loss tend to be overconfident; temperature scaling or calibration is required before clinical use
- Integration requires exercise recognition before quality assessment, progress tracking across sessions, and clinician-readable reports

**Ethical considerations**: Video capture raises privacy concerns; AI system should supplement (not replace) professional physiotherapy; webcam-based access is more equitable than Kinect-based but still requires device, internet, and physical space.

## Limitations

1. **Exercise specificity**: Model trained on 10 UI-PRMD exercises only; classifies any out-of-set exercise as incorrect without recognizing what exercise is being performed.
2. **Single-person assumption**: MediaPipe BlazePose processes one person per frame; no multi-person support.
3. **Camera viewpoint sensitivity**: Angle features are theoretically rotation-invariant, but MediaPipe's 2D landmark detection is viewpoint-sensitive. Frontal-view training data may not generalize to oblique camera angles.
4. **Occlusion handling**: Self-occlusion (arm crossing torso) causes inaccurate MediaPipe landmarks; no mechanism for detecting or handling occlusion events (5-frame grace period only for complete tracking dropout).
5. **Clinical validity gap**: UI-PRMD incorrect movements are performed by healthy subjects simulating errors; real patient deviations may be more varied, subtle, and idiosyncratic.
6. **Subject-independent accuracy uncertainty**: Reported 96.3% uses random (subject-permeable) splits; true subject-independent accuracy estimated at 90–96%.
7. **Confidence calibration**: No calibration applied; raw confidence scores should not be used clinically without temperature scaling.

## Notable Quotes

> "The use of joint angles as a sensor-agnostic representation has been explored in several contexts. Angular features are inherently invariant to coordinate system transformations, body scale differences, and camera perspective distortions, making them attractive for cross-modal deployment." — §2.4.4

> "The implication for future work is clear: advancing the state of the art in webcam-based rehabilitation assessment requires investment in large-scale RGB video dataset collection." — §6.1.2

> "For clinical applications, it is important that the model's confidence scores are well-calibrated — that is, when the model predicts 'correct' with 90% confidence, the exercise should actually be correct approximately 90% of the time. Neural networks, particularly those trained with cross-entropy loss, are often poorly calibrated, tending toward overconfident predictions." — §6.3.2

> "The standard transfer learning paradigm assumes that early layers learn general features that transfer across domains. However, in this setting, several factors undermine this assumption." — §6.1.3

## Connections

- [[concepts/sensor-agnostic-features]] — the joint angle feature engineering approach and its invariance properties
- [[concepts/efficientgcn-architecture]] — EfficientGCN-B0 architecture, layers, training configuration
- [[concepts/graph-convolutional-networks]] — ST-GCN lineage and spatial-temporal graph convolution theory
- [[concepts/skeleton-based-action-recognition]] — field overview, benchmark datasets, method comparison
- [[concepts/pose-estimation]] — MediaPipe vs Kinect comparison, landmark mapping
- [[concepts/domain-adaptation]] — domain gap problem and why transfer learning failed
- [[entities/mediapipe-blazepose]] — inference-time pose estimator
- [[entities/ui-prmd-dataset]] — primary training dataset
- [[entities/microsoft-kinect]] — sensor used for UI-PRMD data collection
- [[entities/kimore-dataset]] — rehabilitation benchmark used in literature review comparison
- [[entities/rehab24-6-dataset]] — cross-dataset transfer learning target
- [[entities/wlu-dataset]] — combined with REHAB24-6 for ~1,020 sequence experiments

## Open Questions

- What is the true subject-independent accuracy under strict leave-subjects-out evaluation (not post-hoc)?
- Would InstanceNorm or LayerNorm as a BatchNorm replacement enable successful cross-dataset fine-tuning?
- How does the 2D angle approximation perform on out-of-plane exercises vs full 3D angle computation?
- Can a smaller architecture (fewer params than 690K) outperform fine-tuning when target data is scarce (~400 sequences)?
- Would domain adversarial training (DANN, CORAL, MMD) further close the residual angle std gap (0.389 vs 0.440) without labeled target data?
- What is the minimum dataset size needed for EfficientGCN to achieve the 96.3% accuracy obtained with 2,000 UI-PRMD sequences?
