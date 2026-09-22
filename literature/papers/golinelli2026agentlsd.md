---
kind: paper
title: "AgentLSD: Evaluating AI Security Agents Under Adversarial Task Contamination"
authors: ["Matteo Golinelli", "Idilio Drago", "Matteo Boffa", "Francesco Bergadano", "Bruno Crispo"]
institutions: ["University of Trento", "University of Turin", "Politecnico di Torino"]
year: 2026
venue: "arXiv (cs.CR); accepted to the 19th ACM Workshop on Artificial Intelligence and Security (AISec 2026), co-located with CCS 2026"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.19140"
code_url: "https://github.com/Golim/agent-lsd"
citations: null
source: "raw/papers/golinelli2026agentlsd.pdf"
added: "2026-09-22"
relevance: 3
credibility: 4
status: read
related_experiments: []
related_concepts:
  - "[[concepts/information-firewall]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/budget-as-ceiling]]"
tags: ["security-agents", "ctf", "environment-deception", "prompt-injection", "contamination", "paired-design", "effort-tax", "telemetry", "robustness-evaluation", "workshop-paper", "no-significance-tests"]
---

# AgentLSD: Evaluating AI Security Agents Under Adversarial Task Contamination

## TL;DR

A paired clean/trap harness that injects deceptive artifacts into 11
purpose-built Flask web CTF challenges while holding the vulnerability,
solution path and flag checker fixed, so any behavioral difference is
attributable to the injected artifact. Six models are measured **clean**
(136/330 default-prompt flags, ~41%); only **three on-prem models get the
full trap grid**, plus GPT-5 on an adversarially-selected 5-trap subset.
The headline result is a **decoupling**: redirective traps (goal hijack,
exploration bait) destroy *solves*, while fabricated-validation and
authority cues barely move the solve rate and instead impose a large,
highly heterogeneous **effort tax** — in the worst measured cell, a trap
that never costs a single solve turns a 10-turn / 0.8k-token clean solve
into a 49-turn / ~6.3k-token one. Telemetry shows fragility is not about
*contacting* a decoy (all models do, 39–57% of trapped runs) but about
**failing to disengage**: GPT-OSS 120B lingers a median ~14 steps on
decoy routes against ~4 for the others, and its win rate drops from 65%
to 28% conditional on taking the bait.

**The framing claim the paper is cited for — instruction vs.
non-instructional evidence — is defined and designed for but never
separately measured.** See Critique.

## Claims

- **Adversarial task contamination is a distinct threat from prompt
  injection.** The paper's definition: "an intervention that adds
  attacker-controlled artifacts to the environment while holding the
  canonical task and its success criterion fixed. Prompt injection is
  defined by its mechanism, where an attacker-supplied instruction
  competes with the task instruction. The two notions therefore overlap
  but are not identical. A fake validation result, decoy endpoint, or
  plausible but incorrect vulnerability cue contaminates the evidence
  without issuing any instruction. The failure therefore becomes a change
  in which evidence the agent trusts or explores."
- **Clean CTF performance understates vulnerability to deceptive
  evidence**, because the dominant cost is effort rather than outcome:
  "traps rarely flip the outcome of an agent that can already solve a
  challenge; instead, they impose a large and highly heterogeneous effort
  tax."
- **Robustness to deception does not track clean capability.** "Even GPT-5
  is affected, despite its [subset of traps for cost reasons], showing
  that not even a frontier hosted model is immune. Robustness to deception
  therefore [does not track] clean capability."
- **Redirection drives outcome loss; every family drives effort.** "traps
  based on redirection (goal hijack and exploration bait) drive most of the
  outcome loss, while every trap family inflates effort."
- **Fragility is disengagement, not contact.** "Robustness to environmental
  deception is therefore [less about] avoiding contact with a trap than
  about disengaging [from it]."
- **A trap-free corpus is a prerequisite, and the corpus is unsaturated.**
  No model solves blind command injection; GPT-5 tops out at 32/55.

