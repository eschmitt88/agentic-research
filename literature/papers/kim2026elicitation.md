---
kind: paper
title: "Elicitation or Expansion? Demystifying Gains from Agent Harness Optimization"
authors: ["Younghyun Kim", "Wooseok Seo", "Minseo Kim", "Seunghyuk Oh", "Minjae Lee", "Donghoon Kim", "Hyung Il Koo", "Jaehyung Kim", "Wonjun Kang"]
institutions: ["FuriosaAI", "Yonsei University", "University of California, Berkeley"]
year: 2026
venue: "arXiv 2610.27889 (cs.LG), v1 2026-10-20, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.27889"
code_url: "https://github.com/furiosa-ai/elicitation-or-expansion"
citations: null
source: "raw/papers/kim2026elicitation.pdf"
added: "2026-10-27"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/pass-at-k]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/evolutionary-search-grain]]"
tags: ["harness-optimization", "meta-harness", "pass-at-k", "coverage", "elicitation-vs-expansion", "stronger-reference-arm", "model-scaling", "reasoning-effort", "difficulty-stratification", "hard-tasks", "ood-transfer", "scorer-validity", "exact-match-false-negatives", "known-ordering-check", "hotpotqa", "appworld", "lawbench", "gpt-oss", "qwen3", "claude-opus-5-proposer", "trajectories-released"]
---

# Elicitation or Expansion? Demystifying Gains from Agent Harness Optimization

## TL;DR

The paper takes the RL pass@k question (does training widen what a model can
solve, or only make it solve the same things more often?) and asks it of
harness optimization. It runs Meta-Harness, with Claude Opus 5 as the
proposer, for 20 iterations of up to 2 proposals each. The solver weights
stay fixed. It then compares the initial harness H0 and the validation-selected
H⋆ on held-out test sets at k = 64 (text classification) or k = 128
(HotpotQA, AppWorld). It also compares both with a **stronger reference**:
the same H0 on a larger model of the same family (GPT-OSS-120B for 20B,
Qwen3-32B for 8B), or at high reasoning effort (AppWorld).

Three results matter for this graph:

1. **On static tasks the gain is mostly elicitation.** On LawBench the
   optimized GPT-OSS-20B harness lifts pass@1 from 18.86% to 44.00%
   (+25.14 pp). 21.02 pp of that comes from tasks H0 already solved at
   least once in 64 tries. At pass@64 the gain is +3 pp (53 → 56 of 100
   tasks: 7 gained, 4 lost). Its 16.05 pp pass@1 lead over the 120B
   reference reverses at k = 11 and ends at 56% vs 60%. The 20B harness
   is below the 120B reference at pass@64 on all three static datasets.
2. **On interactive tasks the harness does widen reach over its own
   baseline, but not past the stronger reference on hard tasks.** AppWorld
   hard (63 tasks) goes from 52/63 to 56–58/63 at pass@128, which ties the
   high-effort reference at best (58/63). HotpotQA hard (41 questions) goes
   from 29/41 to 29–30/41 against 32/41 for Qwen3-32B. All 40 candidates
   from one HotpotQA run stay below 32B on hard questions at every k.
3. **Redirecting the optimizer at hard questions raised hard pass@1, not
   hard coverage.** A hard-focused task mixture moved hard pass@1 from
   56.88% to 59.28%, against the 32B reference's 59.51%. Hard pass@128
   stayed at 73.17% (30/41), the same as before. This is one re-optimization run.

The paper's stated scope is "bounded by the evaluated benchmarks, solver
families, and optimization runs examined". The hard-subset differences are
1–2 questions on HotpotQA and 0–6 tasks on AppWorld, and no interval is
reported anywhere.

## Claims

- "pass@1 advantages of harness over stronger interventions can reverse at
  larger attempt budgets and aggregate gains conceal shortfalls on hard
  tasks."
- Harness optimization "concentrates gains on tasks with existing baseline
  successes". The paper offers this as an analogy to the Matthew effect
  Noukhovitch et al. (2026) reported for RL.
- Environment interaction gives the harness room to expand. A harness that
  changes what is retrieved or done can reach evidence H0 never sees. In one
  HotpotQA example, H0 never reaches the needed abstract in 128 rollouts.
  H⋆ reaches it in 125 and answers correctly in 122.
- The candidate with the best pass@1 is not the one with the best pass@128.
  This is shown in Fig. 2 (one run, 300 HotpotQA tasks, k = 128). "Repeated
  evaluation can improve the precision of pass@1, but does not make it a
  substitute for measuring task coverage at larger budgets."
