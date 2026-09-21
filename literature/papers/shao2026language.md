---
kind: paper
title: "Language-model groups overstate consensus when replaying human deliberation on a reasoning task"
authors: ["Tengfei Shao"]
institutions: ["Waseda University (Global Education Center)"]
year: 2026
venue: "arXiv (cs.AI/cs.CL/cs.CY/cs.MA)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.20543"
code_url: "https://doi.org/10.5281/zenodo.21318346"
citations: null
source: "raw/papers/shao2026language.pdf"
added: "2026-09-21"
relevance: 4
credibility: 4
status: read
related_experiments: []
related_concepts:
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/information-firewall]]"
tags: ["multi-agent", "consensus", "over-convergence", "herding", "construct-validity", "measurement-definition", "pre-registration", "held-out", "counterfactual-control", "synthetic-participants", "negative-result"]
---

# Language-model groups overstate consensus when replaying human deliberation on a reasoning task

## TL;DR

100 held-out human Wason-task groups were replayed by LLM agent groups, one
belief-anchored agent seeded from each real participant's pre-discussion
answer, and scored with the same code. Agent groups reached full consensus
**+34.1 points** (chat) and **+44.4 points** (reasoner) above the matched
humans on the participation-matched lurker-free subset, and the gap survived
disabling early stopping, removing the memorizable answer, and a
cross-family model swap. The two halves of the result come apart under the
paper's own no-peer control: hiding peer messages collapses chat consensus
from **87.0% to 31.0%** (channel-driven agreement) but moves reasoner
consensus only from **100.0% to 91.0%** (agreement that needs no channel at
all). And the agreement is not competence: on the isomorphic
reparameterization the reasoner still agrees near-totally, with **74.0% of
all groups agreeing on a wrong answer**. Separately, the paper is a
measurement lesson — the *human* consensus rate on the same 100 groups is
anywhere from **24.0% to 57.0%** depending purely on how participation and
final states are operationalized.

## Claims

- **Belief-anchored LLM groups are biased estimators of the human
  group-outcome distribution**, in the direction of excess unanimity. The
  bias is "not zero-mean noise removable by averaging."
- **Consensus does not track collective accuracy.** Full consensus is
  defined on identical final answers only and "never references the correct
  answer," so a near-total agreement rate says nothing about whether the
  agreed answer is right — demonstrated by the isomorphic task, where
  agreement stays near-total and moves onto wrong card sets.
- **Identical scoring code is not identical measurement.** DeliData's human
  final state is a carry-forward tracker over a possibly-disengaged
  participant; every agent is force-elicited. Running the same scorer over
  both populations does not make the comparison like-for-like.
- **How you operationalize participation changes the human number by more
  than a factor of two** (24.0% → 57.0%), so the operationalization "is part
  of the validity argument rather than a reporting detail."
- **Over-convergence is not an artifact of the stopping rule or of
  recognizing the canonical Wason answer** — both were tested directly.
- **Role-fidelity instruction bounds the effect; per-turn belief re-injection
  does not.** The registered monotonic ordering none > fidelity >
  fidelity_memory is classified unsupported.
- **The direction of the distortion may be task-regime-dependent.** On a
  subjective task with no correct answer, the same machinery
  *under*-converges (exploratory, N = 20).

## Methods

- **Corpus.** DeliData (Karadzhov, Stafford & Vlachos, 2023): "a public
  corpus of 500 group discussions of the Wason card selection task,
  comprising 1,974 participants and 14,003 utterances." Each participant has
  a pre-discussion answer, free-text group chat, and a post-discussion
  answer; the corpus stores a per-message solution tracker.
