---
title: Retrieval-Augmented Generation (RAG)
type: concept
tags: [rag, llm, vector-search, document-qa, semantic-retrieval]
sources: [mindforge-report]
last_updated: 2026-04-15
---

# Retrieval-Augmented Generation (RAG)

## Definition
RAG is a framework that grounds large language model outputs in externally retrieved content. Rather than relying purely on parametric knowledge baked into model weights, a RAG system embeds a user query into vector space, retrieves the most semantically similar document passages from a vector index, and provides those passages as context to the LLM for answer generation.

## Why It Matters
RAG directly addresses LLM hallucination by constraining generation to retrieved evidence. It also allows knowledge to be updated by modifying the document index rather than retraining the model. It is the dominant paradigm for enterprise document question-answering. Introduced by Lewis et al. (NeurIPS 2020).

## Key Properties / Dimensions

**Basic RAG pipeline:**
1. Embed user query → retrieve top-K passages from vector index → generate answer conditioned on passages

**Limitations of first-generation RAG (from [[sources/mindforge-report]]):**
- **Single-pass retrieval**: one retrieval pass cannot handle multi-hop questions where answering one aspect depends on first understanding another
- **No self-evaluation**: no mechanism to detect whether retrieved content adequately covers the question
- **No persistent memory**: every session starts from scratch, repeating the same retrievals
- **Opacity**: black-box reasoning with no visibility into why an answer was generated
- **Short, uncited outputs**: a single LLM call produces a short answer without structured sections or explicit source attribution

**Advanced extensions:**
- Fusion-in-Decoder (FiD, Izacard & Grave 2021): processes multiple retrieved passages jointly at generation time
- Agentic RAG (MindForge): decomposes query before retrieval; adds a Critic agent for quality evaluation; stores agent findings in a separate memory index for cross-session continuity

## Evidence & Examples
[[sources/mindforge-report]] benchmarks MindForge (agentic RAG) vs. basic RAG across ten features — architecture, reasoning depth, self-evaluation, gap correction, persistent memory, vector index count, transparency, output structure, confidence signal, and query decomposition. MindForge outperforms on all dimensions for complex multi-faceted questions.

Shi et al. (2023) showed that more retrieved passages can hurt performance by introducing noise — quality-aware filtering (like MindForge's Critic Agent) is more important than simply retrieving more.

## Contradictions & Open Questions
- The cost of agentic RAG is latency: 12–15s for MindForge vs. near-instant for basic RAG. For simple questions, basic RAG remains the right tool.
- Structured Critic evaluation requires the LLM to produce well-formed JSON — malformed outputs can break the pipeline.

## Related
- [[concepts/vector-databases]] — the infrastructure underlying all retrieval
- [[concepts/multi-agent-systems]] — the architectural extension MindForge applies over RAG
- [[concepts/react-agent-pattern]] — the per-agent reasoning loop
- [[entities/endee]] — the vector database used in [[sources/mindforge-report]]
