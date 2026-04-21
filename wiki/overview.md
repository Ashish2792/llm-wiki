---
title: Wiki Overview
type: overview
last_updated: 2026-04-21
---

# Wiki Overview

This wiki is a personal knowledge base documenting Ashish's B.Tech CSE research at Lovely Professional University. It spans two active research threads: AI-powered movement analysis on consumer hardware, and multi-agent research intelligence systems for document question-answering.

## Research Thread 1 — AI Movement Analysis on Consumer Hardware

**Central question: How can clinically useful AI movement analysis run on consumer hardware?**

The core problem: high-quality labeled motion data exists in research settings (Kinect sensors, motion capture labs) but deployment targets are consumer devices (webcams, smartphones). The training/inference sensor gap creates a domain adaptation challenge that, naively approached, causes catastrophic model failure.

The primary answer documented here: **sensor-agnostic joint angle features** — computing joint angles from vector dot products produces features that are mathematically invariant to coordinate system, scale, and translation, bridging the gap between Kinect 3D millimeter coordinates and MediaPipe normalized 2D coordinates without requiring any target-domain training data.

## Architecture of Knowledge

```
Deployment challenge
    ↓
Domain gap (Kinect → webcam)
    ↓
Solution: Sensor-agnostic features (joint angles)
    ↓
Model: EfficientGCN-B0 on skeleton graph
    ↓
Inference: MediaPipe BlazePose → angle computation → classification
    ↓
Result: 96.3% accuracy, ~15 fps, CPU-only
```

## Key Entry Points

### By Problem

- **What is the domain gap and how is it bridged?** → [[concepts/domain-adaptation]], [[concepts/sensor-agnostic-features]]
- **How does the GCN model work?** → [[concepts/graph-convolutional-networks]], [[concepts/efficientgcn-architecture]]
- **How are joint positions obtained at inference?** → [[concepts/pose-estimation]], [[entities/mediapipe-blazepose]]
- **What dataset was used for training?** → [[entities/ui-prmd-dataset]], [[entities/microsoft-kinect]]

### By Concept

- [[concepts/sensor-agnostic-features]] — the central technical contribution: joint angles as cross-modal invariant features
- [[concepts/efficientgcn-architecture]] — the ~690K parameter model that runs in real-time on CPU
- [[concepts/graph-convolutional-networks]] — the theoretical foundation: skeleton as spatial-temporal graph, ST-GCN lineage
- [[concepts/skeleton-based-action-recognition]] — field context and rehabilitation-specific challenges
- [[concepts/pose-estimation]] — MediaPipe vs Kinect comparison and coordinate system incompatibility
- [[concepts/domain-adaptation]] — why transfer learning failed and how representation engineering succeeded

### Primary Source

- [[sources/capstone-rehab-exercise-assessment]] — the B.Tech capstone project (Ashish, April 2026) that provides all empirical evidence

---

## Research Thread 2 — Multi-Agent Research Intelligence (MindForge)

**Central question: How can agentic AI overcome the core limitations of basic RAG for complex document question-answering?**

Basic RAG (single-pass retrieval + LLM generation) fails on complex questions due to shallow retrieval, no self-correction, and no memory across sessions. MindForge addresses all three through a four-agent pipeline: Planner (query decomposition) → Researcher (two-phase retrieval via Endee dual-index) → Critic (structured completeness evaluation with confidence score) → Synthesizer (five-section structured report). A self-correcting loop re-runs research when the Critic returns `insufficient`.

```
Complex query
    ↓
Planner → ≤4 sub-questions
    ↓
Researcher → memory-first retrieval (mf_memory) → fallback to document index (mf_knowledge)
    ↓
Critic → JSON verdict (sufficient/insufficient + confidence + gaps)
    ↓ [re-research loop if insufficient, max 3 iterations]
Synthesizer → structured report with citations and confidence score
```

### Key Entry Points

- **What are the system's components?** → [[concepts/multi-agent-systems]], [[sources/mindforge-report]]
- **How does retrieval work?** → [[concepts/retrieval-augmented-generation]], [[concepts/vector-databases]], [[entities/endee]]
- **How does each agent reason?** → [[concepts/react-agent-pattern]]
- **What is the project?** → [[domains/personal/projects/mindforge]]

### Primary Source

- [[sources/mindforge-report]] — the B.Tech seminar report (Ashish, Jan–April 2026) documenting the full MindForge design, implementation, and empirical results

---

## Current State of Knowledge

**Well-established:**
- Sensor-agnostic joint angles bridge the Kinect→webcam gap (empirically validated end-to-end)
- EfficientGCN-B0 achieves competitive accuracy at real-time CPU speeds
- The field is data-limited, not architecture-limited (accuracy scales with sequence count)
- BatchNorm-based models fail at cross-domain transfer learning when domain shift is large

**Open questions:**
- True subject-independent accuracy (leave-subjects-out) is not yet rigorously measured
- Whether 2D angle features lose clinically important information for out-of-plane exercises
- Whether InstanceNorm could enable successful transfer learning
- How well the system generalizes to real patient populations vs healthy subjects simulating errors

## Research Thesis

See [[domains/research/thesis]] for the evolving synthesis across both research threads.

## Personal Projects

See [[domains/personal/projects/capstone]] for B.Tech capstone project tracking (AI rehabilitation exercise assessment, EfficientGCN + MediaPipe, 96.3% accuracy).

See [[domains/personal/projects/mindforge]] for MindForge project tracking (4-agent RAG system, Endee dual-index, Streamlit UI).
