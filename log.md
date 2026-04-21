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

---

## [2026-04-21] update | capstone_report.pdf — continuation of 2026-04-14 ingest

**Operation**: update
**Domain**: research
**Summary**: Continued and completed the capstone ingestion that was partially finished in a prior session (pages dated 2026-04-20). The source page and four concept pages (sensor-agnostic-features, efficientgcn-architecture, domain-adaptation, skeleton-based-action-recognition) had already been substantially rewritten with full claim-level citation protocol. This session created the three missing entity pages referenced in those pages but not yet written, created the personal projects tracking page for the capstone, and updated index.md with the new pages and dates.
**Pages touched**:
- wiki/entities/kimore-dataset.md (CREATED)
- wiki/entities/rehab24-6-dataset.md (CREATED)
- wiki/entities/wlu-dataset.md (CREATED)
- domains/personal/projects/capstone.md (CREATED — closes open item from 2026-04-15 log)
- index.md (UPDATED — 22→27 pages, new entities and project listed)
**Claims added**: ~25 total (10 direct, 8 paraphrase, 5 inferred, 2 uncertain) across new entity pages
**Open items**:
- True subject-independent (leave-subjects-out) accuracy not yet evaluated — current 96.3% may include subject leakage
- Transfer learning with InstanceNorm/LayerNorm not tested — open architectural fix for cross-dataset adaptation
- WLU dataset provenance (institution, subjects, exercises) not documented in the source — detail unknown
- KIMORE dataset: why it wasn't used in training is unexplained in the source

---

## [2026-04-21] lint | Full wiki health check

**Operation**: lint
**Domain**: all
**Summary**: First full LINT pass of the wiki. Read all 27 pages. Zero broken wiki-links. Zero broken anchors on the capstone source page. Nine post-protocol pages (2026-04-20+) are clean. The dominant finding is that 13 pre-protocol pages (created before the §2 citation protocol was adopted) have untagged factual claims and unanchored citations throughout. The mindforge-report source page is the structural blocker — it uses non-standard section headings, which prevents all MindForge concept/entity pages from adding anchored citations.
**Pages touched**:
- outputs/lint-2026-04-21.md (CREATED)
- index.md (UPDATED — output added, page count 27→28)
**Open items**:
- H1: Rewrite wiki/sources/mindforge-report.md to §3.1 format (unblocks all MindForge anchor citations)
- H2: Migrate wiki/concepts/graph-convolutional-networks.md (pre-protocol, entirely untagged)
- H3: Migrate wiki/concepts/pose-estimation.md (pre-protocol, significantly untagged)
- H4: Migrate 4× MindForge concept pages after H1
- H5: Migrate 3× pre-protocol entity pages (mediapipe, kinect, ui-prmd)
- M3: Add capstone project link to overview.md and thesis.md

---

## [2026-04-21] update | Migration batch — mindforge-report, graph-convolutional-networks, overview, thesis

**Operation**: update
**Domain**: research
**Summary**: Three-part migration batch per §10. Task 1: Rewrote wiki/sources/mindforge-report.md from scratch — the pre-protocol version had non-standard headings (TL;DR, Key Points, Notable Details) preventing any anchored citations into it; replaced with the full §3.1 format (TL;DR, Background, Methodology, Results, Discussion, Limitations, Notable Quotes, Connections, Open Questions), added missing `last_updated` frontmatter, and tagged all ~70 claims. This unblocks M1 from the lint report: 20+ unanchored `[[sources/mindforge-report]]` citations across six concept and entity pages can now be anchored. Task 2: Migrated wiki/concepts/graph-convolutional-networks.md from pre-protocol (entirely untagged) to §2-compliant format. Tagged all claims against capstone-rehab-exercise-assessment; three detailed architecture descriptions (2s-AGCN internals, MS-G3D internals, CTR-GCN internals) that go beyond the capstone source were tagged {uncertain} with source-gap notes rather than silently stripped. Task 3: Added `[[domains/personal/projects/capstone]]` links to wiki/overview.md (Personal Projects section) and domains/research/thesis.md (Thread 1 section inline reference); bumped both files' `last_updated` to 2026-04-21. Closes lint items H1, H2, M3, and partially resolves L1.
**Pages touched**:
- wiki/sources/mindforge-report.md (REWRITTEN — §3.1 format, all claims tagged)
- wiki/concepts/graph-convolutional-networks.md (MIGRATED — full tag+anchor pass)
- wiki/overview.md (UPDATED — capstone project link added, last_updated bumped)
- domains/research/thesis.md (UPDATED — capstone project link added, last_updated bumped)
**Claims added**: ~95 total across all pages
  - wiki/sources/mindforge-report.md: ~70 claims (38 direct, 24 paraphrase, 8 inferred)
  - wiki/concepts/graph-convolutional-networks.md: ~25 claims (8 direct, 9 paraphrase, 4 inferred, 4 uncertain)
**Open items**:
- Remaining H3: wiki/concepts/pose-estimation.md still pre-protocol
- Remaining H4: four MindForge concept pages (RAG, multi-agent, vector-databases, react-agent-pattern) still untagged but now unblocked after H1
- Remaining H5: mediapipe, kinect, ui-prmd entity pages still pre-protocol
- Remaining L3: endee, sentence-transformers entity pages still pre-protocol
