---
title: AI-Powered Rehabilitation Exercise Assessment System Using EfficientGCN and Webcam-Based Pose Estimation
type: source
domain: research
date_ingested: 2026-04-14
source_file: raw/research/capstone_report.pdf
tags: [rehabilitation, graph-convolutional-network, pose-estimation, domain-adaptation, efficientgcn, mediapipe, skeleton-based-action-recognition, deep-learning]
last_updated: 2026-04-14
---

# AI-Powered Rehabilitation Exercise Assessment System

## TL;DR

This B.Tech capstone project (Ashish, April 2026) builds a real-time system that classifies physical therapy exercises as correct or incorrect using only a standard webcam. The central technical contribution is a **sensor-agnostic joint angle feature representation** — by computing angles between limb vectors rather than using raw joint positions, the system bridges the domain gap between Kinect depth-sensor training data (UI-PRMD) and MediaPipe RGB-based inference, achieving 96.3% accuracy without requiring any webcam-domain training data.

## Key Points

- **Four-stage pipeline**: MediaPipe BlazePose (33 landmarks) → joint angle computation (9 channels × 22 joints) → EfficientGCN-B0 classifier (~690K params) → binary prediction at ~15 fps on CPU.
- **Sensor-agnostic features**: Joint angles are invariant to translation, scale, and coordinate system. Feature distributions between Kinect (mean 0.041, std 0.389) and MediaPipe (mean 0.038, std 0.440) align closely; raw position features are catastrophically misaligned (Kinect: [-1842, 1756] mm vs MediaPipe: [0.01, 0.99]).
- **Results on UI-PRMD**: 96.3% accuracy, 0.963 macro F1, balanced across correct/incorrect classes. Competitive with SOTA EGCN ensemble (96.9%) while being a single model directly deployable to webcam.
- **Negative result — transfer learning fails**: Fine-tuning the UI-PRMD model on REHAB24-6 (~400 sequences) achieved only 63.8%, worse than training from scratch (72%). BatchNorm statistics mismatch and data scarcity are the primary culprits.
- **Data is the bottleneck**: Accuracy scales directly with sequence count — 2,000 seqs → 96.3%, 1,020 seqs → 80.4%, 400 seqs → 72%. The field is data-limited, not architecture-limited.
- **Multi-stream features matter**: Angle-only → 93.1%; Angle+Velocity → 95.2%; Angle+Velocity+Relative → 96.3%. Each stream adds distinct discriminative signal.
- **Real-time validation**: Shoulder abduction classified 100% correct; out-of-distribution side lunge classified 89% incorrect — expected and clinically appropriate behavior.

## Notable Details

- Pre-computed Kinect Euler angles yield 98.0% accuracy but cannot be reproduced from MediaPipe landmarks — the 1.7% gap over vector-computed angles (96.3%) is the literal cost of sensor-agnosticity.
- 200-frame temporal resampling is empirically optimal: 100 frames → 94.1%; 300 frames → 96.4% (+0.1%) at +50% compute cost.
- MediaPipe is the runtime bottleneck at ~38 ms/frame; EfficientGCN inference is ~25 ms (CPU) / 4.5 ms (GPU).
- UI-PRMD uses healthy subjects simulating incorrect movements — a limitation for clinical generalization, as real patient deviations may be more varied and subtle.
- Deployed as a FastAPI backend with WebSocket streaming for real-time feedback and REST endpoint for video file processing.
- Training: NVIDIA T4 via Kaggle Notebooks; PyTorch 2.0, MediaPipe 0.10, Python 3.10.

## Connections

- [[concepts/graph-convolutional-networks]] — ST-GCN lineage and how GCNs model skeleton graphs
- [[concepts/efficientgcn-architecture]] — specific architectural details of EfficientGCN-B0
- [[concepts/sensor-agnostic-features]] — the joint angle feature engineering approach
- [[concepts/skeleton-based-action-recognition]] — broader field context
- [[concepts/pose-estimation]] — MediaPipe vs Kinect comparison
- [[concepts/domain-adaptation]] — the domain gap problem and how this project addresses it
- [[entities/mediapipe-blazepose]] — inference-time pose estimator
- [[entities/ui-prmd-dataset]] — primary training dataset
- [[entities/microsoft-kinect]] — sensor used for training data collection

## Open Questions

- What is the true subject-independent accuracy? Post-hoc analysis suggests 90–96% but no strict leave-subjects-out protocol was run.
- Would instance normalization (instead of BatchNorm) allow better transfer learning by avoiding running-statistics mismatch across datasets?
- How does the 2D angle approximation perform on exercises with significant out-of-plane rotation (e.g., shoulder internal-external rotation)?
- Can a smaller architecture close the gap on REHAB24-6 without a data volume fix?
