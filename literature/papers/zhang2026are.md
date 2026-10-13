---
kind: paper
title: "Are We Measuring Scientific Intelligence? Rethinking the Evaluation of AI Scientists"
authors: ["Kate Zhang", "Yuante Li"]
institutions: ["Carnegie Mellon University"]
year: 2026
venue: "arXiv 2610.04915 (cs.AI), v1 2026-10-04; NeurIPS 2026 Workshop: Agentic AI for Biological Discovery (per the PDF footer)"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.04915"
code_url: null
citations: null
source: "raw/papers/zhang2026are.pdf"
added: "2026-10-13"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/refusal-cost-symmetry]]"
tags: ["evidence-grounding", "verified-counterfactual", "evidence-grounded-accuracy", "construct-validity", "prior-knowledge-shortcut", "no-data-baseline", "noticing-without-acting", "counterfactual-validity", "flip-auc", "llm-judge", "abstention-option", "gene-name-anonymisation", "baisbench", "genebench-pro", "single-cell", "ai-scientist-evaluation", "biology"]
---

# Are We Measuring Scientific Intelligence? Rethinking the Evaluation of AI Scientists

## TL;DR

Data-driven science benchmarks score an agent's final answer against a fixed
key and assume a correct answer was derived from the supplied data. The
paper tests that assumption directly. For each item it builds three edited
versions of the raw data files: a **null** edit (shuffle and rename, answer
unchanged), a **withdrawal** (delete the evidence for the key) and a **flip**
(move the evidence to another option). Each edit is checked by recomputing a
pre-registered reference statistic on every file. The same agent then runs
on every version in a fresh sandbox. A correct answer counts as
**evidence-grounded** only if the agent also reacts to withdrawal (changes
its answer or flags the data) and follows the flip.

On 18 single-cell questions from BAISBench's discovery split, the three
Claude Code agents are 95% accurate, 83% accurate with no data at all, and
41% evidence-grounded. After withdrawal the benchmark still awards 77% of
its credit. After a flip, 61% of runs follow the data. The benchmark's
exact-match score and a standard LLM process judge both rank a run that
follows a flip *below* one that ignores it (flip AUC 0.11 and 0.12). A
grounding-focused judge prompt reverses this (0.83).

## Claims

- **Observational evaluation cannot certify grounding (Prop. 1).** Two
  agents can produce the same trajectory and answer on E and still differ
  under an intervention on E. So no evaluator of the form M(τ, a, c*)
  computed on the original run can tell a derived answer from a recalled
  one. Reading the trajectory does not fix this.
- **Accuracy decomposes by route** (Eq. 4): derived (reacts, follows) =
  EGA; eliminated (persists under withdrawal, follows flip); reacts-only;
  recalled (persists, keeps key); other. Only the derived route is
  certified by the data.
- **Agents notice but do not act.** 35% of withdrawal runs state that the
  data cannot decide, and 84% of those give the key anyway.
- **Prior knowledge competes with the data.** Replacing gene symbols with
  opaque IDs on 10 gene tasks drops no-data accuracy from 87% to 27%,
  leaves with-data accuracy roughly unchanged, raises flip-following from
  58% to 93% and EGA from 39% to 69% (Claude models combined).
- **Evaluators share the blind spot.** The official score and LLM judges
  both measure agreement with the key, not responsiveness to evidence.
- **Repairs fix one failure each.** An explicit "data are insufficient"
  option cuts withdrawal credit from 77% to 31% but barely moves flips
  (30% → 21%). A grounding judge prompt fixes flip AUC but leaves part of
  the withdrawal blind spot. Transform consistency helps on synthetic tasks.
  "No repair covers every failure, and none can be validated without paired
  counterfactuals."

## Methods

