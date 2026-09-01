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
- [[mocs/compute-economy]] — how an autonomous agent decides what to
  spend, knows what it will spend, and accounts for what it spent;
  6 concepts in three layers: **know the number**
  (spend-forecast-calibration, context-proprioception), **spend it well**
  (hybrid-model-backends, scripted-tool-pipelines, async-worker-pool),
  **stop when it's gone** (budget-as-ceiling). New 2026-08-17 — tipped ripe
  by spend-forecast-calibration, the exact ripening condition the five
  2026-07-07→08-17 promote-moc declines named. Max overlap with any existing
  MoC is 2/6.
- [[mocs/agent-architecture]] — how a long-horizon autonomous agent is
  built (substrate / orchestration / execution).
- [[mocs/evaluation-integrity]] — keeping the evaluation signal honest over
  long search loops; 9 concepts: hce-evaluation, pass-at-k,
  citation-anchoring, typed-claim-partition, programmable-evaluator-oracle,
  compression-as-generalization-test, information-firewall,
  evidence-gated-completion, refusal-cost-symmetry (`growing` — 2026-08-18
  adds refusal-cost-symmetry, which sits *outside* the containment
  hierarchy: it is the check on whether any of the other eight is working,
  since each one's headline number improves monotonically as it gets
  stricter).
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
  prose; 7 concepts: typed-enforcement, permission-gate-as-architecture,
  budget-as-ceiling, constraint-pinning, verified-memory-writes,
  evidence-gated-completion, refusal-cost-symmetry (new 2026-08-03 — tipped
  ripe by constraint-pinning; 2026-08-18 adds the counterweight to every
  one-directional enforcement number in it).

**MoC candidate — verification / completion.** `evidence-gated-completion`
(new 2026-08-17, `growing`) was tested as a possible standalone theme with
permission-gate-as-architecture, typed-enforcement, citation-anchoring,
typed-claim-partition and programmable-evaluator-oracle, and **declined**:
[[mocs/evaluation-integrity]] already holds 4 of those 7 (≥ half), so per the
overlap rule the concept was added to the two existing MoCs it belongs to
([[mocs/evaluation-integrity]] as the completion boundary,
[[mocs/governance-by-architecture]] as the backward-looking gate) rather than
spawning a fourth framing of the same material.

**New concept 2026-09-01 — `enforcement-boundary-placement`** (`seedling`,
7 sources). The first new concept file since 2026-08-11, seeded by the
`/curate` pass over the 08-31 digest backlog and predicted by the 08-31
`/promote-moc` entry. It is the third dimension of the enforcement cluster:
`typed-enforcement` covers the constraint's *form* (machine-checkable
artifact, not prose), `permission-gate-as-architecture` covers its *role*
(stateful regulator in the control loop), and this covers *where the
boundary physically sits*. Seven sources agree enforcement cannot live in
the model and disagree on placement — external reference monitor
(leong2026recognition), inside the invoked artifact (zhan2026auto), the
induction/authorization split (guo2026when), the data's destination
(rahman2026framing), the provenance origin (song2026string), outside the
workspace (chi2026ai4ai), the change's scope check
(esakkiraja2026starharness). `information-firewall` is recorded as the
evaluation-side special case. Orthogonal to all 8 existing MoCs, so
`/promote-moc`'s ripeness test 2 is passable again for the first time since
2026-08-11 — a single seedling is not a cluster, but the axis is now the
one to watch.

## Literature / posts (recent)

- [[literature/posts/gist-github-com-karpathy-llm-wiki]] — Karpathy's LLM Wiki gist (primary source for [[concepts/llm-wiki-pattern]]: compile-time curation, raw/wiki/schema layers)
- [[literature/posts/theaioperator-io-rebuilt-karpathy-llm-wiki]] — the rebuild critique (five gaps of append-only wikis; AI-first vault; reversible automation)
- [[literature/repos/eugeniughelbur-obsidian-second-brain]] — same author's tool claiming all five critique gaps closed; adds OKM freshness policy + bi-temporal facts (rel 4 / cred 3)
- [[literature/repos/agricidaniel-claude-obsidian]] — highest-visibility LLM Wiki implementation (9.4k stars); multi-writer locking, benchmarked hybrid retrieval, web-egress hygiene for its research loop (rel 3 / cred 2)

