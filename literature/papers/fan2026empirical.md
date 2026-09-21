---
kind: paper
title: "An Empirical Study of Harness Design for Coding Agents"
authors: ["Run-Ze Fan", "Zihao Zhang", "Simin Ma", "Yebowen Hu", "Shouju Wang", "Kaiqiang Song", "Fei Liu", "Hamed Zamani", "Xiaoyang Wang"]
institutions: ["UMass Amherst", "Zoom Video Communications", "Emory University", "UNC Charlotte"]
year: 2026
venue: "arXiv (cs.AI / cs.CL / cs.LG / cs.SE)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.20804"
code_url: null
citations: null
source: "raw/papers/fan2026empirical.pdf"
added: "2026-09-21"
relevance: 4
credibility: 4
status: read
related_experiments: []
related_concepts:
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/lossless-context-offload]]"
  - "[[concepts/scripted-tool-pipelines]]"
  - "[[concepts/context-proprioception]]"
  - "[[concepts/hierarchical-delegation]]"
tags: ["harness-ablation", "context-management", "planning", "action-space", "swe-bench", "terminal-bench", "component-level", "negative-result", "conditional-effects", "cost"]
---

# An Empirical Study of Harness Design for Coding Agents

## TL;DR

Hold the ReAct loop, the model, and the task fixed; vary exactly one
harness component at a time. Across **176 matched settings** (4 models ×
2 benchmarks × 5 context-management tiers × 4 window budgets, plus a
planning ablation and an action-space ablation at T4/128k), no component
is unconditionally good. Context management's benefit is almost entirely
**overflow prevention**, not better selection: the managed-vs-unmanaged
success gap collapses from **35.7 to 2.7 percentage points** on SWE-Bench
as the window goes 32k → 128k, tracking the T0 overflow rate's fall from
**78.7% to 8.7%**, and the five tiers differ far more in *cost* than in
accuracy. The lossless half — storing elided observations and exposing
`recall_event` — is **a null result**: across 32 matched T2-vs-T1
comparisons the mean difference is **−0.36 pp**, and 36 of 64
recall-enabled settings never call the tool once. Planning flips sign
with capability: **+11.6 pp** for the 30B model, **−30%/−32% cost with
−2.0/−0.4 pp accuracy** for the two strongest. Predefined tools are
scaffolding for models that cannot express intent in bash (**+15.0 pp**
for 30B) and pure overhead for one that can (**+3.6 pp and −53% cost**
for bash-only on 550B).

## Claims

- **Harness design is conditional, not default.** "Harness design is thus
  a conditional systems problem in which each component should be
  selected for the target model, task type, and resource budget rather
  than adopted as a default." Every one of the four headline effects
  reverses sign somewhere in the 4-model × 2-benchmark grid.
- **Context management is an overflow guard first and a selection policy
  a distant second.** The abstract states it directly: context management
  becomes "increasingly valuable as the context-window budget tightens,
  with most of its benefit coming from preventing context-overflow
  failures." Trajectory analysis supplies the mechanism: it "primarily
  extends execution trajectories without substantially altering agent
  behavior."
- **Making elision reversible buys nothing.** "making elided content
  recoverable adds machinery that models rarely use and yields no accuracy
  gain." This is a directly matched test (T1 = elide-and-discard vs
  T2 = elide-and-store-with-`recall_event`; everything else identical).
- **Cheap rule-based compaction before expensive LLM compaction is the
  efficiency frontier.** T4 (elide at soft threshold B₁, summarize only
  above hard threshold B₂) matches the other tiers on accuracy at the
  lowest cost, because early elision keeps the summarizer from firing.
- **Planning is an accuracy scaffold for weak models and a stopping rule
  for strong ones.** It "sustains the trajectories of models that abandon
  tasks too early and trims repeated verification" … "in models that verify
  too long" (a page break falls between the two fragments).
- **Action space trades expressive granularity against interface
  competence.** "structured tools support models with limited shell
  proficiency, while bash-only enables capable models to combine multiple
  code modifications in a single tool call." The crossover also moves with
  how shell-centric the *task* is, not just the model.

## Methods