- **Frozen split.** A fixed hash of the group id splits 400 calibration / 100
  held-out. "The study was preregistered on the Open Science Framework on 4
  July 2026 (https://osf.io/5jp7s), before the held-out cells were
  generated." "Both frozen identifier lists were committed by SHA256 in that
  preregistration so that neither could be altered after results were seen."
  Calibration tuned **exactly one** target (the group improvement rate) over
  scaffold arm and temperature; the human calibration rate was 35.8%.
- **Belief anchoring, and why.** A contamination probe showed current models
  solve Wason far above the human individual baseline even under isomorphic
  reparameterization, so free-solving agents would import competence their
  participant did not have. Each agent is therefore seeded with one real
  participant's pre-discussion selection, its own alias, the four visible
  cards and the rule — and is never given the answer key, the human
  discussion, the post-discussion answers, or peers' private pre-answers.
- **Matrix.** 100 groups × 3 scaffold arms (none / fidelity / fidelity +
  per-turn belief re-injection) × 3 seeds × 2 inference modes =
  1,800 confirmatory cells. Both API aliases (`deepseek-chat`,
  `deepseek-reasoner`) returned the same served identifier
  (`deepseek-v4-flash`) and system fingerprint, so they are treated as two
  inference settings of one build, not two models. Qwen3-14B served locally
  is a cross-family check. "Seven deepseek-reasoner cells (0.4%) failed and
  were excluded rather than backfilled."
- **Turn protocol.** Each turn an agent may speak or **pass**, so
  non-participation is permitted rather than forced. Termination on settled
  consensus over two rounds, an all-pass round, or a per-group message cap
  (25, the human median). Turn order rotated by an order seed.
- **Four human consensus definitions** (Table 2, same 100 groups):
  carry-forward 24.0% (n=100), submit-based 52.0% (n=98), active-only 57.0%
  (n=100), lurker-free 51.1% (n=45). The lurker-free row is the
  participation-matched basis for the headline paired comparison and is
  explicitly "a post-unblinding sensitivity estimate, not a preregistered
  confirmatory quantity."
- **Three robustness replays + one diagnostic control**, all on the same 100
  groups, primary arm: isomorphic reparameterization (cards remapped to
  maple/birch/lantern/candle with logical structure fixed), fixed horizon
  (early stops disabled), and **peer-hidden** (every message from other
  agents withheld, same fixed horizon) — the last framed as a diagnostic
  control, not a robustness check.
- **Statistics.** Unit of analysis is the group (seeds averaged within
  group); group-clustered bootstrap with 10,000 resamples at a fixed seed;
  **exact** sign-flip permutation tests enumerated by dynamic programming;
  Holm correction across {H1, H2a, H2b} within each mode. A preregistered
  compatibility rule declares a signature reproduced only when the human
  value lies inside the simulated 95% bootstrap interval.
- **Five preregistration deviations are disclosed in full**, including the
  post-unblinding adoption of the lurker-free restriction as the headline.

## Results

### The consensus gap, by definition

- **Lurker-free (n = 45), the headline.** "Chat agents reached 85.2% full
  consensus against the human 51.1%, a +34.1 percentage-point difference
  (95% CI [19.3, 48.9]); reasoner agents reached 95.6%, a +44.4-point
  difference ([29.6, 59.3])."
- **Submit-based sensitivity (n = 98).** "+34.0 percentage points (95% CI
  [24.1, 44.2]) for chat and +43.9 points ([34.0, 53.7]) for reasoner,
  preserving over-convergence without the carry-forward tracker." Two routes
  that reduce *different* asymmetries land within 0.5 points of each other.
- **All-groups, carry-forward (the preregistered confirmatory H2a).** "+62.0
  points [53.0, 70.7] for chat and +72.0 points [63.3, 80.3] for reasoner,
  both p_holm < 0.001" — but this figure "mix[es] over-convergence with
  participation and carry-forward differences" and should not be quoted as
  the effect. Cohen's h 1.35 / 1.71, descriptive only.
- **Improvement (S1), by contrast, is over-stated only modestly.** Chat
  improved in 60.7% of groups, reasoner 85.3%, against a same-code human
  41.0%. This one is metric-fragile: human incidence "rises from 41.0% with
  exact-set scoring to 77.0% with graded Jaccard," so the authors decline to
  claim agents deliberate better.
