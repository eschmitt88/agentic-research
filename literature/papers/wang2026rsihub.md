---
kind: paper
title: "RSIHub: A Composable Framework for Agent-Evolution Methods"
authors: ["Zimu Wang", "Xiaobo Wang", "Siyu Ye", "Jiaqi Li", "Silin Chen", "Yuling Shi", "Ruobing Wang", "Jingjing Zhang", "Chaofan Wang", "Kai Wu", "Mengnan Qi", "Kai Cai", "Xiaodong Gu", "Zilong Zheng"]
institutions: ["University of California, Berkeley", "University of Science and Technology of China", "Douyin Group", "Shanghai Jiao Tong University", "Tongji University", "Beijing Institute for General Artificial Intelligence", "Simple Agent Lab"]
year: 2026
venue: "arXiv 2610.32253 (cs.AI), v1 2026-10-23, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.32253"
code_url: "https://github.com/simple-agent-lab/RSIHub"
citations: null
source: "raw/papers/wang2026rsihub.pdf"
added: "2026-10-27"
relevance: 3
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/information-firewall]]"
  - "[[concepts/programmable-evaluator-oracle]]"
tags: ["agent-evolution", "harness-evolution", "rsi", "framework", "ahe", "gepa", "hyperagents", "a-evolve", "terminal-bench", "tau-bench", "mutable-surface", "certified-evaluation", "provenance", "train-gate-sealed", "single-run", "seed-draw-noise", "fault-injection", "harbor"]
---

# RSIHub: A Composable Framework for Agent-Evolution Methods

## TL;DR

RSIHub is an open-source (Apache-2.0) framework. Each agent-evolution
method is written as a **recipe**: a YAML file that binds one operator to
each lifecycle slot (Select, Rollout, Analyze, Mutate, Validate, Novelty,
Gate, Record, Reflect) and declares the **mutable surface**. The framework
owns everything else. It builds workspaces, checks the real tree diff
against the surface, commits the candidate, scores a clean checkout with a
frozen Harbor evaluator, and writes framework-certified rows to an
append-only archive that operators cannot overwrite. The authors adapt
AHE, Hyperagents, A-Evolve and GEPA as recipes (47–61 recipe lines, 216–1,087
operator lines, zero framework-file changes) and run each once on two seed
agents (MiniSWE, Codex) × two benchmarks (Terminal Bench 2, τ³ Banking).
GPT-5.4 is the target model and the mutation model in every cell.

Three results matter for this graph:

1. **No method ranking survives a change of agent or benchmark.** The
   largest overall gain, +13.5 pp, is MiniSWE on Terminal Bench 2 (56.2 →
   69.7, Hyperagents and A-Evolve). About 8 of those points come from the
   train and gate splits, which the loop optimized or selected on. Sealed
   gains range from +5 tasks of 27 (Codex, τ³, AHE) to −4 of 27 (MiniSWE,
   τ³, AHE). Each of the 16 cells is **one evolution run and one trial per
   task**. The paper says outright that these are descriptive point
   estimates.
2. **The paper's own noise control is as large as most of the gains.** An
   unevolved seed run three times directly and three times through RSIHub
   disagrees with itself on 12.0–18.9% of tasks. The single seed baseline
   in Table 1 sits up to 8.3 points from the seed's own three-run RSIHub
   mean (our arithmetic). On Codex τ³ the best evolved score (34.0) is
   about the same as the unevolved seed's RSIHub mean (33.0).
3. **The evaluation boundary is real infrastructure, tested only against
   scripted faults.** Seven fault and boundary scenarios × 30 trials × 2
   campaigns (420 trials) all came out correct: evaluator edits, writes
   outside the surface, reading protected splits, and forged archive rows.
   These are injected faults, not an optimizer looking for a way through.
   The authors also say the framework "does not contain hostile code", and
   its guarantees assume "valid evaluators".

## Claims

