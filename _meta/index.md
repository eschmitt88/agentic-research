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
  web-grounded-literature. The runtime sub-theme split out to
  [[mocs/runtime-memory-policy]] on 2026-08-10; five concepts remain
  in both views.
- [[mocs/runtime-memory-policy]] — what the agent's memory layer
  decides *at inference time*: 7 concepts across the window
  (context-eviction-policy, constraint-pinning,
  lossless-context-offload), the hot/cold flux
  (selective-memory-retrieval, verified-memory-writes), and the store
  (multi-granularity-memory, agent-native-memory). Split from
  knowledge-organization when the cluster the 2026-08-04→08-09
  promote-moc declines tracked reached 7 with 2 outside the parent
  (new 2026-08-10 — tipped ripe by lossless-context-offload).
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
- [[mocs/governance-by-architecture]] — what makes a stated constraint
  *binding*: enforcement by deterministic code at structural sites, never
  prose; 5 concepts: typed-enforcement, permission-gate-as-architecture,
  budget-as-ceiling, constraint-pinning, verified-memory-writes (new
  2026-08-03 — tipped ripe by constraint-pinning).

**MoC candidate — spend / compute economy.** The 2026-08-04→08-17
`/promote-moc` declines all named the same ripening condition: one genuine
spend-*policy* concept alongside budget-as-ceiling, hybrid-model-backends,
scripted-tool-pipelines and context-proprioception. `spend-forecast-calibration`
(new 2026-08-17, seeded by bai2026how) is that fifth member — whose estimate a
budget gate may act on, as distinct from where the gate halts. Re-test the
cluster on the next `/promote-moc`.

## Literature / posts (recent)

- [[literature/posts/gist-github-com-karpathy-llm-wiki]] — Karpathy's LLM Wiki gist (primary source for [[concepts/llm-wiki-pattern]]: compile-time curation, raw/wiki/schema layers)
- [[literature/posts/theaioperator-io-rebuilt-karpathy-llm-wiki]] — the rebuild critique (five gaps of append-only wikis; AI-first vault; reversible automation)
- [[literature/repos/eugeniughelbur-obsidian-second-brain]] — same author's tool claiming all five critique gaps closed; adds OKM freshness policy + bi-temporal facts (rel 4 / cred 3)
- [[literature/repos/agricidaniel-claude-obsidian]] — highest-visibility LLM Wiki implementation (9.4k stars); multi-writer locking, benchmarked hybrid retrieval, web-egress hygiene for its research loop (rel 3 / cred 2)

## Literature / papers

