---
kind: paper
title: "Dynamic Skill Lifecycle Management for Agentic Reinforcement Learning"
authors: ["Junhao Shen", "Teng Zhang", "Xiaoyan Zhao", "Hong Cheng"]
institutions: ["The Chinese University of Hong Kong", "Lanzhou University"]
year: 2026
venue: arXiv
peer_reviewed: false
url: https://arxiv.org/abs/2605.10923
code_url: https://github.com/ejhshen/SLIM
citations: null
source: "raw/papers/shen2026dynamic.pdf"
added: "2026-07-13"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/selective-memory-retrieval]]"
tags: [skills, lifecycle, retirement, reinforcement-learning, procedural-memory, curation]
---

# Dynamic Skill Lifecycle Management for Agentic Reinforcement Learning

## TL;DR

SLIM treats an agent's *active external skill set* as a dynamic
optimization variable jointly updated with RL policy learning: it measures
each skill's **marginal external contribution** (MEC) by leave-one-skill-out
validation and applies retain / retire / expand operations, converging to a
compact non-empty skill set that beats both persistent skill accumulation
(SkillRL) and forced zero-skill internalization (Skill0) by an average of
7.1 points on ALFWorld and SearchQA.

## Claims

- The two prevailing skill-RL paradigms — skills as persistent
  ever-growing augmentation, or skills as temporary scaffolds removed
  toward zero-skill inference — are both wrong as defaults: the optimal
  active skill set is **non-monotonic, task- and stage-dependent**, under
  finite parametric capacity and uneven marginal contribution.
- **Retirement is not internalization.** When Skill0 forces the active set
  to zero (~epoch 90), validation drops from 92.2% to 76.6% in the next
  audit — removing a skill does not mean the policy absorbed it.
  Conversely, some broad skills genuinely become near-internalized
  (disabling them costs ~0.06), while low-frequency skills stay externally
  indispensable (cle_003: globally infrequent, −0.250 when disabled).
- Lifecycle decisions must be **contribution-aware, not
  frequency-based**: selection frequency does not predict marginal value
  in either direction; a random-audit ablation (same operations, stochastic
  decisions) is the worst variant (68.8 vs 87.5).
- Endpoint is a *learned external capability boundary*: the active set
  expands 38→46, fluctuates, then stabilizes at 21 skills; with-skill
  performance peaks at 93.8% while no-skill performance rises 29.7%→84.4%
  — policy improvement and external skill retention are complementary,
  not exclusive.

## Methods

- Formulation: capacity-constrained allocation over (policy θ, active set
  A, internalized set I) — maximize task performance minus a monotone
  external support cost Ω(A), subject to parametric memory budget.
  Alternating optimization: GRPO policy update with A fixed, then
  lifecycle management with θ fixed.
- MEC: Δt(s) = Perf(V_t(s); A_t) − Perf(V_t(s); A_t \ {s}) on the
  validation tasks routed to s, EMA-smoothed across audit rounds.
- Three rules with explicit thresholds: **retain** if smoothed MEC ≥
  τ_keep; **retire** if MEC < τ_retire *and* cumulative exposure ≥ n_min
  *and* low-contribution streak ≥ p (protects low-frequency skills from
  premature removal); **expand** when a skill's routed region persistently
  fails and with-skill performance leaves headroom — new task-specific
  skills are created as standalone **Anthropic-style SKILL.md artifacts**
  via a skill-creator workflow.
- Hierarchical skill retrieval: active general pool + task-specific top-K
  (K=3, Qwen3-Embedding-0.6B, τ_emb=0.45) — lifecycle operates on the
  *active* pool the retrieval draws from.
- Audits every d=10 GRPO steps, at most M=4 skills per audit (bounded
  audit budget). Backbone Qwen3-4B; baselines ReAct, Reflexion, Mem0,
  ExpeL, GRPO, EvolveR, SkillRL, Skill0. Train/validation/test protocol:
  lifecycle auditing and tuning on validation, final reporting on test.

## Results

- ALFWorld: SLIM† 87.5 vs SkillRL† 75.0 (+12.5), Skill0 74.2, GRPO 67.2.
  SearchQA: 41.0 vs Skill0 39.3. Gains concentrate on procedural
  state-transformation tasks (Clean 91.4, Cool 88.5).
- ALFWorld needs retained external skills (SLIM† − SLIM = 87.5 − 72.7);
  SearchQA's gap nearly vanishes — the external-vs-internalized division
  of labor is domain-dependent, supporting the lifecycle-over-policy
  claim.
- Ablations (ALFWorld avg): full SLIM 87.5; w/o retirement 73.4
  (accumulation degenerates); w/o expansion 78.9 (pruning can't repair
  missing coverage); random audit 68.8; fixed active-set size (LRU) 75.6
  (prompt-budget control alone is not the mechanism).

## Critique / open questions

- Small backbone (Qwen3-4B) and two benchmarks; whether the
  internalization/externalization boundary behaves the same at frontier
  scale — where parametric capacity is vastly larger — is untested.
- Leave-one-skill-out MEC is a *local* estimate conditioned on the current
  policy, active set, and routing; the paper is careful about this
  (Lemmas give bounded-risk conditions) but global attribution across
  skill subsets remains open.
- Audit cost: LOSO validation per audited skill is expensive; the bounded
  audit budget (M=4, every 10 steps) is a practical compromise whose
  sensitivity is only partially explored.
- RL-training framing; transfer of the retire/expand rules to
  *non-trained* agent memory (a curated skill directory like this
  project's) is by analogy, not demonstrated.

## Trust signals

- **Credibility:** 3 — reputable academic group (CUHK Database Group +
  Lanzhou), code released on GitHub, thorough ablations and a clean
  diagnostic probe; but arXiv preprint (not peer-reviewed), no citations
  yet, and evidence limited to a 4B backbone on two benchmarks.

## Follow-up

- **Relevance:** 4 — gives [[concepts/skill-library-lifecycle]] the
  mechanistic anchor it lacked for **retirement**: an evidence-driven
  retire rule (marginal-contribution threshold + exposure floor +
  persistence streak) plus the sharp negative result that removal ≠
  internalization. Also the first source where skill *expansion* emits
  Anthropic-format SKILL.md artifacts from inside an RL loop.
- The exposure-floor + streak guard before retiring is directly
  transferable to this project's `/lint` heuristics: a note is a retire
  candidate only after *sufficient audited exposure* with persistently
  negligible contribution — not merely because it is old or rarely
  linked.
- Pair with zhao2026generative (this digest): SLIM manages *which skills
  exist* (write-side), the composer decides *which to load, how many, in
  what order* (deploy-side); SLIM's churn is exactly what invalidates a
  trained composer's fixed vocabulary — the open seam between the two.