- Recommendations (the Takeaway boxes):
  - When hard-task coverage matters, test coverage on that subset at the
    intended attempt budget.
  - Compare against feasible larger-model or higher-effort alternatives on
    the same subset.
  - Validate a transferred harness against the target-domain baseline, and
    use code inspection to find domain assumptions.
- Future work: "search and selection methods that reward validated
  successes on previously uncovered hard tasks and retain candidates with
  complementary successes." This is proposed, not tested.

## Methods

- **Optimizer.** Meta-Harness (Lee et al. 2026b, COLM 2026). The proposer is
  a coding agent that reads prior harness code, scores and traces. Here it is
  Claude Opus 5, with 20 iterations × ≤2 proposals, so up to 40
  candidates per run. H⋆ is picked on validation (text classification:
  equal-dataset mean validation accuracy, with the selection set reused from
  validation). On HotpotQA candidates are shortlisted on validation and the
  pick is by EM on a separate selection set. On AppWorld the pick is on the
  dev set at temperature 0 under the original TGC.
- **Settings (Table 1, test sizes in brackets).**
  - Static, jointly optimized single harness: Symptom2Disease [212],
    LawBench [100], USPTO [100]. GPT-OSS-20B solver vs GPT-OSS-120B
    reference, medium effort, **temperature 0**. H0 is few-shot and
    resamples demonstrations per query and repetition. H⋆ builds a memory
    and retrieves from it, rebuilding the memory per repetition.
  - OOD transfer of the ID-selected H⋆, with no further optimization:
    FiNER [100], GoEmotions [150], Banking77 [150].
  - HotpotQA [300 test; 150 prep / 300 search-val / 300 selection]. Qwen3-8B
    vs Qwen3-32B, temperature 0.6. H0 is GEPA's two-hop seed program with a
    fixed BM25 retriever. Caps: 4 solver calls, 2 retrievals, 7 passages.
  - AppWorld [168 test_normal]. GPT-OSS-120B at medium effort, vs the same
    model at high effort. H0 is the simplified ReAct agent, temperature 1 at
    evaluation. Each attempt gets 50 solver calls and 50 environment steps,
    with a 2,400 s timeout (7,200 s for high effort).
- **Estimator.** The unbiased Chen et al. pass@k from n repeated complete
  executions per task (n = k = 64 or 128). Tasks are weighted equally, and the
  text macro average weights datasets equally. The paper states the i.i.d.
  assumption.
- **Scorers (App. B.3).** The same scorer is used for all systems on a
  benchmark. Two were changed after a sanity failure:
  - HotpotQA: EM gave Qwen3-32B *lower* pass@1 than Qwen3-8B. Table 2 lists
    32B answers that state the reference inside a sentence and get EM = 0.
    The evaluation scorer is Judge = answer containment ∧ an LLM correctness
    judgment. The judge model is not named.
  - AppWorld: the original TGC ranked GPT-OSS-120B below 20B at pass@1.
    Arity-fixed TGC also accepts an attempt whose only failed check is an
    unrequested answer, provided all state checks pass and there is no
    execution error.
  In the main runs, search and selection used the *original* scorers (EM,
  TGC), and only evaluation used the fixed ones. App. C.3 reruns
  optimization and selection under the aligned scorers.
- **Robustness (App. C).**
  - Three optimization runs per benchmark (Fig. 12, mean ± sample SD).
  - The same analysis with a GPT-OSS-120B solver on the static tasks, and
    with a GPT-OSS-20B solver on the dynamic ones.
  - Frozen vs rebuilt memory (Table 4) and analytic uniform-guessing
    coverage (Table 5).
  - Aligned scorers, and tight vs relaxed resource caps (Table 6; three runs
    each).
  - A hard-focused task mixture (Fig. 9, one run).
- **Released.** Code (github.com/furiosa-ai/elicitation-or-expansion) and a
  trajectory dataset (HF furiosa-ai/elicitation-or-expansion-trajectories),
  both named in the PDF and not opened at ingest.

## Results

- **Static, GPT-OSS-20B (Table 4, n = 64; Rebuilt is the figure arm).**

  | Dataset | pass@1 H0 | pass@1 H⋆ | pass@1 120B | pass@64 H0 | pass@64 H⋆ | pass@64 120B |
  |---|---|---|---|---|---|---|
  | Symptom2Disease | 82.08 | 84.66 | 85.08 | 95.75 | 94.81 | 95.75 |
  | LawBench | 18.86 | 44.00 | 27.95 | 53.00 | 56.00 | 60.00 |
  | USPTO | 6.25 | 9.64 | 16.94 | 28.00 | 32.00 | 35.00 |

  Freezing the memory gives the same pattern (H⋆ pass@64 92.45 / 53.00 /
  30.00).
