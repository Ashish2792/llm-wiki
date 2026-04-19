---
title: Endee
type: entity
entity_type: tool
tags: [vector-database, hnsw, int8-quantization, open-source, c++]
last_updated: 2026-04-15
---

# Endee

## Overview
Endee is a high-performance, open-source vector database written in C++ with SIMD optimizations (AVX2, AVX512, NEON, SVE2). Released in 2025, it is designed to handle up to 1 billion vectors on a single node with sub-millisecond query latency, using HNSW indexing and INT8 quantization. It exposes a Python SDK and a REST API (default: `localhost:8080/api/v1`), and is deployable via Docker.

## Relevance to This Wiki
Endee is the core infrastructure component of MindForge — it powers both the document knowledge index (`mf_knowledge`) and the agent memory index (`mf_memory`). [[sources/mindforge-report]] is the primary empirical source for Endee's performance characteristics in a real AI pipeline at small scale (14-chunk test corpus).

## Key Facts
- Written in C++ with SIMD optimizations (AVX2, AVX512, NEON, SVE2)
- Supports up to 1 billion vectors on a single node
- HNSW indexing: O(log n) query complexity with high recall
- INT8 quantization: 4× memory savings vs FP32, no measurable accuracy loss at 384-dim cosine similarity
- Python SDK: `from endee import Endee, Precision`
- Docker image: `endeeio/endee-server:latest`
- Version tested: v0.1.22 (in [[sources/mindforge-report]])
- Observed latencies (small-scale, from [[sources/mindforge-report]]): index creation <100ms, top-5 query 15–25ms, top-3 query 10–20ms, batch upsert (14 chunks) ~67ms
- Index is persistent across server restarts

## Appearances
- [[sources/mindforge-report]] — dual-index architecture (mf_knowledge for document chunks, mf_memory for agent findings)

## Connections
- [[concepts/vector-databases]] — Endee in the broader vector DB landscape
- [[concepts/retrieval-augmented-generation]] — the application paradigm Endee enables
- [[entities/sentence-transformers]] — produces the 384-dim embeddings stored in Endee indexes
