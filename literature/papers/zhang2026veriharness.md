---
kind: paper
title: "VeriHarness: Scaling Agentic Verification for Long-Horizon Tasks"
authors: ["Caiqi Zhang", "Rujun Han", "Zifeng Wang", "Zoey CuiZhu", "Nigel Collier", "Tomas Pfister", "Chen-Yu Lee"]
institutions: ["University of Cambridge", "Google Cloud AI Research"]
year: 2026
venue: "arXiv 2610.00972v1 (cs.AI), 2026-10-01; preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.00972"
code_url: "https://github.com/google-research/veriharness"
citations: null
source: "raw/papers/zhang2026veriharness.pdf"
added: "2026-10-06"
relevance: 4
credibility: 4
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/pass-at-k]]"
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/information-firewall]]"
tags: ["test-time-verification", "best-of-n-selection", "same-model-verifier", "agentic-verifier", "consensus-error", "common-mode-error", "claim-entropy", "disagreement-resolver", "consensus-challenger", "evidence-record", "revision", "skill-evolution", "held-out-split", "selection-oracle", "apex-agents", "spreadsheetbench-2", "rollout-dataset"]
---

# VeriHarness: Scaling Agentic Verification for Long-Horizon Tasks

## TL;DR

Given a frozen pool of N = 10 rollouts per task, the generator's own model
(Gemini 3.5 Flash or Claude Opus 4.8) is put inside a verification harness:
a read-only sandbox holding the rollouts and the task environment, file and
shell tools, and an 18-skill library. Two investigators run in separate
contexts:

- a **disagreement resolver** checks claims the rollouts dispute against
  the environment;
- a **consensus challenger** proposes how a unanimous claim could be wrong
  and tests that.

A fresh context then adjudicates both evidence records into a base rollout,
a revision plan and a list of unresolved claims.

On five workspace benchmarks (APEX-Agents, Workspace-Bench Lite, WorkBuddy,
SpreadsheetBench 2, JobBench), selection beats every baseline's mean in all
ten model × benchmark cells. The gain over the pool mean is **+4.4 (Flash) /
+4.1 (Opus)** for selection and **+6.2 / +6.4** after revision. Against the
strongest baseline the margin is **about +2 (selection) and +3 (revision)**.
Selection captures about 30% of the oracle headroom.

For this graph the paper does three things:

- It is a well-instrumented case of "agreement ≠ correctness" within one
  model's samples.
- It shows that environment access alone buys almost nothing: the protocol
  and skills carry the gain.
- It gives an evolved-skill loop with a group-aware dev/held-out split and
  an admission rule.

## Claims

- **Consensus can conceal errors; disagreement exposes alternatives.** On
  APEX-Agents with Opus 4.8 ten-rollout pools: "34% of consensus values are
  judged incorrect, while 74% of disputed claims contain a correct candidate
  and the most frequent value is correct in only 47%."
- **Two checking tasks.** Disputed claims need evidence that discriminates
  among exposed alternatives. Consensus claims need the verifier to "first
  propose how the consensus value could be wrong", which "requires knowledge
  of how artifacts fail, which the pool does not supply". Skills are the
  paper's source of that knowledge.
- **Access is not sufficient.** "the agentic verifier with the same
  environment access but no protocol or skills recovers only about half of
  the harness's gain."
- **Revision gains come from the evidence.** "Aggregating the pool into a
  new artifact without evidence checks stays below the harness on every
  benchmark, so the revision gain comes from the evidence records rather
  than from rewriting itself."
- **Skills evolve from failures.** A library evolved from empty beats the
  human-authored one on held-out tasks. Evolving from the human library is
  best.
- **The challenger is safe.** "it never refuted a claim that every rollout
  had right."
- **Why shared errors survive.** "The verifier tends to stop where the
  rollouts stopped."
- **The paper's strongest wording**, from the abstract: VeriHarness is "a
  novel and critical approach", and the first agentic verification harness
  for long-horizon tasks.

## Methods

- **Setting.** Same-model verification. Generator and verifier share the
  model and tools, and the verifier never sees the grader, rubrics or
  reference answers. All methods get the same frozen pool of 10 rollouts.
  "Single rollout" is the **pool mean**.
- **Benchmarks (Table 1).**
  - APEX-Agents v1.0: 480 tasks, expert rubric graded by an LLM judge
    (Gemini 3 Flash).
  - Workspace-Bench Lite: 100 tasks. Its agent-as-judge is Claude Code
    running **Opus 4.8**.
  - WorkBuddy: 200 tasks across code, office and web; the security domain
    was dropped. A composite of tests, rules and a Gemini 3.5 Flash judge.
  - SpreadsheetBench 2: 321 tasks, deterministic recalculation and cell
    comparison.
  - JobBench: 65 tasks, rubric judge.