- **Deterioration (S2) is a null.** +0.3 pts [−4.7, +4.7] chat, −3.0 pts
  [−7.3, +0.7] reasoner; reported as "no credible evidence," with TOST
  equivalence at a ±5-point margin and an explicit statement that ±3 is
  underpowered at n = 100.

### The gap is not the stopping rule, not memorization, not one family

- **Fixed horizon.** "Full consensus was 87.0% for chat and 100.0% for
  reasoner (n = 100 groups each), essentially unchanged from 86.0% and
  96.0%."
- **Isomorphic reparameterization.** "Full consensus persisted at 80.0% for
  chat and 98.7% for reasoner." For the reasoner, "correct consensus fell
  from 84.0% of groups on the classic task to 24.7% while wrong consensus
  rose from 12.0% to 74.0%, a paired within-group increase in
  wrong-consensus incidence of +62.0 percentage points (95% group-bootstrap
  CI [+53.3, +70.7], exact sign-flip permutation p < 0.001, n = 100 paired
  groups)."
- **Cross-family.** "Qwen full consensus was 65.7% against the human 24.0%
  carry-forward rate, a +41.7-point paired excess (p < 0.001, nominal)" —
  comparable to the all-groups deepseek numbers, not the participation-
  matched ones. Qwen "did not inflate improvement incidence (37.7% versus
  human 41.0%)," so the *improvement* half is model-dependent and the
  *convergence* half is not.

### The decomposition that matters: peer-hidden control

- **Chat consensus is produced by the channel.** "For deepseek-chat, full
  consensus was 31.0% without peer messages (26.3% correct, 4.7% wrong),
  below the human lurker-free rate of 51.1%, against 87.0% with peers, where
  wrong consensus rose from 4.7% to 32.0%." Exchange adds +56.0 points of
  agreement and multiplies wrong agreement roughly sevenfold.
- **Reasoner consensus is not.** "For the reasoning mode it was already
  91.0% without peer messages, all correct, against 100.0% with peers (88.3%
  correct, 11.7% wrong)." Agents re-solving *in isolation* already agree
  almost totally.
- The difference-in-differences is "+47.0 percentage points (95%
  group-bootstrap CI [+34.7, +58.7], exact sign-flip permutation p < 0.001,
  n = 100 paired groups)."
- The authors bound this themselves: the control ran only on the classic
  task, and high peer-hidden reasoning agreement "may reflect task-specific
  competence," so it "does not establish a general interaction mechanism."

### Process signatures (correlational)

- **Participation.** "Agent lurking was 0.2% compared with 20.1% among
  humans." Near-full agent participation is itself a reproduced failure and
  a candidate contributor to the higher consensus.
- **Anchor retention leaks.** "Only 17.1% of reasoner agents held their
  assigned belief for the whole discussion, against 34.6% for chat," and
  "reasoner agents' individual accuracy rises from 11% to 85% through
  discussion, so some task competence leaks past the injected persona."
- **Speed.** "In the primary arm, chat and reasoner groups took 4.4 and 2.3
  rounds, respectively, to reach a shared belief." Reasoner groups held a
  shared live belief at some round in 95.3% of groups vs 62.0% for chat; by
  round two, 76.0% vs 26.7%.
- **Belief-change targets.** "85.6% of reasoner changes moved toward the
  logically correct card set and 37.5% toward the current majority; for chat
  the split was 54.8% and 40.5%." The categories overlap when the majority
  already holds the correct set, so this is not a demonstrated cascade.
- **Structural collapse.** Final-state Herfindahl index: "chat 0.94 versus
  0.62, paired difference +0.32, 95% group-bootstrap CI [+0.27, +0.37]";
  reasoner 0.98 versus 0.61, +0.37 [+0.32, +0.41]. Simulated groups collapse
  onto a single final position where humans retain residual disagreement.
- **Correct vs wrong split, of all groups.** Humans: 11.0% correct, 13.0%
  wrong. Chat: 54.0% correct, 32.0% wrong. Reasoner (classic): 84.0%
  correct, 12.0% wrong. "A reader treating either simulation as a human
  proxy would draw opposite conclusions about how often groups agree on
  something false."