- **Success-set churn (Table 3, n = k = 64).** Each entry gives |S0| →
  |S⋆|, then gained / lost / kept, then the share of the pass@1 gain from
  kept tasks:
  - Symptom2Disease: 203 → 201; 3 / 5 / 198; +2.63 of +2.57.
  - LawBench: 53 → 56; 7 / 4 / 49; +21.02 of +25.14.
  - USPTO: 28 → 32; 7 / 3 / 25; +3.25 of +3.39. 68 of 100 USPTO tasks
    are never solved.
- **Across three optimization runs (Fig. 12).** The mean gain over H0
  shrinks from pass@1 to the largest k on all three benchmarks. All three
  text runs end below 120B at pass@64. On hard HotpotQA and hard AppWorld,
  no run exceeds the reference at any k. The SD "includes evaluation and
  preparation; it does not isolate optimizer variance."
- **Dynamic, all tasks.** Both HotpotQA 8B harnesses beat 32B at pass@1.
  Only one stays above it at pass@128. Both AppWorld medium-effort
  harnesses end below high effort at pass@1 and pass@128. One leads briefly at k = 2.
- **Dynamic, hard subsets (§5.2).**
  - HotpotQA: pass@128 is 70.73–73.17% for H⋆, 70.73% for H0 and 78.05% for
    32B, i.e. 29–30/41 vs 29/41 vs 32/41.
  - AppWorld: 88.89–92.06% for H⋆, 82.54% for H0 and 92.06% for high effort,
    i.e. 56–58/63 vs 52/63 vs 58/63.
- **Task-level pass@1 gains, harness vs scaling (Fig. 7).** On every panel
  the hard-group mean favours scaling. For text classification the groups
  are dataset-level proxies, not difficulty labels.
- **OOD transfer (Fig. 10).** At pass@64 the transferred H⋆ is below H0 by
  6.00 pp on FiNER, 8.00 pp on GoEmotions and 11.33 pp on Banking77. Its
  small pass@1 leads on the first two reverse at k = 2. At pass@4 and @8
  both harnesses are well above uniform guessing (Table 5). The mechanism
  was found by reading the code. H⋆ routes to a multi-label "catalog"
  template when there are ≥30 labels with mean length <40. That suits
  LawBench's multi-charge cases, but Banking77 is single-intent.
- **Counter-cases in the appendix.**
  - With a GPT-OSS-20B solver on HotpotQA, H⋆ *exceeds* GPT-OSS-120B at
    pass@128 (89.00% vs 85.33%). On AppWorld it is slightly below at
    pass@64 (91.07% vs 91.67%). This is not discussed in the main text.
  - With a GPT-OSS-120B solver, LawBench pass@1 goes from 27.95% to 38.66%
    while pass@64 goes from 60% to 59%.
  - Relaxed caps (12/12/20) vs tight (4/2/7), three runs each: pass@1
    73.39 ± 1.60 vs 72.50 ± 1.43, pass@128 87.22 ± 1.54 vs 87.78 ± 1.39.

## Critique / open questions

- **Against the digest framing: below framing (mild).** The digest summary
  is the abstract, and the abstract is accurate. The "Why" line, "the RL
  'elicitation not expansion' result carried over to harnesses", goes
  further than the paper in two places.
  1. **The answer is split by task type.** On the static tasks the gain is
     mostly elicitation (LawBench: 84% of the pass@1 gain is on tasks
     already reachable). On both dynamic benchmarks H⋆ widens coverage over
     its own H0: AppWorld hard +4 to +6 of 63 tasks, and all-task HotpotQA
     and AppWorld curves rise above H0 at k = 128. The reversals the
     abstract highlights are against a *stronger reference*, not against
     the pre-optimization harness. The cases where H⋆ falls below its own
     H0 at large k are Symptom2Disease (−2 tasks), OOD transfer, the 120B
     LawBench case (−1 task) and the Fig. 1 example.
  2. **The vivid hard-task loss is a favourable cut.** Fig. 1 shows hard
     HotpotQA pass@128 falling from 71 to 66 (about 29 → 27 of 41). A
     footnote says this "uses an optimized harness under relaxed per-attempt
     resource limits". Those are App. C.4 runs, not the main-setting
     harnesses of Fig. 5. In the main setting hard coverage is flat to
     +1 question. Table 6 shows relaxed caps do not change mean coverage. So
     the figure chosen to lead the paper is the one run that shows coverage
     *loss* against baseline. The main result is coverage *stasis* against
     a stronger reference.