- **Generators** run in each benchmark's own scaffold: the APEX runner,
  opencode, the SB-2 agent, and Claude Code.
- **Verifier runtime.** A thin layer over the open-source `pi` coding
  agent. Rollouts and environment are mounted read-only with no network;
  only the output directory is writable.
- **Protocol (Algorithm 1).**
  - Resolver and challenger run concurrently and cannot see each other.
  - The resolver picks checks by qualitatively judged information gain and
    eliminates candidates that are inconsistent with the evidence.
  - The challenger covers consensus values, readings and omissions.
  - A fresh adjudicator context outputs (base, revision_plan, unresolved).
    Unresolved interpretations are delivered with **both readings** "where
    the artifact format allows".
- **Claim analysis (App. A).** Covers 434 APEX tasks and 1,692 claims, Opus
  only. Opus 4.8 extracts each rollout's value per rubric criterion, and
  "omission [is] a distinct value". A value counts as correct when most of
  the rollouts asserting it pass the criterion.
- **Baselines.**
  - Majority voting (most-consistent rollout).
  - Best-of-N with a judge.
  - A pairwise tournament.
  - LLM-as-a-Verifier (Kwok et al. 2026).
  - Agentic verifier: the same sandbox and tools, but one scoring
    instruction and no protocol or skills.
  - For revision: aggregation over the pool (no environment access) and
    agentic verifier + revision.
  - The selection oracle (best rollout by the grader).
- **Seeds.** 3 verifier seeds; mean ± SD reported.
- **Self-evolution (§5, App. I).**
  - Opus only, on APEX-Agents (file-deliverable tasks excluded) and SB-2
    (template and financial-model tasks).
  - About a 75/25 dev/held-out split. Tasks sharing a workspace, documents
    or a template are grouped before splitting.
  - Each round proposes K = 3–5 candidate libraries. The candidate with the
    largest **non-negative paired dev gain** replaces the current library.
  - An automatic filter rejects task IDs, benchmark names, dev filenames
    and reference values.
  - Round schedules (10 rounds for C, 8 for D) were fixed before any
    held-out trajectory was inspected.
- **Ablations (Table 4)** cover resolver-only, challenger-only, no skills,
  and a single shared context.
- **Cost (App. K).** Metered tokens at list price, with cached input at
  the cache-read rate. Majority voting, best-of-N and the CLI runs are
  estimates, not metered.

## Results

- **Table 2, five-benchmark averages:**

  | | Flash | Opus |
  |---|---|---|
  | Single rollout (pool mean) | 47.2 | 49.6 |
  | Best non-VeriHarness selector | 49.5 (LLM-as-a-Verifier) | 51.6 (agentic verifier) |
  | Agentic verifier (env. access) | 49.4 | 51.6 |
  | **VeriHarness (select)** | **51.6** (+4.4) | **53.7** (+4.1) |
  | Agentic verifier + revision | 50.7 | 53.1 |
  | **VeriHarness (revise)** | **53.4** (+6.2) | **56.1** (+6.4) |
  | Selection oracle | 62.7 | 63.1 |

- **Our arithmetic on Table 2.**
  - Selection captures 4.4/15.5 = **28%** (Flash) and 4.1/13.5 = **30%**
    (Opus) of the oracle headroom.
  - The margin over the best baseline is +2.1 / +2.1 for selection and
    +2.7 / +3.0 for revision.
  - The smallest per-cell selection margin is Flash SB-2: 31.5 ± 2.4 vs
    30.9 ± 1.1, i.e. +0.6, inside one SD.
- **Largest revision gains by cell.** APEX-Agents with Opus gains +11.7
  (35.8 → 47.5), and WSB with Flash +8.9. Figure 1 shows the Flash cells
  (+6.7 APEX, +8.9 WSB).
- **Ablation (Table 4), Opus average 49.6 → 56.1 for the full harness:**
  - resolver only: 54.3;
  - challenger only: 52.6;
  - no skills: 53.8;
  - single context: 55.3 (−0.8).

  Flash shows the same ordering: 51.5, 50.0, 51.3 and 52.6 vs 53.4.
- **Where selection gains (Fig. 5, APEX, Opus).**
  - 154 consensus pools: **+0.6** over the pool mean.
  - 262 disputed pools: +6.1, rising with entropy: low +4.5 (n = 139),
    medium +7.7 (n = 108), high +9.4 (n = 15).
- **Challenger failures (Table 7).** It upheld unanimous errors in these
  proportions:
  - 70%: it tested an upstream intermediate that was correct, while the
    graded quantity lay downstream;
  - 14%: same quantity, different definition, period or version;
  - 12%: it confirmed the wrong value outright;
  - 4%: noise.
- **CLI transfer.**
  - With Flash, both CLIs come within 2 points of the dedicated runtime.
  - With Opus, Claude Code reaches 51.7 (+2.1, about half the gain).
    JobBench falls to the single-rollout level: 42.4 vs 42.8.
