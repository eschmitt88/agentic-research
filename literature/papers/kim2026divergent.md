---
kind: paper
title: "Divergent strategies and convergent outcomes in autonomous materials discovery"
authors: ["Jihan Kim"]
institutions: ["KAIST (Department of Chemical and Biomolecular Engineering)"]
year: 2026
venue: "arXiv (cs.AI / cond-mat.mtrl-sci); journal-format manuscript"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.23957"
code_url: "https://github.com/jihankim929/replicate-study"
citations: null
source: "raw/papers/kim2026divergent.pdf"
added: "2026-09-28"
relevance: 5
credibility: 4
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/pass-at-k]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/information-firewall]]"
tags: ["replicated-agents", "common-mode-error", "reproducibility-vs-validity", "randomized-intervention", "pre-registration", "hidden-answer-key", "materials-discovery", "mof", "claude-code", "population-evaluation", "input-validity"]
---

# Divergent strategies and convergent outcomes in autonomous materials discovery

## TL;DR

Sixteen separately initialized sessions of one model–harness configuration
(`claude-opus-5` via Claude Code 2.1.233), isolated and not told the others
existed, each got the same frozen 12,499-entry MOF database, a
methane-storage objective, a pinned GCMC protocol and a one-week budget.
Their strategies diverged at once (reported screening **106 to 5,006**
structures). They still converged on the frontier an independent reference
calculation finds: the reference's nine best point estimates were all
materials some agent reported. Eight agents were randomized to an appendix
of seven enforceable checks. That raised fresh-run reproduction of the
headline value from **1/8 to 8/8** (Fisher p = 0.001) and total verification
acts from 9 to 19 (p = 0.012). It bought no detectable validity, because
**15 of 16 agents** named the same audit-excluded entry as champion: a
database deposition whose charge-balancing anions had been stripped during
curation, creating artificial pore volume. Their values agreed to an SD of
**0.12 cm³ cm⁻³**, below the simulation uncertainty. The one checked agent
that avoided it had never examined the entry. The paper's thesis:
agreement among replicas "establishes stability to resampling of one model
and workflow, not independent confirmation."

## Claims

- **Divergence emerges without assigned personalities.** "the first five
  logged actions forming a unique sequence for every agent." Four post-hoc
  strategy classes: descriptor-first, breadth-first, serial,
  modification-centered.
- **Outcome convergence on the legitimate frontier.** Eleven agents
  reported the same Cu pts framework (ten independent high-accuracy values
  198.9–200.1). On the reference landscape, "the nine highest point
  estimates are all materials that at least one agent reported"; the first
  structure no agent named is rank ten (190.2 vs 190.3).
- **Recovery did not scale with effort.** Spearman 0.10 (p = 0.77) between
  structures screened and reference top-ten recovered. One agent found 3 of
  the top ten from 30 surviving outputs; the agent with the largest
  footprint (12,465 structures) found none.
- **Enforcement redistributed effort, not validity.** "The checks therefore
  changed behavior but did not target the layer at which the decisive error
  arose." Exploratory behaviors favored unchecked agents, 17 vs 10, but
  n.s. (p = 0.14): "suggest, but do not establish, that verification
  requirements displaced some exploration under a fixed budget."
- **Reproducible is not correct.** "One checked agent reproduced the value
  from archived inputs with a fresh random seed … and passed every enforced
  audit. Nevertheless, no agent recognized that it had reproduced a property
  of an incomplete deposition rather than of the reported material."
- **The evidence was present and reinterpreted.** At least ten agents
  recorded warning signs (bare Cu centers, very high void fraction,
  disconnected 2D sheets, extreme surrogate residuals). "Each sign was
  treated as an explanation for exceptional performance rather than a
  reason to question the structure." One checked agent computed the +8
  charge correctly, then "balanced it with negative groups absent from the
  file." "No agent validated a new chemistry audit against independent
  known cases."
- **Replication tightens both the truth and the error.** "A random sample
  of three agents makes the distinction concrete: there was a 98% chance
  that at least one would report the leading retained material, but
  certainty that a majority would name the excluded entry."
- **Transferable result.** Distinguish "diversity of research trajectories,
  stability of conclusions and independence of the scientific substrate,"
  which task-success evaluations conflate. "Effective checks must test
  whether the object under study is valid, not merely whether the
  calculation is reproducible."

## Methods

