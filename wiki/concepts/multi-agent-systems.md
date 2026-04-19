---
title: Multi-Agent Systems
type: concept
tags: [multi-agent, agentic-ai, llm, orchestration, specialization]
sources: [mindforge-report]
last_updated: 2026-04-15
---

# Multi-Agent Systems

## Definition
Multi-agent AI systems are architectures where multiple specialized LLM-powered agents — each with its own system prompt, tool access, and role — collaborate through structured interfaces to solve tasks that exceed what a single generalist LLM call can reliably accomplish.

## Why It Matters
Agent specialization consistently outperforms single-agent approaches (Wu et al. 2023, AutoGen). Assigning distinct roles (Planner, Researcher, Critic, Synthesizer) allows each agent to be optimized for its specific function. Structured outputs between agents enable principled, programmable orchestration — routing, loops, conditional branching — rather than informal free-form conversation.

## Key Properties / Dimensions

**Design principles (from [[sources/mindforge-report]]):**
- **Role specialization**: each agent has a distinct system prompt enforcing its function
- **Structured inter-agent communication**: agents exchange typed outputs (JSON verdicts, typed findings) rather than free-form text — prevents ambiguity and enables programmatic orchestration (influenced by MetaGPT, Hong et al. 2023)
- **Orchestrator pattern**: a central coordinator wires agents together and manages control flow
- **Modular independence**: each agent is an independent class; any can be upgraded or replaced without modifying others

**MindForge's four agents:**
- **Planner**: receives user question → LLM call → JSON array of ≤4 non-overlapping sub-questions ordered by conceptual dependency
- **Researcher**: two-phase retrieval (memory-first at ≥0.7 threshold, then knowledge index top-5) → synthesizes cited finding → stores finding in memory index
- **Critic**: receives all findings → produces structured JSON: `{verdict, confidence, gaps, follow_up_questions, critique}`
- **Synthesizer**: combines verified findings + Critic evaluation → five-section structured report

**Re-research loop**: if Critic returns `insufficient`, Orchestrator sends follow-up questions back to Researcher (max 3 iterations by default). This is the key differentiator from both basic RAG and simpler sequential multi-agent pipelines.

## Evidence & Examples
[[sources/mindforge-report]] Test Case 2: initial Critic verdict INSUFFICIENT (0.60 confidence), gaps identified in performance benchmark data. After one re-research iteration: SUFFICIENT (0.75 confidence). The system correctly identified what it didn't know and filled the gap.

Related frameworks: AutoGen (Wu et al. 2023) — multi-agent conversation; MetaGPT (Hong et al. 2023) — role-based software development agents; Generative Agents (Park et al. 2023) — LLM agents with memory, planning, reflection.

## Contradictions & Open Questions
- Agentic pipelines are brittle to LLM output failures: malformed JSON from the Critic can propagate through the pipeline. Fallback parsing is a mitigation but not a guarantee.
- More agents = more latency. MindForge's 12–15s is acceptable for research queries but not for conversational UX.
- Optimal number of agents and loop iteration limits are empirical design choices without formal grounding.
- It is unclear whether the Critic agent provides more objective evaluation than self-refine within a single agent (Self-Refine, Madaan et al. 2023) — this is an open empirical question.

## Related
- [[concepts/retrieval-augmented-generation]] — the foundational paradigm multi-agent architectures extend
- [[concepts/react-agent-pattern]] — the reasoning pattern each individual agent implements
- [[entities/endee]] — the storage layer that enables persistent agent memory in MindForge