- **Check-at-ingest: the numbers.** Supplied above. Every headline gap is a
  handful of tasks:
  - LawBench pass@64 −4 vs 120B, of 100 tasks;
  - HotpotQA hard −2 vs 32B, of 41;
  - AppWorld hard 0 to −2 vs high effort, of 63;
  - OOD −6, −12 and −17 tasks, of 100 / 150 / 150.
  No CI or task bootstrap is reported for any pass@k point. Fig. 12 gives a
  ±1 SD band over three runs. By eye it overlaps the reference on several
  hard panels, so "none exceeds" is a statement about point curves.
  [[literature/papers/liu2026what]]'s rule (nothing under about 55 paired
  tasks is decision-grade) applies directly to the hard subsets.
- **On static tasks, part of "coverage" is context resampling, not model
  reach (our inference).** The solver runs at temperature 0. Attempt-to-
  attempt variety comes from the harness: H0 resamples demonstrations and
  H⋆ rebuilds memory. A harness that retrieves more consistently is less
  diverse by construction, so its pass@64 falls even if per-attempt
  competence rose. The frozen-memory ablation points this way: removing
  H⋆'s rebuild lowers its pass@64 further (94.81 → 92.45, 56 → 53,
  32 → 30). The matching control, H0 with frozen demonstrations, is not
  run. The RL analogy is cleaner on HotpotQA and AppWorld, where the solver
  samples at temperature 0.6 or 1.
- **The stronger reference is not compute-matched.** Qwen3-32B costs more
  per attempt than 8B, and the high-effort arm gets a 7,200 s timeout
  against 2,400 s. The optimization spend (up to 40 candidates scored on
  300 search items with an Opus 5 proposer) is never priced. "Harness
  optimization vs model scaling" is therefore a comparison of interventions
  at unequal and unreported cost. It is still a useful arm, but it is not
  the same-budget arm of [[literature/papers/wang2026rethinking]] or the T
  term of [[literature/papers/ding2026harnessdelta]]. It is closest to
  [[literature/papers/zhang2026review]]'s "stronger model alone" baseline.
- **Selection was on pass@1 under a scorer later judged broken.** Main-run
  selection used EM (HotpotQA) and the original TGC (AppWorld). Both
  misranked larger models below smaller ones. A pass@1 selection rule can
  only find the pass@1 leader, and Fig. 2 shows that is not the pass@128
  leader. So "harness optimization gives elicitation" partly restates "we
  selected for elicitation" (our inference). The aligned-scorer rerun
  (Figs. 15–16) keeps the hard-task pattern, which answers the scorer
  half. No arm selects on validation pass@k or on newly covered tasks, so
  the selection half is untested. The authors name this as future work.
- **The scorer finding is the most portable result here, and it sits in an
  appendix.** Two benchmark scorers ranked a bigger model below a smaller
  one at n = 128. In both cases the fault was a false negative (a correct
  answer inside a sentence; an unrequested-answer check). A do-nothing
  probe would not catch this, because both scorers correctly give an
  empty answer 0. A known-ordering check (does the stronger reference
  outscore the base?) caught it. The Judge fix adds an LLM judge whose
  model and agreement rate are not reported.
- **One optimizer, one proposer.** The paper tests only Meta-Harness with an
  Opus 5 proposer, on two solver families. The RSIHub item in the same
  digest (2610.32253) reports method rankings varying by setting, so this
  result may not generalize to GEPA-, ACE- or AHE-style loops.
- **Figure-to-figure inconsistency.** Fig. 3 says one run per red curve,
  Figs. 5–6 say "one selection from two optimization runs", and App. C.1
  says three runs per benchmark. Which runs are shown in the main text is
  not stated.
- **What it does not show.** It covers no ML-research loop, only QA,
  classification and app tasks with short attempts. It has no
  selection-rule arm, so the "reward new coverage" fix is a proposal. It
  does not test whether the extra coverage on AppWorld would feed a better
  next round of RSI, which is the motivation it gives.

## Trust signals

