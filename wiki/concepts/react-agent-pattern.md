---
title: ReAct Agent Pattern
type: concept
tags: [react, agentic-ai, reasoning, llm, chain-of-thought, transparency]
sources: [mindforge-report]
last_updated: 2026-04-15
---

# ReAct Agent Pattern

## Definition
ReAct (Reasoning + Acting) is an agent execution pattern proposed by Yao et al. (ICLR 2023) that interleaves verbal reasoning traces with tool-use actions in LLM-based agents. Each step is typed: **Reason** (think about what to do next) → **Act** (execute a tool call or query) → **Observe** (record the result) → repeat until a **Final** output.

## Why It Matters
ReAct combines chain-of-thought reasoning (explicit intermediate thinking) with action-taking agents (grounded retrieval or tool execution). Unlike CoT-only approaches (which reason but don't act) or action-only agents (which act without reasoning), ReAct enables dynamic, grounded reasoning where the agent adjusts its plan based on what it observes. This makes reasoning transparent, auditable, and correctable.

## Key Properties / Dimensions

**Typed thought steps** (from [[sources/mindforge-report]] implementation):
```python
@dataclass
class Thought:
    agent: str   # "Planner" | "Researcher" | "Critic" | "Synthesizer"
    type: str    # "reason" | "act" | "observe" | "final"
    content: str
```

**Execution cycle:**
1. `reason` — record why an action is needed
2. `act` — execute LLM call or Endee vector query
3. `observe` — record the result of the action
4. `final` — record the agent's terminal output for this step

**UI streaming**: [[sources/mindforge-report]] fires a callback on every Thought object, streaming it to Streamlit in real time. This creates full transparency into what each agent is doing and why — a key trust-building feature.

**vs. alternatives:**
- Chain-of-Thought only (Wei et al. 2022): generates reasoning but takes no actions; cannot ground conclusions in retrieved facts
- Action-only agents: act but don't reason; behavior is opaque and harder to debug
- ReAct: combines both; each action is preceded by explicit reasoning and followed by explicit observation

## Evidence & Examples
[[sources/mindforge-report]] Researcher Agent ReAct loop example:
- reason: "Need to check memory index for prior findings on this sub-question"
- act: query `mf_memory`, top-3, similarity ≥ 0.7
- observe: "No prior findings (first session)"
- act: query `mf_knowledge`, top-5
- observe: "Retrieved 5 passages, top similarity 0.7124"
- act: LLM synthesis call to generate cited finding
- final: finding stored to memory index

## Contradictions & Open Questions
- ReAct's transparency advantage assumes the LLM faithfully records its actual reasoning. If reasoning traces are post-hoc rationalization rather than genuine process traces, the thought log is misleading.
- The number of ReAct cycles per agent is implicitly bounded by max_tokens — complex tasks may need deeper loops than prompt design allows.

## Related
- [[concepts/multi-agent-systems]] — ReAct is the intra-agent pattern; multi-agent is the inter-agent architecture
- [[concepts/retrieval-augmented-generation]] — retrieval is the primary "Act" in MindForge's Researcher Agent
- [[entities/endee]] — the tool being called in most ReAct "act" steps in MindForge
