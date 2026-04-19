---
title: MediaPipe BlazePose
type: entity
entity_type: tool
tags: [pose-estimation, mediapipe, blazepose, google, landmark-detection, real-time]
last_updated: 2026-04-14
---

# MediaPipe BlazePose

## Overview

MediaPipe BlazePose is Google's real-time single-person pose estimation solution, part of the MediaPipe framework. It detects 33 body landmarks (keypoints) from standard RGB images or video, providing 2D screen coordinates plus a heuristically estimated z (depth) coordinate for each landmark. It is designed for CPU-only deployment on mobile and desktop devices, achieving ~25–30 fps on consumer hardware without a GPU.

## Relevance to This Wiki

BlazePose is the inference-time pose estimator in the capstone rehabilitation exercise assessment system. It enables the system to run on a standard webcam without any specialized hardware, solving the practical deployment problem. The key challenge it creates — its normalized coordinate output is structurally incompatible with the Kinect millimeter coordinates used for training — is the primary motivation for the sensor-agnostic joint angle feature design.

## Key Facts

- **33 landmarks**: Full body including face (nose, ears, eyes), shoulders, elbows, wrists, hips, knees, ankles, and feet — more than Kinect's 25 joints
- **Coordinates**: Normalized [0, 1] relative to the image frame; z-coordinate is an estimate relative to hip depth, not true measured depth
- **Speed**: ~38 ms/frame on CPU (Intel Core i7); ~26 ms/frame with GPU acceleration
- **Two-stage pipeline**: (1) Detector stage — localizes the person bounding box; (2) Landmark regression stage — predicts 33 joints within the crop
- **Single-person only**: Not designed for multi-person scenes; loses accuracy when multiple people are visible
- **Version used in capstone**: MediaPipe 0.10 (Python API)
- **Open source**: Apache 2.0 license; available via pip

## Landmark Topology

BlazePose's 33 landmarks include anatomical landmarks not present in Kinect (e.g., face landmarks at indices 0–10, individual finger/toe landmarks at indices 17–32). For the capstone project's 22-joint skeleton:
- Most limb joints have direct 1-to-1 mappings
- Kinect-specific spine joints (SpineBase, SpineMid) are approximated by averaging surrounding BlazePose landmarks
- Face and extremity landmarks (11 total) are discarded

## Limitations Relevant to This Project

- **No true depth**: The z-coordinate is estimated, not measured. For exercises with significant out-of-plane rotation, this limits 3D angle accuracy — the capstone uses 2D angles (ignoring z) as the primary feature.
- **Noise level**: BlazePose's per-landmark noise is higher than Kinect depth tracking — the angle feature std of 0.440 vs Kinect's 0.389 reflects this, though the difference is within BatchNorm tolerance.
- **Speed bottleneck**: At ~38 ms/frame on CPU, BlazePose consumes 60% of the total inference budget, compared to ~25 ms for the full EfficientGCN classification — optimizing the GCN has diminishing returns without also addressing the pose estimation bottleneck.

## Appearances

- [[sources/capstone-rehab-exercise-assessment]] — used as the inference-time pose estimator; characterized as the system's deployment bottleneck

## Connections

- [[concepts/pose-estimation]] — where BlazePose fits in the broader landscape of pose estimation approaches
- [[entities/microsoft-kinect]] — the training-time sensor whose output is structurally incompatible with BlazePose without feature engineering
- [[concepts/sensor-agnostic-features]] — the solution to the BlazePose/Kinect compatibility problem
- [[concepts/domain-adaptation]] — the domain gap BlazePose creates relative to Kinect training data