- **Scaffolding.** "Adding a role-fidelity instruction lowers it (chat 98.7%
  to 81.7%, reasoner 100.0% to 96.0%)," whereas adding per-turn belief
  re-injection "beats plain fidelity in only 32 of 100 chat groups and 17 of
  100 reasoner groups." Page trend significant in both modes but the
  predicted strict order fails.

### Study 2 (exploratory, N = 20, five groups)

Same machinery on a Japanese classroom deliberation about AI-assisted
hiring. With no correct answer to collapse onto, the replay
**under**-converges: human experience-assortativity decay was +1.993
(D(T1) = 2.592 → D(T3) = 0.599, permutation p = 0.0013) while the none and
fidelity arms closed only +0.682 and +0.689 (34% and 35% of the human
value), neither distinguishable from a label-permutation null. Per-turn
belief re-injection, which *worsened* things on the objective task,
performed best here (+1.277, 64%, p = 0.064). Single-seeded and unstable;
the author reports it as a descriptive contrast only.

## Critique / open questions

- **The headline quantity was chosen after unblinding, on a subset.** The
  preregistered confirmatory test is the all-groups paired comparison
  (+62.0 / +72.0); the +34.1 / +44.4 lurker-free values were adopted during
  revision, on 45 groups that are smaller on average. The author discloses
  this prominently and offers two bounds: the independent submit-based route
  (n = 98) lands within 0.5 points, and the registered all-groups gap is
  itself large. That is about as honest as this deviation can be handled,
  but the headline is still a post-hoc estimand.
- **Participation matching does not equalize the endpoint.** Even in
  lurker-free groups, human states are carried forward or submitted while
  agents are force-elicited. A residual measurement asymmetry remains, and
  it points in the direction of the effect.
- **The two "modes" are not two models.** Same served identifier, same
  fingerprint, hosted. So the chat-vs-reasoner contrast is a decoding and
  serving contrast whose weights the author explicitly cannot verify, and
  the hosted transcripts are deposited rather than regenerable. The
  genuinely independent model point is a single Qwen3-14B arm on the same
  100 groups — not an independent-sample replication, and its p-value is
  nominal.
- **The belief anchor is the design's load-bearing assumption and it
  visibly leaks.** Reasoner individual accuracy goes 11% → 85% during
  discussion and only 17.1% of reasoner agents keep their seeded belief.
  That makes "the group converged" hard to separate from "the model solved
  it and the persona gave way." The peer-hidden control is what rescues the
  chat half of the result from this objection: with no channel, chat sits at
  31.0%, *below* the human 51.1%, so the anchor holds well enough that the
  over-convergence cannot be attributed to anchor failure alone. No such
  rescue exists for the reasoner half.
- **One task, and a famously atypical one.** The Wason selection task has a
  demonstrable correct answer, is heavily represented in training data, and
  is known to be framing-sensitive. The isomorphic control shows surface
  reparameterization does not remove the competence — which is a useful
  negative about reparameterization as a contamination defense, but also
  means the contamination was never actually removed. Whether the distortion
  transfers is open; Study 2 suggests the *sign* may even flip.
- **Study 2 is doing a lot of rhetorical work for its size.** N = 20, five
  groups, one seed, one arm ordering that the author says is unstable. It
  motivates task-regime dependence; it establishes nothing.
- **Single-author preprint with an unusual authorship note**: "The
  peer-reviewed journal version of this work is co-authored with" John
  Maurice Gayed, Hidehiro Kanemitsu and Masayuki Goto, who are credited in
  acknowledgements here. Treat this file as a pre-review artifact.
- **Open question the paper names and does not answer.** "What endpoint
  signature is sufficient to certify an agent group as a proxy for human
  collective cognition: full consensus alone is not, and whether jointly
  matching minority survival, participation, and correct-versus-wrong
  consensus out of sample suffices is an open empirical question."

## Trust signals

