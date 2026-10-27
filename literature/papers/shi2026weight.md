---
kind: paper
title: "The Weight of Teamwork: Does Communication Structure Help LLM Scientists Search for Scientific Laws?"
authors: ["Jianing Shi", "Zaibin Zhang", "Shijun Lei", "Zhenfei Yin"]
institutions: ["The University of Chicago", "Dalian University of Technology", "Northwestern Polytechnical University", "University of Oxford"]
year: 2026
venue: "arXiv 2610.24131 (cs.MA), v1 2026-10-16, preprint; acknowledges reviewers at the NeurIPS 2026 FAST workshop"
peer_reviewed: unknown    # workshop reviewers thanked; acceptance not stated in the PDF
url: "https://arxiv.org/abs/2610.24131"
code_url: null            # an "accompanying source package" with CSV/JSON authority exports is described (App. J.5), but no URL is given
citations: null
source: "raw/papers/shi2026weight.pdf"
added: "2026-10-27"
relevance: 3
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/pass-at-k]]"
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/async-worker-pool]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/budget-as-ceiling]]"
tags: ["multi-agent", "communication-topology", "ring", "star", "complete-graph", "independent-sampling", "breadth-vs-depth", "fixed-call-budget", "selection-rule", "delivery-rule", "generation-vs-selection", "symbolic-regression", "llm-srbench", "qwen2.5-7b", "gpt-5.6-terra", "paired-bootstrap", "holm", "exploratory", "single-run-per-cell", "negative-result"]
---

# The Weight of Teamwork: Does Communication Structure Help LLM Scientists Search for Scientific Laws?

## TL;DR

Five LLM "scientists" propose symbolic-regression expressions on
LLM-SRBench problems. Every condition gets the same 50 model calls per
problem. The conditions vary who can read whose earlier responses: nobody
(Independent), the previous agent in a serial loop (Ring), a hub and its
leaves (Star), or everyone's previous round (Complete). A matched Solo
condition makes 50 calls in one trajectory, and an allocation sweep spreads
the 50 calls over N = 1, 2, 5 or 10 independent trajectories. All candidates
are archived, then a selection ("delivery") rule picks one, which is scored
on the benchmark's held-out ID split. Main layer: qwen2.5:7b-instruct on 100
problems. A narrower layer runs gpt-5.6-terra on 41 Transform problems.

Three results:

1. **Breadth was the largest effect.** Ten independent 5-call trajectories
   beat one 50-call trajectory by +0.70 mean held-out Q (95% CI 0.41–1.03;
   median +0.11; 86/11/3 wins/losses/ties). Q is −log10 NMSE, so 0.70 is
   about a fivefold cut in error on average. This also changes trajectory
   length and context resets, so it supports the allocation policy as a
   whole.
2. **Communication did not help, and Ring hurt.** Under the common
   archive-wide selector, Ring minus Independent was −0.33 (CI −0.49 to
   −0.18). Star (+0.03, CI −0.08 to +0.13) and Complete (+0.10, CI −0.08 to
   +0.34) were unresolved. All three contrasts are labelled exploratory
   because the family was written down 11 minutes after the results file.
3. **With the archives frozen, the selection rule moved the architecture
   gap.** Complete minus Independent was +0.10 under archive-minimum training
   error and −0.75 (CI −1.56 to −0.25) under final-round minimum training
   error. 9 of 20 prespecified architecture × rule interactions passed Holm
   correction in Qwen, and 0 of 20 in Terra. The authors' conclusion: report
   generation (the archive) and selection (the returned answer) as separate
   endpoints, and fix or cross the selector when comparing architectures.

Two things limit the result. Each condition was run once, so the
intervals cover problems, not run-to-run variation. And the 7B
backbone mostly fails the task: even at N = 10, mean Q is +0.17 with an
interval spanning zero, and 38/100 problems beat predicting the mean.

## Claims

- A team's delivered score has two stages: generation of a candidate archive
  and a rule that returns one candidate. A single final score cannot tell a
  team that never generated a good candidate from one whose rule failed to
  return it.
