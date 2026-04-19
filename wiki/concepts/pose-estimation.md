---
title: Pose Estimation
type: concept
tags: [pose-estimation, mediapipe, kinect, skeleton, landmark-detection, computer-vision]
sources: [capstone-rehab-exercise-assessment]
last_updated: 2026-04-14
---

# Pose Estimation

## Definition

Pose estimation is the computer vision task of detecting and localizing human body joints (keypoints) in images or video. The output is a set of landmark coordinates — typically 17–33 joints per person — representing the skeleton configuration at each frame. Depending on the system, these coordinates may be 2D (pixel coordinates), 3D (estimated depth), or normalized (scale-independent fractions of image dimensions).

## Why It Matters

Pose estimation is the first stage of any skeleton-based movement analysis pipeline. Its output quality directly bounds downstream task accuracy — garbage joint positions produce garbage action classifications regardless of how sophisticated the downstream model is. Critically, different pose estimation systems produce structurally incompatible outputs (different joint counts, coordinate systems, units), creating domain gaps when training data from one system is used with inference from another.

## Key Properties / Dimensions

### 1. Approach Categories

**Depth sensor-based** (e.g., Microsoft Kinect): Uses an infrared structured-light or time-of-flight depth camera to estimate true 3D coordinates. Output is in millimeters in a camera-centered coordinate system. High accuracy, occlusion-robust, but requires specialized hardware and controlled environments.

**RGB-based top-down** (e.g., MediaPipe BlazePose): Detects a bounding box around the person, then runs a landmark regression network on the cropped region. Output is in normalized coordinates [0,1] relative to the bounding box (plus a pseudo-depth estimate). Works on standard RGB cameras but sensitive to viewing angle and occlusion.

**RGB-based bottom-up** (e.g., OpenPose): Detects all joints in the image simultaneously, then groups them into individuals. Scales better to multi-person scenes but is slower and less accurate for single-person scenarios.

### 2. MediaPipe BlazePose vs Microsoft Kinect

| Property | MediaPipe BlazePose | Microsoft Kinect v2 |
|---|---|---|
| Hardware | Standard RGB webcam | Depth sensor (IR + RGB) |
| Output coordinates | 2D normalized [0,1] + pseudo-z | 3D in mm (true depth) |
| Joints | 33 landmarks | 25 joints (mapped to 22 for UI-PRMD) |
| Depth quality | Estimated (noisy) | Measured (accurate) |
| Processing speed | ~26ms/frame (GPU) / ~38ms/frame (CPU) | ~30ms/frame |
| Deployment | Any device with a camera | Requires Kinect hardware |
| Cost | Free, software-only | ~$100–200 hardware |
| Single-person only | Yes | Yes (standard SDK) |

### 3. The Coordinate System Gap

The domain gap between Kinect and MediaPipe is primarily a coordinate system incompatibility:
- **Kinect**: 3D coordinates in millimeters, body-centered, range approximately [−2000, 2000] mm
- **MediaPipe**: 2D normalized coordinates in [0.01, 0.99], z estimated as a depth hint relative to the hip (not true 3D depth)

This creates a catastrophic feature distribution mismatch for any model trained on one and deployed on the other:
- Raw position feature range: Kinect [−1842, 1756] vs MediaPipe [0.01, 0.99] — no overlap whatsoever
- The solution is computing derived features (joint angles) that are invariant to coordinate system, scale, and translation

See [[concepts/sensor-agnostic-features]] for the mathematical details of the invariance construction.

### 4. Landmark Topology Mapping

Kinect v2 provides 25 joints; MediaPipe provides 33 landmarks. The capstone project uses 22 joints from Kinect (matching the UI-PRMD dataset protocol). Mapping MediaPipe's 33 to these 22 requires:
- **Direct 1-to-1 correspondences**: Most limb joints (elbow, knee, wrist, ankle) have direct equivalents
- **Averaging for synthetic joints**: Some Kinect joints (SpineBase, SpineMid) have no direct MediaPipe equivalent and are estimated by averaging nearby landmarks — introducing a systematic approximation error at those joints
- **Discarding excess MediaPipe landmarks**: Face landmarks (nose, ears, eyes) and foot landmarks not present in Kinect are dropped

### 5. MediaPipe Pseudo-3D Limitation

MediaPipe estimates a z-coordinate (depth) as a heuristic relative to the hip joint, not as true measured depth. For exercises with significant out-of-plane rotation (e.g., shoulder internal-external rotation, hip internal rotation), this z-estimate is unreliable. The capstone project uses 2D angles (ignoring z) as the primary feature, which is clinically appropriate for most rehabilitation exercises that occur predominantly in the sagittal or frontal plane.

### 6. Inference Speed and System Bottleneck

In the full pipeline (MediaPipe → angle features → EfficientGCN → prediction):
- MediaPipe pose estimation: ~38 ms/frame on CPU (~26 ms with GPU) — **the bottleneck, 60% of total inference time**
- Joint angle computation: ~0.5 ms/frame
- EfficientGCN inference: ~25 ms/prediction (CPU) / ~4.5 ms/prediction (GPU)
- **Total: ~64 ms → ~15 fps on CPU**

Optimizing the GCN architecture has diminishing returns compared to improving or replacing MediaPipe for CPU-only deployment scenarios.

## Evidence & Examples

From [[sources/capstone-rehab-exercise-assessment]]:
- Real-time validation: shoulder abduction classified correctly on 100% of frames using a model trained on Kinect data — demonstrating that MediaPipe's output, after angle feature extraction, is distribution-compatible with the training domain.
- MediaPipe's higher landmark noise (std 0.440 vs Kinect's 0.389 in angle features) is within BatchNorm tolerance — the residual distribution difference doesn't cause classification failure for in-distribution exercises.

## Contradictions & Open Questions

- The 2D angle approximation (ignoring z) may lose clinically important information for out-of-plane exercises. Whether this matters depends on which exercises appear in deployment — most standard physiotherapy exercises are predominantly in-plane.
- MediaPipe BlazePose has been updated to version 0.10+ with improved accuracy, but the version-specific behavior may differ from what's documented in benchmarks — always test on the deployment MediaPipe version.
- Newer RGB-based estimators (e.g., ViTPose, RTMPose) claim higher accuracy on standard benchmarks; whether they maintain the single-person speed advantage of BlazePose for real-time use hasn't been tested in the rehab context.

## Related

- [[entities/mediapipe-blazepose]] — the specific pose estimator used for inference
- [[entities/microsoft-kinect]] — the sensor used for training data collection
- [[concepts/sensor-agnostic-features]] — how pose estimation output is transformed into domain-invariant features
- [[concepts/domain-adaptation]] — the broader problem created by cross-sensor deployment
- [[concepts/skeleton-based-action-recognition]] — the downstream task that consumes pose estimation output