- **Harness.** Purpose-built on LangGraph, ReAct loop (reason → act →
  observe), benchmarks driven through Harbor which owns each task's
  container and verifier. Three components vary; workspace guard,
  read-before-write check, permission layer (allow/ask/deny), ruff/pyflakes
  post-edit diagnostics, and stuck detection are **held fixed** so they form
  a common substrate. Max 300 steps/task, tool results truncated at 24k
  chars, up to 8 read-only tools in parallel per step.
- **Three mechanisms, five tiers.** M1 elision (replace a stale tool
  observation's body with a stub), M2 recall (store the original on disk,
  expose `recall_event(id)`), M3 summarization (fold old middle events into
  a running natural-language summary via a separate tool-free call to the
  same model). **T0** = none (run dies on overflow); **T1** = M1;
  **T2** = M1+M2; **T3** = M3; **T4** = all three, staged — elide at soft
  threshold B₁ = 0.6 of window, summarize only above hard threshold
  B₂ = 0.85. T1–T3 act only at B₂. Preamble plus a recent window (0.3 of
  budget, floor 2 turns) is always verbatim; only the middle region is
  touched.
- **Grid.** T0–T4 × {32k, 64k, 96k, 128k} = 20 settings per
  model–benchmark pair, all with predefined tools and planning on. Two
  further ablations at the T4/128k baseline: planning off, and bash-only.
  22 × 4 models × 2 benchmarks = 176 settings.
- **Models.** Nemotron-3 30B / 120B / 550B as a within-family capability
  axis, plus Mistral-Medium-3.5-128B cross-family. Served locally with
  SGLang in BF16, T = 0, top-p 0.95, 16,384 output tokens/turn. Costs are
  *priced* at OpenRouter rates, not paid.
- **Benchmarks.** SWE-Bench Verified (500 tasks) and Terminal-Bench 2.1
  (89 tasks). Metrics: success rate and mean cost per task.
- **Statistics.** Three comparison families per benchmark (tier vs T0,
  planning on vs off, tools vs bash). Two-sided exact **McNemar** tests on
  task-paired outcomes with **Benjamini–Hochberg** FDR control at q < 0.05.
- **Trajectory analysis.** Every turn labeled by a GPT-5.5 judge
  (reasoning effort high, T = 0.6) into SWE-Bench phases
  (Localize/Reproduce/Fix/Verify/Other) or a 10-symbol Terminal-Bench action
  taxonomy, plus a failure-stage judge (file loc / line loc / patch /
  verification) run only on unresolved trajectories. Validated against
  **three human annotators over 15,610 labeled units**.
- **Action-space intervention is deliberately bundled.** The paper says so:
  it "jointly varies tool availability, interface-specific prompts,
  file-state tracking, and automatic post-edit diagnostics, so it does not
  isolate the effect of tool count or action granularity."

## Results

### Context management: the benefit is the overflow guard

- **The gap collapses with the window.** Averaged over models, the
  managed(T1–T4)-minus-T0 success gap goes **35.7 → 15.9 → 5.5 → 2.7 pp**
  on SWE-Bench and **9.5 → 7.5 → 4.8 → 2.8** on Terminal-Bench across
  32k/64k/96k/128k.
- **And it tracks overflow exactly.** The model-averaged **T0 overflow rate
  falls from 78.7% to 8.7%** on SWE-Bench and **61.0% to 12.1%** on
  Terminal-Bench over the same sweep, "while all managed tiers have zero
  overflow failures throughout."
- **Which tier you pick barely moves accuracy.** At 32k on SWE-Bench, 550B
  scores 51.40 / 53.60 / 58.40 / 55.60 under T1/T2/T3/T4 against T0's 6.40
  (Table 3) — a 7-point spread among managed tiers against a 45-point
  spread against no management. At 128k the tiers are within ~2 points of
  each other for every model.
- **Behavior is unchanged; only length changes.** At 128k, "the median
  trajectory length and the mean number of tool calls vary only within a
  narrow band for every model": 39–42 turns for 30B, 70–74 for 550B, with
  re-patch counts differing by fewer than two per task. At 128k the
  failure-*stage* distribution also barely moves, so "the tier comparison at
  128k reflects small differences in how many runs fail rather than where
  they fail." At 32k, T0's medians are 20–30 turns and most runs die in
  Localize; managed tiers stretch medians to ~50–180 turns.
