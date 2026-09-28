---
kind: paper
title: "Who Holds the Pen? Let Specifications, Not Agents, Sign Off"
authors: ["Haiqing Li", "Xin Ma", "Yinhao Wu", "Wenliang Zhong", "Feng Jiang", "Thao M. Dang", "Xiao Hu", "Hehuan Ma", "Yuzhi Guo", "Junzhou Huang"]
institutions: ["The University of Texas at Arlington", "Monash University", "Kent State University"]
year: 2026
venue: "arXiv (cs.AI), AAAI-format preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.29921"
code_url: null
citations: null
source: "raw/papers/li2026who.pdf"
added: "2026-09-28"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/refusal-cost-symmetry]]"
tags: ["completion-gating", "specification-following", "skills", "skillsbench", "guidebench", "obligation-ledger", "runtime-enforcement", "autoformalization", "freshness", "over-refusal", "reference-monitor"]
---

# Who Holds the Pen? Let Specifications, Not Agents, Sign Off

## TL;DR

On all 87 SkillsBench tasks, seven frontier agents in OpenHands leave
**13.6–20.4%** of 509 source-grounded task directions unrealized. They also
claim completion far more often than they pass: claim rates run 86.2–100%
against official pass rates of 52.9–71.3%, a gap of **28.7–37.9 pp**. The
three strongest models claim "done" on **100%** of runs, so their claim
carries no information at all.

SpecHarness is the fix. An LLM compiler, frozen before evaluation, turns the
agent-visible prompt, workspace and `SKILL.md` into source-linked obligations.
Only obligations with a qualified deterministic validator become hard; the
rest stay advisory or are abstained. The agent proposes actions, repairs and
finalization. A ledger commits obligation state only from admissible,
version-fresh evidence, and finalization requires every fresh mandatory
obligation to be satisfied.

Macro Pass rises **61.1% → 73.1%** (paired bootstrap CI +9.1 to +14.9 pp)
and S–A falls **32.8% → 12.8%**. The authors say plainly that this measures
the whole runtime, including online validator feedback and repair that Raw
never gets, not the effect of commitment alone.

## Claims

- **Two structural gaps, not one.** The *understanding–execution* gap: a
  requirement can be understood and still not govern execution. The
  *state–authority* gap: "an agent's interpretation or completion claim does
  not prove that the required state has been achieved."
- **The question is who signs, not when checking happens.** "The key question
  is therefore not only when or how execution is checked, but who may
  establish specification-governed state and authorize completion." They
  state it as the *State Authority Principle*: agents may plan, act and
  request completion, but "only admissible evidence from qualified providers
  may establish specification-governed state."
- **Existing paradigms each cover one stage.** Post-hoc verification detects
  after the fact. Completion gating (VeriMAP; Nguyen & Tran's verify-gated
  completion) "block[s] unsupported completion claims but verif[ies]
  primarily at terminal admission." Runtime enforcement (AgentSpec, VIGIL,
  FORGE) constrains actions without committing effects. SpecHarness makes
  "completion the final commit rather than the sole verification point."
- **Checkable vs not is a disposition, set before runtime.** "Ambiguous,
  subjective, conflicting, or unverifiable requirements remain advisory
  rather than becoming hard obligations." An obligation is hard only if it
  is mandatory, its parameters are source-grounded, it has an authorized
  evidence provider, and its validator passes a qualification suite.
- **Scoped guarantee.** "The guarantee covers grounded mandatory
  obligations, not the full natural-language specification." Preventive
  no-bypass is claimed only on closure-audited surfaces.
- **Capability does not close the gaps.** "Greater baseline capability does
  not eliminate the gaps when specifications remain transient context."

## Methods

- **Benchmarks.** SkillsBench: 87 tool-use and artifact tasks with injected
  skills and held-out verifiers (585 official test functions). GuideBench:
  1,042 guideline-constrained decision tasks, compiled into 297 obligation
  templates and 5,817 task-level instances. Held-out verifiers and oracles
  are used only for final evaluation, never for obligation construction or
  runtime feedback.
- **Compiler selection and freezing.** Seven candidate LLM compilers were
  scored on a task-disjoint development set: 14 SkillsBench `tasks-extra/`
  tasks, 65 source units, 81 hand-annotated directions. GPT-5.6 Sol matched
  77/81 (95.1% dev coverage), was selected, and was frozen. In a
  post-selection audit its 509 directions align with 573/585 official tests
  broadly and 406/585 at fine grain. The authors note this "measures
  alignment, not recovery of verifier semantics."
- **Validator qualification.** A validator may block only if its inputs are
  observable inside the trusted boundary, its criterion is deterministic and
  source-linked, and it is reproducible. Each validator is tested before
  deployment on satisfying cases, targeted violations, malformed inputs and
  provider errors. On SkillsBench, 250 of 278 candidate validators qualified.
  Across both benchmarks, 1929/1946 and 720/728 cases produced the expected
  outcomes.
