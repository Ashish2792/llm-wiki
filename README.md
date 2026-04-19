# LLM Wiki

A personal knowledge base built and maintained by an LLM agent (Claude Code).
Inspired by Andrej Karpathy's LLM-Wiki concept.

## What this is

Instead of RAG (re-retrieving from raw docs every query), this system builds
a persistent, compounding wiki — structured markdown files that get richer
with every source ingested and every question asked.

## Architecture

| Layer | Purpose |
|---|---|
| `raw/` | Immutable source documents — never edited |
| `wiki/` | LLM-generated pages: concepts, entities, sources |
| `domains/` | Synthesis pages per domain |
| `outputs/` | Saved query answers and analyses |
| `CLAUDE.md` | Operating schema — the LLM reads this first every session |

## Three domains

- **Research** — papers, articles, evolving thesis
- **Personal** — goals, habits, self-model
- **Business** — projects, ideas, market notes

## Operations

- `INGEST <file>` — process a new source into the wiki
- `QUERY <question>` — synthesize an answer with citations
- `LINT` — health check: orphans, contradictions, stale pages

## Multi-agent INGEST pipeline (in progress)

Extending CLAUDE.md with a 6-agent pipeline to prevent hallucination propagation:

`Reader → Critic → Writer → Linker → Verifier → Lint Agent`

## Stack

- [Obsidian](https://obsidian.md) — wiki viewer and editor
- [Claude Code](https://claude.ai/code) — LLM agent that maintains the wiki
- Git — version history of the entire knowledge base