- **T4 is the cost frontier.** T4 matches T1–T3 on success "with the lowest
  cost in seven of eight model–benchmark panels," has the lowest mean
  peak-context ratio at all four budgets, and "T4 has the lowest mean cost
  per task at every window budget." Mechanism: early cheap elision means the
  metered LLM summarizer fires less often. Concretely, 550B at 32k SWE-Bench
  costs $2.57 (T1) / $2.70 (T2) / $1.25 (T3) / $1.45 (T4) vs $0.26 for a T0
  that mostly dies early.

### Recall (M2): a clean null

- **Accuracy.** Across 32 model–benchmark–window cells, "T2
  outperforms T1 in 15 settings, underperforms in 14, and ties in three;
  the equal-weight mean difference is −0.36 percentage points (+0.40 on
  SWE-Bench, −1.12 on Terminal-Bench)."
- **Usage.** "Among the 64 T2 and T4 settings, 36 (56.3%) never call
  recall_event, the median invocation rate is zero, and the equal-weight
  mean falls from 0.540 calls per task at 32k to 0.069, 0.011, and 0.007 at
  64k, 96k, and 128k; the 16 planning and action-space settings at T4/128k
  record no recall calls at all."
- **Usage is concentrated in the weakest model under the tightest budget.**
  Table 13: Nemotron-3 30B on Terminal-Bench at 32k is the only cell above
  1.0 calls/task (T2 = 4.326, T4 = 2.708). Every 550B and Mistral cell at
  64k and above is **exactly 0.000**. And the heaviest-use cell is a loser:
  it "averages 4.326 calls per task and scores 3.37% below T1."

### Planning: sign flips with capability

| Model (SWE-Bench, T4/128k) | plan off | plan on | Δ SR | cost off → on |
|---|---|---|---|---|
| Nemotron-3 30B | 13.60 | 25.20 | **+11.6*** | $0.02 → $0.09 |
| Nemotron-3 120B | 46.60 | 44.00 | −2.6 | $0.25 → $0.34 |
| Nemotron-3 550B | 67.80 | 65.80 | −2.0 | $3.31 → $2.33 (−30%) |
| Mistral-3.5-128B | 69.00 | 68.60 | −0.4 | $4.65 → $3.14 (−32%) |

- **Weak model: planning keeps the run alive.** Disabling planning drops the
  30B median SWE-Bench trajectory from **40 to 5 turns**; "68.6% of runs
  terminate without an edit and 58.4% terminate during" … "Localize,
  compared with 27.8% and 10.4%, respectively, with planning (Table 6)"
  (a page break falls between the two fragments). Its failure-stage
  distribution shifts too: removing planning "pushes the file-localization
  share from 56.3% to 73.8%."
- **Strong models: planning is a stopping rule.** It "reduces the median
  trajectory length from 108 to 74 turns for Nemotron-3 550B and from 68 to
  53 turns for Mistral-Medium-3.5-128B," and "The phase composition
  attributes most of this reduction to verification rather than localization
  or fixing." Turns, tool calls, and mean input tokens per call all fall
  (−24.3% / −24.5% / −12.4% for 550B; −23.2% / −23.3% / −15.1% for Mistral,
  Table 5), while the without-edit rate stays below 3%.
- **On Terminal-Bench planning has two opposing effects** — extending runs
  that would die early and truncating over-long tails — so the cost sign is
  set by which dominates: −26% for 120B, −3.6% for 550B, ~−40% for Mistral,
  but **+75% for 30B**.

### Action space: predefined tools are training-data alignment, not ergonomics

| Model | SWE-Bench tools / bash | Terminal-Bench tools / bash |
|---|---|---|
| Nemotron-3 30B | 25.20 / 10.20* | 13.48 / 3.37* |
| Nemotron-3 120B | 44.00 / 42.40 | 28.09 / 23.56 |
| Nemotron-3 550B | 65.80 / **69.40*** ($2.33 / $1.11) | 44.94 / **50.56** ($2.43 / $1.70) |
| Mistral-3.5-128B | 68.60 / 45.40* | 37.08 / **43.82** |