- **Self-evolution (Fig. 4a; held-out; point estimates; one run).**

  | Library | APEX | SB-2 |
  |---|---|---|
  | A: empty | 38.6 | 32.8 |
  | B: human-authored | 45.3 | 35.7 |
  | C: evolved from A | 49.6 | 38.5 |
  | D: evolved from B | 52.1 | 39.4 |

  Of the 95 evolved checks, 5 restate a human check, 48 concretize one and
  42 are new. Both evolved libraries independently learned not to treat a
  recomputation that matches the pool as confirmation.
- **Cost.**
  - Selection costs **$1.44/task** (Flash) and **$3.92/task** (Opus).
    Revision costs about 3× selection.
  - 86–91% of input tokens are cache hits. At full input price, selection
    costs $5.40 and $13.06.
  - VeriHarness is on the cost–gain frontier in both models.

## Critique / open questions

- **Running digest check: this one also reads at or below its framing
  (32/32).** The digest's numbers match the abstract, but each is the
  favourable cut:
  - "+6.2/+6.4 over a single rollout" is the **revision** number measured
    against the **pool mean**. Selection alone gives +4.4/+4.1. Against the
    best competing selector the margin is **+2.1 for both models**, and
    against agentic verifier + revision it is +2.7/+3.0.
  - Selection still leaves about 70% of the oracle headroom on the table.
  - The paper says this itself. It is not misleading, but the number
    that travels is the large one.
- **"Best selection on five benchmarks" holds for the means, on 3 seeds.**
  The Flash SB-2 margin (+0.6, SD 2.4) and several +1.1–1.6 margins (Opus
  WorkBuddy, Opus/Flash JobBench, both WSB) are within about one to two
  SDs. "gains more than the cross-seed spread in every cell" refers to the
  gain over the pool mean, not over the runner-up.
- **Revision gains concentrate where an LLM judges.** This is our
  inference, not the paper's.

  | Grader | Benchmark | Revision adds over selection |
  |---|---|---|
  | Deterministic recalculation | SB-2 | +0.7 (Flash), +0.9 (Opus) |
  | Mostly deterministic | WorkBuddy | +0.4, +0.7 |
  | LLM rubric judge | APEX | Opus +6.3 |
  | LLM rubric judge | WSB | Flash +3.4 |
  | LLM rubric judge | JobBench | +2.7, +1.5 |

  Three things could inflate judged scores:
  - Delivery writes **both readings** of an unresolved interpretation into
    the artifact. A per-item rubric judge can credit either.
  - WSB's judge is Opus 4.8, the same model as the Opus verifier.
  - WorkBuddy-office uses a Gemini 3.5 Flash judge for Flash outputs.

  The aggregation control has no environment access, so it does not rule
  out judge-pleasing by an environment-aware reviser. The fair control,
  agentic verifier + revision, closes about half the gap. A hedging-off
  ablation or a deterministic-grader-only breakdown would settle it.
- **The consensus statistic is one benchmark, one model, model-extracted.**
  The 34% / 74% / 47% figures come from APEX with Opus only. Opus 4.8 itself
  extracts the per-claim values, and Gemini 3 Flash grades correctness.
  Omission counts as a value, so a criterion all ten rollouts skipped
  counts as a "wrong consensus". No breakdown separates shared omissions
  from shared wrong values.
- **The challenger's catch rate is never reported.** We get two things:
  - its precision, "never refuted" a correct unanimous claim;
  - a breakdown of the errors it **missed** (Table 7).

  We do not get how many unanimous errors it fixed, nor the denominator of
  unanimous errors. Challenger-only adds +3.0 (Opus) and +2.8 (Flash).
  That figure includes omission repair and general revision. Because a
  shared error can only be fixed by revision, it rides on the ~3× revision
  cost. Selection on consensus pools gains +0.6. So "consensus conceals
  errors" is well motivated, but the evidence that this challenger
  *un-conceals* them in volume is indirect.
- **This is not contagion.** Ten rollouts of one model have no channel
  between them. The 34% is **common-mode error**, the no-channel arm on the
  independence axis, like kim2026divergent's 15/16 wrong champion. It is
  not propagation through a shared substrate. The digest's "contagion
  thesis applied to sampling" overstates the link.
- **Environment access alone is worth almost nothing.**
  - Flash: the agentic verifier (49.4) is level with no-environment
    LLM-as-a-Verifier (49.5).
  - Opus: 51.6 vs 51.3 for the pairwise tournament.

  The gain is the protocol and skills on top of access. The paper's
  "recovers about half" is measured against the pool mean and hides this.
