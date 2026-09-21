---
kind: paper
title: "Agora: Git as Shared Memory for Collective AutoResearch"
authors: ["Yifan Zhang", "Yunheng Zou", "Shaokun Zhang", "Jian Hu", "Hao Zhang", "Binfeng Xu", "Jan Kautz", "Yi Dong"]
institutions: ["NVIDIA"]
year: 2026
venue: "arXiv (cs.LG)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.18094"
code_url: null
citations: null
source: "raw/papers/zhang2026agora.pdf"
added: "2026-09-21"
relevance: 5
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/file-as-bus]]"
  - "[[concepts/async-worker-pool]]"
  - "[[concepts/llm-wiki-pattern]]"
  - "[[concepts/budget-as-ceiling]]"
tags: ["git-as-memory", "provenance", "contribution-dag", "multi-agent", "coordination", "verification", "diversity", "monoculture", "autoresearch", "negative-result", "uncontrolled", "n-of-1"]
---

# Agora: Git as Shared Memory for Collective AutoResearch

## TL;DR

This repository's founding thesis — *git is the memory layer* — built and
run by NVIDIA as a multi-worker institution rather than a single-operator
vault. Research is an append-only DAG in Git: every result, insight,
hypothesis, verification and report is a commit, parent edges mean "builds
on," and a SQLite index derived from (and rebuildable from) the history
exposes the frontier, neglected branches and per-claim verification status.
The first sustained run put 13 LM worker sessions with **no assigned tasks
and no central planner** on a no-training weight-transfer problem for
**11 days 19 hours**; they published **1,703 contributions** and moved the
evaluator from **3.3923 to 1.899044 bpb**. What the paper actually
establishes is much narrower than the framing suggests, and the authors say
so: **98% of the total reduction came from the first 18 scored
contributions**, the remaining 1,106 found 0.03, the community then spent
**five days refining one recipe by 10⁻⁵ bpb per step** — roughly two orders
of magnitude *below* the paper's own cross-hardware noise floor of
1.3 × 10⁻³ — and left that basin only after a human deployed new views
mid-run. There is **no control arm**: no matched run without Agora, without
the diversity views, or with a plain leaderboard. The matched comparison is
Appendix C, and it was not run.

## Claims

- **Git is the only state.** "Git is the only state the system depends on:
  the SQLite index that answers queries, the analyze views below, and every
  figure in this report are derived from the Git history and can be rebuilt
  from it." Workers "never share a filesystem, model, or conversation" —
  they publish through a CLI/HTTP API against a server that mints the
  canonical commit and timestamp.
- **Quality comes from downstream evidence, not votes.** A contribution's
  evidence score is the weighted count of what *other accounts* built on it
  (Eq. 2, with an explicit `1[a(u) ≠ a(v)]` self-citation exclusion).
  Reserved tags carry the weights: `result`/`insight`/`hypothesis`/`report`
  +5, `verification` **+20 / +10 / −20** (confirmed / partial / failed),
  and — the deliberate move — `endorsed` **0**, "Visible, but excluded from
  fitness" (Table 2). Social approval is structurally barred from becoming
  evidence.
- **The score is explicitly not a truth signal.** "The score is not a truth
  signal. It encodes a narrower claim: work that others have reproduced or
  built on is more actionable than work that has only been voted for."
- **Verdicts are revocable without rewriting history.** "If a verifier
  changes its verdict on a target, the newest verdict replaces the old
  one's effect on the score, and both commits stay in the history."
- **A leaderboard is an exploitation signal and a poor map.** "It pulls
  every worker toward the same parent, hides negative results, and makes a
  saturated basin look like progress." The remedy offered is a
  diversity-aware UCB (Eq. 3) that penalises near-duplicate descriptions
  (ρ) and three explicit slots: *exploit*, *explore known*, *explore
  novel*. The authors' own hedge: "The split matters more than the exact
  score."
- **Agora is a coordination substrate, not a lab manager.** It "does not
  try to decide what is true," and prescribes no end-to-end researcher —
  positioned as complementary to AutoResearch / AI Scientist / Agent
  Laboratory, and as serving "asynchronous participants that share no
  conversation, manager, role graph, runtime, or filesystem."
- **The causal claim is explicitly withheld.** "What the run does not settle
  is causal: we did not run the same models and compute without Agora or
  with a plain leaderboard, and the community left its first basin only
  after we showed it a map."

## Methods

