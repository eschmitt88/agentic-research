---
kind: paper
title: "What Can Be Enforced? A Theory of Certified Runtime Safety for Tool-Using Agents"
authors: ["Shawn Ray"]
institutions: ["Carnegie Mellon University"]
year: 2026
venue: "arXiv 2607.22868v1, cs.AI (preprint, 24 Jul 2026; AAAI-style formatting with proof supplement)"
peer_reviewed: false
url: https://arxiv.org/abs/2607.22868
code_url: https://github.com/shawnray-research/certified-agent-guardrails
citations:
source: "raw/papers/ray2026what.pdf"
added: "2026-08-18"
relevance: 5
credibility: 4
status: read
related_concepts:
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/refusal-cost-symmetry]]"
related_experiments: []
tags: [enforcement, formal-methods, runtime-guardrails, decidability, calibration, conformal, closed-loop, evaluation-integrity, provenance, governance]
---

# What Can Be Enforced? A Theory of Certified Runtime Safety for Tool-Using Agents

## TL;DR

The enforcement cluster's missing **upper bound**. Every other source in
this graph builds a gate and reports its violation rate; this one asks
which policies a gate can enforce *at all*, and answers in three separate
regimes — symbolic (what a deterministic checker represents), statistical
(what a fallible judge can certify), and **closed-loop** (what happens once
blocking changes what the agent proposes next). The third is the one that
bites hardest here: "static scores and ungated trajectories need not
identify the closed-loop frontier," which means measuring a permission gate
on traces collected without it can be *unidentified*, not merely noisy.
Its own experiments demonstrate the effect — gates that reduce attack
success also **create attacks that would not have occurred ungated** — and
the safety it buys costs 78% of calls blocked and a task-success drop from
.254 to .180.

## Claims

- **T1 — the expressiveness boundary.** Relative to a fixed oracle
  predicate interface Π, a deterministic pre-execution gate effectively
  enforces a nonempty policy P *if and only if* P is a **safety** property
  and its good prefixes `Good(P)` are recognized by the gate's register
  model (`Good(P) ∈ RA[T; Π]`). "Effective" = sound (never commits a bad
  prefix) **and** transparent (never blocks or alters an action of a
  compliant stream). Enforcement is witnessed constructively: simulate the
  recognizer, refuse exactly the one-step extensions it rejects.
- **Gates are strictly weaker than edit automata, and irreversibility is
  why.** Witness (Prop. 1): "every `pay` is eventually followed by a
  `confirm`" is a non-safety renewal property an edit automaton enforces by
  buffering `pay` until `confirm` — but no pre-execution gate can. Once the
  irreversible `pay` commits it cannot be retracted, and blocking it breaks
  transparency on the compliant stream `pay confirm`. **Adding substitution
  (`sub(a′)`) does not enlarge the enforceable class**; prefix-safe rewrites
  can only improve utility within it. Human escalation is analogous only
  when modeled as a prefix-safe oracle adjudicating blocks, and no separate
  escalation theorem is proved.
- **T3 — the decidability boundary.** Asking whether a policy is even
  nontrivial (does it forbid anything?) is **undecidable with two
  decrementable zero-test counters**, but lands in **PSPACE** for a
  separable, key-local, constant-guard **monotone** fragment. That
  fragment is not a toy: it is exactly what deployed **caps, quotas, and
  rate limits** use. Registers alone cannot count; unrestricted counters
  are undecidable; monotone per-key counters are the deliberate middle
  ground.
- **T2 — the statistical boundary.** For a *fixed exogenous* one-step
  decision, Neyman–Pearson gives the exact false-block/miss ROC frontier
  attainable by any judge-score-measurable rule (not just thresholds).
  Label-conditional exchangeability yields a finite-sample marginal
  conformal certificate — **possibly satisfied only by the vacuous
  block-all rule**, which the paper treats as a real and frequently
  realized outcome rather than a degenerate case.
- **The closed-loop non-identifiability result — the most importable
  claim.** Once blocking changes future proposals, the system is a
  *controlled* process, and a static ROC no longer describes it. A
  specified finite controlled model yields a belief-state **occupancy
  program** for the true frontier; without that specified kernel, "the
  evaluated policies' outcomes are identified, but the closed-loop frontier
  is not." Measured policy-class gap on an exactly enumerated two-step
  retry model: at δ = .05 the occupancy-optimal cost is **.551 with perfect
  recall**, versus **.863** for current-score/time policies and **1.137**
  for stationary thresholds.
- **Benign calibration does not transfer under attack.** A single
  *predeclared* 16-token "administrator preapproval" suffix inserted into
  the judge-visible string — canonical item and label held fixed —
  increases miss by **.200–.600 for 8 of 11** fully cached judges, with
  simultaneous intervals excluding zero for all 8. Robustness costs an
  explicit margin; a certificate calibrated on benign data does not carry.