- **Tasks.**
  - BAIS-SD has 193 multiple-choice questions from 41 published
    single-cell studies. 22 can be decided from the supplied h5ad data by
    one reference statistic. 4 were dropped because the original data do
    not support the benchmark key. That leaves 18 questions from 13
    studies, in five evidence families (abundance, composition change,
    expression localisation, marker gene, condition DE).
  - GeneBench-Pro contributes 4 of its 10 public synthetic genetics
    problems, the ones whose ground truth transforms exactly under an edit.
- **Counterfactual construction.** Edits touch only the quantity the
  statistic reads, applied to X, all layers and `.raw`. Every file of every
  version, the original included, goes through one canonical writer. It
  strips precomputed summaries and uses canonical gzip headers, because
  one agent in development recovered a swap from a filename in a gzip
  header.
- **Agents.** Claude Code (Opus 5.5, Sonnet 5, Haiku 4.5) and Codex CLI
  (GPT-5.6-Sol, GPT-5.6-Terra), 3 replicates each; Biomni A1 on Sonnet 5,
  1 replicate. Fresh sandbox per run with no web, retrieval, memories or
  MCP; 40-minute limit; the agent writes an answer, a 0–100 confidence and
  a data-support label. An audit of all 2,944 runs (16,091 tool calls) for
  out-of-sandbox access, key material or network use flagged none.
- **Metrics.** Invariance I (null vs original, compared with rerun
  agreement), withdrawal responsiveness R_rm and its flag-only variant,
  flip-following R_flip, EGA and a strict EGA (requires a flag under
  withdrawal). Evaluator tests: null sensitivity S_null, blind spot B_c
  (credit against the original key after an edit) and flip AUC (does a
  flip-following run outscore a key-keeping one).
- **Evaluators.** The official BAIS score; an Opus 5.5 process judge
  given the task, key, condensed trajectory and answer; a GPT-5.6-Sol
  cross-family judge (Spearman 0.91 with the Opus judge); the agents' own
  self-reports.
- **Uncertainty.** Task-clustered bootstrap intervals (Table 8).

## Results

- **Accuracy vs EGA (Table 2, BAIS-SD).**

  | agent | acc. E | no data | R_flip | B_rm | EGA |
  |---|---|---|---|---|---|
  | Opus 5.5 | 98 | 85 | 59 | 85 | 35 |
  | Sonnet 5 | 100 | 89 | 52 | 89 | 31 |
  | Haiku 4.5 | 87 | 74 | 72 | 56 | 56 |
  | GPT-5.6-Sol | 100 | 81 | 65 | 81 | 33 |
  | GPT-5.6-Terra | 94 | 80 | 63 | 78 | 35 |
  | Biomni (Sonnet 5) | 100 | – | 67 | 72 | 39 |

  Null edits change answers no more than reruns (98% agreement both ways).
  The strict EGA is 17–28% for every agent. The EGA ordering roughly
  inverts accuracy (Haiku least accurate, most grounded; Kendall τ = −0.54,
  not significant with six agents).
- **Routes (Fig. 4).** Fewer than half of correct runs are derived. About
  one in five is elimination, and a similar share is recall. Sonnet 5 has
  the largest recall share (35%).
- **By family (Table 7).** Marker-gene questions are almost all answerable
  without data (96%), flag a deleted marker in 24% of runs and keep 69% of
  withdrawal credit. Condition-DE questions follow flips least (17%).
- **Per task (Fig. 8).** Withdrawal leaves full credit in every run on 6 of
  18 tasks, and 2 tasks never follow the flip. The authors say the headline
  is not driven by a few items. Excluding the two tasks with an alternative
  valid analysis moves flip-following from 61% to 65% and withdrawal credit
  from 77% to 76%.
- **Bootstrap intervals (Table 8) are wide per model.** R_flip: Opus 59
  [39, 80], Sonnet 52 [30, 74], Haiku 72 [52, 91]. B_rm: Haiku 56 [39, 74],
  the others 78–89 with lower bounds 61–76.