- **Node record.** `v = (h, a, T, d, x, m, P, τ)` — canonical commit hash,
  publishing account, tag set, description, structured metadata, optional
  project metric, parent set, server timestamp. Two publication paths:
  *light* (JSON metadata, server creates the commit) and *heavy*
  (participant commits locally, uploads a Git bundle, server validates then
  mints a canonical server-timestamped commit). Both yield the same node
  kind. Server-minted identity and timestamp mean a participant cannot
  forge its own provenance.
- **Derived index.** Go service + Next.js UI + SQLite (eight tables). 26
  HTTP routes, 15 CLI command groups. Bearer auth on writes/clones/fetches,
  rate limits on registration, contribution creation, search, project
  creation and bundle size. "The contribution index can be rebuilt from
  Git; project metadata and authentication state still need ordinary
  database backups."
- **Clustering / diversity views.** Single-link clusters over description
  embeddings; reports cluster count and sizes, top-cluster share, an
  entropy-based effective cluster count, evenness, a metric histogram, and
  a frontier of promising nodes in small clusters. "Clustering requires at
  least 50% embedding coverage, uses a cosine threshold of 0.90 by default,
  and caps pairwise analysis at the 5,000 most recent contributions."
- **Task.** Initialise a frozen 119,572,320-parameter 14-layer
  attention/SSM hybrid (hidden 672, 7 heads, untied embeddings; dimensions
  chosen so **no donor matches any of them**) from a zoo of "141 open-weight
  models (534 GB) from 32 architecture families," with **no training corpus
  and no gradient update on the target**. Submission is a Python
  `transfer(model, config)`.
- **Evaluator.** Seeds every RNG with 42, runs `transfer()`, scores 200
  FineWeb-Edu texts in non-overlapping 512-token chunks under the GPT-2
  tokenizer; summed next-token loss / UTF-8 byte count. The FineWeb-Edu
  loader raises if called from inside `transfer()`. Pretraining,
  fine-tuning, and editing the evaluator or target config are forbidden.
  **"Two runs of the same code on the same hardware are bit-identical;
  across GPU types the score can differ in the third decimal place."**
- **Workers.** "The workers were coding-agent sessions running frontier
  language models: Claude Code … with Claude Opus 4.7 … and Codex …
  with GPT-5.5" (inline reference citations elided from the quote).
  Each ran in a container with one 80 GB GPU, one Agora
  credential, and a one-line headless prompt pointing at a two-page
  `program.md` brief committed as the project's first node. "Nothing in the
  prompt or brief names a method, assigns a role, or ranks the
  participants." When a session ended the launcher started a new one on a
  free credential.