- **Credibility:** 3.
  - For:
    - An industry lab (FuriosaAI) with Yonsei and Berkeley co-authors.
    - Code and the full trajectory dataset are released and named in the
      PDF.
    - Test splits are disjoint from search and selection, and the split
      sizes are tabulated.
    - Three optimization runs per benchmark, with individual runs plotted.
    - Ablations that hit the obvious objections: frozen memory, a
      random-guess floor, aligned scorers, resource caps, and a second
      solver size in each direction.
    - It reports at least one result against its own story (20B HotpotQA
      beats 120B at pass@128).
    - A clearly specified estimator.
  - Against:
    - Not peer reviewed; v1 is a week old.
    - No intervals on any pass@k value.
    - Hard subsets of 41 and 63 tasks, with headline gaps of 0–2 tasks.
    - One optimizer and one proposer.
    - The lead figure comes from a non-main setting.
    - The main runs were selected under scorers the paper itself found
      biased.
    - The LLM judge is unnamed and unvalidated.
    - The stronger reference is not compute-matched.
  - Not 4 because the decisive comparisons rest on a few tasks with no
    uncertainty and a cherry-picked lead example. Not 2 because the design
    is careful, the robustness appendix is unusually complete, and the
    trajectories are public, so every number can be rechecked.

## Follow-up

- **Relevance:** 4. It is the first source in the graph to apply the
  pass@k-across-budgets lens to harness optimization, the same class of
  loop as `/elevate`'s self-application. It adds three things:
  - [[concepts/pass-at-k]]: report a harness or loop gain at the intended
    k, stratified by difficulty, with the success-set transition counts.
    This is a third independent separation of pass@1 from pass@k, after
    [[literature/papers/li2026acm]] and [[literature/papers/agarwal2026fire]].
  - [[concepts/evolutionary-expansion]]: fitness = validation pass@1
    selects reliability, and redirecting the task mixture did not buy reach.
  - [[concepts/hce-evaluation]]: a stronger-reference arm, and a
    known-ordering check that caught two broken scorers which a do-nothing
    probe would have passed.

  [[concepts/evolutionary-search-grain]] gets no edit. The paper does not
  vary the mutation unit. Its candidate-level result (Fig. 8: all 40
  candidates below 32B on hard questions) is about selection and reach, and
  is filed under evolutionary-expansion. Not 5: there is no ML-research
  loop, no intervals, and one optimizer.
- **For the `hce-known-bad-preflight` proposal (flag only).** The proposal's
  do-nothing probe catches scorers that pass the empty answer. This paper's
  two broken scorers fail the other way: correct answers scored 0. A
  third probe would close that gap: score a known-correct answer in a
  non-canonical form, such as wrapped in a sentence or carrying extra
  fields. The 10-20 hce-evaluation section's known-good arm would not catch
  it if the known-good answer is the canonical gold string, which EM
  accepts. If a stronger model is cheap to run, add a known-ordering
  check (stronger model ≥ base on a small validation slice). One paper,
  one group, so this is a note for the proposal's evidence list, not a new
  proposal.
- **For redirect-before-halt.** Tangential. The "redirect" here changes the
  optimizer's *training mixture*, not a stalled loop's approach. It
  raised hard pass@1 by 2.4 pp with no added hard coverage (one run, 41
  questions). It is weak evidence that re-weighting what a loop is scored
  on moves reliability, not reach.
- **Cheap local check.** The trajectory dataset should let the
  success-set transition matrix (Table 3 style) be computed for HotpotQA
  and AppWorld hard subsets, with a task bootstrap on the hard-subset
  pass@128 gaps. That would show whether "none exceeds the reference" is
  distinguishable from a tie. It needs no GPU.
- **Candidates** (cited, not yet in the graph):
  - Lee et al. 2026b, "Meta-Harness: End-to-end optimization of model
    harnesses" (COLM 2026). The optimizer studied here, and also used in
    ding2026harnessdelta and wang2026rethinking.
  - Yue et al. 2025, "Does reinforcement learning really incentivize
    reasoning capacity in LLMs beyond the base model?" (NeurIPS 2025). The
    source of the pass@k elicitation diagnostic.
  - Noukhovitch et al. 2026, "Learning to solve hard problems in RL for
    LLMs by never giving up" (arXiv 2609.13443). The Matthew-effect
    analogy.
  - Lin et al. 2026b, "Harness updating is not harness benefit" (arXiv
    2605.30621). Already cited by xia2026rrsi and ding2026harnessdelta.
  - Gong & Wen 2026, "Why prompt optimization works, and why it sometimes
    doesn't: a causal-inspired edit-level analysis" (arXiv 2605.26655).
  - Yang et al. 2026, "One recipe, many harnesses: what self-evolution
    encodes across languages and models" (arXiv 2608.10178).
  - Zhao et al. 2026, "Beyond prompts: measuring and optimizing LLM
    tool-agent harnesses" (arXiv 2609.05736). On the repeatability of
    harness gains.
  - Lee et al. 2026a, "Recursive harness self-improvement" (arXiv
    2607.15524). Already cited by wei2026scale.
