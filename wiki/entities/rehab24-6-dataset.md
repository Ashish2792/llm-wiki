---
title: REHAB24-6 Dataset
type: entity
entity_type: product
tags: [dataset, rehabilitation, mediapipe, webcam, exercise-assessment, cross-dataset, transfer-learning]
last_updated: 2026-04-21
---

# REHAB24-6 Dataset

## Overview

REHAB24-6 is a rehabilitation exercise assessment dataset collected using MediaPipe BlazePose from standard webcam video — in contrast to Kinect-based datasets like UI-PRMD and KIMORE. {paraphrase} [[sources/capstone-rehab-exercise-assessment#results]] It contains approximately 400 sequences of 6 rehabilitation exercises, evaluated with a strict subject-independent train/test split. {direct} [[sources/capstone-rehab-exercise-assessment#results]] It serves as the target domain in the capstone's cross-dataset transfer learning experiment.

## Relevance to This Wiki

REHAB24-6 is the primary target-domain dataset in the capstone project. It represents the "real deployment scenario" — webcam-captured RGB video processed through MediaPipe, with no Kinect sensor involved. Results on this dataset directly test whether sensor-agnostic joint angle features generalize from Kinect training data to webcam inference. The transfer learning failure (63.8% with fine-tuning vs 72% training from scratch) on REHAB24-6 is a central negative finding of the project.

## Key Facts

- **Sequences**: ~400 total {direct} [[sources/capstone-rehab-exercise-assessment#results]]
- **Exercises**: 6 rehabilitation exercises (subset differs from UI-PRMD's 10) {paraphrase} [[sources/capstone-rehab-exercise-assessment#results]]
- **Sensor**: Standard webcam → MediaPipe BlazePose (2D normalized coordinates) {paraphrase} [[sources/capstone-rehab-exercise-assessment#results]]
- **Split protocol**: Strict subject-independent train/test split {direct} [[sources/capstone-rehab-exercise-assessment#results]]
- **Labels**: Binary correct/incorrect {paraphrase} [[sources/capstone-rehab-exercise-assessment#results]]

## Results on REHAB24-6 (Capstone)

| Approach | Accuracy | Notes |
|---|---|---|
| Training from scratch | ~72% | Native MediaPipe data only; ~400 sequences |
| Transfer learning from UI-PRMD | ~63.8% | Frozen blocks 0–3, fine-tuned blocks 4–7 — worse than scratch |
| Combined WLU + REHAB24-6 (larger model, CV) | 80.4% | ~1,020 sequences, reduced model, 5-fold CV |

{direct} [[sources/capstone-rehab-exercise-assessment#results]]

The transfer learning failure (63.8% vs 72% from scratch) is a negative result identifying BatchNorm running statistics, data scarcity (~400 sequences for ~380K parameters), and exercise set mismatch as the primary causes. {direct} [[sources/capstone-rehab-exercise-assessment#discussion]]

## Appearances

- [[sources/capstone-rehab-exercise-assessment]] — target domain for cross-dataset transfer learning experiment; key negative result (transfer learning failure)

## Connections

- [[entities/ui-prmd-dataset]] — source domain for transfer learning; REHAB24-6 is the transfer target
- [[entities/wlu-dataset]] — combined with REHAB24-6 for the ~1,020 sequence experiment
- [[entities/mediapipe-blazepose]] — pose estimator used to collect REHAB24-6 features
- [[concepts/domain-adaptation]] — REHAB24-6 is the target domain where the cross-modal deployment is tested
- [[concepts/sensor-agnostic-features]] — the feature engineering approach applied to REHAB24-6 data

## Open Questions

- Why does training from scratch on only 400 sequences (72%) perform better than fine-tuning from a 2,000-sequence model (63.8%)? The discussion identifies BatchNorm statistics as the primary cause, but InstanceNorm replacement hasn't been tested. {paraphrase} [[sources/capstone-rehab-exercise-assessment#discussion]]
- Whether the 6 exercises in REHAB24-6 overlap with UI-PRMD's 10, and which exercises are shared vs. different, is not detailed in the source. {uncertain} [[sources/capstone-rehab-exercise-assessment#results]]