- **Provenance sensitivity is real but wildly uneven across judges.** A
  counterfactual source-role diagnostic pairs the culminating call of all
  123 nonempty AgentDojo task plans with **byte-identical action and
  instruction text**, swapping *only* whether the instruction is
  trusted-user input or untrusted tool output — so action-only concordance
  is .5 by construction. Context-rich paired concordance: **.561** ([.533,
  .589]) for the 3B judge, **.967** ([.939, .988]) for a 7B corroboration,
  and **.500** for a 0.5B negative control (Holm-adjusted exact sign tests:
  p < .001, p < .001, p = 1).
- **An evaluation contract for runtime guardrails** — a four-item minimum
  reporting standard implied by the boundaries: (i) the policy language,
  oracle interface, and trusted metadata; (ii) the target **miss/false-block
  operating point**, not AUC alone; (iii) **whether blocking is visible to
  the agent**, with paired reruns that can expose changed trajectories; and
  (iv) the calibration population, exchangeability claim, and adversarial
  margin behind any certificate.
- **AUC is not an actionable target.** It "cannot be inverted into a
  required deployment quality because it does not identify a particular ROC
  operating point." The correct spec is an operating-point constraint —
  e.g. at H = 1, miss ≤ .05 within a .10 false-block budget requires
  `roc(.05) ≤ .10` — "rather than model size or AUC alone."
- **Explicit scope discipline.** "Certified" means *either* streamwise
  enforcement relative to fixed oracle predicates *or* a marginal risk bound
  under stated stochastic assumptions — "not universal semantic safety."
  The paper repeatedly declines to claim one theorem removes all four
  departures (fallible predicates, irreversibility, unbounded data,
  untrusted content) at once.

## Methods

Theory plus four deliberately separate empirical probes, each targeting one
regime. Static diagnostic: **AgentDojo** (97 user-task and 35 injection-goal
instruction strings = 132 items), scored by **23 open, closed, and dedicated
guard judges**; supplementary probes on InjecAgent, AgentHarm, and a
60-call probe. Calibration: 500 shared balanced two-fold splits with a
suite × label outer bootstrap. Controlled model: exact enumeration of the
occupancy polytope of a fully specified two-step retry model. Representation
attack: one predeclared suffix at fixed θ = .5, simultaneous intervals over
11 fully cached judges. Closed loop: a local 7B agent rerun for **949
episodes per condition** with an inline gate, plus a task-paired rerun of
the three injectable suites (N = 389). T1 and T3 are formal results,
instantiated constructively by the running example's policies rather than
validated empirically.

## Results

- **Static discrimination.** AUC .660–.858 across the 23 judges. Paired
  DeLong contrasts against Gemma-2-9B (AUC .813) with max-T intervals and
  Holm checks over 22 comparisons find **no familywise-adjusted
  superiority**; max-T flags only Llama-3.1-8B as inferior (Holm
  sensitivity p = .062). No comparison establishes familywise equivalence
  at the predeclared ±.05 margin, and the median detectable difference is
  .104 — so the data support "non-monotonic in parameter count" and
  support *neither* a tie nor a capacity ceiling.
- **Calibration is mostly vacuous.** At δ = .05, **every** judge's split
  threshold is block-all. For Gemma-2-9B at δ = .10: miss .010 (simultaneous
  upper .021) with false block **.780** ([.660, .901]). At δ = .20 the miss
  point estimate is .138 but its upper bound is .264, "so it is not
  certified by this empirical diagnostic" — an explicit refusal to read a
  low observed miss as proof.
- **Closed loop (C4).** The 3B gate cuts attack success from 45/949 = .047
  ([.036, .063]) to 4/949 = **.004** ([.002, .011], p = 2.2e-9) — while
  gating **78%** of calls. Task success: **.254** ([.227, .283]) ungated,
  .243 with the 7B gate, **.180** ([.157, .206]) with the 3B gate. The
  lighter 7B gate gates 12%, reaches .032, and is **not** significant across
  suites (p = .09).
- **Gates create attacks.** On the paired rerun (N = 389) the 3B gate shows
  **34 baseline-only vs 5 gate-only** attacks (p = 2.4e-6); the 7B gate
  shows **29 vs 22** (p = .40). Nonzero gate-only outcomes are the direct
  demonstration that decisions change transitions — the empirical face of
  the non-identifiability theorem.

## Critique / open questions

- **Safety is bought by over-blocking, and the paper says so first.** A
  78% gating fraction with a 7.4-point absolute task-success drop is not a
  deployable operating point; it is a demonstration that the frontier is
  real. Whether any judge in the surveyed class sits at a *usable* point is
  not established.
- **C4 estimates three selected policies, not the controlled frontier.**
  The paper is explicit: gate-created attacks are exactly why ungated data
  cannot identify that frontier, so its own closed-loop numbers describe
  the policies it ran, not the achievable envelope.
- **The static diagnostic is instruction strings, not context-rich tool
  calls** (stated in-text). The judges are scored on a proxy for the
  deployment task, and the source-role diagnostic is explicitly "not
  natural-trace or deployment-safety accuracy."