- **Human verification of the record (§4.8).** Three levels: re-export the
  full contribution stream and **recompute every count, statistic and
  figure** ("no number here is taken from an agent's summary or from the
  leaderboard display"); check the best score algebraically against its
  reported loss, token and byte counts; and read the winning commit's chain
  of 83 Python modules from final edit back to root, confirming no step
  touches evaluation data or applies a gradient. Algorithm 1 and Table 3
  were written from that reading, not from agent prose. **"We did not rerun
  the winning method; the primary result is the archived evaluator output,
  corroborated by the agents' cross-hardware reproductions."**

## Results

- **Volume.** 1,703 contributions over 11 days 19 hours (Apr 26 – May 8
  cutoff): "1,124 scored results, 284 insights, 203 hypotheses, 165
  verifications, and one report, with tag overlaps; 233 of the scored
  results set a new best." 13 worker accounts wrote 1,699 of them; 17
  accounts total. Graph: 1,703 nodes, 1,894 edges, 149 multi-parent nodes,
  "one component holding 98.9% of all nodes." Sustained "roughly 170
  contributions per day once all 13 workers were running" (figure caption).
- **The headline, stated carefully by the authors.** The best `transfer()`
  reaches **1.899044 bpb** against **3.3923** random and "about 1.0" for a
  trained GPT-2 124M, "closing 62% of that gap." But the trained model
  "sets the scale; it is not an achievable no-training baseline," and
  "Because every component was selected on the same 200-text development
  evaluator, the number to trust is the improvement from 3.39 to about 1.90
  rather than the final decimal places."
- **Almost all of it happened in the first day.** "Those 18 scored
  contributions account for about 98% of the total reduction," and "The
  first eight improvements account for roughly 70% of the total descent."
  The remaining 1,106 scored results "found the next 0.03."
- **Five days below the noise floor.** "For five days they also refined one
  recipe by 10⁻⁵ bpb per step, each reading the same leaderboard, and left
  that basin within a day of being shown a map of it." The last recorded
  change "moved the score by 9 × 10⁻⁶, below cross-hardware variation"
  (which is up to 1.3 × 10⁻³). Nothing in the system had a noise floor, so
  the leaderboard kept certifying sub-noise deltas as new bests.
- **Monoculture, measured.** "A shared leaderboard did not stop duplicate
  work, and the graph is heavily exploitation-biased." Of 696 pairs of
  different accounts posting identical scores, **63% are within an hour of
  each other and 80% within six**. At the intervention point "more than a
  third of all activity sat in a single semantic cluster and the
  leaderboard had stalled."
- **Lineage is genuinely collective.** "The best contribution at cutoff has
  145 commits in its ancestry, written by 15 of the 17 accounts; 115 of the
  144 parent edges cross account boundaries, so no single worker assembled
  the recipe."
- **Verification produced zero negative verdicts.** "Participants posted 165
  verification contributions covering 95 distinct targets. Each names its
  target, each verifier differs from the author, and **none reports a
  failure**." Coverage of the winning lineage is partial: "Forty of the
  winner's 144 scored ancestors were independently reproduced" (≈28%).
  Same-hardware reproductions are bit-identical; cross-hardware "differ by
  up to 1.3 × 10⁻³ bpb, within the tolerance the brief set for a
  confirmed verdict."
- **Negative results were published and reused.** "The window contains 53
  contributions explicitly tagged as negative results" — doubling the
  prefix set to 48, flattening the singular-value spectrum, transplanting
  native Mamba blocks, copying GPT-2's embedding matrix, and building the
  prior from a different-tokenizer donor (Pythia) all regressed and were
  posted with their scores. "The first scored attempt … scored 4.68, worse
  than random; its author published it as a negative result with an
  explanation."
- **Unrequested protocol emerged from the brief.** "From April 28 onward
  more than 400 descriptions declare a prediction band before the result,
  and later workers explicitly close follow-ups named by earlier ones. We
  did not ask for any of this. It emerged from the brief's reproducibility
  requirement and from the visibility of other accounts' posts."
- **The one intervention.** "On May 2, when the analysis views showed that
  more than a third of all activity sat in a single semantic cluster and
  the leaderboard had stalled, we deployed the clustering, diversity
  summary, and diversity-aware UCB of Section 3.3. Workers adopted the new
  views immediately, and the first sub-1.90 result was published the next
  morning by a worker that chose to follow the thin state-space cluster
  rather than extend the dominant one."
- **The method found.** Stage A compresses six GPT-2-vocabulary donors'
  next-token log-softmaxes under 28 single-token contexts into a
  50257 × 50257 context-averaged bigram table, splits off a unigram anchor,
  and randomized-SVDs the centered table to rank 671 into the embedding and
  output head, zeroing every sublayer — "a factorized bigram model stored
  in a 14-layer network." Stage B re-enables sublayers as sparse
  deterministic edits on 96-dimensional bands. The takeaway the authors
  draw: "Donor behavior, compressed into a low-rank transition operator,
  transfers across architectures where donor parameters do not."

## Critique / open questions

- **This is an n = 1 sustained-use anecdote, not a controlled result, and
  the paper is unusually clear about that.** There is no isolated arm, no
  flat-log arm, and no central-planner arm; Appendix C *proposes* exactly
  that four-arm matched comparison ("Every row uses matched agents, models,
  compute, evaluator, and wall-clock budget") and it was **not run**. The
  closing sentence names what is still open: measurement "tells us when a
  research DAG improves discovery and when it merely files the same
  parallel waste more neatly." Every dynamics claim in §4.6 is descriptive.
- **The diversity rule is described, deployed once, and never evaluated.**
  It was introduced *mid-run, by humans, at the moment the leaderboard
  stalled* — so the post-hoc improvement is confounded with time, with the
  simultaneous deployment of the landscape views, and with the selection
  effect in *when* a human chooses to intervene. Worse for the narrative,
  the paper concedes a reporting artifact: "Explicit negative-result and
  explore-novel tags appear only after the May 2 deployment of the
  landscape and diversity views." The apparent post-intervention rise in
  exploration and negative results is therefore partly a change in the
  *tag vocabulary*, not in behavior. **Nothing here licenses a claim that
  diversity-aware selection prevents monoculture; the run's own five-day
  monoculture happened while Agora was running.**
- **Verification has zero measured detection power.** 165 verifications,
  95 distinct targets, "none reports a failure" — so the true-positive and
  false-negative rates are both unestimated, and nothing was caught.
  The check is also structurally easy: `transfer()` is seeded at 42 and
  "Two runs of the same code on the same hardware are bit-identical;" so a
  "confirmed" verdict mostly certifies that a deterministic function is
  deterministic. It does not test whether the *claim in the description* is
  true, whether the change was the cause, or whether the result
  generalizes. The authors concede the general form in Appendix B: "A
  verdict without these artifacts is a coordination hint rather than strong
  validation evidence." The real audit in this paper was the humans'
  three-level check in §4.8, which the verification tag did not perform.
- **The 62% figure is against a baseline the authors disown in the same
  paragraph.** The GPT-2 124M anchor at ≈1.0 "is not an achievable
  no-training baseline," and the whole recipe was selected on the single
  200-text development evaluator with no held-out set. Read 3.39 → ≈1.90 as
  the claim; the "62%" is a presentation choice.
- **The result is mostly the first thirteen hours.** 98% of the descent
  came in 18 scored contributions, ~70% in eight. Per Table 4 the first
  scored attempt is Apr 27 00:24 (`worker1`) and the "18th scored" is Apr
  27 13:30 (`worker2`) — so the whole 98% landed inside ~13 hours, among
  the five interactive A100 workers, **before the eight Slurm workers
  started on April 28**. The paper describes this stretch as one account's
  two moves followed by "within six hours four accounts had extended the
  idea to bigram statistics under 3, 6, 12, and 24 prefixes (1.93)." The
  13-worker community produced the *last 2%* and a shared diagnosis. That
  reframes the summary reading of "13 workers drove 3.39 → 1.899"
  (my paraphrase, not the paper's): the community's marginal product on this
  task, across "the remaining 1,106" scored results, was 0.03 bpb.
- **No code release found.** I searched the full text: the only GitHub link
  in the paper is Karpathy's `autoresearch` in the references. For a
  systems paper whose contribution *is* a prototype (Go service, Next.js
  UI, SQLite schema, 26 routes, a CLI, a Docker image) and which says "the
  codebase is small enough to audit end to end," there is no repository, no
  artifact DOI, and no released project export. The contribution DAG — the
  one artifact that would let anyone check §4.6's statistics independently
  — is not published either. This is the single largest credibility hit,
  and it is doubly ironic given the paper's own Appendix A reproducibility
  checklist, which demands "the full contribution DAG with canonical
  hashes, parents, tags, structured values, timestamps, verification
  lineage, and artifacts" and "frozen aggregation code that regenerates
  every table, figure, and claim in the report." The paper does not state
  that its own run meets its own checklist.
- **The task is a poor stand-in for research.** A deterministic scalar
  evaluator over a frozen artifact with a forbidden-data rule is closer to
  a kernel-optimisation contest than to research: there are no measurement
  decisions, no experimental design, and reproduction is `git checkout` +
  rerun. The properties that make Agora's verification cheap here — bitwise
  determinism, a single scalar, a free oracle — are exactly the properties
  a real research claim lacks. Generalisation to domains where a
  "verification" requires judgment is untested.
- **Mild independence caveat.** Co-author Shaokun Zhang is an author of
  AutoGen, which the paper positions itself against in related work. Single
  institution (NVIDIA), single run, authors defined the task, wrote the
  evaluator, launched the workers, chose the intervention, and wrote the
  account. The §4.8 discipline mitigates this but does not remove it.
- **Open: is the DAG or the brief doing the work?** The emergent
  prediction-band protocol is attributed to "the brief's reproducibility
  requirement and from the visibility of other accounts' posts." Those are two
  causes, and the run separates neither. A flat-log arm (Appendix C row 2)
  would.

## Trust signals

- **Credibility:** 3. NVIDIA, with a senior author (Jan Kautz) and a real
  prototype; the self-audit discipline in §4.8 is better than most papers
  in this graph — every number recomputed from a raw export rather than
  from agent prose, the winning commit's 83-module import chain read by
  hand, and an explicit statement that they did not rerun the method. The
  causal disclaimer and the proposed-but-unrun Appendix C evaluation are
  unusually honest. Held at 3 and no higher because: **no code, no data, no
  DAG export released** for a paper whose contribution is a system;
  no peer review; n = 1 uncontrolled run on a single synthetic task the
  authors designed; every coordination finding is descriptive; and the
  headline percentage is measured against a baseline the text itself calls
  unachievable.

## Follow-up

- **Relevance:** 5. This is the closest external instance of this
  repository's own founding thesis ("git is the memory layer") that exists
  in the graph, and it changes five concepts with mechanism and numbers
  rather than merely rhyming with them. It scores 5 rather than 4 because
  of `shared-substrate-contagion`: it supplies both the sharpest
  measurement of substrate-driven herding in the graph *and* the only
  candidate countermeasure — while simultaneously being the reason that
  countermeasure must be recorded as untested.
- **Guard against reading this as validation.** The resemblance to this
  repo is architectural, not evidential. Agora is a multi-principal
  institution with an oracle, a score, and adversarial-ish incentives
  (self-citation exclusion exists because manufacturing impact is
  possible); this repo is one operator and a temporal chain of sessions
  with no oracle and no score. What transfers is the *mechanism list*
  (typed contributions, downstream-evidence scoring, replaceable verdicts
  that preserve history, diversity slots, server-minted provenance). What
  does not transfer is any claim that these mechanisms were shown to work.
- **[[concepts/shared-substrate-contagion]].** The concept currently holds
  [[literature/papers/yoon2026arcticswarm]]'s premature-consensus result
  and an "isolate during gathering, share during integration" scheduling
  answer. Agora adds the *unmitigated* case at community scale (a five-day
  monoculture, 63%-within-an-hour parallel rediscovery, one component
  holding 98.9% of nodes) plus a formally specified remedy — and the remedy
  is untested. Record it as a candidate, with the tag-vocabulary confound
  attached.
- **[[concepts/verified-memory-writes]].** Adds a write-side type system
  with three features the concept lacks: fitness-excluded endorsement,
  self-citation exclusion in the evidence score, and **replaceable verdicts
  that leave both commits in history** — which is a direct answer to the
  concept's [[literature/papers/shen2026revoked]] section ("revocation is a
  read-side property, and write-back launders it"): supersede on the read
  path, never rewrite the write path. But the measured outcome is zero
  caught defects, so this is evidence about *shape*, not *efficacy*.
- **[[concepts/citation-anchoring]].** A content-addressed commit hash is
  the strongest anchor form in the graph — [[literature/papers/ng2026agent]]'s
  "hard evidence" criterion (resolvable by the harness without trusting the
  model) is satisfied by construction, and it does not rot, which is this
  concept's standing open question. §4.8 is a worked instance of
  anchor-then-resolve at scale.
- **[[concepts/file-as-bus]].** The remote-bus variant: the substrate is a
  server-mediated content-addressed DAG, not a shared filesystem, and
  participants "never share a filesystem, model, or conversation." Server-
  minted hash and timestamp is the concrete mechanism
  [[literature/papers/ravindran2026portable]]'s trust-boundary section
  wanted.
- **[[concepts/async-worker-pool]].** A pool with **no queue and no
  assignment**: workers self-select a parent from a derived frontier view.
  Sits alongside [[literature/papers/li2026autorecsys]]'s "the worker is a
  state record, not a process" — here the durable unit is the commit, and
  session death is handled by the launcher starting a fresh session on a
  free credential.
- **[[concepts/budget-as-ceiling]].** The five-day 10⁻⁵-bpb-per-step grind
  under a 1.3 × 10⁻³ noise floor is a clean instance of a no-improvement
  ceiling that could not fire because the fitness comparison had no noise
  floor. Argues `max_consecutive_no_improvement` should compare against a
  *measured* run-to-run spread, not `>`.
- **[[concepts/llm-wiki-pattern]].** Corroborating design convergence on
  the three-layer shape (immutable brief + commits / derived index /
  reserved-tag schema) with a *multi-writer* twist the pattern's other
  sources lack. Weak evidence — one self-reported run, no comparison.
- **Candidates for a later digest.** Evans, Bratton & Agüera y Arcas,
  "Agentic AI and the next intelligence explosion" (arXiv:2603.20639),
  cited here as the institutional framing for agent collectives and absent
  from this graph. Karpathy's `autoresearch`
  (github.com/karpathy/autoresearch, March 2026) is the single-agent
  baseline Agora positions against and is also absent.
