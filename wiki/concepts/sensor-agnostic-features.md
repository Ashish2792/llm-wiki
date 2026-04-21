---
title: Sensor-Agnostic Joint Angle Features
type: concept
tags: [feature-engineering, domain-adaptation, joint-angle, pose-estimation, skeleton]
sources: [capstone-rehab-exercise-assessment]
last_updated: 2026-04-20
---

# Sensor-Agnostic Joint Angle Features

## Definition

Sensor-agnostic joint angle features are skeletal motion representations computed as the angles between limb vectors at each joint, using vector dot products. {paraphrase} [[sources/capstone-rehab-exercise-assessment#methodology]] Because angles are invariant to coordinate system, scale, and translation, they produce consistent feature distributions regardless of whether the underlying skeleton positions were captured by a Kinect depth sensor (3D, millimeter units) or estimated by MediaPipe from an RGB webcam (2D, normalized units). {paraphrase} [[sources/capstone-rehab-exercise-assessment#methodology]] This invariance enables cross-modal deployment: training on Kinect data and running inference via webcam without any target-domain retraining. {inferred} [[sources/capstone-rehab-exercise-assessment#discussion]]

## Why It Matters

Most public rehabilitation exercise datasets (UI-PRMD, KIMORE) were collected with Microsoft Kinect sensors. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]] Deploying a model trained on this data to webcam video causes catastrophic failure when raw joint positions are used as features, because the value ranges have no overlap: Kinect positions range approximately [−1842, 1756] mm while MediaPipe positions are normalized to [0.01, 0.99]. {direct} [[sources/capstone-rehab-exercise-assessment#results]] Joint angle features reduce this to a minimal domain gap: means of 0.041 vs 0.038 and standard deviations of 0.389 vs 0.440 — close enough for BatchNorm layers to tolerate. {direct} [[sources/capstone-rehab-exercise-assessment#results]]

## Key Properties / Dimensions

### 1. Mathematical Formulation

For joint `j` with two connected neighbors `a` and `b`, the joint angle is: {direct} [[sources/capstone-rehab-exercise-assessment#methodology]]

```
v_a = p_a − p_j       (limb vector toward neighbor a)
v_b = p_b − p_j       (limb vector toward neighbor b)

θ_j = arccos( (v_a · v_b) / max(‖v_a‖ · ‖v_b‖, ε) )  ∈ [0, π]
```

Normalized: `θ_j / π ∈ [0, 1]`

Joints at skeleton extremities (hands, feet) with only one neighbor have their angle set to zero. {direct} [[sources/capstone-rehab-exercise-assessment#methodology]] The epsilon guard prevents division by zero when joints are coincident. {direct} [[sources/capstone-rehab-exercise-assessment#methodology]]

### 2. Invariance Properties

The formula is invariant to three transformations that differ between sensor modalities: {paraphrase} [[sources/capstone-rehab-exercise-assessment#methodology]]

| Property | Why it holds |
|---|---|
| Translation invariance | The subtraction `p_a − p_j` cancels absolute position — the angle doesn't depend on where in space the skeleton sits |
| Scale invariance | Dividing by `‖v_a‖ · ‖v_b‖` cancels any uniform scaling — a 90° elbow flexion is 90° whether measured in mm or normalized [0,1] |
| Rotation invariance | The dot product is invariant to rigid rotations of the entire skeleton in 2D; approximately invariant in 3D for common viewing angles |
| Sensor invariance | All of the above combine: Kinect 3D mm coordinates and MediaPipe 2D normalized coordinates produce the same angle for the same body configuration |

A 90-degree elbow flexion produces the same normalized value of 0.5 regardless of sensor modality. {direct} [[sources/capstone-rehab-exercise-assessment#methodology]]

### 3. Multi-Stream Construction

Three complementary streams, each with 3 components (x, y, z decomposition of the angle), giving 9 channels total: {paraphrase} [[sources/capstone-rehab-exercise-assessment#methodology]]

**Angle stream (ch 0–2)**: Raw normalized joint angles `θ_j / π` at each frame — the instantaneous angular configuration of the skeleton. This alone achieves 93.1% accuracy on UI-PRMD. {direct} [[sources/capstone-rehab-exercise-assessment#results]]

**Velocity stream (ch 3–5)**: Frame-to-frame temporal difference `Δθ_j = θ_j^(t) − θ_j^(t-1)` — captures movement dynamics. Correct exercises tend to have smooth, controlled velocity profiles while incorrect exercises often show jerky or irregular patterns. {paraphrase} [[sources/capstone-rehab-exercise-assessment#methodology]] Adds +2.1% accuracy. {direct} [[sources/capstone-rehab-exercise-assessment#results]]

**Relative stream (ch 6–8)**: Each joint's angle minus its parent's angle in the skeleton hierarchy `θ_j^rel = θ_j − θ_parent(j)` — encodes relative joint configuration within kinematic chains, distinguishing isolated joint errors from coordinated multi-joint movement patterns. {paraphrase} [[sources/capstone-rehab-exercise-assessment#methodology]] Adds +1.1% accuracy. {direct} [[sources/capstone-rehab-exercise-assessment#results]]

**Final tensor shape**: `(C=9, T=200, V=22, M=1)` — channels × time × joints × persons. {direct} [[sources/capstone-rehab-exercise-assessment#methodology]]

### 4. Temporal Resampling

Raw exercise sequences vary from approximately 50 to 500 frames. {paraphrase} [[sources/capstone-rehab-exercise-assessment#methodology]] All are resampled to 200 frames using linear interpolation along the temporal axis. {direct} [[sources/capstone-rehab-exercise-assessment#methodology]] This value was empirically validated: {direct} [[sources/capstone-rehab-exercise-assessment#results]]

- 100 frames → 94.1% (too coarse, loses movement dynamics)
- 200 frames → 96.3% (optimal)
- 300 frames → 96.4% (+0.1%) at +50% compute — not worth it

### 5. Empirical Validation of Domain Invariance

| Feature type | Kinect | MediaPipe | Domain gap |
|---|---|---|---|
| Position range (min) | −1842 mm | 0.01 | Catastrophic |
| Position range (max) | 1756 mm | 0.99 | Catastrophic |
| Position mean | 12.4 | 0.48 | Catastrophic |
| **Angle range (min)** | **−1.82** | **−1.73** | **Minimal** |
| **Angle range (max)** | **1.96** | **1.92** | **Minimal** |
| **Angle mean** | **0.041** | **0.038** | **Minimal** |
| **Angle std** | **0.389** | **0.440** | **Minimal** |

{direct} [[sources/capstone-rehab-exercise-assessment#results]]

The residual std difference (0.389 vs 0.440) reflects MediaPipe's higher landmark noise compared to Kinect depth tracking, but this is within the tolerance of BatchNorm layers. {paraphrase} [[sources/capstone-rehab-exercise-assessment#results]]

### 6. Data Augmentation

Applied during training to improve robustness: {paraphrase} [[sources/capstone-rehab-exercise-assessment#methodology]]

- **Temporal speed perturbation**: Stretch or compress sequence by ±10% before resampling to 200 frames — improves robustness to natural variation in exercise execution speed.
- **Gaussian noise injection** (σ = 0.01): Simulates the measurement uncertainty of MediaPipe's higher landmark noise compared to Kinect.

Augmentations deliberately avoided: {paraphrase} [[sources/capstone-rehab-exercise-assessment#methodology]]

- **Spatial flipping** (swap left/right): Some UI-PRMD exercises are side-specific; flipping changes exercise identity.
- **Joint dropout**: Removing joints changes the graph topology and breaks the angle computation at affected nodes.

### 7. Comparison with Pre-Computed Euler Angles

Kinect provides pre-computed quaternion-derived Euler angles alongside joint positions. {paraphrase} [[sources/capstone-rehab-exercise-assessment#methodology]] These yield 98.0% accuracy on UI-PRMD — the highest of any representation tested. {direct} [[sources/capstone-rehab-exercise-assessment#results]] However, they cannot be reproduced from MediaPipe landmarks, making them unusable for the cross-modal deployment goal. {paraphrase} [[sources/capstone-rehab-exercise-assessment#results]] The vector-computed angles (96.3%) close 96% of the gap between the raw-position baseline (85.3%) and the sensor-specific upper bound (98.0%) while remaining sensor-agnostic. {inferred} [[sources/capstone-rehab-exercise-assessment#results]]

## Evidence & Examples

The progression from raw 3D positions (85.3%) → tuned 3D positions (92.3%) → vector angles (96.3%) demonstrates that the feature representation is the single largest lever for accuracy on this task. {inferred} [[sources/capstone-rehab-exercise-assessment#results]]

Live webcam deployment demonstrates the domain gap is practically bridged: shoulder abduction (in-distribution) was classified correctly on 100% of frames using a model trained entirely on Kinect data. {direct} [[sources/capstone-rehab-exercise-assessment#results]]

A side lunge (out-of-distribution) was classified as incorrect on 89% of frames — the expected and clinically appropriate behavior. {direct} [[sources/capstone-rehab-exercise-assessment#results]]

## Contradictions & Open Questions

The residual std difference (0.389 vs 0.440) means angle feature distributions are close but not identical. For exercises involving extreme range-of-motion configurations, distribution tails may differ enough to cause occasional misclassification. {inferred} [[sources/capstone-rehab-exercise-assessment#discussion]]

The invariance argument assumes a consistent joint topology between Kinect and MediaPipe. In practice, some Kinect joints (SpineBase, SpineMid) have no direct MediaPipe equivalent and must be estimated by averaging nearby landmarks — introducing a systematic approximation error at those joints. {paraphrase} [[sources/capstone-rehab-exercise-assessment#methodology]]

Whether 2D angles (computed ignoring the z-coordinate) lose clinically important information for exercises with significant out-of-plane rotation (e.g., shoulder internal-external rotation) is an open question. {paraphrase} [[sources/capstone-rehab-exercise-assessment#limitations]]

## Related

- [[concepts/domain-adaptation]] — the broader problem this approach addresses
- [[concepts/efficientgcn-architecture]] — the model that consumes these features
- [[concepts/pose-estimation]] — how joint positions are obtained from different sensors
- [[entities/mediapipe-blazepose]] — the inference-time pose estimator
- [[entities/microsoft-kinect]] — the training-time sensor