- Agent-evolution systems share one lifecycle: pick a parent, gather
  evidence, propose and evaluate a revision, keep the outcome. Most
  implementations bundle that lifecycle with method policy, so every new
  method rebuilds the runner, and comparing methods means reconciling
  runners.
- "Keep scoring out of the method's reach." Evaluation and the evidence
  record sit outside the declared mutable surface. Method-controlled
  changes to evaluation or record keeping "can make an apparent gain
  ambiguous".
- Parent eligibility is a separate policy (Gate) downstream of the
  certified score. Gate, Record and Reflect "may add policy outputs but
  cannot alter certified evaluation fields".
- The four recipes are "interface adaptation[s] at the level of operator
  choices and configuration". Exact correspondence with the original
  systems "also depends on its prompts, models, budgets, dependencies, and
  runtime adapters". These are adaptations, not reproductions.
- "No method dominates across benchmarks, seeds, and splits." A higher
  overall score "can conceal held-out losses."
- Framework overhead is negligible: a median 0.02% of generation time
  (Terminal Bench 2) and 0.019% (τ³ Banking).
- An agent-driven mode lets an outer agent rewrite the recipe between
  iterations, while the evaluator, splits and surface stay fixed. **It is
  described, not evaluated**: "Our experiments evaluate fixed recipes."

## Methods

- **Lifecycle (Alg. 1).** Select (framework verifies the parent and forks
  its recorded commit) → Rollout on train tasks → Analyze* → Mutate (an
  agent, multi-turn, may trial on train tasks) → Guardrail (framework diff
  check against the surface; then Validate* and Novelty*) →
  CommitReviewedTree → Evaluate on gate tasks with a hash-referenced
  contract → Certify → Gate → Record (annotations filtered against
  protected fields) → Reflect*. Rejections, empty proposals and surface
  violations close as typed, archived attempts.
- **Comparison identity (Table 8).** A candidate can be selected as parent
  only if its evaluation matches the run's evaluator-tree fingerprint,
  task-set hash and runtime fingerprint. It must also have a certified
  receipt, a finite score and a generation tag that matches the recorded
  commit. Sealed evaluation is a separate, non-selectable "anchor" kept out
  of the mutation-feedback bundle. The mutate config exposes
  `expose_gate_data: false`.
- **Operators inside the surface.** When a recipe puts operators inside the
  mutable surface, as Hyperagents does, each stage loads from the checkout
  given to it. Edits to Validate or Novelty can therefore affect the
  current proposal, and "Gate and Record subsequently load from the
  committed child". In that recipe the candidate's own code makes its
  admission decision. The certified score is still protected.
- **Setup.** Seeds: MiniSWE 2.4.5 and Codex CLI 0.143.0, both on GPT-5.4
  (high reasoning). Mutation agent: Codex on GPT-5.4 (xhigh) for every
  recipe. Terminal Bench 2 is split 50/19/20 train/gate/sealed (89 tasks);
  τ³ Banking 50/20/27 (97). One child per generation, at most 10
  generations, static task sampling, **one evaluation per task**. Table 1
  split columns are each split's *contribution* to full-benchmark pass@1,
  so one task is 1.12 pp (TB2) or 1.03 pp (τ³). The appendix recipe has
  `budget_usd: 150`. Tokens, dollars and the number of generations actually
  run per cell are not reported.
- **Fidelity control (Table 2).** For each benchmark × agent, three
  repetitions of Direct-A → RSIHub generation 0 → Direct-B. The paper
  reports Avg@3 pass@1, task-level D–D and D–R agreement, and "excess
  disagreement".
- **Integrity (Table 3).** 7 scenarios, each injected 30 times per
  campaign: terminate during mutation, during evaluation, or after the
  receipt; modify evaluator files; write outside the surface; access
  gate/sealed/anchor canaries during mutation; forge an archive outcome. A
  trial passes if the state is valid and attributable, unauthorized changes
  cannot affect an accepted result, and the champion is reproducible from
  framework evidence.

## Results