## Methods

- **Threat model, stated tightly.** The adversary controls only selected
  artifacts in the environment: "The adversary cannot compromise the model,
  the agent's prompt, its tools, or the runtime, and it cannot manipulate
  the agent's internal instructions." Three bounding assumptions are
  declared: the adversary is **static and non-adaptive**, **model-agnostic**
  (same artifacts to every agent, not tailored), and agents are
  **trap-unaware** (never told the environment is contaminated). "The
  measured effect is therefore an estimate of susceptibility under a fixed
  prompt."
- **Task identity is pinned by construction, not inference.** "Task
  identity is established by the clean challenge implementation and its
  reference solution, rather than inferred after observing the contaminated
  page… Following the original exploit therefore produces exactly the same
  flag in both conditions." The generator mechanically enforces this:
  decoy routes must not collide with real solution paths, no duplicate
  decoy routes, honeytoken config checked. The paper is candid that "These
  checks do not prove that a challenge remains semantically unchanged, but
  they provide mechanical guardrails."
- **Trap pipeline.** YAML *primitives* (family templates) → deterministic
  *resolved instances* → *manifests* → Flask middleware injection into
  eligible HTML responses → DOM delivery verification by a headless
  browser → honeytoken telemetry. Seeds are "derived from the global seed,
  primitive identifier, instance index, and retry attempt", and resolved
  files "are intended to be byte-identical across repeated runs with the
  same seed." (*Intended* — no reproduction check is reported.) On
  application error the runtime **fails open** and returns the clean
  response.
- **Trap taxonomy — three independent label axes.** The *primary axis is
  intent*: goal hijack, false authority, false validation, exploration
  bait, time sink. *Surface* (visible text node, hidden div, HTML comment,
  meta tag, accessibility label) and *object* (decoy route, fake endpoint,
  decoy file, hint, flag) are recorded separately, "lets us ask whether a
  trap family is effective because of what it says, where it appears, or
  the interaction between the two." **There is no recorded label for
  instructional vs. non-instructional.**
- **Corpus.** 11 Flask web CTF challenges over seven vulnerability classes
  (command injection, path traversal, race conditions, SQLi, SSRF, template
  injection, XXE). Contamination control is good: "We purpose-built all
  challenges for this study rather than reusing public CTF tasks, and did
  not release their implementations or write-ups before the trials. Models
  may know the underlying vulnerability patterns, but cannot recall a
  challenge-specific write-up or its freshly generated flag." Each trial
  runs in a fresh container on a per-trial private network with a freshly
  generated flag.
