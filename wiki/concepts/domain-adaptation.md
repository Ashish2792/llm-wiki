---
title: Domain Adaptation
type: concept
tags: [domain-adaptation, transfer-learning, domain-gap, cross-modal, batchnorm, feature-engineering]
sources: [capstone-rehab-exercise-assessment]
last_updated: 2026-04-20
---

# Domain Adaptation

## Definition

Domain adaptation is a machine learning strategy for deploying a model trained on data from one distribution (the *source domain*) to a different but related distribution (the *target domain*). The core challenge is that models learn to exploit statistical regularities in training data that may not hold at test time — a model trained on Kinect 3D skeleton data will have learned features tied to millimeter-scale coordinates and 3D geometry that are meaningless when applied to MediaPipe's normalized 2D coordinates. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]]

## Why It Matters

Most high-quality labeled datasets in specialized domains (rehabilitation, sports biomechanics, clinical gait analysis) were collected with expensive research-grade sensors (Kinect, Vicon motion capture). {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]] Deploying classifiers from these datasets to consumer hardware (webcams) requires bridging a domain gap. For raw joint positions, the distributions have zero overlap, guaranteeing complete model failure: Kinect positions range approximately [−1842, 1756] mm while MediaPipe positions are [0.01, 0.99] normalized — no overlap whatsoever. {direct} [[sources/capstone-rehab-exercise-assessment#results]]

## Key Properties / Dimensions

### 1. Types of Domain Gap

**Covariate shift**: P(X) differs between source and target but P(Y|X) is the same — the benign case where input distributions change but the feature-label relationship is preserved. Standard domain adaptation techniques (sample reweighting, feature alignment) are designed for this case. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]]