- **Credibility:** 4. A single-author Waseda preprint with no track record
  in this graph and no peer review, raised well above the usual preprint
  floor by an unusually disciplined protocol: an OSF preregistration dated
  before cell generation with both split lists SHA256-committed, a
  hash-based 400/100 split where calibration was allowed to tune exactly one
  metric, exact (enumerated) permutation tests, group-clustered bootstraps
  with a fixed seed, a Zenodo deposit of transcripts, configs, prompts and
  MIT-licensed code, a config hash verified by an integrity audit, and five
  preregistration deviations disclosed in full. It reports its own nulls
  (H2b, H3 ordering), its own metric fragility (41.0% → 77.0% under
  Jaccard), and the leak in its own anchor (11% → 85%). Held at 4 rather
  than 5 by: one task, one hosted build whose two "modes" may be one
  configuration, a headline estimand adopted after unblinding on n = 45, a
  cross-family check that reuses the same groups, and a tiny Study 2.

## Follow-up

- **Relevance:** 4. Not a research-agent paper — the domain is
  synthetic-participant psychology, and the transfer to ML-research agents
  is by analogy. It scores 4 anyway because it supplies the one thing
  [[concepts/shared-substrate-contagion]] has never had on its herding axis:
  a **matched independent baseline** (the same 100 human groups) plus a
  **no-channel counterfactual** that separates agreement produced by the
  shared channel from agreement produced by shared weights. Not 5 because
  the setting is one artificial reasoning task with seeded personas.
- **[[concepts/shared-substrate-contagion]].** The concept's third effect —
  "it carries premature agreement (herding)" — rested on
  [[literature/papers/yoon2026arcticswarm]]'s 3.8-point gain from gating peer
  visibility, i.e. a benefit measured on the outcome, with no measurement of
  the convergence itself and no human baseline. This paper measures the
  convergence directly against matched humans and then **splits it in two**
  with the peer-hidden control. That split is the important contribution and
  it cuts both ways: chat's agreement is +56.0 points of channel effect
  (contagion proper), while the reasoner agrees 91.0% with the channel
  entirely removed (correlated priors, not propagation). It also supplies
  [[literature/papers/yang2026sok]]'s demanded counterfactual in a form this
  concept can reuse: **run the no-channel arm before attributing convergence
  to the substrate.**
- **[[concepts/hce-evaluation]].** Adds a third construct-validity failure
  beside [[literature/papers/zhang2026double]]'s scaffold ownership and
  scorer criterion validity: **the same scorer applied to two populations
  can measure two different things**, because the state-generating processes
  differ (carried-forward vs force-elicited). The 24.0/52.0/57.0/51.1 spread
  on one fixed set of 100 groups is the cleanest demonstration in the graph
  that a headline rate can be moved by a factor of two without touching the
  data. Also adds a reusable compatibility rule (the comparison value must
  lie inside the simulated 95% bootstrap CI) and a calibration discipline
  worth copying: tune exactly one signature on a disjoint split and freeze
  everything else.
- **[[concepts/information-firewall]].** The contamination probe and the
  isomorphic replay are a measured negative on **surface reparameterization
  as a firewall**: remapping the cards to neutral tokens removed the
  memorizable answer but not the competence, and agreement stayed at 80.0%
  (chat) and 98.7% (reasoner). The paper is explicit that this "establishes
  robustness to surface reparameterization, not memorization in isolation."
  Reparameterization belongs with wang2026evobrowsecomp's recency boundary
  as a *decaying* defense, not a clean one.
- **Independence.** Cites Ashery et al. (2025, Science Advances) on emergent
  collective bias in LLM populations, Zhang et al. (2024) on conformity and
  groupthink, Liang et al. (2024) on premature convergence in debate, Smit
  et al. (2024) on agreement not guaranteeing a better answer, and
  Taubenfeld et al. (2024) on drift toward built-in positions over assigned
  personas. None is in this graph; Ashery (a peer-reviewed *Science
  Advances* paper on group-level bias not reducible to any individual agent)
  is the strongest digest candidate of the five. DEBATE (Chuang et al.,
  2025) is the closest neighbour — digital twins replayed on subjective
  questions, where the correct/wrong consensus split cannot be defined.