- **Table 1, pass@1 (one run per cell).** Overall Δ vs seed, with sealed Δ
  in tasks (our conversion from contribution points):

  | Bench | Agent | Seed | AHE | Hyperagents | A-Evolve | GEPA |
  |---|---|---|---|---|---|---|
  | TB2 | MiniSWE | 56.2 | +9.0 (sealed +4/20) | +13.5 (+5) | +13.5 (+5) | +7.8 (+4) |
  | TB2 | Codex | 69.7 | +2.2 (+2) | +1.1 (+1) | +1.1 (0) | +0.0 (0) |
  | τ³ | MiniSWE | 27.8 | +1.1 (−4/27) | +5.2 (−2) | +0.0 (0) | +2.1 (−2) |
  | τ³ | Codex | 24.7 | +9.3 (+5) | +4.2 (+4) | +4.2 (+3) | +8.3 (+4) |

  The best method is Hyperagents/A-Evolve (TB2 MiniSWE), AHE (TB2 Codex),
  Hyperagents (τ³ MiniSWE) and AHE (τ³ Codex). On TB2 the gate split, which
  is the selection split, is flat or down in 6 of 8 cells (Codex AHE −3.4,
  Hyperagents −2.2).
- **Table 2, the unevolved seed (3 reps).**

  | Bench | Agent | Direct Avg@3 | RSIHub Avg@3 | Table 1 seed (1 run) | D–D agreement |
  |---|---|---|---|---|---|
  | TB2 | MiniSWE | 59.6 | 56.9 | 56.2 | 83.5% |
  | TB2 | Codex | 64.0 | 61.4 | 69.7 | 88.0% |
  | τ³ | MiniSWE | 24.6 | 22.0 | 27.8 | 81.1% |
  | τ³ | Codex | 29.6 | 33.0 | 24.7 | 82.1% |

  The six direct Codex TB2 executions span 60.7–65.2. Table 1's Codex TB2
  seed (69.7) is above all six, and the Codex τ³ seed (24.7) is 8.3 below
  its RSIHub mean. The paper does not set the two tables side by side.
  That comparison is ours.
- **Integrity.** 420/420 trials correct, with no corrupted state admitted.
  Median handling latency 0.23–5.15 s.
- **Overhead.** Framework spans are 0.02% (TB2 median) and 0.019% (τ³) of
  generation time. Model, mutation and benchmark execution dominate.

## Critique / open questions