- At a fixed invocation budget, spreading calls over more independent
  trajectories "produced the largest measured gain" in this testbed.
- The implemented serial Ring policy underperforms Independent under a
  common selector. Star and Complete are unresolved, and "do not establish
  equivalence".
- "An architecture ranking is not identified by communication structure
  alone when the rule that returns the final answer is left unspecified."
- Three reporting practices, which "do not depend on the present testbed":
  state the selection rule beside the architecture; report archive and
  returned expression separately; hold the selector fixed across
  architectures or cross architecture with selector.
- Explicit non-claims: no new solver, no communication mechanism, no
  universally preferred topology, and "None of this implies that
  multi-agent organization should be avoided."

## Methods

- **Task.** LLM-SRBench. Qwen layer: 50 LSR-Transform (manifest indices
  0–49) + 50 LSR-Synth (111–160), a fixed prefix of each family, not a
  random draw. Terra layer: the 41-problem Transform intersection (0–40);
  Terra Synth Independent never ran (HTTP 429, credit exhausted, 0/50).
  Each prompt shows the target, variable descriptions, five training rows,
  the agent's own previous responses and whatever peer responses the policy
  allows (anonymised as "colleague 1, 2…"; topology not named). The output
  is a short discussion plus one `EQUATION:` line. **Agents never see a fit
  score**: training NMSE is used only for selection, not fed back.
- **Budget.** 50 scheduled calls per problem in every condition. The
  communication comparison uses 5 agents × 10 rounds. The allocation sweep
  uses (N, T) ∈ {(1,50), (2,25), (5,10), (10,5)}. Matching is by call
  count, not tokens. Communicating arms carry peer text in their prompts.
  Token counts are archived but not reported.
- **Policies.** Visibility and update timing vary together. Ring reads the
  latest predecessor response in rotating serial order. Star runs a leaf
  phase, then a hub phase. Complete reads all four peers' previous-round
  snapshot. Peer text does not accumulate in self-history.
- **Models.** qwen2.5:7b-instruct via local Ollama, temperature 0.70,
  512-token output cap. gpt-5.6-terra via OpenAI, temperature 1.00,
  `reasoning_effort=none`, 2,048-token cap. Only within-layer comparisons
  are made.
- **Score.** NMSE = MSE / Var(y) on the official pregenerated held-out ID
  arrays (6,251–20,000 rows for Transform, 500 for Synth), never shown to
  agents. Q = −log10(NMSE + 1e−10). Constant-mean prediction gives Q ≈ 0.
- **Selectors (d1–d6), applied to frozen archives.** d1 is the minimum
  training NMSE over the whole archive (primary). d2 is the minimum training
  NMSE in the final round. d3 is final-round plurality of canonical
  skeletons. d4 is archive-wide plurality. d5 is the last message (stress
  test). d6 is training NMSE + 0.05 × normalised SymPy node count. Training
  NMSE is computed on the full training array (25,006–80,000 rows for
  Transform, 4,000 for Synth). None of the selectors touches held-out data
  or the reference expression.
- **Statistics.** Mean paired difference is primary. 95% percentile
  bootstrap over problems (10,000 resamples, seed 20260904), with medians,
  W/L/T, rank-biserial r and Holm-adjusted Wilcoxon. The Holm families are
  the 3 communication contrasts, the 5 allocation contrasts, and 20
  delivery interactions per layer. **One run per condition**, so "bootstrap
  intervals across problems do not measure repeated-run sampling variation."
- **Provenance (App. J).** The communication family was defined at
  12:04:47 on 2026-07-26, after the results file at 11:53:17, so it is
  exploratory. The primary endpoint was amended from OOD to held-out ID on
  2026-08-28, because Transform has no OOD array; this came after the
  original runs. An earlier draft had copied Star/Complete intervals from a
  different-seed analysis, and this was corrected. The original Solo run had
  only 10 calls per problem and was replaced by a matched 50-call rerun. The
  Qwen runtime hash and SciPy version are unrecoverable. A post-hoc
  diversity analysis hit its own prespecified stopping rule and was then
  run anyway as a recorded deviation.