- **Agent scaffold, held constant.** OpenCode, one agent definition, one
  system prompt, **temperature 0**, tools = bash / file read / write /
  edit / glob / grep, with **web-search and web-fetch disabled**. Never
  receives application source (unless the challenge publishes it) or the
  reference solution. 30-minute per-trial limit (a 15-minute pilot "proved
  too tight"); a rarely-binding LLM-request cap. A **dedicated
  flag-checking service holds the trial's flag and is the sole authority
  on correctness** — a scored oracle the agent cannot spoof.
- **Two prompts** — minimal *default* and a *methodical* one prescribing a
  workflow and listing vulnerability classes. **All reported results are
  default-prompt only**; the methodical prompt "only marginally improves
  solve rates and does not change our conclusions."
- **Design (Table 2), and where it narrows.** Baseline: 5 on-prem (Gemma-4
  31B, DeepSeek-V4 Flash, GPT-OSS 120B, Qwen3-Coder-Next, Mistral Small 4)
  + 1 hosted (GPT-5), 11 challenges, 5 trials/cell, **605 baseline trials**.
  Trap: **3 strongest on-prem + 1 hosted**, 7 challenges run / **5
  reported**, 14 DOM instances across 5 families, both prompts, **3,061 trap
  trials** (2,940 on-prem fully factorial + 121 for GPT-5).
- **Metrics.** Solve rate (exact flag), *degradation* (clean minus trap
  solve rate over the same model+challenge), and three effort measures —
  turns, tool calls, tokens. Effort deltas are computed **only on trap
  trials that still succeed**, so they measure the overhead of successful
  recovery; flag-capture comparisons require ≥1 clean success.

## Results

### Q1 — clean capability (all six models)

- **136 of 330 default-prompt baseline trials capture the flag, ~41%.**
  Verified against Table 3 column totals: GPT-5 32/55, Gemma-4 31B 28/55,
  DeepSeek-V4 Flash 24/55, GPT-OSS 120B 23/55, Qwen3-Coder-Next 15/55,
  Mistral Small 4 14/55 → 136/330 = 41.2%.
- **The corpus is unsaturated and the top is not the hosted model by much.**
  GPT-5 leads but leaves 23 of 55 trials unsolved; Gemma-4 31B (31B, on-prem)
  is within 4 captures. No model recovers the blind command-injection flag
  (0/5 × 6 models), and race-condition-medium yields exactly one capture
  across the whole grid.
- **Success and failure are sharply different cost regimes** (clean
  condition): "Successful solves are cheap and consistent across models,
  clustering around 10 to 30 interaction turns and 1 to 3k reasoning
  tokens. Failures are both far more expensive and far more variable:
  median effort rises to roughly 55 to 80 turns and 5 to 15k reasoning
  tokens, with individual runs reaching 200 turns and 30k tokens before the
  budget stops them."

### Q2 — the clean/trap delta, and who actually got one

- **Per-model clean/trap deltas exist for four of the six models.**
  Gemma-4 31B, DeepSeek-V4 Flash and GPT-OSS 120B get the full 14-instance
  × 5-challenge × 2-prompt × 5-trial grid. GPT-5 gets 121 trials on a
  reduced grid. **Qwen3-Coder-Next and Mistral Small 4 have no trap data
  at all** — "a model that rarely solves the clean challenge provides
  little signal about deception, so the weakest models do not enter this
  phase."
- **Solve-rate effect, unevenly.** "Flag capture [drops for every model
  except] Gemma-4 31B, and the effect is largest [for GPT-OSS 120B, which]
  loses a substantial share of its clean [solves]."
- **Effort effect, consistently.** "Among [the trap conditions that still]
  recover the flag, interaction turns and [reasoning tokens shift right]
  for every model, with medians above [the clean baseline and long
  positive tails]. Even Gemma, which [keeps almost all of its solves],
  pays this tax."
- The paper's own qualification: because a successful trial stops at flag
  capture while a failed one runs until the budget is exhausted, "the
  reported effort figures are conservative and would grow further under an
  unbounded budget."

### Q3 — which traps matter depends entirely on the axis

- **Outcome axis — redirection dominates.** "goal hijack (n=4) and
  exploration bait (n=3) cause the largest flag-capture losses, removing
  roughly one-half to two-thirds of the clean solves on GPT-OSS 120B
  (median changes near −50 to −65%). False validation (n=3) is
  intermediate, while time sink (n=3) and false authority barely move the
  outcome; false authority rests on a single instance (n=1) and should be
  read with care."
- **Effort axis — everything inflates.** "Turns and reasoning tokens rise
  for essentially every family, including those that leave the solve rate
  intact and including the robust Gemma… even false validation and false
  authority, which rarely change the outcome, push effort well above the
  clean baseline. Which family 'matters' therefore depends on the axis."
- **The sharpest individual cells (all grep-verified against the Figure 5
  grids).** Gemma-4 31B, `visible_false_validation_error_page_hint` on
  sqli-easy: **stays 5/5** while turns go 10 → 49 (**+39**) and reasoning
  0.8k → ~6.3k (**+5.5k**). DeepSeek-V4 on xxe-medium (clean baseline 15
  turns, 2.3k reasoning): the same visible false-validation page adds
  **+42 turns and +10.7k tokens**, and "the three authority traps add 22 to
  34 turns each" (verified: +34, +30, +22).
