# LLM Wiki

A personal knowledge base built and maintained by an LLM agent.  
Inspired by [Andrej Karpathy's LLM-Wiki concept](https://github.com/karpathy/llm-wiki).

> Instead of re-deriving answers from raw documents every query (RAG), this system builds a **persistent, compounding wiki** — structured markdown files that grow richer with every source ingested and every question asked.

---

## What This Is

Most RAG systems rediscover knowledge from scratch on every question. This system works differently: when you add a new source, the LLM doesn't just index it — it reads it, extracts knowledge, and integrates it into the existing wiki. Concept pages get updated. Contradictions get flagged. The synthesis evolves. The wiki compounds.

The human's job: curate sources, ask the right questions, direct the analysis.  
The LLM's job: summarize, cross-reference, file, maintain, and keep everything consistent.

---

## Architecture

Three layers:

| Layer | Purpose |
|---|---|
| `raw/` | Immutable source documents — the LLM reads, never writes |
| `wiki/` + `domains/` | LLM-generated pages: concepts, entities, sources, syntheses |
| `CLAUDE.md` | Operating schema — the LLM reads this first every session |

Two special files:

- **`index.md`** — master catalog of every wiki page with claim counts and evidence breakdown
- **`log.md`** — append-only activity log of every ingest, query, and lint pass

---

## Directory Structure

```
wiki/
├── CLAUDE.md                  ← schema & protocols (read first, every session)
├── index.md                   ← master page catalog
├── log.md                     ← append-only activity log
│
├── raw/                       ← immutable sources
│   ├── research/
│   ├── business/
│   └── assets/
│
├── wiki/
│   ├── overview.md            ← high-level synthesis, entry point
│   ├── concepts/              ← one page per concept
│   ├── entities/              ← people, orgs, tools, books
│   └── sources/               ← one summary page per raw source
│
├── domains/
│   ├── research/              ← evolving thesis + synthesis pages
│   └── business/              ← projects, ideas, market notes
│
└── outputs/                   ← saved query answers, analyses, comparisons
```

---

## Three Operations

### `INGEST <filename>`
Process a new source into the wiki.

1. Read `index.md` to understand current wiki state
2. Read the source from `raw/`
3. Discuss key takeaways with user
4. Create source summary page at `wiki/sources/<slug>.md`
5. Create or update concept and entity pages
6. Update domain synthesis pages
7. Update `wiki/overview.md` if the big picture shifts
8. Update `index.md` and append to `log.md`
9. Run pre-commit claim check — nothing commits without passing

A single ingest typically touches 5–15 pages.

### `QUERY <question>`
Synthesize an answer from the wiki with full citations.

- Reads `index.md` → retrieves relevant pages → synthesizes with citations
- Answer confidence cannot exceed evidence confidence
- Offers to save the answer as an output page so explorations compound

### `LINT`
Periodic health check of the wiki.

Checks for: orphan pages, contradictions, stale content, missing pages, broken links, untagged claims, unanchored citations, evidence-thin pages. Produces a prioritized fix report saved to `outputs/lint-YYYY-MM-DD.md`.

---

## Claim-Level Citation Protocol

Every factual claim on every wiki page carries a citation tag and an anchor pointing to its source:

```
[[sources/slug#Section]] {tag}
```

**Five tags:**

| Tag | Meaning |
|---|---|
| `{direct}` | Claim is directly stated in the source |
| `{paraphrase}` | Claim restates the source in different words |
| `{inferred}` | Claim follows logically from source but isn't stated |
| `{uncertain}` | Claim is weakly supported or contested |
| `{synthesis}` | Claim integrates multiple sources |

**The rule: cite-or-don't-write.** If a claim can't be anchored to a source, it doesn't go in the wiki. This is enforced by a pre-commit claim check on every ingest.

This protocol directly addresses the core weakness of LLM-maintained wikis: confident-sounding unsupported claims. Every assertion is grounded, every confidence level is visible, and the Critic Agent (see below) audits compliance before anything is written.

---

## Multi-Agent INGEST Pipeline *(in progress)*

Motivated by the observation that a single LLM doing everything in one pass — reading, summarizing, cross-referencing, writing, verifying — is where errors enter and compound. The fix is separating concerns and adding checks between steps.

```
Raw Source
    ↓
Reader Agent    — extract raw claims with exact source references
    ↓
Critic Agent    — flag contradictions with existing wiki + unsupported claims
    ↓  (human reviews flags if needed)
Writer Agent    — write/update pages; cite-or-don't-write rule enforced
    ↓
Linker Agent    — manage graph, detect duplicates, update cross-references
    ↓
Verifier Agent  — QA gate; nothing persists without passing
    ↓
Wiki Updated ✓
```

**What each agent solves:**

| Problem | Agent |
|---|---|
| Mistakes compound across pages | Critic blocks them at entry |
| Hallucinated connections | Critic + Writer (citation required) |
| Traceability breaks down | Writer (can't write without citation anchor) |
| Duplicate pages at scale | Linker detects before Writer creates |
| Cascading update chaos | Linker owns the graph exclusively |
| Silent quality drift | Verifier as final QA gate |

A separate **Lint Agent** runs periodically (not every ingest) to audit the full wiki for accumulated issues.

---

## Page Types

Every wiki page has YAML frontmatter with `title`, `type`, and `last_updated`.

| Type | Location | Purpose |
|---|---|---|
| `source` | `wiki/sources/` | Summary of one raw source, fixed section headings for citation anchors |
| `concept` | `wiki/concepts/` | Definition, evidence, contradictions, related pages |
| `entity` | `wiki/entities/` | Person, org, tool, or book — key facts + appearances |
| `synthesis` | `domains/` | Evolving understanding of a topic across sources |
| `output` | `outputs/` | Saved query answer with confidence summary |

Source pages use fixed section headings (`## TL;DR`, `## Background`, `## Methodology`, `## Results`, `## Discussion`, `## Limitations`) so citation anchors are always predictable across the whole wiki.

---

## Why This Works

The tedious part of maintaining a knowledge base isn't reading or thinking — it's bookkeeping. Updating cross-references, flagging contradictions, keeping summaries current. Humans abandon wikis because maintenance grows faster than value.

LLMs don't get bored, don't forget to update a cross-reference, and can touch 15 files in one pass. The wiki stays maintained because the cost of maintenance is near zero.

The claim-level citation protocol and multi-agent pipeline address the main criticism of this approach (from [Mehul Gupta's analysis](https://medium.com/)): that errors propagate silently and connections get hallucinated. The Critic Agent blocks errors at entry. The Writer can't write without a citation. The Verifier ensures nothing slips through.

---

## Comparison with RAG

| | RAG | This Wiki |
|---|---|---|
| Knowledge compilation | Every query | Once at ingest |
| Cross-references | Re-derived each time | Already there |
| Contradiction detection | None | Critic Agent |
| Navigability | Search only | Linked pages + graph view |
| Infrastructure needed | Vector DB | Just markdown files |
| Scales to | Millions of docs | Hundreds of sources |

Best for: research going deep over weeks/months, personal knowledge accumulation, project tracking where synthesis matters more than raw recall.

---

## Stack

- [Obsidian](https://obsidian.md) — wiki viewer, editor, and graph view
- [Claude Code](https://claude.ai/code) — LLM agent that maintains the wiki
- Git — version history of the entire knowledge base

---

## Setup

1. Clone the repo and open the root folder as an Obsidian vault
2. Read `CLAUDE.md` — this is the full operating schema
3. Drop a source file into `raw/research/` or `raw/business/`
4. Open Claude Code pointed at the wiki root
5. Type `INGEST <filename>`

The wiki builds itself from there.

---

## Domains

This instance covers two public domains:

- **Research** — papers, articles, deep learning topics. Maintains an evolving `domains/research/thesis.md`.
- **Business** — projects, ideas, market notes. Tracks active projects in `domains/business/projects.md`.

Personal domain is maintained in a separate private vault using the same `CLAUDE.md` schema.

---

## Roadmap

- [x] Foundation — directory structure + `CLAUDE.md` schema
- [x] First INGEST verified in Obsidian
- [x] Claim-level citation protocol (`{direct}`, `{paraphrase}`, `{inferred}`, `{uncertain}`, `{synthesis}`)
- [x] Pre-commit claim check gate
- [ ] Multi-agent INGEST pipeline (Reader → Critic → Writer → Linker → Verifier)
- [ ] QUERY operation with confidence propagation
- [ ] First LINT pass
- [ ] CLI search with qmd for scale

---
