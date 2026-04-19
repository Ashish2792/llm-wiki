---
title: UI-PRMD Dataset
type: entity
entity_type: product
tags: [dataset, rehabilitation, skeleton, kinect, vicon, exercise-assessment, benchmark]
last_updated: 2026-04-14
---

# UI-PRMD Dataset

## Overview

The UI-PRMD (University of Idaho — Physical Rehabilitation Movement Data) dataset is a publicly available skeleton sequence dataset for physical rehabilitation exercise analysis. It was collected at the University of Idaho and provides synchronized Kinect v2 and Vicon motion capture recordings of 10 subjects performing 10 rehabilitation exercises, with binary correctness labels (correct vs incorrect execution).

## Relevance to This Wiki

UI-PRMD is the primary training and evaluation dataset for the capstone rehabilitation exercise assessment system. It provides the Kinect skeleton sequences on which EfficientGCN-B0 is trained and evaluated. Its size (~2,000 sequences), sensor (Kinect v2), and labeling protocol (binary correct/incorrect) define the entire design space of the capstone project.

## Key Facts

- **Subjects**: 10 healthy subjects (not patients)
- **Exercises**: 10 standard rehabilitation exercises (e.g., shoulder abduction, elbow flexion, squat, lunge)
- **Sequences**: ~2,000 total (~100 per exercise: ~50 correct, ~50 incorrect per exercise per subject)
- **Sensors**: Microsoft Kinect v2 (25 joints, 3D mm coordinates) + Vicon motion capture (gold standard, not used in capstone)
- **Joint count used**: 22 joints (subset of Kinect v2's 25, matching the UI-PRMD protocol)
- **Labels**: Binary — correct (1) / incorrect (0) per sequence
- **Incorrect generation method**: Healthy subjects instructed to deliberately perform movements with specific deviation patterns (e.g., limited range of motion, trunk compensation)
- **Frame rate**: 30 fps; sequence lengths vary from ~50 to ~500 frames
- **Availability**: Publicly downloadable

## Dataset Split Used in Capstone

- Train: ~1,600 sequences (80%)
- Validation: ~200 sequences (10%)
- Test: ~200 sequences (10%)
- Split is sequence-level; subject-independence of the split is not guaranteed by the dataset protocol

## Results on UI-PRMD (Capstone)

| Method | Accuracy | F1 |
|---|---|---|
| Raw 3D positions (baseline) | 85.3% | — |
| Tuned 3D positions | 92.3% | — |
| EGCN ensemble | 96.9% | — |
| **EfficientGCN-B0 (angle features)** | **96.3%** | **0.963** |
| Kinect Euler angles (not sensor-agnostic) | 98.0% | — |

## Limitations

**Healthy subject simulation**: Incorrect executions are performed by healthy individuals following deliberate deviation instructions, not by real patients with injury or motor control limitations. Real patient errors may be subtler, more variable, and biomechanically different from simulated errors — potentially reducing generalization to clinical deployment.

**Subject-independence**: With only 10 subjects, true leave-subjects-out cross-validation produces high-variance estimates. The capstone's test accuracy (96.3%) may be inflated by subject leakage between train/test splits.

**Single sensor**: Although synchronized Vicon data was also captured, the capstone only uses Kinect data — Vicon's higher accuracy is not exploited.

## Appearances

- [[sources/capstone-rehab-exercise-assessment]] — primary training and evaluation dataset; all main results reported on UI-PRMD

## Connections

- [[entities/microsoft-kinect]] — the sensor used to collect UI-PRMD
- [[concepts/skeleton-based-action-recognition]] — the benchmark datasets section and accuracy comparison
- [[concepts/sensor-agnostic-features]] — features computed from UI-PRMD Kinect data for training
- [[concepts/domain-adaptation]] — UI-PRMD is the source domain; webcam deployment is the target domain