**Cross-modal gap** (the rehabilitation case): Source and target domains use entirely different measurement modalities (depth sensor vs RGB camera). The coordinate system, units, dimensionality, and noise characteristics are all different. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]] Standard covariate shift assumptions break down entirely for raw positional features. {inferred} [[sources/capstone-rehab-exercise-assessment#background]]

### 2. Strategies

| Strategy | Description | Applicability |
|---|---|---|
| Feature engineering | Design features invariant to domain shift | Works when a mathematical invariant exists — the approach taken in the capstone |
| Fine-tuning | Train on source, fine-tune on small target dataset | Requires labeled target data; BatchNorm statistics failure can undermine this |
| Domain adversarial training | Add a discriminator that cannot distinguish source/target features | Requires unlabeled target data during training |
| Instance re-weighting | Upweight source samples similar to target distribution | Requires estimating P(target)/P(source) |
| Normalization adaptation | Reset/adapt BatchNorm statistics to target domain | Lightweight; targeted fix for BatchNorm-specific failures |

{paraphrase} [[sources/capstone-rehab-exercise-assessment#background]]

### 3. The Feature Engineering Approach (Sensor-Agnostic Angles)

For the rehabilitation domain, the cross-modal coordinate gap has a clean mathematical solution: compute joint angles using vector dot products. {paraphrase} [[sources/capstone-rehab-exercise-assessment#methodology]] Since arccos((v_a · v_b) / (‖v_a‖ · ‖v_b‖)) is invariant to scale, translation, and coordinate system, the same formula applied to Kinect mm coordinates and MediaPipe normalized coordinates produces the same angle for the same body configuration. {direct} [[sources/capstone-rehab-exercise-assessment#methodology]]

This converts a cross-modal domain gap into a minimal residual gap: angle distributions differ slightly due to sensor noise (Kinect std 0.389 vs MediaPipe std 0.440) but overlap substantially. {direct} [[sources/capstone-rehab-exercise-assessment#results]] The residual gap is small enough for BatchNorm to tolerate during inference. {paraphrase} [[sources/capstone-rehab-exercise-assessment#results]]

See [[concepts/sensor-agnostic-features]] for the mathematical detail.

### 4. Why Transfer Learning Failed

The capstone project attempted transfer learning: train on UI-PRMD (Kinect, 2,000 sequences), then fine-tune on REHAB24-6 (webcam, ~400 sequences). Result: 63.8% accuracy — *worse* than training from scratch on REHAB24-6 (72%). {direct} [[sources/capstone-rehab-exercise-assessment#results]]

Three causes were identified: {paraphrase} [[sources/capstone-rehab-exercise-assessment#discussion]]

**Primary cause — BatchNorm statistics mismatch**: EfficientGCN uses BatchNorm layers that accumulate running mean and variance statistics during training. These statistics encode properties of the UI-PRMD angle distribution (Kinect-derived). Even with unfrozen affine parameters (scale and shift), the frozen running mean and variance create a systematic normalization error throughout fine-tuning. {paraphrase} [[sources/capstone-rehab-exercise-assessment#discussion]]

**Secondary cause — data scarcity**: ~400 fine-tuning sequences are insufficient for the ~380K trainable parameters in blocks 4–7, leading to severe overfitting even with early stopping. {direct} [[sources/capstone-rehab-exercise-assessment#discussion]]

**Tertiary cause — exercise set mismatch**: UI-PRMD and REHAB24-6 cover different exercises (10 vs 6), so exercise-specific patterns learned in early blocks may not transfer effectively. {paraphrase} [[sources/capstone-rehab-exercise-assessment#discussion]]

**Potential fix**: Replace BatchNorm with InstanceNorm or LayerNorm before transfer — these normalize per-sample rather than accumulating running statistics, eliminating the domain-specific running-stats problem. {inferred} [[sources/capstone-rehab-exercise-assessment#discussion]]

### 5. The Data Bottleneck

Accuracy scales strongly with training data volume: 400 sequences → 72%; 1,020 sequences → 80.4%; 2,000 sequences → 96.3%. {direct} [[sources/capstone-rehab-exercise-assessment#results]] This implies that collecting more labeled rehabilitation video is more impactful than domain adaptation research for this specific task. {inferred} [[sources/capstone-rehab-exercise-assessment#discussion]] However, collecting labeled data requires clinical expertise (physiotherapist labeling) and controlled conditions — which is exactly why cross-modal domain adaptation (reusing existing Kinect datasets) remains valuable. {inferred} [[sources/capstone-rehab-exercise-assessment#discussion]]

### 6. Domain Gap Quantification

Empirical comparison of feature distributions: {direct} [[sources/capstone-rehab-exercise-assessment#results]]

| Feature | Kinect (source) | MediaPipe (target) | Gap severity |
|---|---|---|---|
| Position min | −1842 mm | 0.01 | Catastrophic — no overlap |
| Position max | 1756 mm | 0.99 | Catastrophic — no overlap |
| **Angle mean** | **0.041** | **0.038** | **Minimal** |
| **Angle std** | **0.389** | **0.440** | **Minimal** |

The gap goes from catastrophic to minimal purely through representation choice, with no target-domain data required. {inferred} [[sources/capstone-rehab-exercise-assessment#results]]

## Evidence & Examples

Sensor-agnostic angles achieve 96.3% accuracy with zero webcam training data, bridging 96% of the gap to the sensor-specific upper bound (98.0% from Kinect Euler angles). {inferred} [[sources/capstone-rehab-exercise-assessment#results]]

Fine-tuning failure (63.8%) is a negative result that usefully constrains the design space: BatchNorm-based models are fragile to cross-domain fine-tuning when the domain shift is large and fine-tuning data is scarce. {inferred} [[sources/capstone-rehab-exercise-assessment#discussion]]

100% of shoulder abduction frames were classified correctly at runtime with a model trained entirely on Kinect data. {direct} [[sources/capstone-rehab-exercise-assessment#results]] The domain gap is practically bridged for in-distribution exercises.

## Contradictions & Open Questions

Whether domain adversarial training (DANN, CORAL, MMD-based methods) could further close the residual angle distribution gap (std 0.389 vs 0.440) without labeled target data hasn't been explored. {paraphrase} [[sources/capstone-rehab-exercise-assessment#open-questions]]

Instance normalization as a BatchNorm replacement for transfer learning hasn't been tested — whether it would enable successful fine-tuning from the 63.8% baseline toward the 90%+ range remains an open question. {paraphrase} [[sources/capstone-rehab-exercise-assessment#discussion]]

The 2D angle approximation introduces systematic error for out-of-plane movements; whether this constitutes a second domain gap (depth accuracy vs estimated depth) for exercises like shoulder internal-external rotation is an open question. {inferred} [[sources/capstone-rehab-exercise-assessment#limitations]]

## Related

- [[concepts/sensor-agnostic-features]] — the feature engineering approach to domain adaptation
- [[concepts/pose-estimation]] — where the domain gap originates (sensor differences)
- [[concepts/efficientgcn-architecture]] — the model whose BatchNorm layers cause transfer learning failure
- [[entities/mediapipe-blazepose]] — the target-domain sensor
- [[entities/microsoft-kinect]] — the source-domain sensor
- [[entities/ui-prmd-dataset]] — source domain dataset
- [[entities/rehab24-6-dataset]] — target domain dataset where transfer learning failed