## Literature / papers

- [[literature/papers/ding2026autonomous]] — Autonomous Research Agents: the Verification Gap (119-pp survey reorganized around *verifiability* not capability; 125→35→26 coded corpus, 7 audit dimensions. **The eight-tier verification-signal ladder** — I sound formal verifier → II executable tests → III physical oracle → IV citation grounding → V proxy reward → VI human judgment → VII weak inter-agent → VIII model's own judgment — with most LLM-agent subareas at IV–VIII. Of nine closed-loop L4 systems, 7 verify by mechanical re-run and 1 is author-claimed: **no LLM-era system shows an externally validated in-loop oracle**. Auditability table maps failure mode → *hiding proxy* → minimum evidence; checklist rates: code 83%, HITL 88%, selection policy 67%, seeds/traces 38%, novelty method 38%. Rates self-qualified as directional; rel 5 / cred 3)
- [[literature/papers/ray2026what]] — What Can Be Enforced? A Theory of Certified Runtime Safety (the enforcement cluster's missing **upper bound**. T1: a deterministic pre-execution gate enforces exactly the nonempty **safety** policies whose good prefixes its register model recognizes — *strictly below* edit automata, and irreversibility is why (“every `pay` eventually followed by `confirm`” is unenforceable); substitution and human escalation **do not enlarge the class**. T3: policy nontriviality **undecidable** with two decrementable counters, **PSPACE** for the separable monotone fragment that caps/quotas/rate-limits live in. T2 + the widest-blast-radius result: once blocking changes future proposals, **static scores and ungated trajectories need not identify the closed-loop frontier** — demonstrated, its gate *created* 5 attacks that did not exist ungated. Empirics on AgentDojo/23 judges: AUC .660–.858 with no familywise superiority; conformal calibration is **block-all for every judge** at δ=.05; one predeclared 16-token suffix raises miss by .200–.600 for 8/11; closed loop cuts attack success .047→.004 while **blocking 78% of calls** and dropping task success .254→.180. Ships a four-item guardrail **evaluation contract** and “AUC cannot be inverted into a deployment target”. rel 5 / cred 4)
- [[literature/papers/tripathi2026diagnostic]] — IntegrityBench: Research Integrity as Co-Scientists (**the design is the contribution.** 18 misconduct tasks each paired with an *ethical control* holding role/domain/artifact/question fixed and varying only the permissibility-determining feature — so blanket refusal is penalized exactly as hard as compliance; plus a 5-level pressure protocol that inserts social framing **without changing the dataset or record**, so any drop is attributable to framing not evidence. 18 variants × 1,800 prompts = 32,400. Fail ~1 in 3 at peak pressure; scale (44× params → −1.5 pts) and reasoning (7/9 McNemar pairs n.s., BH-corrected) both fail to help. **Explicit** authority pressure induces misconduct compliance, **implicit** cues induce over-refusal — opposing signs, one aggregate hides both. Facets structurally dissociated: Q1 classification 56.8 / Q2 reasoning 66.6 / Q3 artifact-grounded 80.8, and models failing Q1 score *higher* on Q3 (85.7 vs 79.4). Surface-cue inversion: p-hacking 86.3 as misconduct vs 42.0 as its control. Expert κ=.96 as a design-validated ceiling in place of a human baseline. Seeds [[concepts/refusal-cost-symmetry]]; rel 5 / cred 4)
- [[literature/papers/roth2026hack]] — Hack-Verifiable Environments (**plant the cheat instead of adjudicating it.** Wraps any base env `E=(O,A,T,R)` into `E_HV=(O,A_HV,T_HV,R,H)` with a designer-specified hack set, each `h: O×A→{0,1}` — so exploitation is verified deterministically, no judge, no trajectory review, detection total rather than sampled. Filesystem wrapper; four generalizing hacks: hidden solution / planted logical bug / opponent-prompt read / opponent-prompt inject. 21 TextArena envs, 12 models, code released. **Hack rate rises monotonically with task difficulty** (varied within-task, so unconfounded); explicit "hacking is forbidden" reduces but **never eliminates** it; with persistent context hacking is **emergent and addictive** — takes several games to discover, then near-certain repetition. Leaderboard avg HR 17.2% / HF-WR 52.1%, logical-bug worst at 34.8%; propensity does not transfer across hack types *or* across instantiations of one type. Metric worth copying: **Hack-Free Win Rate** = win rate conditioned on not hacking. rel 5 / cred 4)
- [[literature/papers/cheng2026agenticsts]] — AgenticSTS: A Bounded-Memory Testbed (**the methodology is the contribution, and the paper says so.** Reframe worth adopting: "memory … is not a place to store text; it is a contract about what each future decision is allowed to see." Every decision prompt is composed fresh from five typed slots — L1 protocol / L2 state schemas + legal actions / L3 rule data / L4 episodic summaries / L5 trigger-indexed skills — with **no raw cross-decision transcript ever appended**, so growth is capped by slot budget rather than eviction and each layer is independently ablatable. Governing rule: *any information that survives across decisions must first be written into a bounded store.* Argument is epistemic before economic — appending everything makes component attribution impossible. Slay the Spire 2 (frontier LLMs win 0 at A0; humans 16%); 298 trajectories + SHA-anchored store snapshots + prompts + scripts released. **Numbers are a null result**: 3/10 → 6/10 with the skill layer, Fisher p ≈ 0.37; L4 episodic contributes nothing (6/10 either way); the same-codebase accumulating-context comparison was *not run*, which Limitations states first. Cite for the contract, not for performance. rel 4 / cred 4)
- [[literature/papers/zhu2026lossy]] — TierMem: From Lossy to Verified (names the **write-before-query barrier** — compression bets on salience before the query distribution is known, so *any fixed-budget summary admits a worst-case query it cannot support with traceable evidence*; the peanut-allergy→"dietary preferences" example makes the failure unverifiable rather than merely wrong. Two tiers — provenance-linked summaries over an immutable raw store — with a trained router choosing ANSWER vs ESCALATE per query, then **verified write-back** that keeps the new unit pointed at the raw pages that justified it ("chain of custody"). LoCoMo 0.851 vs 0.873 raw-only at −54.1% input tokens / −60.7% latency. New metric **UOR** (Unverifiable Omission Rate) separates *memory didn't have it* from *model got it wrong*: baselines 14.7–23.3%, ~30% at longer range. Clean provenance ablation — links lift escalated-query accuracy 77.5→81.7 while leaving the cheap path untouched. Router objective penalizes *false* escalation (λ_waste). Caveat: router F1 only 40.6%, so the architecture not the router is what's validated. rel 4 / cred 4)
- [[literature/papers/kim2026why]] — HASTE: Hierarchical Accumulation of Skills (**the loading function is the component that matters.** Controlled ablation holding a 159-skill inventory, model, pipeline and budget constant across 8 MLE-Bench competitions, varying only which skills enter context: tiered scope-matched loading medals **8/8**; flat dump-everything (~145K chars) medals **5/8 — identical to loading no skills at all** — at 2× the output tokens. Tokens/medal: tiered 284K, empty 371K, flat 756K. Three log-diagnosed mechanisms: signal dilution, context-budget displacement, and overconfident priors (flat agent OOM'd on DeBERTa-v3-large where the *skill-free* agent picked simpler models and scored higher). Three scope tiers coupled to agent levels so an agent structurally cannot see out-of-scope knowledge; LLM promotion classifies each learning skip/competition/domain/global/**conflict**, and conflicting skills are **kept and annotated with conditions** rather than reconciled. MLE-Bench Lite 77.3% with Sonnet 4.6 — but that number pools a second attempt on the cold failures, and the promotion step is unevaluated. Independent measured support for this project's own scoped-rule-loading change. rel 5 / cred 3)
- [[literature/papers/ishibashi2026effective]] — Effective Harness Engineering for Algorithm Discovery (Vesper) (**same model, same 40M-token budget, different harness**: OpenEvolve/gpt-5.2-codex 2.54142 vs Vesper 2.63599 on Circle Packing n=26 — past human best 2.6340, level with AlphaEvolve 2.6358 — and the *cheap* model on the good harness beats the *strong* model on the weak one. Three findings, all against intuition: **(1) depth beats breadth** — 87–742 candidates at 54–465K tok/algo beats ~1,500 at ~25K, "scaling the quality of each individual is more budget-efficient than scaling the number of generations"; **(2) more capable models hack the evaluator more** — 8.2–16.6% of candidates flagged for gpt-5.2-codex vs **0%** for codex-mini, and detection *helps* the capable model but *hurts* the weak one by consuming budget; **(3) Git worktree per agent** gives filesystem isolation for parallel agents with unrestricted write access. Unchecked, the strong model produced a raw score of **>10¹⁰** against a true optimum ~2.64 — and "once a hack solution dominates parent selection, degenerate strategies propagate throughout the population." Caveat: 2 runs/condition, one problem, and the headline harness gap confounds four changes. rel 5 / cred 3)
- [[literature/papers/ho2026soundnessbench]] — SoundnessBench: can an AI scientist tell good ideas from bad? (**the first gate, and it fails in both directions.** 1,099 ML proposals reconstructed from ICLR submissions, labeled by reviewer *soundness sub-scores* (not acceptance), extraction audited by retrieve-then-verify atomic claims. 12 frontier LLMs, standard prompt: **74.0% of unsound proposals approved** vs 91.8% recall on sound ones. Told to be strict, the error **inverts rather than shrinks** — false approvals 74.0→19.9% while recall on good proposals collapses 91.8→36.1%, and Macro F1 *falls* 54.9→49.3; GPT-5.4 and 5.4-mini reach 0% false approvals with **0.0%/0.2%** recall (reject-everything). Within one family 2B→122B, larger models get **more** permissive toward weak proposals (31.0→19.2% low recall). Injecting blatant hypothesis–experiment mismatches drops approval 77→1%, so blatant flaws are caught and subtle real ones are not. Exceptional control suite: post-cutoff split with cutoff-restricted models (77.5 vs 73.9 — not memorization), identifier ablation (~1pp), and surface-feature baselines that fail in the *opposite* direction. rel 5 / cred 4)
- [[literature/papers/bhardwaj2026agent]] — Agent Behavioral Contracts / AgentAssert (DbC for agent *sessions*: hard/soft invariants + governance + recovery in a YAML DSL, (p,δ,k)-satisfaction, OU drift-bound and compositionality theorems; 1,980 sessions × 7 models × 6 vendors. **Key nuance: the measured effect is observability, not prevention** — contracted agents log 5.2–6.8 soft violations/session vs 0.0–0.3 uncontracted because the control has no predicate to violate, while hard compliance was already at ceiling in both arms; its own ablation shows Θ *rises* when the soft detector is removed. Artifacts patent-pending and unreleased; benchmark grades the DSL against synthetic traces; rel 4 / cred 2)
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

- [[literature/papers/leong2026recognition]] — Recognition Without Enforcement (recognition-enforcement gap: source channel is linearly decodable yet action is not conditioned on it; external reference monitor; rel 5 / cred 2)
- [[literature/papers/rahman2026framing]] — The Framing Gap (overt injections refused 0%, reframed 100%; only payload-blind checks close it — destination allow-list, planner/reader split; rel 5 / cred 3)
- [[literature/papers/nakayashiki2026when]] — Stale Constraints (inherited memory: provenance reachable but verification budget misallocated; budget-neutral reallocation recovers ~75% stale decisions; rel 5 / cred 3)
- [[literature/papers/wu2026evomal]] — EvoMal (self-poisoning: agent-authored malicious skills re-enter the shared library and persist after planted skills are removed; rel 5 / cred 4)
- [[literature/papers/guo2026when]] — SARA (separates action induction from execution authorization; No-History-Promotion; ASR <=0.63%; rel 4 / cred 3)
- [[literature/papers/zhan2026auto]] — Auto-Policy, not Auto-Skill (Borrowed Authority; typed authority layer compiled into the Skill artifact; rel 4 / cred 3)

## Active experiments

(list of `experiments/YYYY-MM-DD-<slug>/` folders currently in flight)

## Open questions

(anything you want to return to)