- **Substrate.** CoRE MOF 2024 computation-ready set, frozen at N = 12,499
  and SHA-256 manifest-pinned; coordinate-identical entries merged, giving
  9,167 distinct structures (name matching both over-merged and missed 80
  coordinate-identical groups). RASPA v2.0.37, TraPPE methane, UFF, hash-pinned
  force-field files, 298 K, deliverable capacity 65 bar minus 5.8 bar.
- **Pre-launch integrity audit, hidden.** A three-pass charge-accounting
  procedure, validated on 70 ZIF-like structures (net charge exactly zero),
  flagged **406** charge-inconsistent entries as a lower bound. Six combined
  the imbalance with anomalous void space and were excluded prospectively
  in the answer key. Agents got neither the list nor the tool. The
  excluded entry's value (207.17 ± 1.24) was computed before launch.
- **Design.** 16 agents, 8 checked / 8 unchecked by seeded assignment,
  sealed by a public-hash commit before launch (a pre-launch budget
  amendment cut the plan from 20 agents / 10 days to 16 / 7). Both groups
  got the same scientific norms, including the charter's charge-balance
  rule and instructions to "prefer a correct negative conclusion over an
  inflated record." Only the checked group got Appendix A (two capacity
  thresholds, structural sanity, protocol validity, modification hygiene,
  finalist reproduction, random auditing). How enforcement was mechanized,
  and whether a harness refused a noncompliant report, is in Supplementary
  Texts S1/S2, not the main text.
- **Isolation and control.** Isolation by charter and audited from
  scheduler records and transcripts, **not** an OS boundary; one shared
  cluster account. The investigator made no scientific decisions. A
  controller handled 58 escalations (54 infrastructure, 4 charter) with
  operational answers only.
- **Budgets.** 1,610 CPU-hours per agent (vs ~22,873 for exhaustive
  screening), 32M tokens, US$280 hard stop. The dollar limit bound for 12
  of 16 agents, because cache reads were 59% of smoke-phase cost.
  Scheduler-only compute accounting under-reported use by more than twofold,
  so compute was metered from completed job records. Active sessions lasted
  33–79 of 168 allowed hours.
- **Scoring.** Conduct measures extracted by mechanical text rules from
  final reports (replacing planned hand scoring via a pre-analysis
  amendment filed before any report was read). Interim reference landscape:
  1,497 random + 847 descriptor-tail audit-retained structures (2,344),
  frozen 6 Sept 2026; the full census continues.
- **Smoke phase (2 agents, excluded by pre-registration).** It exposed
  "more than a dozen instances of one recurring defect class in which a
  success signal referred to the wrong object." Examples: RASPA exiting 0
  after a fatal input error, and a provisioner that would have supplied the
  wrong database while reporting successful verification. This led to a
  standing rule that success is judged from the produced artifact.

## Results

- Reproduction 8/8 vs 1/8 (Wilson 0.68–1.00 vs 0.02–0.47). Verification
  total 19 vs 9 (Mann–Whitney p = 0.012). Without the mandated reproduction
  item the other four measures total 11 vs 8, so almost the whole delta is
  the item Appendix A explicitly required.
- Champion = excluded entry: 7 checked + 8 unchecked = 15/16 (Wilson
  0.72–0.99). Reported values 206.7–207.2. Named a retained MOF as champion:
  1 vs 0, p = 1.0. Identified the deposition as incomplete: 0 vs 0.
  Tested tools against chemistry known from outside the database: 0 vs 0.
- 11/16 reported the leading retained material (0.44–0.86). "Eleven agents'
  best retained claim was the structure with the highest reference point
  estimate." For 15 agent-reported structures at floor fidelity, reference
  minus agent value = −0.2 ± 0.8.
- Modification: 8 agents built 2,253 hypothetical structures (unchecked
  2,207 vs checked 46, dominated by one program). All were unrelaxed and
  simulated as built.
- Sensitivity: dropping the two workspaces that received foreign files gives
  13/14, 10/14 and 7/7 vs 1/7 (p = 0.005); the champion contrast stays at
  p = 1.0. Dropping the four agents that acted on a retracted
  infrastructure notice leaves both primary contrasts unchanged.

## Critique / open questions

