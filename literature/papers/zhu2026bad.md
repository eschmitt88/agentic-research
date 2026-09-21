---
kind: paper
title: "Bad Genius: Counterfactual-Guided Harness Evolution Beyond Task-Specific Shortcuts"
authors: ["Guojun Zhu", "Xunheng Huang", "Peng Yin", "Jiahui Xie", "Sanguo Zhang", "Doudou Zhou"]
institutions: ["University of Chinese Academy of Sciences", "National University of Singapore", "Institute of Automation, Chinese Academy of Sciences"]
year: 2026
venue: "arXiv (cs.AI)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.18366"
code_url: null
citations: null
source: "raw/papers/zhu2026bad.pdf"
added: "2026-09-21"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/information-firewall]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/evolutionary-expansion]]"
tags: ["evaluation", "benchmark-validity", "harness-optimization", "shortcut-learning", "counterfactual", "metamorphic-testing", "protocol-holdout", "adversarial-subagent", "constrained-search", "theory"]
---

# Bad Genius: Counterfactual-Guided Harness Evolution Beyond Task-Specific Shortcuts

## TL;DR

Every holdout discipline in this graph varies **tasks**. This paper names
the axis nobody splits: the **protocol** — file names, directory layout,
metadata, tool aliases, demonstration order, feedback format — which is
shared by the search set and the test set alike. A harness optimizer that
reads a released benchmark's scores and traces can therefore write a
correlation that holds across *every* task into the harness code, and no
task split can see it. Their instance: **58.1%** of OfficeQA Full
questions mention a numerical scale, and in its 697-document corpus the
next nonblank line after **95.2%** of unit statements begins a table — so
*read just above the table* is a benchmark-wide rule, not a per-task
cheat. CHASE adds a **Challenger** that, after each Proposer round,
searches for an executable, firewall-checked transformation of the
protocol that destroys the harness's *gain over H0*; a confirmed
transformation enters a finite archive and becomes a feasibility
constraint on all later rounds. The measured reversal: a first-round
harness gains **+8.16%** on the released protocol and **−5.10%** once the
same retrieved text is re-nested as table context, with questions,
answers, documents, budgets and scorer all unchanged. Three theorems
convert the finite archive into bounds on the unknown exact neutralized
benchmark B0. The catch is the scale of the real-benchmark evidence:
across three rounds on OfficeQA **exactly one** counterfactual was ever
confirmed, and CHASE's certification score on the released protocol is
**68.86%** against **67.98%** for changing nothing at all.

## Claims

- **Two shortcut classes, formally separated.** Let ∆_TS be the excess
  search-set gain that task holdout measures and ∆_BS the gain that
  disappears under exact shortcut neutralization. "A benchmark-wide
  shortcut can persist in both Ds and Dh because both use the same
  (Vrel , Qrel ). Therefore ∆TS (H; H0 ) ≈ 0 does not imply ∆BS (H; H0 )
  ≈ 0." Passing a held-out-task check is evidence about one axis only.
- **The optimizer is the bad actor, not the agent.** The target agent is
  fixed. The Proposer edits "prompts, memory, retrieval, tools, and
  control code" against benchmark feedback, and "Harness evolution adds an
  executable route: the Proposer can write the shortcut into the agent’s
  protocol." This is a different actor from
  [[literature/papers/ludwig2026shortcutting]]'s agent-cloning-the-repo
  and from [[literature/papers/roth2026hack]]'s planted hacks.
- **Existing defenses are the wrong shape.** "Existing methods ban task
  identifiers, filenames, and per-task repair recipes, then evaluate on
  held-out tasks" Those are fixed filters over *harness edits*; "the
  latter can only cover shortcut patterns anticipated by the filter."
- **The estimand is gain destruction, not score change.** ∆_b(H; H0) =
  G_rel(H; H0) − G_b(H; H0). A protocol change usually moves the evolved
  *and* the initial harness, so the score delta is uninformative; what
  matters is how much of the *advantage over H0* survives.
- **Validity is executable, never argued.** Seven Boolean invariants
  (task and answer, available information, access and resources, scoring,
  declared changes, task independence, replay and inversion) must all
  pass. "The Challenger’s natural-language validity claims are retained in
  the audit record but are not used as substitutes for these executable
  checks."
- **A confirmed counterfactual becomes a constraint, not a penalty term.**
  The Proposer's later rounds are a constrained maximization: highest
  released gain subject to ∆̂_b ≤ ε for every archived Bb. Final selection
  re-applies the whole archive to every candidate ever evaluated, because
  "a high-scoring harness may violate counterfactual constraints
  discovered in later rounds."