- **The 30B failure is an interface failure, not a reasoning failure.**
  Without predefined tools it "falls back on tool-call patterns acquired
  during training rather than translating intended operations into bash"
  and "the harness cannot resolve emitted calls into executable actions"
  (two fragments of one passage, separated by one sentence). On
  Terminal-Bench, **66% of bash-only trajectories terminate after such
  out-of-interface emissions, shortening the average trajectory from 71 to
  15 turns.**
- **Strong models are taxed by the structured interface.** For 550B,
  "bash-only improves success rate by 3.6% on SWE-Bench and 5.6% on
  Terminal-Bench while reducing cost by 53% and 30%, respectively," issuing
  32% fewer calls on SWE-Bench and 24% fewer on Terminal-Bench.
- **Granularity is the mechanism (Table 7).** Bash-only cuts re-patches of
  already-edited files for all four models (3.3 → 0.4 for 30B, 2.8 → 2.2 for
  120B, 4.6 → 1.5 for 550B, 3.0 → 1.3 for Mistral) and shifts Terminal-Bench
  file writes toward coarse create-or-replace (28 → 64%, 39 → 76%, 51 → 76%,
  28 → 57%). For 550B the median largest SWE-Bench edit "grows from 18 to 54
  lines." The paper's summary: "predefined tools lower the complexity of each
  individual action, but they often do so by increasing the number of
  interactions and incremental repair cycles needed to express the same
  operation."
- **Task type, not just capability, sets the crossover.** Mistral is the
  clean case: "The full tool set improves success by 23.2% on SWE-Bench, but
  bash-only adds 6.7% on Terminal-Bench." With all tools it issues 71.9% of
  Terminal-Bench workspace actions through bash vs 40.4% on SWE-Bench. Its
  bash-only SWE-Bench collapse is a *localization* failure, occurring before
  repair: "32.8% of Mistral's bash-only SWE-Bench runs end without editing a
  file, versus 1.2% with the full tool set," and its file-localization
  failure share rises from 16.0% to 41.4% (Table 10).

### Judge validation

Three annotators, non-overlapping splits, 15,610 units (10,254 SWE-Bench
action labels, 5,306 Terminal-Bench, 50 failure-stage). Per-split agreement
88.7–98.4%, κ 0.858–0.980; aggregate "approximately 94.2% raw agreement and
a weighted mean Cohen's κ of 0.929."

## Critique / open questions

- **The "most of its benefit" claim is well supported, but it is not the same claim as *selection quality does not matter*.** What the
  paper actually shows is that *variation among four reasonable compaction
  mechanisms* is worth little accuracy once the window stops binding. Every
  tier shares the same selection *target* — preamble and a recent window
  pinned verbatim, only the middle region touched. There is no random-eviction
  or drop-the-recent-window arm, so the common rule's contribution is never
  measured. The honest statement is: **given a sane anchor-plus-recency rule,
  the marginal return on a cleverer middle-region policy is near zero, and the
  return on not dying is large.** A digest gloss of *rather than from better
  selection* overshoots this slightly.
- **The residual at 128k is real but undecomposed.** The model-averaged gap
  at 128k is only 2.7 pp, yet all four managed tiers beat T0 for Nemotron-3
  550B at 128k with FDR-adjusted significance (T0 59.80 vs T1 65.20*, T2
  67.40*, T3 65.80*, T4 65.80*). T0's overflow rate at 128k is 8.7%
  model-averaged, which could account for the whole 6-point 550B gap — but
  the per-model overflow rate at 128k is only in Figure 3, not in text or a
  table, so this cannot be checked from the paper. The overflow-only story is
  *consistent with* every number reported and *proven* by none of them.
