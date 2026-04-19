---
title: MindForge — Personal Project
type: synthesis
domain: personal
last_updated: 2026-04-15
sources_count: 1
---

# MindForge — Multi-Agent Research Intelligence System

## Current Understanding
MindForge is a B.Tech seminar project (LPU CSE, Jan–April 2026) — a fully implemented, production-ready multi-agent research intelligence system built to demonstrate the architectural advantages of agentic AI over basic RAG for complex document question-answering.

The system is complete and tested end-to-end. It includes a Streamlit web UI with real-time agent thought streaming, a CLI, Docker-based Endee deployment, and has been empirically evaluated on two test cases (standard query → 0.90 confidence; complex query → re-research loop → 0.75 confidence). It is publicly hosted on GitHub as a fork of the Endee repository.

**Status**: Completed (seminar submission, Jan–April 2026)
**Repository**: github.com/Ashish2792/endee/tree/master/mindforge
**LinkedIn**: linkedin.com/in/ashish2792

## Evidence Base
- [[sources/mindforge-report]] — the full 28-page seminar report

## Key Decisions and Rationale
- **Why Endee**: high-performance open-source vector DB chosen to demonstrate the novel dual-index architecture; submission required forking the Endee repo
- **Why dual-index**: the central architectural contribution — separating document knowledge from agent memory enables cross-session knowledge accumulation without additional infrastructure
- **Why Groq/llama-3.1-8b-instant**: fast inference API, no GPU required locally — consistent with the "consumer hardware" theme across Ashish's B.Tech projects
- **Why four agents**: AutoGen research shows specialization outperforms generalist agents; Planner+Researcher+Critic+Synthesizer maps cleanly to the Plan→Research→Evaluate→Report workflow
- **Why structured JSON between agents**: MetaGPT-influenced design — structured inter-agent communication enables programmatic orchestration (conditional loops, routing) vs free-form text

## Confidence Level
**High** — System is implemented, tested, and submitted. Results are empirical from actual pipeline runs, not theoretical claims.

## Next Steps (from Future Scope section)
1. Multi-document cross-referencing (comparative literature analysis across ingested docs)
2. Automated memory pruning (similarity-based deduplication in `mf_memory`)
3. Streaming report generation (Groq streaming API for token-by-token output)
4. Domain-specific agent personas (legal, medical, financial system prompts)
5. Hybrid search (BM25 + dense vectors for better recall on specific terminology)
6. RAGAS evaluation framework (faithfulness, answer relevancy, context precision/recall)
7. Cloud deployment (Endee Cloud + AWS ECS / Google Cloud Run, multi-tenant)