- **Anonymisation (Table 11, App. G).** Claude combined: no-data 87 → 27,
  flips followed 58 → 93, credit after flip 36 → 4, EGA 39 → 69. GPT
  models move the same way. One task (S03_Q3) stays answerable from the
  study background. Without it, no-data falls 85 → 22 (Claude), and flips
  followed rise 54 → 93.
- **Evaluator stress test (Table 6, n = 75 per judge).** Flip AUC: Opus
  judge 0.12, GPT judge 0.10, benchmark score 0.11. Grounding prompt: 0.83
  and 0.73, at unchanged null sensitivity. Self-reports separate better than
  the judges: 84% of flip-following runs report full support against 13% of
  key-keeping runs.
- **Abstention option (Table 12, 486 extra runs).** Agents abstain on 0% of
  intact-data runs, 57% of withdrawal runs and 6% of flip runs. Withdrawal
  key credit falls 77 → 31 and flip credit 30 → 21.
- **GeneBench-Pro.** Accuracy is low (Opus 42%, Sonnet 17%, Haiku 0%). The
  grader passes almost no run on either version, so a run that tracks the
  transform exactly (Opus on Hi-C swaps the case and control estimates to
  the third decimal) scores 0, the same as one that ignores the swap.
  Transform consistency (53%) is about the same as rerun agreement (47%).

## Critique / open questions

- **House-pattern check: the paper reads AT the digest framing.** Every
  digest number checks out. 95% / 83% / 41% are the means of the three
  Claude Code agents (98/100/87, 85/89/74, 35/31/56). 58% → 93% is the
  anonymisation result for the Claude models combined on the 10 gene tasks.
  Neither is a best cut: the strict EGA (17–28%) is lower, and dropping the
  residual-prior task makes the anonymisation effect larger (54 → 93). The
  abstract also reports what does not hold up, namely that no single repair
  works.
- **What the digest leaves out.** First, the repairs. A grounding judge
  prompt lifts flip AUC from 0.11 to 0.83, so "LLM judges reward answers
  that ignore the change" holds for a *standard* judge prompt, not judges
  as such. Second, the denominator. The 18 questions are the 9% of BAIS-SD
  that a single statistic can decide, so "83% with no data" describes
  that subset, not BAISBench. Third, a side result the digest skips: 4 of
  the 22 data-decidable items (18%) have keys the supplied data contradict.
- **The variance worry is partly answered.** The pooled finding that agents
  keep credit after withdrawal and follow flips only in part holds in
  every model's task-bootstrap interval. The per-model *ordering* does not:
  CIs on R_flip span about 40 points, and the accuracy-vs-EGA inversion
  is explicitly not significant. Read the paper as "accuracy does not
  certify grounding", not as a ranking of agents. Judge results rest on
  75 runs per judge, with no human validation of the judges (conceded).