- **Placement is assigned per obligation by a closure audit.** An action
  class gets preventive *mediate-and-commit* only if a capability graph
  shows no reachable effect-equivalent path around its authorization check.
  Otherwise, safely isolated channels get post-effect *validate-and-commit*.
  On SkillsBench, 110/250 hard obligations (44.0%) are mediated and 140/250
  (56.0%) are validate-and-commit. GuideBench is 100% validate-and-commit.
  Closure stress tests found 11 bypasses in 1628 cases, and those surfaces
  were excluded from mediation.
- **Ledger and freshness.** A commit binds (result, dependency digest,
  provenance) atomically. Both `passed` and `failed` can be committed; an
  `error` cannot. Evidence is fresh only while every dependency version
  matches. A mutation marks the commit stale and forces revalidation.
- **Comparisons.** Raw vs SpecHarness across seven agents on both
  benchmarks. With GPT-5.6 Sol fixed, the authors compare paradigms using
  their own adaptations of baselines. On SkillsBench these are Agentic
  Rubrics (post-hoc), VeriMAP (completion gate) and AgentSpec (runtime
  enforcement). On GuideBench they are RvLLM, VeriMAP and SatLM. There are
  also an architecture ablation (Table 5), 248 freshness mutations
  (Table 6), cost (Table 22) and diagnostic traceability (Table 23).
- **Statistics.** Paired task bootstrap (10,000 replicates) and exact
  per-model McNemar tests with Holm correction, for main results only. The
  paper appears to use one run per task-agent pair (87 paired outcomes per
  model). Ablations use one model and report no intervals.

## Results

- **Raw gaps (Table 1, Fig. 3).** U–E ranges from 13.6% (GPT-5.6 Sol) to
  20.4% (DeepSeek-V4-Pro), macro 17.4%. S–A ranges from 28.7 to 37.9 pp,
  macro 32.8%.
- **SpecHarness, SkillsBench (Table 1).** Macro Pass 61.1 → 73.1, U–E
  17.4 → 9.3, S–A 32.8 → 12.8. All seven models improve (+6.9 to
  +17.2 pp Pass). Every per-model McNemar test survives Holm correction;
  the weakest is Kimi K3, with 6 vs 0 discordant tasks at p = 0.031.
- **SpecHarness, GuideBench (Table 3).** Macro Pass 86.2 → 91.3, U–E
  10.4 → 5.0, S–A 13.8 → 6.9.
- **Paradigm comparison, SkillsBench, GPT-5.6 Sol (Table 2).** Pass / U–E /
  S–A: Agentic Rubrics 74.7 / 12.6 / 23.0; VeriMAP 79.3 / 9.6 / 20.7;
  AgentSpec 78.2 / 8.8 / 24.1; SpecHarness 85.1 / 6.3 / 6.9. The action-time
  placement (AgentSpec) gives lower U–E but higher S–A than the
  completion-time gate (VeriMAP): "Action constraints reduce divergence
  without establishing effects, whereas completion checks filter terminal
  claims without governing prior trajectories."
- **Ablation, GPT-5.6 Sol (Table 5).** Columns are Pass / U–E / S–A / Raw-pass
  preservation. Full: 85.1 / 6.3 / 6.9 / 96.8. w/o Mediation: 80.5 / 10.0 /
  11.5 / 95.2. w/o Effect Validation: 78.2 / 9.4 / 16.1 / 93.5. **w/o
  Commitment: 81.6 / 8.1 / 25.3 / 98.4.** **w/o Blocking Qualification:
  79.3 / 8.8 / 5.7 / 90.3.** w/o Skill Obligations: 77.0 / 12.0 / 14.9 /
  91.9.
- **Freshness (Tables 6, 13).** Without invalidation, all 248 mutations
  leave stale evidence admissible for finalization. With it, all 595
  affected entries are invalidated, 570/595 are revalidated, and 218/228
  affected tasks recover.
- **Cost (Table 22).** 1.53× task-agent tokens, 1.24× condition-execution
  wall-clock, +0.36 calls per task. Compiler selection, validator
  qualification and per-task obligation construction are excluded.
- **Residual failures (Table 20).** 164 SkillsBench official failures under
  SpecHarness: repair or interaction exhausted 51, outside hard-enforcement
  scope 35, qualified obligation unresolved 28, held-out evaluator mismatch
  28, validator error 22.

## Critique / open questions

- **The main comparison bundles the gate with information Raw never
  gets.** Raw gets no validator feedback at all. SpecHarness gets online,
  source-linked validator results and a repair loop. The authors flag this
  three times: the comparison "evaluates the complete governed runtime rather
  than the isolated effect of authoritative commitment". So the +12.0 pp
  Pass is mostly *feedback*, not *authority*. The ablation that comes
  closest to isolating the gate, w/o Commitment, keeps 81.6 of the 85.1
  Pass. The commit/finalization gate therefore adds about 3.5 pp (about 3
  of 87 tasks, one run, one model, no interval). Its large effect is on
  S–A (25.3 → 6.9), which is what the gate governs by construction.
- **S–A under SpecHarness is close to definitional.** Acceptance requires
  the frozen validators to pass, and those validators broadly align with
  573/585 official tests. So a low S–A mostly shows validator/test
  agreement, not agent honesty. The Raw S–A is the informative number.