## Results

- **Allocation (Qwen, d1, Table 1B).**

  | Contrast | Mean ΔQ [95% CI] | Median | W/L/T |
  |---|---|---|---|
  | N 1→2 | +0.23 [0.06, 0.43] | 6.3e−5 | 55/40/5 |
  | N 2→5 | +0.20 [0.07, 0.34] | 4.7e−3 | 64/28/8 |
  | N 5→10 | +0.26 [0.04, 0.58] | 5.2e−4 | 58/37/5 |
  | N 1→5 | +0.43 [0.27, 0.63] | 0.05 | 78/18/4 |
  | N 1→10 | +0.70 [0.41, 1.03] | 0.11 | 86/11/3 |

  Gains are right-skewed (mean much larger than median). Absolute mean Q:
  −0.53 (N = 1), −0.30, −0.10, +0.17 [−0.08, 0.49] (N = 10). Problems with
  NMSE < 1: 11, 12, 19, 38 of 100. The N 1→5 gain holds in both families
  (Transform +0.35, Synth +0.52). Terra Transform Independent − Solo is
  +1.04 [0.06, 2.07] with median 0, 18/3/20.
- **Within one trajectory, iteration barely helps.** Solo's best-so-far Q
  rises by 0.01 between call 10 and call 50. At the agent level, 81% (Solo)
  and 81.8% (Independent) of outputs share the trajectory's modal skeleton.
- **Communication (Qwen, d1, Table 1A).** Ring −0.33 [−0.49, −0.18], 21/75/4,
  Holm p 5.75e−7. Star +0.03 [−0.08, +0.13], 43/50/7. Complete +0.10
  [−0.08, +0.34], 53/42/5. Ring is negative in both families (−0.30, −0.35).
  In Terra, Ring −1.18 [−2.04, −0.48]. Star −0.57 and Complete −0.36 both
  have zero-spanning intervals, and 20–24 of 41 problems are exact ties.
- **Archive diagnostics (exploratory).** Problems where one skeleton is
  ≥ 80% of outputs: Solo 81, Ring 72, Star 56, Complete 46, Independent 0.
  Median distinct skeletons per problem: Solo 2, Ring 2, Star 5, Complete 7,
  Independent 8.5, N = 10 15.5. Ring had 1 invalid slot of 5,000, against
  36–61 for the others. Lower-tail count (Q < −0.76): Independent 10,
  Ring 23, Star 11, Complete 12. A post-hoc test of "fewer distinct
  candidates explains the gap" found no consistent ΔD–ΔQ association
  (Spearman −0.02, +0.26, +0.08), and the stratified follow-up did not
  support it.
- **Selection on frozen archives (Qwen, Table 2 / D1).** Complete −
  Independent under d1/d2/d3/d4: +0.10 / −0.75 / −0.78 / −0.59. Star:
  +0.03 / −0.48 / −0.58 / −0.41. Ring: −0.33 / −0.39 / −0.38 / −0.26. Under
  d5 every architecture beats Independent (+1.2 to +1.6, wide intervals).
  Interactions relative to d1: Complete d2 −0.84 [−1.85, −0.26], Holm p
  1.05e−7. Star d2 −0.51 and d3 −0.61, both Holm-significant. Ring's
  interactions are near zero. Terra: 0/20 interactions survive.

## Critique / open questions

