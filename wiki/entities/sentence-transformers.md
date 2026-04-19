---
title: sentence-transformers / all-MiniLM-L6-v2
type: entity
entity_type: tool
tags: [embeddings, sentence-bert, semantic-search, nlp, dense-retrieval]
last_updated: 2026-04-15
---

# sentence-transformers / all-MiniLM-L6-v2

## Overview
sentence-transformers is a Python library that produces semantically meaningful fixed-length sentence embeddings via BERT-based siamese/triplet network fine-tuning (Reimers & Gurevych, EMNLP 2019). The `all-MiniLM-L6-v2` model used in MindForge is a distilled derivative of Sentence-BERT — it produces 384-dimensional embeddings with strong semantic quality at minimal computational cost.

## Relevance to This Wiki
all-MiniLM-L6-v2 is the embedding model for both Endee indexes in MindForge. It converts every document chunk and every agent query into a 384-dim vector that powers all semantic retrieval in the pipeline. The quality of retrieval is directly contingent on the quality of these embeddings.

## Key Facts
- Output dimensionality: 384
- Normalization: L2-normalized (enables cosine similarity as dot product, compatible with INT8-quantized Endee indexes)
- Batch size in MindForge ingestion: 256 chunks per batch
- Language support: English-only (identified limitation — multilingual corpora require a multilingual embedding model)
- Original paper: Reimers & Gurevych (EMNLP 2019) — BERT fine-tuned with siamese/triplet networks for sentence similarity
- Key advantage over vanilla BERT: fixed-length embeddings at linear inference cost vs BERT's quadratic cost for pairwise sentence comparison

## Appearances
- [[sources/mindforge-report]] — embedding model for MindForge's dual Endee indexes

## Connections
- [[entities/endee]] — the vector database that stores all-MiniLM-L6-v2 embeddings
- [[concepts/vector-databases]] — the infrastructure that makes dense embedding search practical at scale
- [[concepts/retrieval-augmented-generation]] — the application pattern that depends on dense embeddings for semantic retrieval