- **This is not a contagion measurement.** No channel ran between agents:
  they were not told others existed, and the one cross-workspace leak (two
  recipients) was detected and quarantined with no reported value
  affected. The 15/16 is common-mode error from a **shared read-only
  input**, the correlated-priors / source-independence limb of
  shared-substrate-contagion, not propagation over a shared store.
- **Data defect vs model blind spot are not separated.** The defect sits in
  the database, but the failure to reject it despite recorded warning signs
  is a model/harness behavior, and one model was tested. The paper says so:
  "the same isolation protocol applied to a second model is the natural next
  test." "Shared inputs" is the paper's framing. The data equally support
  "shared inputs × one shared model." A heterogeneous-model population is
  the arm that would split them.
- **The validity null is non-identification, not a negative.** "with
  fifteen of sixteen agents naming the same excluded entry the design had
  essentially no power to detect one." The pre-registered second wave
  (triggered at 0.05 < p < 0.25) did not fire at p = 1.0. The paper
  shows these checks did not target the error layer. It does not show
  checks cannot improve validity.
- **n = 8 per arm, single model alias, no snapshot pin.** "identical weights
  across the campaign cannot be guaranteed and the study cannot be rerun on
  a pinned snapshot." Strategy classes were assigned post hoc by the author
  alone. The reference landscape is interim (2,344 measured; the random
  sample covers ~16% outside the tail), so "the frontier" means the measured
  frontier.
- **Enforcement mechanism under-described in the main text.** "Enforced"
  checks are an appendix of requirements plus audit ledgers. The main text
  does not say whether a controller refused noncompliant reports, and that
  matters for whether this counts as a gate whose refusal changes behavior.
- **Apparatus disturbances** (a false fleet-wide notice acted on by four
  agents, a process reaper, a shared-directory failure). All are disclosed
  with sensitivity analyses, which is to the paper's credit, but they show
  isolation-by-charter leaks even in a carefully run study.

## Trust signals

- **Credibility:** 4. Single-author, not peer reviewed, but the author is an
  established computational-MOF PI at KAIST (ChatMOF, SimMOF lineage), and
  the artifact discipline is unusually strong. A public pre-registration
  hash since launch, a sealed group assignment, a hidden answer key computed
  before launch, pre-analysis amendments filed before reports were read,
  mechanical extraction rules, Wilson CIs and sensitivity analyses. Released:
  controller, audit code, answer key, all 16 agent repos with full git
  histories and verbatim transcripts (GitHub repo resolves; Zenodo DOI
  archived). Held below 5 by: no peer review, one author with no second
  coder for strategy classes, n = 8 per arm, one unpinned model, and an
  interim reference census.

## Follow-up

- **Relevance:** 5. The first randomized, pre-registered intervention on a
  population of research agents scored against a hidden answer key, run on
  this repo's own harness family (Claude Code). It anchors two load-bearing
  claims with measurements rather than arguments: replicas sharing one input
  converge on its defect (shared-substrate-contagion's independence point),
  and a gate that verifies derivation reproducibility leaves object validity
  untouched (evidence-gated-completion, pass-at-k).
- **[[concepts/shared-substrate-contagion]].** Adds the read-only-input case
  with *no channel*. It sits next to shao2026language's peer-hidden arm and
  kim2026are's no-channel limit, and does not add to the propagation
  sightings.
- **[[concepts/evidence-gated-completion]].** A new table row: failure mode
  = inherited input defect; hidden by = a reproducible value; minimum
  evidence = a validity check on the input object, itself validated on
  independent known cases. It narrows the concept's hold: the first
  randomized enforcement arm with a measured behavioral delta, where the
  delta landed only on the required evidence.
- **[[concepts/pass-at-k]].** k replicas bound stochastic, not systematic,
  error, and majority selection over k actively picks the common-mode error.
- **[[concepts/hce-evaluation]].** An audited pre-existing defect, sealed
  before launch, works as a planted failure without planting.
- **Repo read.** `/ingest` agents here all read the same `raw/` PDF with the
  same model. Concurrent ingests agreeing is exactly this paper's
  "stability to resampling," not independent confirmation.
- **Candidates.** Liao et al. 2026 PseudoBench (arXiv 2606.18060; "a
  polished autonomous workflow can preserve a misleading scientific
  premise") and Huang et al. 2026 AUTOMAT (arXiv 2605.00803; coding agents
  reproducing computational-materials findings) are cited and not in this
  graph. PseudoBench is the closer fit to this cluster.
