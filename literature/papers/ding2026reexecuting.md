---
kind: paper
title: "Re-executing Coding-Agent Evaluations: Unchanged Decisions and Unresolved Policy Risk"
authors: ["Tianyu Ding", "Yaocheng Tong", "Muzhi Yang", "Fei Jiang", "Lei Yang", "Boya Yu"]
institutions: ["Independent researchers"]
year: 2026
venue: "arXiv 2610.14460 (cs.SE), v1 2026-10-10, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.14460"
code_url: null
citations: null
source: "raw/papers/ding2026reexecuting.pdf"
added: "2026-10-20"
relevance: 3
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/shared-substrate-contagion]]"
tags: ["re-execution", "grader-noise", "evaluation-infrastructure", "environment-drift", "rerun-instability", "cpu-contention", "offline-grading", "hermetic-grader", "gold-bound-cache", "dependency-coverage", "preregistration", "sealed-protocol", "clopper-pearson", "exposure", "darwin-godel-machine", "swe-bench-pro", "swe-bench-verified", "lego-rl", "noise-band"]
---

# Re-executing Coding-Agent Evaluations: Unchanged Decisions and Unresolved Policy Risk

## TL;DR

Two preregistered studies re-run the grading of coding-agent patches that
were already scored. Both return the predicted negative ("KILL") result,
and the authors say plainly that neither negative is a safety assurance.

1. **Study 1 (SWE-bench Pro v1, offline grading).** Does the evaluator's
   `--block_network` option reject patches that the networked official
   grader accepted? On a 120-task prefix, 0 of 281 evaluable accepted
   patches lost their verdict. But only 59 of the 281 (all Go, 25 tasks)
   faced a configuration the offline option changes. The other 222 ran
   under identical arms. The post-hoc upper bound on the exposed rejection
   rate is 4.95%, above the 3% the study was built to detect. A follow-on
   extension over 63 more Go tasks was inconclusive. A separately
   *selected* stratum of 11 tasks showed the mechanism exists: a
   reconstructed Lego-RL gold-bound module cache passed the gold patch on
   11/11 but rejected 6 of 15 accepted alternatives.
2. **Study 2 (Darwin Gödel Machine archive).** Do rerun instability or CPU
   contention reverse DGM's recorded decisions? 1,929 archived
   agent-version/task cells were each run 3× at four CPUs and 3× at a
   quarter CPU (11,574 runs). Best-node selection and the three-arm ranking
   did not change. Only 2 cells (one Sphinx task) were unstable. 23 cells
   (1.19%, on 4 tasks) disagreed with the archive in all six runs. A
   second host undid 7 of those 23, and rebuilt images changed 16 majority
   verdicts (0.83%). Neither changed a decision. The decisions had
   margins of 13 tasks (D1) and at least 7 flips (D2), and both margins
   were known before the seal.

## Claims

- "Two preregistered re-execution studies returned negative results"
  (abstract). This is accurate.
- On Study 1, the authors say: "The KILL therefore gives no safety
  assurance" (§4.2). A "3% exposed-patch risk" remains "unresolved".
- On Study 2, "the test asks whether DGM's recorded endpoints survive the
  noise re-execution can expose and says nothing about close decisions"
  (§5.2). They also say "D2 held because of where drift fell as well as
  because of its margins" (App. E).
- Validating a hermetic grader on the gold patch is not enough. It should
  be validated on accepted alternatives too (Lesson 4). A gold-bound
  cache "rejected accepted alternatives from four models that passed with
  their own dependencies."
- Audits should sample the policy-exposed stratum directly, and check
  before data collection that it holds enough independent units (Lesson 1).
  Random task sampling dilutes the contrast.
- Re-executions should check replay fidelity, meaning patch and grading
  inputs, and report the runtime environment. Matching patch and commit
  records "did not ensure matching verdicts" (Lesson 3).
- Scope, stated by the authors: accepted patches and fixed-history
  decisions only. "General policy safety and adaptive-learning robustness
  require broader evaluations."

## Methods

- **Preregistration and sealing (App. C).** Each study fixes a gate. A
  complete, valid result below threshold is KILL, and a result that fails
  completeness or validity is INCONCLUSIVE. Protocol, code, manifests,
  seeds, sample reservation, analysis script and predictions were sealed
  in a pushed commit before any confirmatory data. The runner refuses to
  start unless the sealed files match. Five amendments were committed as
  dated appendices before the data they affect. A reviewer model from a
  different family ran design and integrity audits. These audits could
  make a result inconclusive but could not pass a gate.
