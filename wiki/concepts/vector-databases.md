---
title: Vector Databases
type: concept
tags: [vector-database, semantic-search, embeddings, hnsw, ann, quantization]
sources: [mindforge-report]
last_updated: 2026-04-15
---

# Vector Databases

## Definition
Vector databases store high-dimensional numerical vectors (embeddings) alongside metadata, and provide efficient approximate nearest-neighbor (ANN) search to retrieve the vectors most similar to a query vector. They are the infrastructure backbone of RAG systems and semantic search applications.

## Why It Matters
Traditional keyword search engines match lexically — they cannot retrieve conceptually related content expressed in different words. Vector databases solve this by operating in a semantic embedding space where meaning, not surface form, determines similarity. Combined with dense embedding models, they enable retrieval of relevant documents regardless of exact phrasing.

## Key Properties / Dimensions

**Core operations**: upsert (id, vector, metadata) → query (vector, top_k) → returns ranked results with similarity scores and associated metadata

**HNSW indexing** (Malkov & Yashunin, 2018): the dominant ANN algorithm. Builds a multilayer graph — upper layers enable fast long-distance traversal, lower layers enable precise local search. Achieves O(log n) query complexity with high recall.

**INT8 quantization tradeoff:**
- FP32: full precision, 4× memory cost
- INT8: 4× memory compression. [[sources/mindforge-report]] observed no measurable accuracy degradation vs FP32 for 384-dim cosine similarity search — INT8 is validated for production RAG applications at this embedding dimensionality.

**Similarity metrics**: cosine similarity is standard for normalized embeddings (reduces to dot product after L2 normalization, enabling hardware-optimized SIMD computation).

**Landscape (2019–2025):**
| System | Year | Notes |
|--------|------|-------|
| FAISS | 2019 | Library only; needs significant engineering for production |
| Pinecone | 2019 | Managed cloud, proprietary, expensive at scale |
| Weaviate | 2021 | Open-source, GraphQL API, built-in vectorization |
| Qdrant | 2021 | Open-source, Rust, production-reliability focus |
| Chroma | 2022 | Lightweight, developer-friendly, local/embedded use |
| Endee | 2025 | C++ + SIMD (AVX2/AVX512/NEON/SVE2), 1B vectors/node |

**Dual-index pattern** (novel in [[sources/mindforge-report]]): using two separate indexes — one for document knowledge, one for agent memory — enables independent optimization of each retrieval task and separates persistent document content from session-generated agent findings.

## Evidence & Examples
[[sources/mindforge-report]] measured Endee performance: top-5 knowledge query at 15–25ms; top-3 memory query at 10–20ms; full ingestion of 14 chunks ~1.5s (dominated by embedding computation, not DB writes at ~67ms for 14 chunks).

## Contradictions & Open Questions
- No independent benchmarks comparing Endee against Qdrant or Weaviate for equivalent workloads — Endee's performance claims in [[sources/mindforge-report]] are self-reported on a small test scale (14 chunks).
- Unbounded memory index growth (identified limitation in [[sources/mindforge-report]]): without pruning, old/stale findings accumulate. Automated similarity-based deduplication is listed as future work.
- The right chunk size and overlap (512 chars, 64 chars overlap in MindForge) are empirical — no principled method for determining these parameters per document type.

## Related
- [[concepts/retrieval-augmented-generation]] — the application layer that vector DBs enable
- [[entities/endee]] — the specific vector database used in MindForge
- [[entities/sentence-transformers]] — the embedding model generating vectors stored in these DBs