- **The claim-rate datum is less a measure of overclaiming than a
  measure of the claim's absence of content.** For three of seven models
  the claim rate is 100%, so the gap is simply 1 − Pass. The text also says
  "all valid Raw runs end in an accepted ungated claim, Raw S–A equals
  1−Pass". That contradicts Fig. 3 for the other four models (claim rates
  86.2–93.1%, where S–A ≠ 1 − Pass), unless "valid run" silently excludes
  non-claiming runs from the S–A denominator only. The inconsistency is
  unresolved in the text.
- **The checkable/advisory split is a design choice, not a measurement.**
  The paper never reports how many of the 509 directions ended up advisory
  or abstained, and never measures adherence on the advisory side. All 509
  directions are validator-measured units. So "79.6–86.4% satisfied" is
  adherence on the *checkable* surface only. Nothing here corresponds to
  [[literature/papers/nepal2026faithful]]'s dispositional-rule compliance
  rate. If anything, it shows that checkable, source-grounded directions
  from skills *also* go unrealized 14–20% of the time.
- **Most "verifiable" requirements are not mediated.** Only 44% of
  SkillsBench hard obligations qualify for preventive mediation. On
  GuideBench the share is 0%. The rest are checked after the effect.
- **Self-selection risk in the measurement surface.** GPT-5.6 Sol is both
  the frozen compiler that defines the 509 directions and the best-scoring
  task agent on U–E and S–A. Selection was done on a disjoint dev set, but
  whether the compiler's model family is advantaged on its own extraction
  is not examined.
- **Small internal inconsistencies.** Table 5 gives full-SpecHarness Raw-pass
  preservation as 96.8% (≈ 60/62 Raw-passing tasks). Table 17's McNemar row
  for the same model shows only 1 Raw-only discordant task (≈ 98.4%).
  Table 22 gives exact token counts for GPT-5.6 Sol (68,161 / 104,286) but
  round thousands for the other six models. The end-to-end traces are
  "audited protocol replays … not presented as verbatim logs."
- **No released code, compiler prompts, validators or ledger.** The
  architecture is described in detail, but none of it can be rerun.

## Trust signals

- **Credibility:** 3. This is an unreviewed AAAI-template preprint from a
  UT Arlington-led academic group, with no code or artifacts. The
  evaluation protocol is unusually disciplined for the genre. The compiler
  is selected on a disjoint dev set and frozen; validators are qualified
  before task-agent runs; held-out verifiers are kept out of the runtime;
  there are paired bootstrap CIs, Holm-corrected McNemar tests, a
  six-variant ablation, a mutation study, and an explicit statement of what
  the comparison does *not* isolate. It is held at 3 by the lack of
  artifacts, single-run ablations on one model with no intervals, baselines
  that are the authors' own adaptations, and the internal inconsistencies
  above (S–A ≠ 1 − Pass for four models, 96.8% vs one discordant task,
  round-number token counts).

## Follow-up

- **Relevance:** 4. This is the first source under
  [[concepts/evidence-gated-completion]] with a completion gate whose
  refusal drives a repair loop inside a harness, plus an ablation that
  removes the gate while keeping validation. That is the measurement the
  concept's hold has asked for over five cycles. It is a 4, not a 5,
  because the gate's own outcome effect is small (about 3.5 pp, one model,
  one run) and confounded with feedback, and nothing is released.
- **[[concepts/evidence-gated-completion]].** The gate's refusal does change
  what happens next, but most of the downstream outcome gain comes from the
  evidence being *fed back*, not from the refusal itself. The refusal
  mainly buys acceptance integrity. It also adds **freshness** as a gate
  requirement: evidence bound to dependency versions, measured at 248/248
  stale admissions without invalidation.
- **[[concepts/enforcement-boundary-placement]].** This gives a fourth
  head-to-head placement comparison (action-time vs completion-time vs
  post-hoc): action placement cuts divergence and completion placement
  cuts unsupported acceptance, and neither covers both. It also gives a
  *per-obligation* placement rule (closure audit) with a measured split:
  preventive placement is available for only 44% of checkable requirements.
- **[[concepts/typed-enforcement]].** This partly answers the open question
  on false positives of autoformalized policy. Raw-pass preservation is
  96.8% with validator qualification and 90.3% without it, and compiler
  misreading is measured against dev annotations (77/81).
- **[[concepts/refusal-cost-symmetry]].** Admitting unqualified
  deterministic validators into the blocking set buys 1.2 pp less
  unsupported acceptance for 6.5 pp worse preservation. The paper reports
  S–A "with Pass and Raw-pass preservation to expose over-refusal", which
  is this concept's pairing discipline adopted as a metric.
- **Candidates (not in graph).** Nguyen & Tran 2026, *Verify-Gated
  Completion as Admission Control in a Governed Multi-Agent Runtime*
  (arXiv 2605.17998), is the closest prior art for the completion gate
  itself. Also: VIGIL (arXiv 2606.26524, runtime enforcement of behavioral
  specs in agent skills); Tang et al. 2026, verification-gated mission-state
  governance (arXiv 2606.31339); SkillsBench itself (arXiv 2602.12670).