- **EGA scores some rational behaviour as ungrounded.** Two cases.
  1. *Elimination.* In a closed option set, choosing C because the data
     show that A, B and D are not endothelial markers is valid inference
     from the data, yet it counts against EGA. The withdrawal edit removes
     evidence *for* the key but not *against* the alternatives, so a
     purely data-driven agent can persist. The flag-only variant handles
     this better, but it is strict.
  2. *Implausible flips.* Swapping PECAM1 and PAX7 values in every cell
     makes data that contradict well-established biology. The best
     response is arguably "these data look mis-annotated", which gets no
     EGA credit. The authors concede this ("a scientist might reasonably
     distrust data that contradict known biology"). Their counter is
     partial: ordinary-looking reversals such as proportion shifts are
     followed only 67% of the time, and most key-keeping runs never
     mention a conflict.
  So EGA is a lower bound on grounding in the closed-option setting, and
  the 41% should not be read as "59% of answers are recalled". The recall
  route itself is 7–35% of runs, depending on the agent (Fig. 4).
- **Domain and format.** All tasks are single-cell biology plus 4 synthetic
  genetics problems. Answers are multiple choice or tolerance-graded. Agents
  run 5–6 turns and 30–50 s median. These are short analyses, not research
  loops. Transfer to ML-research agents is by analogy: a loop that "finds"
  a known-good architecture or reports numbers close to published SOTA has
  the same unidentified route. The method (edit the data, verify the edit
  with the statistic the task implies, rerun) transfers directly.
- **The self-report result is under-used.** The agents' own data-support
  labels separate flip-followers from key-keepers (84% vs 13%) better than
  either judge. The labels carry the signal, but the forced-choice answer
  discards it. That is a harness-design point, not only an evaluation one.
- **Open for this graph:** the paired-counterfactual test checks the
  *evaluator* as well as the agent, on the same runs. No ML-research
  benchmark in the graph (MLE-bench, PaperBench) has been checked this
  way. The nearest analogue in the graph is
  [[literature/papers/huang2026reward]]'s finding that offline label
  distillation passes a held-out recompute. In both cases the answer
  transfers to the check without coming from the run.

## Trust signals

- **Credibility:** 3.
  - For: CMU School of Computer Science. Careful construct design: the
    reference statistic is pre-registered per item, every edit is verified
    on every file and on raw counts, and a canonical writer closes metadata
    leaks found in development. All 2,944 runs were audited for leakage.
    Task-clustered bootstrap intervals, a cross-family judge-agreement
    check, a worked example traced through every metric, and verbatim
    case quotes that were checked automatically. Limitations are stated
    plainly.
  - Against: a two-author workshop paper (NeurIPS 2026 workshop, which
    counts as not peer-reviewed under the house rule). No code, data
    versions or run records are named in the PDF. Small samples (18 + 4
    tasks, 3 replicates, 1 for Biomni), no human validation of the judges,
    and one domain.
  - Not 4 because nothing is released and the sample is small. Not 2
    because the method is careful and its claims are kept to what the data
    show.

## Follow-up

- **Relevance:** 4. It strengthens [[concepts/hce-evaluation]] and
  [[concepts/programmable-evaluator-oracle]] with a new, measured failure:
  a correct key-match oracle on clean held-out data still cannot tell
  derived answers from recalled ones, and both the oracle and a standard
  judge are *counterfactually invalid* (flip AUC ≈ 0.1). It also gives the
  test that detects this: verified paired counterfactuals. It is the
  converse of [[literature/papers/tripathi2026diagnostic]]'s design, which
  holds the evidence fixed and varies the framing; this one holds the
  framing fixed and varies the evidence. Not 5 because the domain is
  biology MCQ and the samples are small.
- **Adoptable for this project's loops.**
  - Before trusting a "the data showed X" claim from an autonomous
    analysis, rerun it on a version of the data with X's support withdrawn.
    If the claim survives unchanged, it did not come from the data.
  - Validate any grader or judge on paired counterfactuals. A judge's flip
    AUC is a cheap, decisive admissibility check.
  - Give the answer format a "data insufficient" channel. It cost nothing
    on intact data here.
  - Run a no-data baseline per item, not just per benchmark.
- **Candidates** (cited, not in the graph):
  - Ren et al. 2026, "Right answer, wrong method: shortcut hacking misleads
    the evaluation of LLM reasoning on frontier science benchmarks"
    (arXiv 2608.02442).
  - Luo et al. 2026, BAISBench (Bioinformatics 42 Suppl. 1).
  - Li & Ho 2026, GeneBench-Pro (bioRxiv 2026.06.29.735386).
  - Huang et al. 2025, Biomni (bioRxiv 2025.05.30.656746).
  - Zhong, Raghunathan & Carlini 2026, ImpossibleBench (ICLR 2026).
  - Bean et al. 2026, "Measuring what matters: construct validity in LLM
    benchmarks" (NeurIPS 38).