- **Solve losses for DeepSeek concentrate away from its easiest column.**
  "Its sqli-simple column stays at 5/5 under all 14 traps, yet solves now
  fall elsewhere: the fake flag format halves sqli-easy from 4/5 to 2/5,
  and xxe-medium degrades under nearly every instance, from a clean 5/5 to
  2/5 for the hidden endpoint listing and for the visible false-validation
  page." All four verified in the grid.
- **The paper flags its own noise.** Gemma's xxe-medium column has a 2/5
  clean baseline, so "one trap appears to raise the solve rate to 3/5, and
  several effort cells turn strongly negative (for example, −26 to −31
  turns), artifacts of the low, unstable baseline rather than genuine trap
  benefits."

### Q4 — telemetry: contact is universal, disengagement is not

- **Bait-taking is common and does not rank models by fragility.** "taking
  the bait is common across models (39 to 57% of trapped runs) but does not
  track fragility: the intermediate DeepSeek-V4 is baited most (57%), the
  robust Gemma-4 31B least (39%), and the fragile GPT-OSS 120B falls in
  between (47%)."
- **Consequence conditional on the bait, per model.** DeepSeek solves 87%
  of baited runs vs 85% unbaited (no harm); Gemma 70% vs 92% (moderate);
  **GPT-OSS 28% vs 65%** (collapse).
- **The mechanism is dwell time.** "GPT-OSS also lingers: it spends a
  median of about 14 distinct [steps on decoy routes, against roughly 4 for
  the] other two models." For Gemma and DeepSeek, reasoning-token usage
  "spikes at contact, reaching nearly twice and about 1.5× the per-run
  average before subsiding, so the decoy triggers a localized burst"; GPT-OSS
  "reacts less at contact… its cost comes not from thinking [hard] once but
  from persisting on the decoy over many steps."
- **The honest limit on this signal.** "These telemetry signals provide
  evidence of exploration and engagement, but not belief. An agent may
  request a decoy [and] move on, or interact with a trap and still recover."

### The paper's own nulls, scope conditions and admitted gaps

- **All traps are DOM-channel, activated one at a time, static.** "They are
  static, with no adaptation to the agent's actions." Pixel and hybrid
  primitives are explicitly future work.
- **No benign-injection control.** "we also include no benign-injection
  control experiments to quantify the effect of traps increasing page size.
  We anticipate that this effect is small… but it must still be quantified."
  This is the single most important missing control: the entire effort-tax
  result is confounded with "the page got longer."
- **No statistics.** "five trials per cell is coarse, and we report
  statistics without significance tests."
- **No Claude-family or broader frontier coverage**, "a limitation driven
  by cost."
- **Agents are never warned.** "providing instructions on how to identify
  and escape from traps may reduce the impact of the deceptions" —
  untested.
- **Mitigations are proposals, not results.** "These are design
  implications rather than evaluated mitigations." The proposed set:
  preserve provenance and trust labels across tool boundaries, require
  independent evidence before accepting success or changing goals, validate
  final results against a trusted service, and bound exploration of
  hypotheses that repeatedly fail to corroborate. The paper notes its own
  flag checker "already prevents a fake flag from being scored as success,
  but cannot prevent the agent from wasting its budget."

## Critique / open questions

- **The instruction/evidence distinction is the paper's headline
  contribution and it is never measured as a factor.** The definition is
  crisp and the trap set is deliberately built to straddle the line —
  "Some AgentLSD traps are also indirect prompt injections, while others
  carry no instruction and test whether deceptive evidence changes which
  hypothesis the agent trusts or which routes it explores." But the three
  recorded label axes are **intent × surface × object**; instructional vs.
  non-instructional is not among them, no instance is tagged with it, and
  no figure or table contrasts the two. Figure 4 groups by *intent*. So
  the paper **asserts and designs for** the distinction and **does not
  test** it. A reader wanting "are fabricated results harder to resist
  than fabricated instructions?" must read it off the intent families as
  a proxy — and see the next point.