- **Against the digest framing: below framing (mild).** The digest's
  numbers are all in the PDF and quoted correctly. Four things are framed
  more strongly than the paper supports, or left out:
  1. **"Independence beats sharing at matched budget" is only half
     shown.** Only Ring is worse than Independent. Star and Complete are
     unresolved, with intervals that allow a gain of up to +0.34. The +0.70
     is a breadth effect (N = 1 vs 10, no sharing in either arm), not a
     sharing-vs-independence contrast. It is also confounded with shorter
     trajectories and context resets.
  2. **The +0.70 is reported under one selector only, and it is close to an
     oracle.** The allocation sweep is scored under d1 alone (App. E). Our
     inference: the expressions have no fitted constants, and the training
     and held-out ID arrays are large i.i.d. draws from one generator. So
     minimum training NMSE should rank candidates almost exactly as held-out
     NMSE would, which makes d1 a near-oracle best-of-archive. N = 10
     produces 15.5 distinct skeletons against Solo's 2, so the breadth gain
     is largely a best-of-more-distinct-draws gain under a near-oracle
     picker. That is the same generation-vs-selection confound the paper
     warns about in its architecture comparison, and it applies to its own
     headline. Whether breadth still wins under a weak rule (plurality, an
     LLM judge) is untested.
  3. **Single run per cell, weak backbone, low absolute performance.** None
     of the effects has a rerun-noise estimate. On a 7B model with a
     512-token cap, the system at N = 10 does not beat constant-mean
     prediction with confidence. The Terra layer, the only stronger
     model, is 41 problems with median 0 on every contrast.
  4. **Exploratory status.** The communication family was written after
     the results, and the d2–d4 emphasis is exploratory. The paper says
     this plainly. The digest does not.
- **The selection result is the solid one, and it reads as an oracle-vs-weak
  selector gap.** Our inference, not the paper's: if d1 is near-oracle, then
  d1 contrasts measure archive quality, and d2–d4 measure what weaker rules
  return. Complete and Star lose most under plurality and final-round
  rules. Those are also the conditions whose archives concentrate (46 and 56
  problems with a ≥ 80% modal skeleton, against 0 for Independent), so a
  vote or a last-round pick lands on the herd's answer. Ring's
  interactions are near zero because Ring is already concentrated under
  every rule. The paper declines to name a mechanism, and its post-hoc
  diversity test did not support a simple count-of-distinct-skeletons story.
- **"Depth" here is iteration without feedback.** Agents never see their
  NMSE. A 50-call Solo trajectory is 50 blind re-proposals conditioned on
  its own text, and it plateaus by call 10 (+0.01 after). This does not
  conflict with [[literature/papers/ishibashi2026effective]], where depth
  meant iterating a candidate *against the evaluator* and paid. Our
  reading: breadth beats feedback-free depth. The test of breadth against
  feedback-driven depth at matched calls was not run, and it is the one
  that matters for an `/iterate`-style loop.
- **The call-matching currency favours the communicating arms.**
  Communicating prompts carry 1–4 peer responses, so they cost more input
  tokens per call than Independent. Token counts are logged but not
  reported. Under token matching in the [[literature/papers/oved2026llm]]
  sense, Independent's position could only improve. So "communication did
  not help" is conservative in the token currency. "Ring hurt" stands
  either way.
- **Ring confounds topology with serial timing.** Ring is the only fully
  serial policy, it has the fewest invalid slots (1/5,000) and as few
  distinct skeletons as Solo. It behaves like one long trajectory passed
  around. The authors name this ("separate topology from update timing")
  as future work.
- **Peer messages carry no evaluation.** Agents see colleagues' rationales
  and equations but not their fit. Sharing without scores is a weak form of
  communication. The paper lists "peer messages that include evaluations"
  as future work. Most research-agent pools share results, not just
  proposals.
- **Memorisation is untested.** LSR-Transform is built from transformed
  Feynman equations. Synth "extends the result beyond transformed Feynman
  equations without testing memorization."
- **Fit with the graph.** The breadth result is the matched-call version of
  [[literature/papers/zhong2026automating]]'s inference that insight is a
  per-trajectory draw ("not tested at matched compute"). The selection
  result is [[literature/papers/kim2026are]]'s oracle-vs-vote gap measured
  as an architecture *ranking* flip. Concentration within a trajectory
  (81.8% in Independent) reproduces [[literature/papers/gurkan2026mutation]]'s
  operator attractor at expression grain. The digest's link to
  [[literature/papers/lee2026when]] (whatever breaks ties decides the
  kept candidate) holds in spirit, but here the selectors differ in kind,
  not only in tie-breaking.