- **The recall null has a domain-specific confound the paper does not name.**
  In a coding workspace the environment is already a lossless addressable
  store: a file can be re-read, a command re-run. The harness says so out
  loud. The T1 stub reads "[tool output elided: {N_LINES} lines / {N_CHARS}
  chars. **Re-read or re-run to get it again.**]" and the `recall_event`
  description instructs "Often you can instead just re-read the file or
  re-run the command — use recall_event when the output isn't easily
  reproducible (e.g. a past test log)." So the harness both supplies a free
  alternative recovery path *and explicitly steers the model to prefer it*.
  That makes this a strong negative result for recall machinery **in
  environment-backed domains** and a weak one for irreproducible evidence
  (web captures, non-deterministic logs, a completed experiment's stdout) —
  which is exactly the regime this project's own `raw/` corpus lives in.
- **Cost comparisons against T0 are not like-for-like.** T0's low cost at
  tight budgets is the cost of dying early: 550B at 32k SWE-Bench costs
  $0.26 under T0 at 6.40% success. Any cost-per-task figure here needs
  reading alongside the success rate and the median trajectory length.
- **Non-frontier models, and capability is proxied by size.** The paper
  concedes it: "Model size is only an imperfect proxy for capability, since
  differences in training, prior exposure to tool interfaces, and native
  shell proficiency may also contribute." Mistral is the proof — its
  interface preference flips by *benchmark*, not by size. Nothing here was
  run on a frontier model; the 30B "weak model" regime (out-of-interface tool
  emissions, 5-turn collapses) is unlikely to reproduce on Opus/Sonnet-class
  models, which means the *weak* half of each conditional is the half least
  likely to transfer to this box.
- **Statistical power is thin on half the grid.** "Each setting is run once
  per task, and Terminal-Bench contains only 89 tasks, so many Terminal-Bench
  contrasts do not reach significance under the paired McNemar test; our
  conclusions there rest on consistent directions across models and budgets
  rather than on individually significant cells." One run per task at T = 0
  means no [[concepts/pass-at-k]]-style variance estimate anywhere — the
  scattered non-monotonicities in Table 4 (120B Terminal-Bench T1 swings
  33.71 → 29.21 → 22.47 → 22.47 across budgets) are unattributed noise.
- **Planning and action space are ablated at one point only.** "Planning and
  the action space are ablated only under the default T4/128k configuration
  due to limited computing resources, and a full factorial study would be
  required to determine whether their effects persist under other
  combinations of components and context budgets." The interaction that
  matters most for this project — does planning still save cost when the
  window binds? — is not measured.
- **Interventions are single implementations.** One planning prompt, one
  `update_plan` tool, one threshold policy at fixed 0.6/0.85. A null on
  `recall_event` is a null on *this* recall affordance, whose tool
  description actively discourages use.
- **No code release.** No repository, no trajectory dump, no artifact link
  anywhere in the paper. 176 settings and a GPT-5.5 judge pipeline are not
  reproducible from the text.
- **Mild conflict of interest, disclosed.** Two first authors did the work
  "during internships at Zoom Video Communications," and the corresponding
  address is a Zoom one. (Affiliation 2 renders as a logo and is blank in the
  PDF's text layer — "Zoom Video Communications" in the frontmatter is
  inferred from that footnote and from `Xiaoyang.W@zoom.us`, not read off
  the affiliation line.) Nothing here sells a product, and the conclusions
  are unflattering to harness complexity generally, so the risk is low.
- **A typesetting defect in the source.** The sentence about Mistral's
  bash-only localization failures is truncated mid-number in the PDF's text
  layer ("the share never reaching the correct file rises from 16.0The
  Mistral result is therefore…"). The intended figure (16.0% → 41.4%) is
  recoverable from Table 10.

## Trust signals

- **Credibility:** 4 — arXiv preprint, not peer-reviewed, no code released,
  but UMass Amherst (Hamed Zamani's group) with Emory and UNC Charlotte, and
  the methodology is unusually disciplined for a harness paper: fully matched
  cells with one component varied, exact McNemar on paired outcomes with
  Benjamini–Hochberg FDR control, an LLM judge validated against three human
  annotators over 15,610 units (κ = 0.929), and a Limitations section that
  names the bundled action-space intervention, the single-run power problem,
  the one-point planning ablation, and size-as-capability-proxy without being
  asked. Docked from 5 for no artifacts, one run per task, and non-frontier
  open models.

## Follow-up

- **Relevance:** 4. It changes three memory concepts and one tool-design
  concept with matched evidence, and is the only component-level harness
  ablation in this graph at anything like 176 cells. Held at 4 rather than 5
  because no frontier model was tested, the domain is software engineering
  rather than ML research, and the capability-conditional findings are
  anchored on a 30B failure regime that will not reproduce on this box.
- **[[concepts/lossless-context-offload]].** The concept's twelve sources
  argue the invariant from theory (zhu2026lossy's write-before-query
  barrier, xu2026llm's Prop. 1), from serving cost
  ([[literature/papers/dang2026addressable]]), and from deployment scale
  ([[literature/papers/mason2026missing]]'s 1.39M evictions at a 0.0254%
  fault rate). None of them ran a **matched accuracy comparison against the
  same eviction with the address removed**. This paper does, and finds
  nothing: −0.36 pp over 32 cells, 56.3% of settings never touching the
  tool. Crucially, it is the same finding
  [[literature/papers/semenov2026beyond]] already implies — when "the
  environment itself is the addressable store," a second addressable store
  is redundant — now measured. The concept should keep the invariant and
  gain an explicit **scope condition**: pay for recall machinery when the
  evicted content is *not* re-derivable from the environment.
- **[[concepts/context-eviction-policy]].** The concept treats the rule as
  a fidelity, cost, and governance surface. This paper adds a fourth and
  blunter reading of the same evidence: at a generous window the rule is
  mostly a *liveness* device. It also puts a price on the
  cheap-before-expensive ordering — T4's staged elide-then-summarize is the
  cost frontier in 7 of 8 panels — which is a concrete implementation
  recommendation the concept currently lacks.
- **[[concepts/scripted-tool-pipelines]].** Strong, quantified support with
  a capability precondition. Bash-only *is* the un-scripted form of this
  pattern, and for a capable model it wins on both axes (550B: +3.6 pp and
  −53% cost on SWE-Bench, 32% fewer calls, median largest edit 18 → 54
  lines). But the same interface destroys a model that cannot express intent
  in shell (30B: 25.20 → 10.20 on SWE-Bench, 66% of Terminal-Bench runs dying
  on out-of-interface emissions). The concept should carry the precondition.
- **[[concepts/context-proprioception]].** Weaker than the digest implied.
  This paper never ablates *visibility* — every tier shows the model the same
  elision stubs — so it does not test VISTA's claim, which is about the
  archive *manager's* observability rather than the task solver's. The one
  transferable observation: the T2/T4 stubs report `{N_LINES}` and
  `{N_CHARS}` plus the event id, which is per-item cost proprioception in
  miniature, and it produced essentially no recovery behavior. Consistent
  with [[literature/papers/bai2026how]]'s finding that access to state is not
  the same as acting well on it, and worth logging in Open questions rather
  than as a rebuttal.
- **[[concepts/hierarchical-delegation]] / planning generally.** The
  capability-conditional result is the interesting part: a persistent plan
  scaffold's job changes from *keeping the run alive* to *making it stop*.
  For a strong model, the measured benefit of planning is almost entirely
  trimming redundant post-edit verification (108 → 74 median turns, with the
  reduction attributed to the Verify phase, at −2.0 pp accuracy). That is
  the first quantified statement in this graph that planning scaffolds can
  cost a strong model accuracy while saving it money.
- **A conditionality concept may be ripe.** "each component should be
  selected for the target model, task type, and resource budget rather than
  adopted as a default" is the paper's real thesis and has no home in the
  35 existing concepts. It converges with
  [[literature/papers/wang2026act]]'s 16-combination harness × model grid
  (where the minimalist harness beat feature-rich ones) and with
  [[literature/papers/esakkiraja2026starharness]]'s premise that harnesses
  should be *searched* rather than designed. Hold for a third attestation
  before seeding.
- **Independence / digest candidates.** None of the closest converging works
  are in this graph: Liu (2026), "More is not always better: Cross-component
  interference in LLM agent scaffolding" (arXiv:2605.05716) — the nearest
  prior work, full-factorial but short-horizon and without a window sweep;
  Lewis (2026), "Same model, different harness" (arXiv:2608.26218);
  Rombaut (2026), "Inside the scaffold: a source-code taxonomy of coding
  agent architectures" (arXiv:2604.03515); Bogavelli et al., AgentArch
  (arXiv:2509.10769); Mehtiyev & Assunção (arXiv:2604.02547), whose
  trajectory-phase taxonomy this paper builds on; and Ehrlich & Blackman,
  "LCM: Lossless context management" (arXiv:2605.04050), which is the
  direct counterparty to the recall null.
