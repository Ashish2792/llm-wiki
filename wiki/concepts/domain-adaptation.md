---
title: Domain Adaptation
type: concept
tags: [domain-adaptation, transfer-learning, domain-gap, cross-modal, batchnorm, feature-engineering]
sources: [capstone-rehab-exercise-assessment]
last_updated: 2026-04-14
---

# Domain Adaptation

## Definition

Domain adaptation is a machine learning strategy for deploying a model trained on data from one distribution (the *source domain*) to a different but related distribution (the *target domain*). The core challenge is that models learn to exploit statistical regularities in training data that may not hold at test time — a model trained on Kinect 3D skeleton data will have learned features tied to millimeter-scale coordinates and 3D geometry that are meaningless when applied to MediaPipe's normalized 2D coordinates.

## Why It Matters

Most high-quality labeled datasets in specialized domains (rehabilitation, sports biomechanics, clinical gait analysis) were collected with expensive research-grade sensors (Kinect, Vicon motion capture). Deploying classifiers from these datasets to consumer hardware (webcams) requires bridging a domain gap. The gap is not just "similar but slightly different" — for raw joint positions, the distributions have zero overlap, guaranteeing complete model failure without adaptation.

## Key Properties / Dimensions

### 1. Types of Domain Gap

**Covariate shift**: P(X) differs between source and target but P(Y|X) is the same. This is the "benign" case — the input distribution changes but the relationship between features and labels is preserved. Domain adaptation techniques that reweight samples or align feature distributions are designed for this case.

**Dataset shift**: Both P(X) and P(Y|X) differ. More severe — the model's decision boundaries may be fundamentally wrong for the target domain.

**Cross-modal gap** (the rehabilitation case): Source and target domains use entirely different measurement modalities (depth sensor vs RGB camera). The coordinate system, units, dimensionality, and noise characteristics are all different. Standard covariate shift assumptions break down entirely for raw features.

### 2. Strategies

| Strategy | Description | Applicability |
|---|---|---|
| Feature engineering | Design features that are invariant to domain shift | Works when domain shift is systematic and a mathematical invariant exists |
| Fine-tuning | Train on source, then fine-tune on small target dataset | Requires labeled target data; BatchNorm failure can undermine this |
| Domain adversarial training | Add a discriminator that cannot distinguish source/target features | Requires unlabeled target data during training |
| Instance re-weighting | Upweight source samples similar to target distribution | Requires estimating P(target)/P(source) |
| Normalization adaptation | Reset/adapt BatchNorm statistics to target domain | Lightweight; effective for BatchNorm-specific failures |

### 3. The Feature Engineering Approach (Sensor-Agnostic Angles)

For the rehabilitation domain, the source/target coordinate system gap has a clean mathematical solution: compute joint angles using vector dot products. Since `arccos((v_a · v_b) / (‖v_a‖ · ‖v_b‖))` is invariant to scale, translation, and coordinate system, the same formula applied to Kinect mm coordinates and MediaPipe normalized coordinates produces the same angle for the same body configuration.

This converts a *cross-modal* domain gap into a *minimal residual gap* (angle distributions differ slightly due to sensor noise, but overlap substantially). The residual gap (Kinect std 0.389 vs MediaPipe std 0.440) is small enough for BatchNorm to tolerate during inference.

See [[concepts/sensor-agnostic-features]] for the mathematical detail.

### 4. Why Transfer Learning Failed

The capstone project attempted a second approach: train on UI-PRMD (Kinect, 2,000 sequences), then fine-tune on REHAB24-6 (webcam, ~400 sequences). Result: 63.8% accuracy — *worse* than training from scratch on REHAB24-6 (72%).

**Primary cause — BatchNorm statistics mismatch**: EfficientGCN uses BatchNorm layers that accumulate running mean and variance statistics during training. These statistics encode properties of the source domain's feature distribution. When the model is fine-tuned on a new domain, these running statistics update slowly (momentum 0.1), causing the normalization to be systematically wrong throughout fine-tuning, distorting all activations.

**Secondary cause — data scarcity**: 400 sequences is insufficient to update 690K parameters without severe overfitting, even with early stopping. The pre-trained initialization may be actively harmful if it places the model in a local minimum far from the REHAB24-6 optimum.

**Potential fix**: Replace BatchNorm with InstanceNorm or LayerNorm before transfer, which normalize per-sample rather than accumulating running statistics — eliminating the domain-specific running stats problem.

### 5. The Data Bottleneck

The capstone finding that accuracy scales strongly with data volume (400 seqs → 72%, 1,020 → 80.4%, 2,000 → 96.3%) implies that collecting more labeled rehabilitation video is more impactful than domain adaptation research for this specific task. However, collecting labeled data requires clinical expertise (physiotherapist labeling) and specialized equipment — which is why domain adaptation (avoiding the need for labeled target data) remains valuable.

### 6. Domain Gap Quantification

Empirical comparison of feature distributions:

| Feature | Kinect (source) | MediaPipe (target) | Gap severity |
|---|---|---|---|
| Position min | −1842 mm | 0.01 | Catastrophic — no overlap |
| Position max | 1756 mm | 0.99 | Catastrophic — no overlap |
| **Angle mean** | **0.041** | **0.038** | **Minimal** |
| **Angle std** | **0.389** | **0.440** | **Minimal** |

The gap goes from catastrophic to minimal purely through representation choice, with no target-domain data required.

## Evidence & Examples

From [[sources/capstone-rehab-exercise-assessment]]:
- Sensor-agnostic angles achieve 96.3% with zero webcam training data, bridging 96% of the gap to the sensor-specific upper bound (98.0%).
- Fine-tuning failure (63.8%) is a negative result that usefully constrains the design space: BatchNorm-based models are fragile to cross-domain fine-tuning when the domain shift is large.
- 100% of shoulder abduction frames were classified correctly at runtime — the domain gap is practically bridged for in-distribution exercises.

## Contradictions & Open Questions

- Whether domain adversarial training (DANN, CORAL, MMD-based methods) could further close the residual angle distribution gap (std 0.389 vs 0.440) without labeled target data hasn't been explored.
- Instance normalization as a BatchNorm replacement for transfer learning hasn't been tested — it's an open question whether it would enable successful fine-tuning from 63.8% toward the 90%+ range.
- The 2D angle approximation introduces a systematic error for out-of-plane movements; whether this constitutes a second domain gap (depth accuracy vs estimated depth) for certain exercises is an open question.

## Related

- [[concepts/sensor-agnostic-features]] — the feature engineering approach to domain adaptation
- [[concepts/pose-estimation]] — where the domain gap originates (sensor differences)
- [[concepts/efficientgcn-architecture]] — the model whose BatchNorm layers cause transfer learning failure
- [[entities/mediapipe-blazepose]] — the target-domain sensor
- [[entities/microsoft-kinect]] — the source-domain sensor
- [[entities/ui-prmd-dataset]] — source domain dataset
