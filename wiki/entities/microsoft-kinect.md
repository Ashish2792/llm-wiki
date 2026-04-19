---
title: Microsoft Kinect
type: entity
entity_type: product
tags: [kinect, depth-sensor, skeleton-tracking, microsoft, pose-estimation, 3d-capture]
last_updated: 2026-04-14
---

# Microsoft Kinect

## Overview

Microsoft Kinect is a depth-sensing camera peripheral originally developed for the Xbox 360 game console (Kinect v1, 2010) and later released as a standalone sensor (Kinect v2, 2014). It combines an RGB camera with an infrared depth sensor (structured light in v1; time-of-flight in v2) to produce real-time 3D body skeleton tracking without requiring markers or special clothing. Kinect v2 was discontinued in 2017, but its influence on skeleton-based action recognition research persists — the NTU RGB+D and UI-PRMD datasets, which define the field's benchmarks, were both collected with Kinect v2.

## Relevance to This Wiki

Kinect is the training-time sensor for the capstone rehabilitation exercise assessment system. The UI-PRMD dataset was collected with Kinect v2, and the model is trained entirely on Kinect skeleton data. The fundamental deployment challenge — deploying a Kinect-trained model on webcam input — exists because Kinect was discontinued and is unavailable in home settings, making webcam adaptation an essential research problem.

## Key Facts

- **Kinect v2 output**: 25 body joints in 3D Cartesian coordinates (millimeters), 30 fps
- **Coordinate range**: approximately [−2000, 2000] mm in x and y, deeper in z — varies with subject distance
- **Tracking quality**: Robust to moderate occlusion due to infrared depth sensing; accurate to within a few millimeters for large joints
- **Pre-computed skeletal data**: In addition to raw positions, the Kinect SDK provides pre-computed quaternion-derived Euler angles for each joint — these achieve 98.0% accuracy on UI-PRMD but cannot be reproduced from RGB-only sensors
- **Joint set**: 25 joints including SpineBase (pelvis), SpineMid, SpineShoulder, Head, 4 limb joints per arm/leg, Hands, Feet, Thumbs
- **Discontinuation**: Kinect v2 discontinued in 2017; Kinect Azure (2019) briefly succeeded it, also discontinued 2023

## The Kinect vs MediaPipe Gap

The core domain adaptation challenge in the capstone:

| Property | Kinect v2 | MediaPipe BlazePose |
|---|---|---|
| Sensor type | Depth (time-of-flight IR) | RGB camera |
| Coordinates | 3D mm, true depth | 2D normalized [0,1] + estimated z |
| Position range | [−1842, 1756] mm | [0.01, 0.99] |
| Noise level | Low (depth-measured) | Higher (regression-estimated) |
| Hardware cost | ~$100–200 (discontinued) | Free (software-only) |
| Deployment context | Research lab, clinic | Home, any device |

This gap is what motivates the sensor-agnostic angle feature design — the only way to bridge a cross-modal gap of this magnitude without target-domain labeled data is to find features that are mathematically invariant to the coordinate system difference.

## Appearances

- [[sources/capstone-rehab-exercise-assessment]] — training-time sensor; source domain for all Kinect data
- [[entities/ui-prmd-dataset]] — the rehabilitation dataset collected with Kinect v2

## Connections

- [[entities/mediapipe-blazepose]] — the inference-time sensor that replaces Kinect in deployment
- [[concepts/pose-estimation]] — how Kinect fits in the broader pose estimation landscape
- [[concepts/sensor-agnostic-features]] — features designed to be invariant to the Kinect/MediaPipe gap
- [[concepts/domain-adaptation]] — the domain gap Kinect's discontinuation creates