- **Against the digest framing: below framing (mild).** The digest quotes
  the abstract correctly and flagged "evidence depth uncertain". What it
  calls "the controlled head-to-head the field lacks" is controlled on
  model, evaluator, splits and seed artifact. It is not controlled on
  budget or replication:
  - **One run per cell, one trial per task, one seed baseline draw.** The
    paper's own fidelity control shows how far one draw of the unevolved
    seed can move: within-cell D–D disagreement of 12–19% (cf.
    [[literature/papers/liu2026what]]'s 13% rerun flip rate on SWE-bench).
    The single baseline sits up to 8.3 points either side of its own three-run
    RSIHub mean. The sign of the sealed deltas follows the baseline draw
    in 3 of 4 (bench, agent) groups. Where the seed drew low (Codex τ³),
    all four methods "gain" on sealed (+3 to +5 of 27). Where it drew high
    (MiniSWE τ³), three of four "regress" (−2 to −4). Codex TB2 is the
    exception (high draw, small gains). Our reading: a large part of the
    method-ranking variation the paper headlines is regression toward the
    seed's mean, not policy. The clearest gain is TB2 MiniSWE. There the
    seed draw is near its mean (56.2 vs 56.9), and all four methods add
    4–5 of 20 sealed tasks. With ~16% discordance a 20-task rerun has a
    net-change SD of about 1.8 tasks (our arithmetic), so +4/+5 is
    probably real, but the four arms share the one baseline.
  - **The headline +13.5 pp is mostly in-sample.** "Overall" sums the
    train split (rollout feedback), the gate split (admission) and the
    sealed split. For A-Evolve, 7.9 of its 13.5 points are train.
    Held-out-only, the largest gain is +5.6 contribution points (5 of 20
    TB2 tasks).
  - **Budget is matched in generation count only.** At most 10
    generations and one child per generation, but GEPA runs an extra
    minibatch Validate, and Mutate may trial on train tasks. No token or
    dollar totals per cell are given, so by [[literature/papers/oved2026llm]]'s
    standard the methods are not compute-matched. There is no same-budget
    search arm on the seed (best-of-K or pass@K at the loop's rollout
    spend), which [[literature/papers/wang2026rethinking]] showed erased
    AHE's held-out gain on Terminal-Bench 2.1. There is no neutralization
    arm ([[literature/papers/ding2026harnessdelta]]), and the sealed split
    is drawn from the same benchmark.
  - In [[literature/papers/chen2026auditing]]'s codebook this is a
    same-start, product-claim comparison with no stated budget match and
    one run of each loop. The framework makes the missing arms cheap. It
    does not run them.
- **What "evaluation outside the mutable surface" buys.** It implements
  HCE clause 4's tamper half as infrastructure, and does it well. The
  frozen evaluator tree is fingerprinted into every row. A parent must
  match the run's comparison identity, so a candidate scored under drifted
  conditions cannot be selected. Operator annotations are filtered before
  they reach the archive. Three limits:
  1. **The tests are scripted faults, not search pressure.** 30 identical
     injections of each scenario test regression, not robustness.
     [[literature/papers/sanayei2026evidencefirst]]'s caution applies:
     a suite built next to the mechanism it tests covers that mechanism's
     own cases. No arm lets an evolving Hyperagents candidate, whose
     surface includes operators, try to reach the score.
  2. **Admission can sit inside the surface.** With operators in the
     surface, Gate loads from the committed child, so a candidate's own
     code decides whether it becomes a parent. The score fields are
     protected, but the acceptance rule is not. The paper does not discuss
     this. Our inference: under Hyperagents the "separate Gate operator"
     is a candidate-authored policy over a protected number.
  3. **Unchanged is not correct.** The guarantees assume "valid
     evaluators". Harbor's per-task verifiers are taken as given, and no
     do-nothing or known-answer submission is scored. This is the exact
     gap [[literature/papers/chen2026when]] found in τ-bench's own
     grading: a pristine evaluator that was wrong.
- **The protected-evidence test checks the bundle, not the network.** The
  canary scenario confirms that gate and sealed material never enters the
  mutation bundle. Terminal Bench 2 and τ-Knowledge are public, and the
  paper does not say whether the Codex mutation agent had network or
  filesystem access to the dataset. A bundle firewall does not stop an
  agent from fetching a public benchmark (cf. liu2026what: 312 of 439
  web-enabled trials fetched the upstream fix).
- **Host execution.** "Editable operators and candidate adapters may
  execute on the host, beyond task-container isolation." The authors say
  so plainly, and it means the integrity claims hold only against
  non-adversarial code.
- **Sealed selection.** The champion is "the best accepted candidate".
  The paper does not say by which score, though it is presumably gate. It
  also does not say whether Table 1's train and gate numbers are a fresh
  final run of the champion or the scores recorded during search. If they
  are recorded scores, the gate column carries the winner's curse, and the
  flat gate results are even weaker.
- **Fit with the graph.** This is the second Terminal-Bench harness-
  evolution datum after wang2026rethinking (held-out +0.0 for AHE, mean of
  2 runs). Here MiniSWE gains 4–5 sealed tasks of 20 and Codex gains 0–2.
  The paper reads the Codex result as "gains are smaller from the stronger
  Codex baseline". Part of that strength is a high single draw (69.7
  against a three-run mean of 61.4), so the scaffold-strength reading is
  confounded with baseline noise (our inference). The
  Fork-and-Flush ([[literature/papers/cai2026forkandflush]]) inspection
  workflow is cited as the intended use of the viewer.