- **Study 1 arms.** Each artefact runs three times per arm, in fresh
  `--network none` containers (4 CPUs, 8 GiB, one-hour timeout).
  - **A0** emulates the networked contract with a read-only Go module
    mirror filled request by request. It must reproduce the official
    verdict.
  - **A1 (P_BLOCK)** is the offline option: shipped image only. This is
    the confirmatory policy.
  - **A2** adds the candidate's own modules, except the task's module
    path.
  - **A1_LEGO** is a reconstruction of Lego-RL's gold-bound cache, run
    descriptively on gold-bump Go tasks.
- **Study 1 rules.** A task is *eligible* if the gold patch passes all
  three restricted runs and the empty patch fails all three. A *confirmed
  reversible loss* needs four things: A0 resolves 3/3, A1 rejects 3/3, A2
  resolves 3/3, and the A1 logs show the network-error signature.
  Candidates get inverse inclusion-probability weights. The gate requires
  ≥200 evaluable candidates from ≥2 models, a weighted rate ≥3%, a
  one-sided 95% lower bound >1%, and losses on ≥5 tasks. The
  preregistered power was 0.48 (independent) or 0.40 (clustered) at a true
  rate of 3%, and 0.95 / 0.88 at 5%. The prior on KILL was about 0.85,
  revised to about 0.65 before confirmatory data.
- **Study 1 sample.** The SWE-bench Pro v1 public split, with patches from
  nine published agent runs. The sample is the first 120 tasks by seeded
  hash, reserved from metadata before any patch was fetched. Each task
  contributes up to 4 candidates plus gold, empty and one officially
  unresolved control.