- **Finite-archive guarantees.** Theorem 1 bounds ∆_BS with respect to the
  unknown exact B0 in terms of the archive's worst observed gain
  destruction, scaled by K(A) — the smallest number of archived
  transformations whose composition *is* B0 — and ρ, the worst-case excess
  destruction under composition. If B0 ∈ A then K(A)=1 and every ρ term
  vanishes. Theorem 2 bounds the number of confirmations by (C/(γ−r))^d,
  so admissions cannot continue forever. Theorem 3 makes a *non*-confirmed
  round an approximate stopping certificate over the whole valid family.

## Methods

- **Roles and loop.** Fixed target agent A; Proposer edits the harness
  against Devo; Challenger searches Ddisc for a transformation Φ_b
  maximizing ∆̂_b(Ht; H0); host-side validity firewall + duplicate screen;
  fresh confirmation on Dconf,t; archive update; repeat. Sealed Dcert is
  "accessed only after the final harnesses, A3 , and the analysis code are
  fixed."
- **Four disjoint task roles, not two.** Devo, Ddisc, Dconf,1..T, Dcert
  are mutually disjoint. Adding an adversary to the loop costs two extra
  splits, because the adversary’s own search overfits — the paper notes
  that because "the Challenger searches over Ddisc", a large discovery-set
  gain destruction "may be inflated by selection".
- **OfficeQA.** 246 questions over a 697-document U.S. Treasury Bulletin
  text corpus, 1939–2025; a separate Pro V2 release of 90 questions over
  1,435 parsed documents. Split 49 / 49 / 3×24 / 76 by *source component*
  (100 components from 296 unique source filenames), not at random, so
  shortcut reproduction can be tested both within and beyond the sources
  used for discovery. Scoring is the released fuzzy-match function at 0%
  numerical tolerance, held fixed across every Bb.
- **Syn-Ledger.** 320 synthetic multi-document arithmetic tasks, five task
  families, 12 documents each (2 evidence + 10 structurally matched
  distractors), ≤12 tool calls. Five observable features — filename,
  directory depth, search rank, candidate-record position, serialization
  — each get a favorable and an unfavorable level, stored in a rendering
  manifest separate from the ledger contents. Under Brel, "56/64 = 7/8 of
  the evidence documents have the favorable level for each feature";
  under B0 the features are balanced and every 5-feature configuration
  occurs equally often. Five canonical neutralizers commute and compose to
  B0 exactly, so K(A) is knowable and a **B0-Access** comparator exists.
  Five placebo transformations audit the construction. Gold answers are
  recomputed from both canonical records and rendered documents under all
  twelve variants and must agree; both evidence documents are verified
  necessary.
- **Budget and thresholds.** Three Proposer rounds for all optimized
  methods. OfficeQA: ε = 0.05, η_conf = 0.075; the Challenger emits eight
  proposals per round, the firewall and duplicate screen run *before any
  rollout*, at most two survivors are scored on Ddisc, and the winner is
  frozen for "2 harnesses × 2 benchmarks × 24 questions × 5 rollouts = 480
  confirmation runs."
- **Baselines.** RawHarness (H0, unchanged). HarnessCompass — the same
  Proposer backbone under that paper's **fixed Generalization Gate**, a
  prompt-level rule admitting only task-agnostic edits, here re-specified
  by these authors for OfficeQA. B0-Access (Syn-Ledger only) selects
  candidates directly on B0.
- **Models.** "all methods use gpt-5.6-sol with high reasoning effort for
  every model role, including the target agent, Proposer, and Challenger."
  Syn-Ledger drops to the same model in the non-thinking setting for cost.

## Results

- **OfficeQA certification (Table 1; 76 sealed questions, 3 rollouts,
  every benchmark in A3).** RawHarness 27.04 / 67.98 / 66.23 / 64.47;
  HarnessCompass 26.30 / 64.04 / 63.16 / 62.28; CHASE 30.37 / 68.86 /
  68.42 / 67.98 — columns are Pro V2, released protocol, archive average,
  archive worst case. So CHASE beats doing nothing by **+0.88 points** on
  the released protocol and by **3.51 points** on the archive worst case,
  and "HarnessCompass performs below the initial harness"