## Trust signals

- **Credibility:** 3.
  - For: several groups (Berkeley, SJTU, BIGAI, ByteDance/Douyin, USTC,
    Tongji), code released under Apache-2.0 with runnable recipes
    (github.com/simple-agent-lab/RSIHub, named in the PDF; not opened at
    ingest), and pinned agent versions with content-pinned benchmark
    assets. A real fidelity control with three repetitions and two direct
    bracketing runs. Limitations, responsible-use and "descriptive point
    estimates" caveats are stated plainly, and the paper does not claim a
    method winner.
  - Against: not peer reviewed and four days old. The framework comes from
    the authors' own lab (Simple Agent Lab), and the paper is in part a
    release announcement. Every evolution cell is a single run with no CI.
    No compute accounting. The integrity suite is self-built scripted
    injections. The headline mixes in-sample splits. The four methods are
    adaptations, not reproductions. The AI4AI survey it cites for
    measurement validity is by six of its authors.
  - Not 4 because the empirical comparison is one run per cell and
    unmatched on tokens. Not 2 because the infrastructure claims are
    concrete, released and tested, and the paper's own reading of its
    numbers is cautious.

## Follow-up

- **Relevance:** 3. It contributes a worked architecture for HCE clause 4
  as framework code (certified rows, comparison-identity parent predicate,
  filtered annotations, surface diff check) to
  [[concepts/enforcement-boundary-placement]] and
  [[concepts/hce-evaluation]]. It also gives a clean in-paper measure of
  single-draw baseline noise in harness evolution, and a four-method
  same-start comparison for [[concepts/evolutionary-expansion]]. Not 4
  because the method comparison is too thin to rank anything. The one new
  structural finding, admission policy inside the mutable surface, is ours,
  not the paper's.
- **For the `hce-known-bad-preflight` proposal.** RSIHub is a
  well-engineered, independent instance of clause 4's tamper protection
  that explicitly *assumes* the evaluator is valid. That supports the
  proposal's premise that the hash/pristine half and the correctness half
  are separate. It is not new evidence that a do-nothing check catches
  defects.
- **Noise band.** The fidelity table is a reusable recipe: bracket the
  framework run with two direct runs and report D–D next to D–R. For
  `/iterate`, our inference: a single-draw baseline can sit 5–8 points off
  its mean on ~90-task suites, so the "no improvement" counter should use
  a repeated baseline.
- **Cheap local check.** Pair Table 1 with Table 2 per cell from the
  released run records. Re-scoring each champion and the seed three times
  on the sealed split would settle whether the sealed deltas outlast the
  baseline draw. That needs API spend only, no GPU.
- **Candidates** (cited, not yet in the graph):
  - Lin et al. 2026, "Agentic harness engineering" (AHE, arXiv
    2604.25850). Used by wang2026rethinking and ding2026harnessdelta, not
    ingested itself.
  - Zhang et al. 2026, "Hyperagents" (arXiv 2603.19461).
  - Agrawal et al. 2026, "GEPA: Reflective prompt evolution can outperform
    reinforcement learning" (ICLR 2026).
  - Lin et al. 2026, "Position: Agentic evolution is the path to evolving
    LLMs" (A-Evolve, arXiv 2602.00359).
  - Liu et al. 2026, "Rethinking self-evolving agent skills: feedback
    dynamics over multiple rounds" (arXiv 2608.02636). Says persistent
    skill improvement is sparse and conditional.
  - Wu et al. 2026, "AI4AI survey: from long-horizon agents to recursive
    self-improvement" (Preprints.org 202608.2108). Distinguishes measured
    gain, retention and held-out transfer. Overlapping authors.
  - Harbor Framework Team 2026, Harbor v0.18.0 (Zenodo
    10.5281/zenodo.21249820). The evaluator substrate several papers in
    this graph run on.
  - Same digest, item 14, 2610.27889 "Elicitation or Expansion?" A pass@k
    view of the same harness-optimization gains.
