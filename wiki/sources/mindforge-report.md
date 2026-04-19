---
title: MindForge — Multi-Agent Research Intelligence System (Seminar Report)
type: source
domain: research
date_ingested: 2026-04-15
source_file: raw/research/MindForge_Report.pdf
tags: [multi-agent, rag, vector-database, agentic-ai, react-pattern, llm, endee]
---

# MindForge — Multi-Agent Research Intelligence System

## TL;DR
MindForge is a four-agent AI pipeline (Planner, Researcher, Critic, Synthesizer) that overcomes the core limitations of basic RAG — shallow single-pass retrieval, no self-correction, and no persistent memory — by combining a self-correcting orchestration loop with Endee's novel dual-index vector database architecture. Each agent implements the ReAct (Reason-Act-Observe) pattern with full thought-trace streaming to a Streamlit web UI.

## Key Points
- **Four-agent pipeline**: Planner decomposes queries into ≤4 sub-questions; Researcher performs two-phase retrieval (memory-first, then knowledge index); Critic evaluates completeness via structured JSON verdict with confidence score; Synthesizer writes a five-section structured report.
- **Dual-index Endee architecture**: `mf_knowledge` stores document chunks; `mf_memory` stores agent findings across sessions. Researcher checks memory (similarity ≥ 0.7) before querying the document index — enabling progressive, cross-session knowledge accumulation.
- **Self-correcting loop**: If Critic returns `insufficient`, the Orchestrator sends follow-up questions back to the Researcher (up to 3 iterations). Test Case 2 showed confidence rise from 0.60 to 0.75 after one re-research pass.
- **Sub-25ms Endee latency**: INT8 quantization (4× memory savings vs FP32) with HNSW indexing delivers no measurable accuracy degradation. End-to-end pipeline runs in 12–15s, dominated by Groq LLM API latency.
- **Technology stack**: Endee v0.1.22, all-MiniLM-L6-v2 (384-dim embeddings), Groq llama-3.1-8b-instant, Streamlit, Docker, Python 3.10+.

## Notable Details
- The Critic's output schema is fully structured JSON: `{verdict, confidence, gaps, follow_up_questions, critique}`. This structured interface between agents is a deliberate design choice influenced by MetaGPT — it enables principled orchestration rather than free-form conversation between agents.
- "The system knows what it doesn't know" — MindForge reports low confidence and explicit limitations instead of hallucinating, which is its strongest differentiating feature vs. basic RAG.
- Ingestion speed: ~14.87 batches/second; 512-char chunks with 64-char overlap; embeddings generated in parallel batches of 256.
- The memory index has no pruning mechanism — it grows unbounded and may accumulate stale findings over time (identified limitation, listed as future work).
- Test environment: Windows 11, 16 GB RAM, Python 3.10 (Miniconda), Endee via Docker on localhost:8080.
- Test Case 1 (standard query): SUFFICIENT at 0.90 confidence, 12s end-to-end. Test Case 2 (complex query): INSUFFICIENT at 0.60 → re-research → SUFFICIENT at 0.75.

## Connections
- [[concepts/retrieval-augmented-generation]] — the baseline paradigm MindForge extends
- [[concepts/multi-agent-systems]] — the architectural approach: Planner/Researcher/Critic/Synthesizer
- [[concepts/vector-databases]] — Endee, HNSW, INT8 quantization
- [[concepts/react-agent-pattern]] — the reasoning loop each agent implements
- [[entities/endee]] — the vector database at the core of the dual-index architecture
- [[entities/sentence-transformers]] — all-MiniLM-L6-v2 embedding model
- [[domains/personal/projects/mindforge]] — personal project tracking page

## Open Questions
- How does MindForge scale when the knowledge index grows to thousands of documents? (Only tested on small document sets.)
- Does memory-first retrieval ever degrade performance by returning stale prior findings that contradict newer information in the knowledge index?
- Can the Critic's confidence scores be calibrated against human judgements of answer completeness?
- Would a larger LLM improve Critic accuracy at the cost of pipeline latency?