- **Study 2 object.** DGM's released archive has 245 nodes and 6,006
  evaluations on 200 SWE-bench Verified tasks, in three arms: the full
  system, no self-improvement, and no open-ended exploration. Recorded arm
  bests are 100/200, 23/59 and 14/60.
  - **D1** is best-node selection among the eight full-system nodes
    (100/200 vs a runner-up of 87/200).
  - **D2** ranks the three arms by best score.
  - Replay covers 15 nodes (the eight D1 nodes plus each arm's top three).
    All 1,929 cells come from three repositories: 1,399 Django, 357 Sphinx
    and 173 Astropy.
- **Study 2 quantities.**
  - Q1 rerun instability: the three nominal runs disagree.
  - Q2 drift: the recorded verdict differs from the nominal majority.
  - Q3 contention disagreement: the contention majority differs from the
    nominal majority.

  Only Q1 and Q3 reversals count toward the hypothesis, and a reversal
  must reproduce in a confirmation batch. Drift is reported separately.
- **Replay fidelity (App. E).** DGM strips test-file and `tox.ini` diffs
  before grading. The authors reimplemented that filter and matched the
  archived evaluated patch by size and CRC32 on all 6,006 cells.
  Regrading archived logs with the pinned grader had to agree on ≥99%.
- **Exploratory follow-ups X1–X5 (App. I).** These are second-host replay,
  rebuilt images, grader version 5.0.2, suite-incompleteness flags, and
  removing test-infrastructure diff blocks. All are labelled exploratory,
  and some are "exploratory-unsealed" because their scripts were revised
  after the data was seen.

## Results

- **Study 1 confirmatory (Table 1).** 664 artefacts, 5,997 runs.
  - Eligibility: 113 of 120 tasks.
  - Candidates: 312 drawn, 281 evaluable, from 7 models on 110 tasks.
  - Losses: 0 verdict losses. The weighted rate and its lower bound are
    both 0.00%. Result: KILL.
  - Sensitivity: counting the 11 baseline-unreproduced candidates on
    eligible tasks as losses gives 3.90% weighted. That is above 3%, but
    it fails the lower-bound and five-task conditions.
  - The 31 unevaluable candidates, including OOM kills, drop out of the
    count. They "may include policy-sensitive cases".
- **Exposure (Fig. 1).** Only the 59 Go patches faced different
  configurations. Python (152), JavaScript (69) and TypeScript (1) patches
  ran under identical arms. One-sided 95% Clopper–Pearson upper bounds,
  post hoc and unweighted:

  | Unit | Upper bound |
  |---|---|
  | 59 exposed patches, independent | 4.95% |
  | 25 exposed tasks, independent | 11.29% |
  | Counting the 13 unevaluable Go draws as rejections | 27.17% |
  | Hypothetical: all 91 Go tasks in the frame eligible, none rejecting | 3.24% |

  Reaching 3% needs 99 independent exposed units with zero rejections.
  This frame cannot supply that at task level.
- **Extension (App. F).** 63 more Go tasks with a looser rule (any failed
  restricted run counts). Inconclusive: one gold baseline run failed,
  which the control rule does not allow. The descriptive tally is 2
  rejection tasks among 59 eligible. Both rejections, and the failed gold
  baseline run, logged the same Teleport nil-pointer panic, so the panic
  is not confined to the offline policy.
- **Gold-bound cache (selected feasibility stratum, 11 tasks).** P_BLOCK
  failed the gold patch on 8 of 11. The reconstructed cache passed it on
  11/11. Of 15 baseline-reproduced accepted candidates, 6 failed all three
  cache runs and passed once their own modules were added. They came from
  five runs of four models on three tasks. In one Flipt task, three models
  each added a `Masterminds/semver` version absent from the gold `go.sum`.
  This shows the error exists, not how common it is.
- **Provisioning safety failed.** One officially unresolved Python
  candidate passed one of three A2 runs after failing every other run.
  This is consistent with flakiness, and the frozen rule counts it.
- **Study 2 confirmatory.**
  - Regrade agreement: 5,999 of 6,006 (99.88%). All seven disagreements
    are on Astropy 7606, where a later revision of the SWE-bench Verified
    reference test list dropped a required test.
  - All 11,574 runs completed, 68.55 container-hours in total.
  - Instability: 2 unstable cells, both on sphinx-10466, and they are the
    only contention disagreements. Upper bounds are 0.33% per cell and
    2.35% per task. The substitutions narrowed D1's margin from 13 to 12
    tasks. Result: KILL.
- **Drift (Fig. 2).** 23 cells (1.19%) on 4 tasks, each stable across all
  six runs.
  - 7 Astropy cells, from the reference test list revision.
  - 9 Sphinx cells, where tests fetch an external URL and the replay has
    no network.
  - 7 Django cells, cause not isolated.

  Applying all drift at once leaves D1 and D2 unchanged. 22 of the 23
  drift cells fell on D1 nodes, and they cancel on the incumbent and
  runner-up (net 0 each).
- **Follow-ups (exploratory).**
  - X1 second host: verdicts match on 11,515 of 11,574 runs (99.49%).
    Majorities differ on 10 cells. The 7 Django drift cells resolve in
    all 42 original-host runs and in none of the 42 second-host runs.
    The registered reproduction criterion is NOT MET.
  - X2 rebuilt images (2026-10-05): 16 majorities change (0.83%): 8
    Astropy, 7 Django, 1 Sphinx.
  - X3 grader 5.0.2: it rejects all 6,006 logs unadapted. With a
    delimiter adapter, verdicts match on every cell, and 709 `not_found`
    outcomes are treated as unresolved.
  - X4 suite-incompleteness flags: log-based heuristics flag 189 of 1,929
    cells (9.8%), or 150 (7.8%) under the stricter variant.
  - X5 test-infrastructure blocks removed: 53 of 59 selected resolved
    cells still resolve. The 6 that fail also failed with the full patch.
  - D1/D2 are unchanged in every comparison.

## Critique / open questions

- **Against the digest framing:** AT. It is mildly over-read only in the
  "Why" line. Every number in the digest entry is in the paper and
  correct: 1,929 cells, 0.83%, 7 of 23, 0 of 59. "Both negative" is
  correct, and so is the title's own "unresolved policy risk". Three
  things are left out or added:
  1. The digest omits the power and exposure qualifiers that the paper
     puts up front. Study 1 had about 0.48 power at the target rate, its
     bound (4.95%) sits above the 3% it tested, and the authors say the
     KILL "gives no safety assurance". Study 2's decisions had 13-task and
     ≥7-flip margins known before the seal. So "preserved selection and
     ranking" is a low-power test of a robust endpoint, not evidence that
     DGM-style decisions are stable in general.
  2. It omits the one positive finding: a gold-validated hermetic cache
     rejected 6 of 15 accepted alternatives in the selected stratum.
  3. The "Why" claim, that infrastructure noise is small next to
     item-sampling noise, is not measured here. The paper never compares
     with agent or item resampling. The comparison holds only against
     another paper: [[literature/papers/liu2026what]] reports about 13%
     agent-rerun task flips on SWE-bench Verified, and Study 2's per-cell
     instability bound is 0.33%. That is our inference across two papers.
- **What this means for the noise band (accepted 10-11).** For a fixed
  artifact, grader re-execution noise is an order of magnitude below
  agent-rerun noise. The instability bound is 0.33% per cell and drift is
  ≤1.2%. So a band built from agent seeds and implementations
  ([[literature/papers/ning2026implementation]]) is sized by the right
  term. But grader-side error has a different shape: it clusters by task
  and is *stable* across repetitions on one host. All 23 drift cells held
  in 6 of 6 runs, and only a second host or rebuilt image moved them. A
  same-host `--seeds` std cannot see it. In a paired champion/candidate
  comparison on one host it mostly cancels, as it did for D1 (net 0). It
  does not cancel when a candidate's change interacts with the broken
  item, for example a test that needs the network.
- **What this means for shared-substrate correlated error.** Repetitions
  that share a host, image and reference labels share their errors. Three
  reruns measured the same 23 wrong verdicts three times. This is
  correlated error from shared infrastructure, not the artifact-store
  contagion that [[concepts/shared-substrate-contagion]] defines. It
  supports the same rule: diversity of substrate is the only detector.
  Concept not edited.
- **What this means for HCE.** A zero-event audit certifies only the
  units it exposed. In Study 1, 222 of 281 "clean" patches never met the
  policy under test. This is the same failure as
  [[literature/papers/giampieri2026cortex]]'s plants that never reached
  the regression gates. The sealing mechanism, a runner that refuses to
  start unless sealed files match, is a working instance of the binding
  [[literature/papers/waks2026what]] asks for.
- **What it does not show.**
  - Anything about adaptive search. Intermediate DGM selections cannot be
    reconstructed (38 of 89 non-root nodes fall below the released staged
    threshold), so replaying fixed patches says nothing about how
    re-executed verdicts would have steered evolution. The authors
    concede this.
  - Prevalence of the gold-cache error. The stratum was selected for
    exposure.
  - Semantic correctness. Acceptance is the operational criterion
    throughout, so a 0% loss rate is a loss rate against tests, not
    against correct fixes.
  - Any research-agent or ML-training loop. Study 2's 1,929 cells come
    from three Python repositories.
- **Redirect-before-halt proposal: no bearing.** The paper studies
  graders, not loop control.
- **Small points.** The study runs on Docker 25.0.16 with cgroup v1, and
  contention is a single CPU-quota profile with three runs. The authors
  say this "isolates no causal contention effect". Raw logs and some
  patch hunks are withheld. The first author here (Tianyu Ding) is not
  the first author of [[literature/papers/ding2026harnessdelta]]
  (Wenxuan Ding).

## Trust signals

- **Credibility:** 3.
  - For: the method is unusually disciplined.
    - Both studies were preregistered, sealed by commit, and amended only
      by dated appendices. The runner enforces the seal.
    - Kill rules were stated with power simulations, and the prior
      probabilities were disclosed and revised.
    - Integrity audits were run by a model from another family.
    - A version-coverage table dated 2026-10-07 lists every external pin.
    - A second-host replication was run, and its failure (NOT MET) is
      reported.
    - The authors repeatedly decline to over-claim ("no safety
      assurance", "says nothing about close decisions").
  - Against:
    - No peer review.
    - The authors list themselves as independent researchers. The author
      statement names current or former employers (Amazon, Airbnb,
      Microsoft, Warner Bros. Discovery, Capital One), and correspondence
      goes to a pitt.edu address.
    - The replication package is anonymized supplementary material with
      no public URL, and it was not checked here. Raw logs are withheld.
    - AI assistants were used for writing, coding and internal review.
    - Several follow-ups use scripts revised after the data.
  - Not 4 because the authorship cannot be verified, nothing is peer
    reviewed and the artifacts are not publicly checkable. Not 2 because
    the preregistration, sealing and candour are well above the median
    preprint here.

## Follow-up

- **Relevance:** 3. This is useful, careful prior art on evaluator
  re-execution noise for coding agents. It gives a measured floor for
  grader-side noise on fixed artifacts, about 0.3% unstable and ≤1.2%
  drift. Next to liu2026what's agent-rerun flips, this says the noise band
  should be sized from agent resampling. It also gives the existence
  proof that a gold-validated hermetic grader can reject correct
  alternatives. It does not shift a load-bearing concept, and no task is
  an ML-research loop. Not 4 because both headline results are
  low-power negatives and the one positive result is from a selected
  stratum.
- **For this repo's graders.** Any hermetic or offline scorer we build,
  for example an `/iterate` evaluator with network disabled, should be
  checked in two ways. Check the reference passes and the null fails, as
  in this paper's eligibility rule. Also check that it accepts at least
  a few *alternative* correct solutions whose dependencies or approach
  differ from the reference.
- **Candidates** (cited, not yet in the graph):
  - Wang et al. 2026a, "Auditing Harness Tampering in Self-Improving
    Agents" (arXiv 2609.00069).
  - Du et al. 2026, Lego-RL: harness-native RL for coding agents (arXiv
    2608.17393).
  - Zheng et al. 2026, SWE-Bench Pro Verified (arXiv 2609.08149).
  - Huang et al. 2026b, DSec (arXiv 2609.22978): agents fetching upstream
    fixes through module proxies.
  - Cai et al. 2025, RL with verifiable but noisy rewards under imperfect
    verifiers (arXiv 2510.00915).
