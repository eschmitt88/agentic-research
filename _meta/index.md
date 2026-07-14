---
name: index
description: Entry-point index for this project's knowledge graph.
---

# Index

Orientation for the project knowledge graph. Updated by `/wrap`, `/ingest`,
and `/new-experiment`.

## Maps of Content

- [[mocs/knowledge-organization-for-research-agents]] — substrate /
  write-side / read-side of agent memory; 11 concepts spanning
  agent-native-memory, file-as-bus, structured-world-model,
  skill-library-lifecycle, typed-claim-partition, citation-anchoring,
  selective-memory-retrieval, context-eviction-policy,
  multi-granularity-memory, verified-memory-writes,
  web-grounded-literature.
- [[mocs/agent-architecture]] — how a long-horizon autonomous agent is
  built (substrate / orchestration / execution).
- [[mocs/evaluation-integrity]] — keeping the evaluation signal honest over
  long search loops; 6 concepts: hce-evaluation, pass-at-k,
  citation-anchoring, typed-claim-partition, programmable-evaluator-oracle,
  compression-as-generalization-test.
- [[mocs/autonomous-search-loop]] — how an autonomous agent searches a vast
  candidate space productively; 5 concepts: evolutionary-expansion,
  evolutionary-search-grain, programmable-evaluator-oracle, async-worker-pool,
  budget-as-ceiling.

## Literature / posts (recent)

- [[literature/posts/gist-github-com-karpathy-llm-wiki]] — Karpathy's LLM Wiki gist (primary source for [[concepts/llm-wiki-pattern]]: compile-time curation, raw/wiki/schema layers)
- [[literature/posts/theaioperator-io-rebuilt-karpathy-llm-wiki]] — the rebuild critique (five gaps of append-only wikis; AI-first vault; reversible automation)

## Literature / papers

- [[literature/papers/xu2025amem]] — A-Mem (Zettelkasten agent memory: LLM link generation + retroactive memory evolution; NeurIPS 2025, code)
- [[literature/papers/shen2026dynamic]] — SLIM (skill retirement by leave-one-skill-out marginal contribution; removal ≠ internalization)
- [[literature/papers/santosgrueiro2026lingering]] — Lingering Authority / PORTICO (permissions with lifetimes; epoch-bound revocable capability handles)
- [[literature/papers/zhao2026generative]] — Generative Skill Composition (which/how-many/what-order as sequence prediction; 3.9M specialist beats LLM-judge)
- [[literature/papers/bertran2026fits]] — What Fits (Into Few Tokens) Doesn't Overfit (compression test for research-agent overfitting)
- [[literature/papers/yang2026trustmem]] — TrustMem (write-time verification of memory consolidation)
- [[literature/papers/zhao2026agenticos]] — AgenticOS (OS as intent filter; budgets inside capability tokens)
- [[literature/papers/belikova2026managing]] — Managing Procedural Memory in LLM Agents (AFTER skill-transfer benchmark)
- [[literature/papers/jain2026agentic]] — Agentic AutoResearch for Space Autonomy (in-loop credibility gate, aerospace)
- [[literature/papers/yang2026graph]] — Graph-based Agent Memory (survey)
- [[literature/papers/wu2026gam]] — GAM: Hierarchical Graph-based Agentic Memory
- [[literature/papers/du2026memory]] — Memory for Autonomous LLM Agents (survey)
- [[literature/papers/qu2026coral]] — CORAL: Autonomous Multi-Agent Evolution
- [[literature/papers/starace2025paperbench]] — PaperBench (OpenAI)

## Active experiments

(list of `experiments/YYYY-MM-DD-<slug>/` folders currently in flight)

## Open questions

(anything you want to return to)
