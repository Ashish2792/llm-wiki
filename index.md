# Wiki Index
Last updated: 2026-04-15 | Pages: 22 | Sources: 2

## Overview
- [[wiki/overview]] — Master synthesis and entry point: two research threads (movement analysis + multi-agent RAG)

## Sources
| Page | Domain | Summary | Date |
|------|--------|---------|------|
| [[sources/capstone-rehab-exercise-assessment]] | research | B.Tech capstone: EfficientGCN + MediaPipe webcam rehab classifier, 96.3% accuracy via sensor-agnostic joint angles | 2026-04-14 |
| [[sources/mindforge-report]] | research | B.Tech seminar: MindForge 4-agent RAG system with dual-index Endee architecture, self-correcting Critic loop | 2026-04-15 |

## Concepts
| Page | Summary |
|------|---------|
| [[concepts/sensor-agnostic-features]] | Joint angles via dot product — invariant to sensor coordinate system; bridges Kinect→webcam domain gap |
| [[concepts/efficientgcn-architecture]] | EfficientGCN-B0: ~690K params, early fusion, depthwise separable convs, real-time on CPU |
| [[concepts/graph-convolutional-networks]] | GCN theory for skeleton action recognition; ST-GCN lineage from 2018 to BlockGCN 2024 |
| [[concepts/skeleton-based-action-recognition]] | Field overview, benchmark datasets, rehabilitation-specific challenges, accuracy comparisons |
| [[concepts/pose-estimation]] | MediaPipe BlazePose vs Kinect: coordinate systems, speed, noise, landmark mapping |
| [[concepts/domain-adaptation]] | Domain gap strategies; why transfer learning failed; feature engineering as the successful approach |
| [[concepts/retrieval-augmented-generation]] | RAG paradigm: limitations of basic single-pass retrieval; agentic extensions |
| [[concepts/multi-agent-systems]] | 4-agent pipeline design: Planner/Researcher/Critic/Synthesizer; specialization and orchestration |
| [[concepts/vector-databases]] | HNSW, INT8 quantization, dual-index pattern; landscape from FAISS to Endee |
| [[concepts/react-agent-pattern]] | ReAct: Reason-Act-Observe loop with typed thought steps and live UI streaming |

## Entities
| Page | Type | Summary |
|------|------|---------|
| [[entities/mediapipe-blazepose]] | tool | Google's real-time RGB pose estimator; inference-time sensor; 33 landmarks, ~38ms/frame CPU |
| [[entities/ui-prmd-dataset]] | product | Primary rehab benchmark: 2,000 sequences, 10 exercises, 10 subjects, Kinect v2, binary labels |
| [[entities/microsoft-kinect]] | product | Discontinued depth sensor; training-time sensor for UI-PRMD; 3D mm coordinates, 25 joints |
| [[entities/endee]] | tool | High-performance open-source vector DB (C++, HNSW, INT8); dual-index backbone of MindForge |
| [[entities/sentence-transformers]] | tool | all-MiniLM-L6-v2: 384-dim dense embeddings for semantic retrieval in MindForge |

## Domain Syntheses
| Page | Domain | Confidence |
|------|--------|------------|
| [[domains/research/thesis]] | research | Medium — two threads: movement analysis (sensor-agnostic features) + agentic RAG (MindForge) |

## Personal Projects
| Page | Status | Summary |
|------|--------|---------|
| [[domains/personal/projects/mindforge]] | Completed | MindForge seminar project: 4-agent RAG system, Endee dual-index, Streamlit UI |

## Outputs
| Page | Date | Query |
|------|------|-------|
| *(none yet)* | | |