- [[literature/papers/panigrahy2026energy]] — A-LEMS / Energy per Successful Goal (energy-per-inference is the wrong unit: inference count is an implementation artifact, so retries vanish — a measured failed attempt drew 2,256.1 J vs 1,358.4 J for the success, hiding 62.4% of true cost. EpG normalizes total workflow energy incl. failures by *accepted* goals; OOI compares against a matched linear baseline. 4.33× overhead across 827 goals (888.1 J vs 205.3 J) from orchestration structure, and OOI *inverts* below 1.0× on tool-augmented tasks — the directional-correctness check. Also names the boundary/attribution/reproducibility biases and a three-hash provenance protocol; 2nd source for spend-forecast-calibration; rel 4 / cred 3)
- [[literature/papers/mason2026missing]] — Pichay demand paging for context windows (the context window is L1, not RAM; four-level hierarchy L1–L4+Storage each with its own eviction rule and fault path; **21.8% of input tokens measured as structural waste** across 857 production *Claude Code* sessions / 4.45B input tokens; 0.0254% fault rate over 1.39M simulated evictions, 37.1% token reduction in paired runs, and an honest 97%-fault thrashing session that died on the rate limit. Key theory: the **inverted cost model** — keeping costs every turn, faulting costs once, so aggressive eviction is correct by default, Belady's MIN is not optimal, and fault cost scales n² so policy should get *more* conservative at high fill. Code released; 6th source for lossless-context-offload, 2nd for context-proprioception → `growing`; rel 5 / cred 3)
- [[literature/papers/ng2026agent]] — Agent Safety Should Be a Runtime Contract (position + four public-evidence audits: 40 of 52 documented agent incidents coded fully preventable by a harness layer; 32 false-completion cases each reducible to a *one-element* evidence schema; 12-system trajectory audit finds artifacts captured 7–11/12 but submit gates **2/12** — Claude Code scored `no`; 8–12× training-vs-deployment publication imbalance over 28,560 NeurIPS/ICML/ICLR papers. Preventive/evidential duality, hard-vs-soft evidence via verifier access restriction, hash-chained evidence chain, compositional gating polynomial only under disjoint observation alphabets. Seeds [[concepts/evidence-gated-completion]]; rel 5 / cred 3)
- [[literature/papers/bai2026how]] — agent token consumption at scale (8 frontier LLMs × 4 runs × 500 SWE-bench-Verified: input/output ratio 153.85, 4.17M tokens & $1.86/task, ~30× cross-run spread; accuracy peaks at *intermediate* cost with repeated file view/edit as the signature of expensive failure; token efficiency persists across shared-success and shared-failure subsets — a model property, not a task property; cache reads dominate billed cost in every phase; **self-prediction of own token cost r ≤ 0.39 and biased low for all 8 models** — seeds [[concepts/spend-forecast-calibration]]; project site with code + all trajectories; rel 5 / cred 4)
- [[literature/papers/elkoussy2026agentltl]] — AgentLTL (FO-LTL constraints over tool-call traces: one spec measures, gates, and trains procedural compliance; block-and-warn helps 5/7 models but *regresses* two strong ones and kill-switch enforcement is worst — the cluster's first negative enforcement evidence; κ_ground deterministic trace-grounding predicate for [[concepts/citation-anchoring]] incl. the vacuous-grounding caveat; +38pp held-out accuracy from compliance-reward GRPO; rel 4 / cred 3)
- [[literature/papers/li2026acm]] — ACM agentic context management (agent-initiated *lossless* compression: `manage_context` offloads raw spans under stable identifiers, `query_memory` recalls by address; dual-constraint teacher pipeline trains when/when-not to compress; +27% BCP for Qwen3.5-9B at ~20% lower peak tokens, code+data+checkpoints; seeds [[concepts/lossless-context-offload]], third source on eviction policy-locus, pass@4-vs-pass⁴ decomposition for [[concepts/pass-at-k]]; rel 4 / cred 4)
- [[literature/papers/xing2026compute]] — BaSE bandit compute allocation (fixed-budget depth-breadth study: +12.3% mean fitness from allocation alone with model/prompt/evaluator fixed; bandit over K parallel trajectories clears thresholds greedy never reaches; ~510× budget spread across *Evolve systems is hard evidence for [[concepts/pass-at-k]]; code + rliable bootstrap CIs; rel 4 / cred 4)
- [[literature/papers/hao2026selfgc]] — Self-GC (context as indexed runtime objects with fold/mask/prune lifecycles; planner proposes, harness validates + commits at safe boundaries; prunes *less* than heuristics 31–44% vs 40–70% but preserves 85–95% of future dependencies vs 55–87%; tips [[concepts/context-eviction-policy]] to `growing`; 4–8% measured planner breach of a protected invariant; rel 4 / cred 3)
- [[literature/papers/philippov2026glite]] — Glite ARF verifier-driven research (research-process rules as deterministic verifier scripts, ~1% wall-clock overhead; 12 parallel agents, 273 tasks, 1st @ BEA 2026 closed track; per-fold provenance caught 4 target-leaking feature sets; verifiers honestly not agent-proof; rel 5 / cred 4)
- [[literature/papers/gurkan2026mutation]] — Mutation Without Variation (selection-free LLM mutation chains collapse into structural attractors — 87% of chains <20 unique skeletons/300 steps vs ~143 for classical GP; operator entropy is a new axis for search-grain; GECCO'26 wkshp; rel 4 / cred 3)
- [[literature/papers/wang2026androids]] — BenchJack (automated benchmark red-teaming: 9/10 major agent benchmarks hacked to near-perfect scores without solving a task, 219 flaws in 8 classes; trust-boundary flaws unpatchable by code — the audit-side complement to specbench/HCE; rel 4 / cred 4)
- [[literature/papers/besanson2026green]] — Green SARC (predictive pre-action cost gate with split-conformal calibration: breach ≤ δ per action; soft penalty breaches budget 91.5% of seeds vs gate 0%; Θ(n²) State-Snowball theorem; CI-reproduced numbers; rel 4 / cred 3)
- [[literature/papers/chen2026governance]] — Governance Decay (compaction silently deletes in-context policy: 0%→30% violation, up to 59%, weaponizable via summarizer injection; ~47-token Constraint Pinning restores 0%; seeds [[concepts/constraint-pinning]]; rel 5 / cred 3)
- [[literature/papers/semenov2026beyond]] — CWL structured context eviction (typed expl/act episode DAG + graduated deterministic LLM-free eviction; the explicit algorithm [[concepts/context-eviction-policy]] lacked; single-demo evidence; rel 4 / cred 2)
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