## Trust signals

- **Credibility:** 3.
  - For: unusually careful statistics and disclosure. Paired bootstrap
    with medians, W/L/T and Holm families stated per analysis. Every
    contrast is labelled confirmatory or exploratory, and the chronology
    that makes the communication family exploratory is timestamped. A
    stopping rule was honoured and its later override recorded as a
    deviation. A copied-interval error was disclosed and corrected.
    Archives are frozen with a published tree SHA-256. Held-out arrays are
    the benchmark's own, never shown to agents. Selectors are fixed and
    held-out-blind. The authors refuse mechanism claims their data cannot
    support. Corresponding author at Oxford. Workshop reviewers are
    acknowledged.
  - Against: one run per condition. A 7B backbone that mostly fails the
    task (11/100 problems beat constant-mean at N = 1). Call-matched, not
    token-matched. Fixed-prefix problem cohorts. The headline breadth
    effect is reported under a single near-oracle selector. The Terra
    layer is small and incomplete. The Qwen runtime hash and SciPy version
    are unrecoverable. No public code URL. Not peer reviewed beyond a
    workshop (acceptance unstated). No citations yet (11 days old).
  - Not 4 because there are no rerun replicates, the backbone is weak and
    the key contrasts are exploratory. Not 2 because the design is clean,
    the selection-dependence result is strong within its own archives (Holm
    p ~1e−7), and the reporting is more candid than most sources in this
    graph.

## Follow-up

- **Relevance:** 3. The setting is symbolic regression with a 7B model, not
  ML engineering, so the effect sizes do not transfer. Two points do. First,
  the generation-vs-selection split: an architecture ranking is only
  defined together with its selector. This adds to
  [[concepts/pass-at-k]] and [[concepts/hce-evaluation]]. Second, breadth
  over feedback-free depth at a matched call budget. This adds to
  [[concepts/evolutionary-search-grain]],
  [[concepts/shared-substrate-contagion]] (a no-channel arm with
  measured concentration) and [[concepts/async-worker-pool]] (sharing did
  not raise the best). Not 4 because there is a single run per cell, the
  domain and backbone are far from this project's, and the depth arm has
  no evaluator feedback.
- **For the noise-band / compute-matching front.** If this box compares
  `/iterate` variants or pool sizes, log tokens as well as calls, and report
  the archive best and the delivered pick as two numbers.
  Neither costs anything extra.
- **Missing arms worth asking for.** (a) The allocation sweep under d2–d4,
  to see whether breadth survives a weak selector. (b) Solo with NMSE
  feedback injected, i.e. feedback-driven depth against breadth at 50
  calls. (c) Ring adjacency on a parallel schedule. (d) 3+ reruns per cell.
- **Candidates** (cited, not yet in the graph):
  - Kim, Yubin et al. 2026, "Capable language models can outgrow the
    benefits of collaboration" (Nature Machine Intelligence 8(7)). Single
    vs multi-agent at matched reasoning tokens; coordination gains shrink
    as single-agent capability rises. Peer-reviewed.
  - Tran & Kiela 2026, "Single-agent LLMs outperform multi-agent systems on
    multi-hop reasoning under equal thinking token budgets" (arXiv
    2604.02460).
  - Li 2026, "Dictionaries, not Darwin: Set-level selection beats LLM
    evolution in scientific equation discovery" (arXiv 2607.04108). Cited
    as the basis for shared-selector archive comparison, and directly on
    the evolution-vs-sampling front.
  - Shojaee et al. 2025, "LLM-SRBench" (ICML 2025). The benchmark.
  - Su et al. 2025, "Many heads are better than one" (VirSci, ACL 2025),
    which compares sequential and random speaking modes.
  - Kim, Junsol et al. 2026, "Reasoning models generate societies of
    thought" (arXiv 2601.10825).