- **Attacks lower-bound worst-case margins only.** Robust calibration would
  need a *verified uniform upper bound* over the representation space; one
  predeclared 16-token suffix establishes vulnerability, not the margin.
- **The closed-loop agent is weak.** .254 ungated task success means the
  feedback effects are measured on a system that fails three-quarters of the
  time regardless; whether the same feedback pathology dominates for a
  competent agent is untested.
- **T1/T3 are formal and unvalidated empirically** — appropriate, but it
  means the headline "what can be enforced" rests on a modeling choice
  (the keyed-counter register class) that the paper argues is the right
  middle ground rather than deriving it.
- Single-author preprint. The statistical hygiene is unusually good, which
  partially substitutes for review, but nothing here has been independently
  checked.

## Trust signals

- **Credibility:** 4 — code and cached scores released; **predeclared**
  attack template and equivalence margin; negative controls (0.5B judge at
  exactly .500); familywise (Holm / max-T / DeLong) corrections over 22–23
  comparisons; simultaneous rather than pointwise intervals; and — the
  strongest signal — repeated *refusal to certify* when the upper bound
  doesn't support it ("not certified by this empirical diagnostic"), plus a
  limitations section that leads with its own over-blocking. Held below 5:
  unreviewed single-author preprint, no independent reproduction, and every
  closed-loop number comes from one small local agent.

## Follow-up

- **Relevance: 5** — this is the upper bound [[concepts/typed-enforcement]]
  has been missing. The cluster's existing sources all say "here is a
  checker and here is its violation rate"; T1 says which policies admit a
  checker at all, and the answer — *safety properties with
  register-recognizable good prefixes, strictly below edit automata* — is
  the formal statement of the escape-hatch pattern that concept already
  documents empirically. The `pay`-then-`confirm` witness is the cleanest
  available demonstration that **irreversibility, not model capability, is
  the binding constraint** on a pre-execution gate.
- **The decidability result names which of this box's constraints are
  cheap.** `budget.yaml`'s ceilings (`max_tokens`, `max_wall_hours`,
  `max_experiments`) are monotone per-key counters with constant guards —
  T3's PSPACE fragment, precisely. That is a direct formal argument for
  [[concepts/budget-as-ceiling]]'s shape, and a warning about the one
  ceiling that isn't monotone: `max_consecutive_no_improvement` **resets**,
  making it a decrementable counter, the construct T3 shows tips
  nontriviality into undecidability. Worth stating in that concept.
- **The sharpest warning is for [[concepts/hce-evaluation]].** "Static
  scores and ungated trajectories need not identify the closed-loop
  frontier" says that evaluating a permission gate on traces collected
  without it is not conservative — it is *unidentified*. Every ablation in
  this graph that compares "with harness feature X" to a baseline trace
  recorded without X inherits this problem when X changes what the agent
  proposes next. The gate-only-attack result (5 attacks that existed
  *only* because the gate was there) is the concrete failure. This deserves
  a check in `/lint` or at minimum a stated caveat in the HCE rule.
- **The four-item evaluation contract is directly adoptable.** Item (iii) —
  "whether blocking is visible to the agent, with paired reruns" — is a
  reporting field nothing in this repo currently records for its own
  permission-gate behavior, and it is the one that distinguishes the
  identified case from the unidentified one. A candidate `/elevate` item,
  paired with ding2026autonomous's reporting checklist, which this
  complements exactly: that one covers *research-claim* disclosure, this one
  covers *guardrail* disclosure.
- **"AUC is not an actionable target" generalizes past guardrails.** The
  same non-invertibility applies to any aggregate score this project uses to
  pick a component — an operating-point constraint is the honest spec.
  Relevant to [[concepts/programmable-evaluator-oracle]], which argues for
  deterministic oracles over judge scores; this gives the argument a precise
  form (a judge is admissible only against a stated `roc(β) ≤ α` target).
- **The source-role diagnostic measures something
  [[concepts/permission-gate-as-architecture]] assumes and never tests:
  that a gate can actually read provenance.** Holding action and
  instruction text byte-identical and varying *only* trusted-user vs
  untrusted-tool-output, the 3B judge scores .561 — barely above the .5
  that is chance by construction — while a 7B reaches .967 and a 0.5B
  control sits at exactly .500. Provenance-sensitivity is a **capability
  threshold, not a free property of being a judge**, which means any
  design keying a gate on "did this instruction come from the user or from
  tool output" must verify the checker can see the difference. (Note this
  is *runtime* trust provenance, distinct from the evaluation-time
  boundaries in [[concepts/information-firewall]] — same word, different
  construct.)
- Directly refines [[concepts/permission-gate-as-architecture]] on the
  substitution question: `sub(a′)` — rewrite-instead-of-block — **does not
  enlarge what a gate can enforce**, only its utility. Any design that
  reaches for "rewrite the dangerous call" as a capability upgrade is
  mistaken about what it buys.
- Its `RA[T; Π]` predicate-bit interface ("this interface does not grant
  access to any other semantic fact") is the formal version of the
  minimal-oracle discipline. Cross-reference if a gate-interface concept
  ever ripens.