- **Read as a proxy, the intent families point the *opposite* way from the
  usual expectation, on outcomes.** The most instruction-like families
  (goal hijack, exploration bait — "authoritative-looking cues redirect
  the solver toward a decoy objective") cause the **largest** solve
  losses; the purely non-instructional ones (false validation — "a
  message, badge, or flag-format cue makes a decoy artifact appear
  validated") are intermediate-to-null on outcome. The pattern inverts on
  the effort axis, where false validation and false authority are the
  costliest single cells measured. Neither reading is the paper's own
  claim, and the proxy is loose because goal-hijack payloads are
  quasi-imperative in surface form.
- **The abstract materially overstates coverage.** "We evaluate six models
  on 11 web CTF challenges" is true of the *clean* phase only. The trap
  phase is **four models on five reported challenges**, and the paper says
  so plainly in §5.2 and Table 2 — but the abstract does not.
- **GPT-5's trap numbers are measured on adversarially-selected traps.**
  "We instead run it on the five trap instances that most reduce the
  on-prem solve rate." The conclusion drawn from them — "not even a
  frontier hosted model is immune" — is therefore a worst-case statement
  on traps pre-selected for damage against *different* models, on the five
  easiest challenges, default prompt only, with "Four of its 25 cells
  completed four trials instead of five due to infrastructure problems."
  It is not comparable to the on-prem deltas.
- **The headline effort number is not reconstructable from the reported
  tables.** "+20 turns and +2k reasoning tokens" appears in the abstract
  and once in the introduction ("models that still recover the flag
  typically use an additional 20 turns and 2,000 reasoning tokens") and
  **nowhere in the results section**. The two per-instance grids that *are*
  reported (Gemma-4 31B, DeepSeek-V4) show typical cell deltas in the
  single digits — most rows between −3 and +11 turns — with a handful of
  large outliers (+39, +42, +34, +30, +22). GPT-OSS 120B's grid is not
  shown. The +20/+2k figure is plausibly a mean dragged by GPT-OSS and the
  outlier cells, but **I could not verify it against any reported table**,
  and the paper states it carries no significance test. Treat it as an
  envelope, not a typical effect.
- **Reporting is selected twice over.** 11 challenges → 7 run with traps
  (4 excluded as unsolvable, defensible) → **5 reported** ("omitting two
  where models show moderate performance. This reduces uncertainty and
  concentrates the analysis on the impacts of deceptions"). Dropping the
  two mid-difficulty challenges is the direction that inflates apparent
  effect cleanliness, and no numbers are given for the omitted two.
- **A small prose/table mismatch.** "Authoritative-hint traps levy a
  smaller but consistent tax, adding 7 to 10 turns across sqli-easy,
  sqli-simple, and ssrf-medium" for Gemma — true of the hidden and visible
  variants (+7…+10) but not the accessibility one, which is +5/+4/+3 in
  the same grid. Minor, but the paper is quoting a range its own figure
  does not fully support.
- **Determinism is asserted, not demonstrated.** Resolved instances are
  "intended to be byte-identical across repeated runs with the same seed";
  no hash or re-generation check is reported. Agent-side determinism is
  bounded by temperature 0 only, which does not make hosted inference
  reproducible.
- **The runtime fails open.** "If an application error occurs, the
  [runtime will] fail open and return the original response." Combined
  with DOM delivery verification this is the safe direction for validity
  (a silently-undelivered trap becomes a clean trial rather than a
  mislabelled one), but the rate of such fail-opens is not reported.
- **Domain gap, acknowledged.** "The measured effect sizes should not be
  extrapolated directly from web CTFs to production." Eleven purpose-built
  Flask puzzles with short-to-moderate solution paths, one scaffold, one
  delivery channel.

## Trust signals

- **Credibility:** 4 — Three established European security groups
  (Trento, Turin, Politecnico di Torino; Crispo and Bergadano are
  long-standing names in the field), EU-funded (SPECTRO, grant 101123118),
  and **accepted to AISec 2026 co-located with CCS 2026** with an ACM DOI
  already registered (10.1145/3847352.3848111) — so the `peer_reviewed:
  false` flag above follows this project's arXiv/workshop convention and
  under-reports the actual review status. The reproducibility package is
  the strongest signal: framework, trap specifications, all 14 resolved
  YAML instances, experimental configs and **raw traces** released at
  github.com/Golim/agent-lsd, with deterministic seeded generation, a
  purpose-built and deliberately unreleased challenge corpus, per-trial
  fresh flags in isolated containers, temperature 0, web tools disabled,
  and an independent flag-checking oracle. The limitations section is
  unusually forthright. Held at 4 rather than 5 by: **no significance
  tests and five trials per cell (self-declared)**, **no benign-injection
  page-size control** for the paper's own headline effect, a headline
  number (+20 turns / +2k tokens) that does not appear in the results
  section and cannot be reconstructed from the reported grids, an abstract
  that overstates trap-phase model coverage 6→4, GPT-5 evaluated only on
  traps pre-selected for maximum damage, and a reported-challenge set
  narrowed from 11 to 5. `citations: null` — posted 16 Sep 2026.

## Follow-up

- **Relevance:** 3. Cite-worthy prior art on an active theme, not a
  concept-shifter. Two reasons it does not go higher. First, the domain is
  offensive/defensive security agents on web CTF puzzles; this project
  curates architecture for **autonomous research agents**, and the
  transfer is by analogy. Second and more decisive, **the one contribution
  that would have made it load-bearing here — the instruction vs.
  non-instructional-evidence split — is asserted in the framing and never
  measured as a factor** (see Critique). What it does contribute is real
  but narrow: a clean paired-design template, an honest demonstration that
  a contaminated source can cost an agent a large multiple of its budget
  *without changing its answer*, and the contact-vs-disengagement
  decomposition. It does not go to 2 because the effort-tax mechanism is
  directly instantiated by this repo's own ingest loop.
- **[[concepts/information-firewall]] — record, and decline to edit.**
  The concept's four boundaries (file space, retrieval space, time,
  protocol) are all about **withholding** information so that a measured
  number means what it claims; the `shao2026language` entry adds surface
  reparameterization as a boundary that *fails*. AgentLSD is a different
  axis entirely: not "did the answer leak in?" but "did an adversary push
  *false* evidence in?" Adding it to `sources:` would blur a concept whose
  precision is its value. The paper is worth naming in a future edit only
  if a second source arrives that actually measures the instruction/
  evidence split — at which point the right framing is that a firewall
  governs *what reaches* the agent, while this governs *what the agent
  trusts once it has arrived*, and the two failure modes need different
  controls. Note also the one place AgentLSD does strengthen the existing
  concept: its contamination control is textbook — purpose-built
  challenges never publicly released, per-trial freshly generated flags,
  and web search disabled in the scaffold, exactly the NatureBench
  retrieval-time stipulation.
- **[[concepts/evidence-gated-completion]] — propose adding to `sources:`.**
  The paper supplies a *measured* boundary condition for the concept, from
  the failure side. Its dedicated flag-checking service is a textbook
  evidence gate — an independent oracle holding the trial's ground truth,
  "the sole authority on correctness" — and the paper states exactly what
  such a gate does and does not buy: it "already prevents a fake flag from
  being scored as success, but cannot prevent the agent from wasting its
  budget." That is the cleanest statement in the graph that **a completion
  gate protects the verdict and not the cost of reaching it**, and it is
  backed by the false-validation cells where the gate holds the solve at
  5/5 while the agent burns ~5× the turns. The concept's proposed
  mitigation list is also stated here in one sentence: require independent
  evidence before accepting success *or changing goals*, and bound
  exploration of hypotheses that repeatedly fail to corroborate. The
  "or changing goals" half is new — this concept has only ever gated
  *completion*, not *goal revision*, and goal hijack is the family that
  causes the largest outcome loss.
- **[[concepts/typed-claim-partition]] — record; weak proposal to add to
  `sources:`.** The paper's first named mitigation is "preserve provenance
  and trust labels across tool boundaries", which is this concept's
  provenance annotation (tool-observed vs. signal-observed vs.
  model-inferred) stated as a security requirement rather than a grounding
  one, and it supplies the motivating threat: §2.1's observation that
  "Even when inputs are marked with roles or message types, the trust
  boundary is only syntactic. The model must still decide semantically
  which content is an authoritative instruction and which content is
  potentially adversarial." That is a sharp articulation of why a typed
  partition needs enforcement rather than labelling. But it is proposed,
  not evaluated — the paper runs no provenance-labelling arm — so this is
  a framing citation at best. I lean toward record-only.
- **[[concepts/budget-as-ceiling]] — record, and decline to edit for now.**
  The interesting inversion: this project treats a budget ceiling as a
  *safety* mechanism that halts a runaway chain, and AgentLSD shows the
  same ceiling is an **attack surface** — an adversary who cannot change
  your answer can still consume your ceiling, and because failed runs cost
  5–8× successful ones (55–80 turns vs 10–30 clean), burning budget is how
  a contaminated source converts into a failed run. Worth adding only when
  a source measures this against a *research* agent rather than a CTF
  solver; the effect size here is confounded with page-size inflation by
  the paper's own admission.
- **Why this matters for this repo's ingest pipeline — honestly bounded.**
  This project's agents read `raw/` (downloaded PDFs) and web captures,
  i.e. exactly the non-instructional evidence channel the paper names, and
  this very session had subagents read a PDF fetched from arXiv. The paper
  does **not** support the specific practice the digest hoped for — it
  offers no evidence that fabricated *results* are harder to resist than
  fabricated *instructions*, because it never measures that contrast, and
  the intent-family proxy points the other way on outcomes. What it *does*
  support, with measured effect: (i) a fabricated "this is validated"
  artifact in a source can impose a 4–5× effort multiple while leaving the
  final answer correct, so **an unexplained cost spike during ingest is a
  contamination signal worth logging even when the output looks fine**;
  (ii) fragility is dwell time, so the cheap instrumented defense is a
  **bound on how many steps an agent may spend on a lead that has produced
  no corroborating evidence**, not a detector for deceptive content; and
  (iii) all of this is measured on adversarial artifacts planted by a
  designer, whereas this repo's threat is closer to honest-but-wrong
  papers — a weaker adversary the paper says nothing about. Concrete and
  low-cost takeaway: nothing here justifies changing `/ingest`'s reading
  behaviour, but the effort-tax finding argues for recording per-ingest
  turn/token cost so an outlier becomes visible.
- **Independence / future digest candidates.** The most valuable cited
  work for this graph is **Shi et al., "Large Language Models Can Be
  Easily Distracted by Irrelevant Context" (ICML 2023)** — cited here as
  the precedent that "non-instructional artifacts are known to change the
  behavior of language models", and the closest thing in the literature to
  the instruction/evidence contrast AgentLSD declines to run. Not in this
  graph; worth a digest. Also cited and absent as first-class notes:
  **CTFExplorer** (Rani et al., arXiv:2602.08023, 2026 — multi-target web
  CTF benchmarking) and **NYU CTF Bench** (Shao et al., NeurIPS 2024
  D&B) for the CTF-agent baseline, and **Juels & Rivest, "Honeywords"**
  (CCS 2013) as the source of the honeytoken instrumentation trick, which
  is the paper's neatest methodological import — decoys repurposed from a
  defense into an *attribution* signal. AgentDojo (Debenedetti et al.,
  NeurIPS 2024) and PoisonedRAG (USENIX Security 25) are both already
  referenced inside this graph via
  [[literature/papers/ray2026what]], [[literature/papers/guo2026when]] and
  [[literature/papers/karamchandani2026your]], so they need no new entry.
