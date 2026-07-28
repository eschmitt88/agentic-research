---
name: index
description: Entry-point index for this project's knowledge graph.
---

# Index

Orientation for the project knowledge graph. Updated by `/wrap`, `/ingest`,
and `/new-experiment`.

## Maps of Content

- [[mocs/knowledge-organization-for-research-agents]] — substrate /
  write-side / read-side of agent memory; 12 concepts spanning
  agent-native-memory, llm-wiki-pattern, file-as-bus, structured-world-model,
  skill-library-lifecycle, typed-claim-partition, citation-anchoring,
  selective-memory-retrieval, context-eviction-policy,
  multi-granularity-memory, verified-memory-writes,
  web-grounded-literature.
- [[mocs/agent-architecture]] — how a long-horizon autonomous agent is
  built (substrate / orchestration / execution).
- [[mocs/evaluation-integrity]] — keeping the evaluation signal honest over
  long search loops; 7 concepts: hce-evaluation, pass-at-k,
  citation-anchoring, typed-claim-partition, programmable-evaluator-oracle,
  compression-as-generalization-test, information-firewall (`growing` —
  third attestation 2026-07-28 adds *time* as a boundary axis alongside
  file space and retrieval space).
- [[mocs/autonomous-search-loop]] — how an autonomous agent searches a vast
  candidate space productively; 5 concepts: evolutionary-expansion,
  evolutionary-search-grain, programmable-evaluator-oracle, async-worker-pool,
  budget-as-ceiling.
- [[mocs/capability-layer]] — the agent's action surface: how procedural
  capability is authored, ported, executed, and gated; 6 concepts:
  skill-library-lifecycle, shared-skill-namespace, scripted-tool-pipelines,
  hybrid-model-backends, permission-gate-as-architecture, typed-enforcement
  (new 2026-07-28 — the gate's policy as a statically checkable artifact,
  factored out from the gate's placement).

## Literature / posts (recent)

- [[literature/posts/gist-github-com-karpathy-llm-wiki]] — Karpathy's LLM Wiki gist (primary source for [[concepts/llm-wiki-pattern]]: compile-time curation, raw/wiki/schema layers)
- [[literature/posts/theaioperator-io-rebuilt-karpathy-llm-wiki]] — the rebuild critique (five gaps of append-only wikis; AI-first vault; reversible automation)
- [[literature/repos/eugeniughelbur-obsidian-second-brain]] — same author's tool claiming all five critique gaps closed; adds OKM freshness policy + bi-temporal facts (rel 4 / cred 3)
- [[literature/repos/agricidaniel-claude-obsidian]] — highest-visibility LLM Wiki implementation (9.4k stars); multi-writer locking, benchmarked hybrid retrieval, web-egress hygiene for its research loop (rel 3 / cred 2)

## Literature / papers

- [[literature/papers/lu2026meta]] — The Meta-Agent Challenge (HCE as *enforcement*: holdout on a separate container filesystem, test scoring gated behind a cryptographic secret injected only after the development phase ends; aligned agents refuse instructions to cheat but violate under resource starvation, 7/8 trials; rel 5 / cred 4)
- [[literature/papers/palumbo2026formal]] — FORGE (policy as Datalog + reference monitor with an assume/guarantee correctness theorem; injection 100%→0%, τ²-bench compliance 58%→98%, at 19–38% latency; anchors [[concepts/typed-enforcement]]; rel 5 / cred 4)
- [[literature/papers/zhao2026specbench]] — SpecBench (reward-hacking gap Δ = validation − held-out; every frontier agent saturates the visible suite while Δ grows ~27pp per 10× code size; rel 5 / cred 4)
- [[literature/papers/wang2026search]] — Search-Time Contamination (holdout leaks through the agent's own search tool; three-tier taxonomy where only answer-level leakage inflates, so URL-matching audits measure the wrong thing; rel 5 / cred 4)
- [[literature/papers/tang2026memory]] — MSCE (governed memory→skill promotion: evidence + positive gain + consistency as an admission gate; skills keep evidence anchors; flat-memory ablation shows the hierarchy outweighs the skill layer; rel 4 / cred 4)
- [[literature/papers/gao2026mempoison]] — MemPoison (write gates have a structural ceiling: compositional + dormant corruption is invisible per-record; L3 dormant is the *worst* attack at 76.7%; all 10 models vulnerable, GPT-5 worst; rel 5 / cred 4)
- [[literature/papers/sharma2026smsr]] — SMSR (independent impossibility route to write-time provenance; 0% ASR unsigned but 8% residual against the *authenticated* writer, qualifying TMA-NM's sufficiency claim; Consistent Minority Effect; rel 4 / cred 3)
- [[literature/papers/ye2026agent]] — Agent Contracts (conservation law Σ R_i ≤ R_parent for delegated budgets; settles pre-flight reservation as *not implementable* — a ceiling is ceiling-plus-one-call; COINE 2026 @ AAMAS; rel 5 / cred 3)
- [[literature/papers/khan2026token]] — Token Budgets (63-incident overrun catalog; first empirical anchor for budget-as-ceiling; rel 5 / cred 3)
- [[literature/papers/louck2026securing]] — TMA-NM (machine-checked separation theorem: content/lineage authority signals are malleable; origin-bound write-time authority holds at 0% ASR; rel 4 / cred 3)
- [[literature/papers/wang2026naturebench]] — NatureBench (discovery-vs-reproduction split via information firewall; sealed host-side evaluator; rel 4 / cred 4)
- [[literature/papers/xu2025amem]] — A-Mem (Zettelkasten agent memory: LLM link generation + retroactive memory evolution; NeurIPS 2025, code)
- [[literature/papers/chhikara2025mem0]] — Mem0 (production memory layer, 60.8k stars; ADD/UPDATE/DELETE/NOOP write policy, 91% lower p95 latency vs full-context; rel 4 / cred 4)
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