- **The round-1 reversal.** H1 adds prompt reminders, unit/precision
  guidance, and a conditional query rewrite. On Devo its gain is +8.16%
  under Brel and **−5.10%** under Bb1 (Table C.4, ∆̂ = 13.27%). Φ_b1 moves
  every text-bearing search result's string into
  `table_context.content` with `placement: "before"` and deletes the
  original `text` field — "no table parsing or within-string reordering is
  performed" and documents, counts, budgets, questions and scoring are
  untouched.
- **The constraint bites, and costs raw gain.** Round 2 produced three
  candidates at +15.31% / +9.18% / +15.31% released gain, with ∆̂_b1 of
  6.12% / 6.12% / 17.35% — all above ε = 0.05, all rejected, and "the
  Proposer returns to H0 while retaining feedback on these failures."
  Round 3's integrated candidate passes at +4.08% released gain with
  ∆̂_b1 = 4.08% and is selected as H3. The archive-feasible harness
  therefore gives up roughly **11 points of search-set gain**.
- **The adversary overfits its own split, measurably.** Round 3's
  Challenger found a ten-line passage-boundary shift with 10.20% gain
  destruction on Ddisc; on the frozen Dconf,3 (24 questions × 5 rollouts)
  "the recorded gain destruction is 4.08%, which is below the 7.5%
  confirmation threshold. The transformation is therefore not added to the
  archive." Discovery-set estimate ≈ 2.5× the confirmation estimate.
- **The firewall fires before any rollout.** Round 1: "the Challenger
  first returned eight typed transformation proposals. The host-side
  firewall accepted seven and rejected one before any target-agent rollout
  on Ddisc ." The rejection was for **inactivity**, not invalidity: a
  seeded permutation of search results that left the order unchanged.
- **Syn-Ledger (Table 2; 208 certification tasks).** R̂_rel / R̂_0:
  RawHarness 81.89 / 15.22, HarnessCompass 80.45 / 14.74, B0-Access
  83.01 / 32.69, CHASE 87.18 / 38.30. Ĝ_rel −1.28 / +1.28 / +5.29 and Ĝ_0
  +2.24 / +20.19 / +23.08 for the three optimized methods, with ∆̂_BS
  −3.53 / −18.91 / −17.79. CHASE, with no access to B0 during
  optimization, reaches a **higher R̂_0 than the B0-Access comparator that
  optimizes on B0 directly**. The ∆̂_BS values are negative because the
  gain is *larger* under B0, not because B0 is easier — B0 scores far
  lower in absolute terms. (This table and its surrounding paragraph sit
  on a page whose embedded font breaks text extraction; see the critique.)
- **Cost.** At certification CHASE is cheap: 98.15M total tokens vs
  HarnessCompass's 118.50M and RawHarness's 93.82M, and "CHASE used the
  fewest uncached input plus output tokens: 22.39 million"; median wall
  time 22.9 min/run against 20.7 and 24.5. During *search* it is not:
  "Compared with HarnessCompass, CHASE runs 2.47 times as many attempts
  and uses 2.43 times as many total tokens."

## Critique / open questions

