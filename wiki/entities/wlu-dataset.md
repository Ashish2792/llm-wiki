---
title: WLU Dataset
type: entity
entity_type: product
tags: [dataset, rehabilitation, mediapipe, webcam, exercise-assessment]
last_updated: 2026-04-21
---

# WLU Dataset

## Overview

The WLU dataset is a webcam-based rehabilitation exercise dataset recorded using MediaPipe BlazePose. {paraphrase} [[sources/capstone-rehab-exercise-assessment#results]] In the capstone project, WLU was combined with REHAB24-6 to form a larger ~1,020-sequence native MediaPipe corpus, used to test EfficientGCN performance at an intermediate data scale between the 400-sequence REHAB24-6 alone and the 2,000-sequence UI-PRMD. {direct} [[sources/capstone-rehab-exercise-assessment#results]]

## Relevance to This Wiki

WLU provides the middle data point in the capstone's data volume scaling analysis. The finding that accuracy increases from 72% (400 sequences, REHAB24-6 alone) to 80.4% (1,020 sequences, WLU+REHAB24-6 combined) supports the data-bottleneck thesis: more native MediaPipe training data improves webcam-deployment accuracy, irrespective of model architecture changes. {inferred} [[sources/capstone-rehab-exercise-assessment#results]]

## Key Facts

- **Collection modality**: Standard webcam → MediaPipe BlazePose (2D normalized coordinates) {paraphrase} [[sources/capstone-rehab-exercise-assessment#results]]
- **Combined size with REHAB24-6**: ~1,020 sequences total {direct} [[sources/capstone-rehab-exercise-assessment#results]]
- **Experiment configuration**: Reduced EfficientGCN model (channels=[48,48,96,96,192,192,192,192], ~370K params), label smoothing ε=0.1, 5-fold cross-validation {direct} [[sources/capstone-rehab-exercise-assessment#results]]

## Results in Combined Experiment (WLU + REHAB24-6)

| Metric | Value |
|---|---|
| CV mean F1 | 0.811 ± 0.044 |
| Test accuracy | 80.4% |
| Training sequences | ~1,020 |
| Model | EfficientGCN, channels=[48,48,96,96,192,192,192,192], ~370K params |

{direct} [[sources/capstone-rehab-exercise-assessment#results]]

The reduced model (~370K vs 690K params) was chosen because the training set (~1,020 sequences) is insufficient for the full B0 model without severe overfitting. {paraphrase} [[sources/capstone-rehab-exercise-assessment#results]]

## Appearances

- [[sources/capstone-rehab-exercise-assessment]] — combined with REHAB24-6 for the ~1,020 sequence experiment; middle point in data volume scaling analysis

## Connections

- [[entities/rehab24-6-dataset]] — combined with WLU for the larger-scale experiment
- [[entities/mediapipe-blazepose]] — pose estimator used to collect WLU features
- [[concepts/domain-adaptation]] — WLU+REHAB24-6 represents the native-webcam training scenario
- [[concepts/sensor-agnostic-features]] — angle features computed from WLU sequences

## Open Questions

- The full provenance of the WLU dataset (institution, number of subjects, exercise types, collection protocol) is not described in the capstone source. {uncertain} [[sources/capstone-rehab-exercise-assessment#results]]
- Whether WLU and REHAB24-6 cover the same exercises is not specified; label compatibility may be assumed. {uncertain} [[sources/capstone-rehab-exercise-assessment#results]]
