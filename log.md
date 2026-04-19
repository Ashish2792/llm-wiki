# Wiki Activity Log

---

## [2026-04-14] ingest | AI-Powered Rehabilitation Exercise Assessment System (capstone_report.pdf)

**Operation**: ingest
**Domain**: research
**Summary**: First-ever session. Ingested the B.Tech capstone project report by Ashish (April 2026) on AI-powered rehabilitation exercise assessment. The report describes a four-stage pipeline: MediaPipe BlazePose pose estimation → sensor-agnostic joint angle feature extraction (9 channels × 22 joints) → EfficientGCN-B0 classifier (~690K params) → binary correct/incorrect prediction at ~15 fps on CPU. User requested emphasis on GCN architecture, so the graph-convolutional-networks.md and efficientgcn-architecture.md concept pages received deep technical coverage including mathematical formulations, layer-by-layer architecture tables, and inference efficiency comparisons. Key findings captured: 96.3% accuracy via sensor-agnostic features (vs catastrophic failure with raw positions), transfer learning failure on REHAB24-6 (63.8% vs 72% scratch), and data-is-the-bottleneck finding. Created all wiki pages from scratch as this was the initial wiki population.
**Pages touched**:
- wiki/sources/capstone-rehab-exercise-assessment.md (CREATED)
- wiki/concepts/graph-convolutional-networks.md (CREATED — emphasized per user request)
- wiki/concepts/efficientgcn-architecture.md (CREATED — emphasized per user request)
- wiki/concepts/sensor-agnostic-features.md (CREATED)
- wiki/concepts/skeleton-based-action-recognition.md (CREATED)
- wiki/concepts/pose-estimation.md (CREATED)
- wiki/concepts/domain-adaptation.md (CREATED)
- wiki/entities/mediapipe-blazepose.md (CREATED)
- wiki/entities/ui-prmd-dataset.md (CREATED)
- wiki/entities/microsoft-kinect.md (CREATED)
- domains/research/thesis.md (CREATED)
- wiki/overview.md (CREATED)
- index.md (CREATED)
- log.md (CREATED)
**Open items**:
- True subject-independent (leave-subjects-out) accuracy not yet evaluated — current 96.3% may include subject leakage
- Transfer learning with InstanceNorm/LayerNorm not tested — open architectural fix for cross-dataset adaptation
- KIMORE dataset not yet ingested — would enrich the rehabilitation benchmark comparison
- Out-of-plane exercise limitation (shoulder internal-external rotation) not yet explored empirically

---

## [2026-04-15] ingest | MindForge — Multi-Agent Research Intelligence System (MindForge_Report.pdf)

**Operation**: ingest
**Domain**: research (+ personal project tracking)
**Summary**: Ingested Ashish's B.Tech seminar report (LPU CSE, Jan–April 2026) on MindForge, a 4-agent agentic RAG system. The system uses a Planner→Researcher→Critic→Synthesizer pipeline with a self-correcting re-research loop, backed by Endee's dual-index vector architecture (mf_knowledge for documents, mf_memory for agent findings). Each agent implements ReAct with live thought streaming to Streamlit. Empirical results: Test Case 1 — SUFFICIENT at 0.90 confidence; Test Case 2 — INSUFFICIENT at 0.60 → re-research → SUFFICIENT at 0.75. All new concept pages created from scratch (no overlap with prior rehab research). A new `domains/personal/projects/` folder was created per user request to track Ashish's own projects separately from the research synthesis. The research thesis was broadened to cover both threads and a cross-cutting insight was added: both projects demonstrate that architecture/representation design matters more than raw model capability.
**Pages touched**:
- wiki/sources/mindforge-report.md (CREATED)
- wiki/concepts/retrieval-augmented-generation.md (CREATED)
- wiki/concepts/multi-agent-systems.md (CREATED)
- wiki/concepts/vector-databases.md (CREATED)
- wiki/concepts/react-agent-pattern.md (CREATED)
- wiki/entities/endee.md (CREATED)
- wiki/entities/sentence-transformers.md (CREATED)
- domains/personal/projects/mindforge.md (CREATED — new folder structure)
- domains/research/thesis.md (UPDATED — second thread added, cross-cutting thesis added)
- wiki/overview.md (UPDATED — second research thread added)
- index.md (UPDATED — 10 new pages, personal projects section added)
- log.md (APPENDED)
**Open items**:
- MindForge only tested on 14-chunk corpus — scalability to large document collections unvalidated
- No RAGAS evaluation run yet — relative improvement over basic RAG is qualitative, not quantitative
- Memory index pruning unimplemented — stale finding accumulation risk not yet observed but theoretically present
- capstone project not yet added to domains/personal/projects/ — consider ingesting again with project tracking focus