- **The real-benchmark result is one confirmed counterfactual.** Across
  three OfficeQA rounds, only Bb1 ever entered the archive, so |A3| = 2
  (the certification budget is stated as "Across two benchmarks and 76
  questions, this yields 456 evaluation runs per method" — i.e. 2 × 76 × 3). Round 3's
  candidate failed confirmation; round 2's Challenger is not described at
  all. Every OfficeQA claim about "confirmed counterfactuals" plural rests
  on n = 1.
- **The headline gain is +0.88 points, single-run.** 68.86% vs 67.98% on
  76 questions × 3 rollouts. There is not a single confidence interval,
  seed replicate, or significance test anywhere in the paper. The archive
  worst-case gap (67.98 vs 64.47) is the more defensible number, and it is
  partly definitional: CHASE selected its harness *to satisfy* the
  constraint that R̂_min,A3 measures.
- **∆_TS is defined and never measured on OfficeQA.** The paper's whole
  motivating claim is that task holdout misses this. It builds the formal
  separation and then runs no task-holdout arm — no experiment shows a
  harness with ∆_TS ≈ 0 and ∆_BS large on a real benchmark. The only place
  both quantities are observable is Syn-Ledger, where the shortcut was
  planted by the authors.
- **On OfficeQA the shortcut is hypothesized, not demonstrated.** The
  95.2% unit-before-table statistic is a corpus regularity, not evidence
  that H1 used it; the paper says "We therefore hypothesize that the
  regular placement of such context in OfficeQA may itself signal that
  enough evidence has been found." What is actually shown is that H1's
  gain is fragile to a protocol change. Fragility is weaker than
  shortcut-dependence: a harness tuned on one evidence representation and
  scored on another can lose its gain without any cue-exploitation.
- **Φ_b1 edits the harness's own tool-output schema, not the corpus.** It
  renames and re-nests fields in the agent-visible JSON search results.
  Under the paper's definition Q_rel ("how the agent interacts with these
  components") that is protocol, but a skeptic can read the counterfactual
  as *breaking the interface H1 was written against* rather than
  neutralizing a benchmark-wide cue. The Syn-Ledger manipulations
  (filename, directory depth, search rank, record position,
  serialization) do not have this problem, which is exactly why the
  synthetic half carries the argument.
- **The main baseline is a re-implementation on a domain its authors never
  used.** The paper says so: HarnessCompass "evaluates this design on
  SWE-bench Verified but does not report an OfficeQA-specific Gate" and
  these authors "construct one OfficeQA-specific Generalization Gate
  before optimization." So *prompt-level gate underperforms doing nothing*
  is suggestive about *where* the constraint should live, not a clean
  head-to-head.
- **Table 2's R and G columns do not reconcile for two of four rows.**
  CHASE checks exactly (87.18 − 81.89 = 5.29; 38.30 − 15.22 = 23.08), but
  HarnessCompass and B0-Access are each off by 0.16 on Ĝ_rel and by 2.72
  on Ĝ_0, in the same direction. Caveat on the caveat: that table sits on
  a page whose embedded font breaks text extraction (decimal points and
  minus signs are dropped — the raw string is `81 89%` etc.), so the
  digits are trustworthy but the reconstruction is mine. Worth a re-read
  from the rendered PDF before the numbers are quoted downstream.
- **One model in all four roles.** Target agent, Proposer and Challenger
  are all gpt-5.6-sol. A Challenger that shares the Proposer's inductive
  biases is the least likely to find the shortcuts the Proposer prefers —
  the same self-audit worry as
  [[literature/papers/zhang2026double]], one level up.
- **No code.** The paper releases pseudocode for the Syn-Ledger firewall
  (Table B.3) and prose for the OfficeQA one, and points only at the
  public OfficeQA HuggingFace datasets. Nothing here is re-runnable.
- **The Challenger is a second overfitting surface the paper handles
  well.** Discovery / confirmation separation plus a duplicate hash over
  canonicalized operator sequences plus a threshold η > ε is a complete
  design, and they report the case where it *rejected* their own finding.
  That candor is the strongest credibility signal in the paper.
- **Coverage is admitted to be budget-bound.** The Discussion concedes that the
  claim is relative to the evaluated counterfactual family, whose coverage
  is limited by the three-round search budget. (That page’s embedded font
  breaks text extraction, so this is a reading of the rendered text, not a
  quotable string.) With ρ and K(A)
  both unestimated on OfficeQA, Theorem 1 gives no numeric bound there —
  the theory is exercised only where B0 is constructed by hand.
- **Open question for this graph.** Nobody has asked what the protocol
  surface *is* for an ML-research benchmark. For MLE-bench/AIDE it would
  be something like: the competition directory layout, the name of the
  submission file, whether `train.csv` always precedes `test.csv` in a
  listing, the format of the grader's feedback string. A CHASE-style
  Challenger for that setting is a concrete, unbuilt experiment.

## Trust signals

- **Credibility:** 3. Statistics and applied-math groups at NUS, UCAS and
  CAS Institute of Automation — reputable, no track record in this graph,
  arXiv v2, not peer reviewed, no code. Raised to 3 by unusually careful
  construction: a sealed certification set opened once, source-component
  rather than random splits, seven executable validity invariants with
  positive and negative controls, a synthetic benchmark whose ground truth
  is recomputed under all twelve variants, three theorems with stated
  conditions, reported token and wall-time cost, and a candidly reported
  non-confirmation that rejected their own round-3 finding. Held at 3
  because the real-benchmark evidence is a single run with one confirmed
  counterfactual and a +0.88-point headline, there are no confidence
  intervals anywhere, the motivating ∆_TS-vs-∆_BS contrast is never
  measured on a real benchmark, the main baseline is the authors'
  re-implementation on a domain its originators never evaluated, and
  Table 2 has an unexplained internal inconsistency.

## Follow-up

- **Relevance:** 4. It adds a genuinely new axis to the graph's most
  load-bearing concept and touches four others. It is not a 5 because the
  domain is document-grounded financial QA rather than ML research, the
  real-benchmark demonstration is thin, and the mechanism it names is
  currently a hypothesis with one supporting reversal.
- **[[concepts/hce-evaluation]].** That concept's every defense — hidden
  test split, evidence-gated completion, external oracle validation,
  gated-trace identification, scaffold ownership, sealed-test
  certification — varies **tasks** while holding the protocol fixed. This
  is the first source in the graph to name the protocol as a distinct leak
  axis and to give the constructed-not-hidden repair: transform the
  protocol under executable validity checks and measure how much of the
  *gain over the initial system* survives. It composes with
  [[literature/papers/zhang2026double]] rather than repeating it — that
  paper asks whether the scaffold owns the decision; this one asks whether
  the scaffold was *tuned to a layout the test split also has*. Both
  failures are invisible to a clean task holdout.
- **[[concepts/information-firewall]].** Two additions. (1) A **fourth
  boundary**, after file space, retrieval space and time: **protocol
  space**. The leaked quantity is not content at all — it is layout,
  ordering and schema, and nothing is withheld, so no curation or refresh
  loop defends it. (2) The firewall has to cover **the red team too**. An
  adversarial subagent searching for a violation needs its own disjoint
  split and its finding needs a fresh confirmation split, with the
  confirmation threshold strictly above the constraint tolerance
  (η = 0.075 > ε = 0.05). Their one measured instance of adversary
  selection inflation: 10.20% on the discovery set, 4.08% on the frozen
  confirmation set.
- **[[concepts/programmable-evaluator-oracle]].** The object graded here
  is a **benchmark transformation**, not an answer — a use of the
  deterministic-oracle pattern the concept has not recorded. Two
  transferable details: the invariant list includes *scorer hash equality
  and scorer verdicts on fixed correct and incorrect output fixtures*, so
  the oracle checks that the oracle did not move; and replay/inversion
  (render twice, require byte equality, require the inverse to recover the
  original records) catches nondeterministic transformations. The
  "inactive transformation" rejection — a permutation that permuted
  nothing — is a class of failure an LLM-judge validity check would not
  catch, since the proposal's *description* was valid.
- **[[concepts/enforcement-boundary-placement]].** A new placement, and a
  new threat model: the constrained component is a subagent *we spawned on
  purpose* to attack our own measurement. The rule is stated cleanly — the
  proposing agent's natural-language validity claim is kept in the audit
  record and never substituted for the host's executable checks, and the
  checks run before any expensive rollout. The concept's "the constrained
  component cannot reach the constraining one" design rule holds here:
  Ht and H0 are read-only to the Challenger, and the firewall is
  host-side.
- **[[concepts/evolutionary-expansion]].** The archive here **prunes** the
  population rather than diversifying it — the inverse of the islands /
  MAP-Elites / novelty-archive machinery that concept records. And the
  trade is measured: the archive-feasible harness took +4.08% search-set
  gain where infeasible candidates offered +15.31%, and that smaller gain
  is what transferred to a separate corpus (Pro V2: CHASE 30.37%,
  RawHarness 27.04%, HarnessCompass 26.30%).
  Relevant to this project's own `/elevate` loop, which selects proposals
  against a fitness signal with nothing sealed and no adversary.
- **Independence.** Converges with [[literature/papers/ludwig2026shortcutting]]
  and [[literature/papers/roth2026hack]] on *explicit prohibition covers
  only the channels you named*, reached from a third direction: not a
  cheating agent but a cheating *optimizer*, and not a prohibition but a
  generated counterfactual. Its literature is almost entirely outside this
  graph — Meta-Harness (Lee et al. 2026), Harness-Bench (Yao et al. 2026),
  HarnessCompass (Zhang et al. 2026), HarnessEvolve (Jiang et al. 2026),
  HarnessLens (Xu et al. 2026), AutoSaddler (Park et al. 2026), Auditing
  Harness Tampering (Wang et al. 2026a), HackProbe (Yang et al. 2026),
  and Cai et al. 2026's "Safe harness self-evolution: A theoretical
  analysis of feasibility and limits" (arXiv:2609.08175). That last one
  and Harness-Bench are the two best digest candidates; the graph has
  [[literature/papers/esakkiraja2026starharness]] from the same cluster
  already.