- **Equal generation cost is not deployment cost.** Every method gets the
  same ten rollouts, so the comparison against a *single* rollout ignores a
  10× generation bill. The release cost ($100k+ for ~26k rollouts) implies
  roughly $3.8 per rollout on average, so about $38 per task for the pool
  vs $1.44–$12 for verification. That is our arithmetic, averaged across
  models and benchmarks. There is no budget-matched comparison, e.g. one
  rollout with more effort, or a 3-rollout pool. The paper cites Wang et
  al. 2026 ([[literature/papers/wang2026rethinking]]) on exactly that
  baseline for harness evolution.
- **Bookkeeping that does not reconcile.**
  - Table 1 gives 1,166 tasks × 10 rollouts × 2 models = 23,320 rollouts,
    not "approximately 26,000".
  - APEX has 480 tasks in Table 1, 434 in the claim analysis, and 416 pools
    in Fig. 5.
- **The self-evolution result is thin.** It is one run, Opus only, on two
  benchmark subsets, scored on a ~25% held-out slice. The authors
  themselves note that "The figures report point estimates and do not
  establish statistical significance."
  - The "human-authored" library B was itself written using development
    failures, so it is not an untuned prior.
  - C beats B by +4.3 (APEX) and +2.8 (SB-2).
  - Both trajectories include held-out regressions.

  The protocol is clean (group-aware split, pre-fixed schedules,
  identifier filter, held-out never fed back). The effect size is the weak
  part.
- **Same-model framing has an asymmetry.** The CLI deployments put the Opus
  verifier inside Claude Code, and Claude Code also generated the WSB and
  WorkBuddy rollouts. That is fine for "same model", but it means harness
  familiarity is uncontrolled.
- **Domain.** These are professional-document workspaces, not ML research.
  The transfer to ML loops is architectural. For a research agent the
  analogous artifact would be a results table or a paper's claims, checked
  against logs and code.

## Trust signals

- **Credibility:** 4.
  - For: Google Cloud AI Research and Cambridge. The verification code and
    the full rollout pool are released, with grader scores, which makes
    the single-rollout, oracle and Fig. 6 numbers reproducible without a
    model. The paper has five benchmarks, two frontier models, 3 seeds
    with SDs, component ablations, metered cost accounting, and an
    unusually careful held-out protocol for skill evolution.
  - Against: a preprint, not peer reviewed. Runner-up margins are
    within about 1–2 SD in several cells. The motivating statistic covers
    one benchmark and one model. The challenger's recall is unreported.
    The rollout counts do not reconcile. Revision gains are not separated
    from LLM-judge effects.

## Follow-up

- **Relevance:** 4. It strengthens existing concepts with material new
  evidence and ablations rather than seeding a new one:
  - [[concepts/pass-at-k]]: selection harvests ~30% of the oracle gap, and
    agreement ≠ correctness within one model's samples;
  - [[concepts/shared-substrate-contagion]]: common-mode error on the
    no-channel arm;
  - [[concepts/evidence-gated-completion]]: claim → check → evidence →
    verdict records, unresolved claims kept open, and access alone being
    insufficient;
  - [[concepts/skill-library-lifecycle]]: a gated evolution loop with a
    held-out split.

  It is not a 5: the domain is workspace tasks, not ML research, and the
  architectural novelty over agentic judges is the two-context split plus
  skills.
- **The design transferable to this box.** For any best-of-k selection in
  an `/iterate`-style loop:
  1. compute which claims the k runs disagree on;
  2. send those to a resolver that must cite a log or file;
  3. separately, have a challenger test what all k agree on, especially
     downstream quantities rather than the intermediate the runs stopped
     at.

  The learned rule "a recomputation that matches the pool is not
  confirmation" belongs in any verifier prompt.
- **Connections.**
  - Converges with [[literature/papers/kim2026are]] and
    [[literature/papers/wang2026rethinking]]: selection, not candidate
    generation, is the bottleneck.
  - Converges with [[literature/papers/kim2026divergent]]: replicas agree
    on wrong answers.
  - Converges with [[literature/papers/xu2026dont]]: the verifier is
    pulled by the account. Here the Table 7 survivors stopped where the
    rollouts stopped.
  - The fresh-context adjudicator without the investigators' history is a
    mild instance of [[concepts/information-firewall]]. Merging contexts
    costs 0.8 points.
- **Candidates.**
  - Kwok et al. 2026, LLM-as-a-Verifier (arXiv 2607.05391), the strongest
    pool-only baseline.
  - Lee et al. 2026b, Agentic Aggregation (COLM 2026).
  - Long et al. 2026, Self-verification dilemma (arXiv 2602.03485):
    self-checks are mostly confirmatory.
  - Goel et al. 2025, "Great models think alike" (ICML 2025): correlated
    errors between a judge and a generator from the same family.
  - The released rollout pool (huggingface.co/datasets/caiqizh/veriharness)
    could test the "both readings" judge-inflation hypothesis offline.
