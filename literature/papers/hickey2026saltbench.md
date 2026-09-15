---
kind: paper
title: "SaltBench: A Referee-Gated Protocol for Measuring Method Effects in Machine-Checked Software Work"
authors: ["Jason Hickey"]
institutions: []    # none stated in the PDF author block
year: 2026
venue: "arXiv (cs.SE)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.11076"
code_url: "https://github.com/jyh/saltbench"
citations: null
source: "raw/papers/hickey2026saltbench.pdf"
added: "2026-09-15"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/information-firewall]]"
  - "[[concepts/enforcement-boundary-placement]]"
tags: ["evaluation", "benchmark-design", "preregistration", "isolation", "sandbox", "budget", "censoring", "formal-verification", "claude-code", "negative-result"]
---

# SaltBench: A Referee-Gated Protocol for Measuring Method Effects in Machine-Checked Software Work

## TL;DR

The paper asks what a working method costs a coding agent when a machine
referee (proof kernel, program verifier, or withheld test suite) decides
the outcome. The subject is the "seat" (Claude Code plus workspace), not
the model. The contribution is the **protocol**, not an effect. Isolation
is probed before any scored run. Every run is authorized by a dated freeze
commit with its predictions registered. A budget stop is scored as a
**halt, never a failure**. On five authored Rust/Verus components at Opus 5,
a reduced ("diet") specify-and-verify arm **cost more on 5 of 5 problems**
(one-sided sign test p = 0.0312). The registered reading **forbids quoting
any ratio or range** as the method's cost: three of five premiums sit below
the design's resolvable floor of 2.0072×. The gold pair (both arms handed
the formal statement) **reached no verdict** (3 of 4, p = 0.3125). A
post-hoc correctness pass **did not separate the arms**. Across all four
substrates, nothing shows the method changing what a referee accepts. The
lasting results are sixteen **instrument findings**. The most pointed for
this project: Claude Code's file-read tool sat **outside the Seatbelt
sandbox** in every scored episode, and the probes that certified the fence
could not see the hole.

## Claims

- **An agent's account of its own work is the least reliable evidence
  about it.** The outcome is decided by a referee outside the agent's
  toolchain. The comparison is fixed before the run "so that it cannot be
  narrated afterwards."
- **Isolation is observed, not assumed.** Five smoke probes must pass,
  carrying the freeze's episode-script hash, before the driver starts.
- **A budget stop is a halt, never a failure.** Scoring a stopped episode
  as a failure "would let the budget instrument move the result."
- **Ask of every gate which arm is more likely to trip it.** A grader that
  refuses what the treatment prompt invites "does not measure a small effect
  badly; it manufactures the opposite one."
- **A stop rule correlated with the arm removes the treatment's hardest work
  from every later sample.** A cap is a sampling instrument, not only a
  spending one.
- **A deny list is only as broad as the layer that enforces it.** A fence
  has to be probed "in the language of every tool it is meant to bind."
- **The paper claims no effect of the method on referee acceptance and no
  population magnitude.** It claims the protocol, the seat-as-subject design,
  the sign of the cost reading with four registered qualifiers, and the
  instrument findings.

## Methods

- **Four arms, two-by-two.** Method (plain vs specify-and-verify) × source of
  the specification (agent-written "bare" vs handed a priori as the formal
  "statement"). The a-priori pair is the registered **gold reading**: it
  separates the cost of verifying against a spec from the cost of writing
  one. An equal-length placebo prompt (1,746 bytes vs the treatment's 1,913)
  controls for prompt length. The treatment is a **diet** rendering of the
  author's Salt method. Prescribed acts were cut after the full rendering hit
  the per-cell cap at $42.24, and "diet" is required on every headline. Only
  the instruction file differs between arms, and it is byte-pinned.
- **S3-Systems population (the reported matrix).** Five components authored
  for the benchmark (Crc32, FreeList, LRU, LZW, Paxos). All are Rust under a
  pinned Verus toolchain, frozen in a dated export before any model call,
  each with a withheld test suite and mutant set (44/44 mutants killed, which
  is a ceiling, not a strength). Greenfield task only. n = 3 per condition at
  claude-opus-5. The primary statistic is a cross-problem one-sided sign test
  on premium(P) = median(diet-bare)/median(plain-bare). The outcome table was
  enumerated in advance: 5/5 → p = 0.0312; 4/5 → 0.1875, registered as not
  positive.
- **Four statement-arm conditions refused at build.** Only LZW's card
  carried a `## Statement` section, so 24 of 24 statement-arm builds on the
  other four refused at the harness. The sections were then derived by tool,
  verbatim from each withheld reference, under a dated amendment (LZW
  reproducing byte-identical as the control). Paxos could not fire: no
  arm-neutral statement exists for a proof-obligation task. The gold pair is
  read at k = 4.
- **Fence.** macOS Seatbelt via the harness's sandbox settings: every host
  denied, writes only under the working copy and session temp, reads denied
  on harness state, credential trees and the run root. The checker runs
  under its own `sandbox-exec` profile. A shell-tool command hook is kept as
  an attempt audit, not as the fence. Stage-B/C views ship only after every
  blind stage-A episode lands, verified by content set-hash rather than by
  the copying tool's exit code.
- **Pre-registration.** Before the freeze commit: population rule and seed,
  byte-pinned arms, checker, constants, reading rule, predictions, adverse
  outcomes, stop rules. After it, every change is a dated amendment appended
  before its own first model call, with its own predictions. Failed
  predictions are recorded with their direction (the cost estimator
  over-priced five consecutive stages, then under-priced the sixth).
- **Budget stop.** A per-episode token stop at 4× the regime's p90. The
  multiple was chosen from the record: no passing episode among 177 exceeded
  2.40×, and the one runaway reached 7.79× and failed. The checker writes
  `class = HALT` from the episode's termination and keeps the would-be class
  as a diagnostic.
- **Three earlier reads** shaped the design. S1: SWE-bench Verified at
  Sonnet 5. S2-Lean: CLEVER Task 1, with a full Lean kernel replay, axiom
  allowlist and statement-immutability checks. S2-Rust: VeruSAGE-Bench proof
  targets, 207 records, with screen, count guard and AST comparison because
  Verus emits no proof object.
- **Scale.** Placebo arm alone $193.42 over 15 cells. Systems-matrix cells
  run $5–$38 each. Code Apache-2.0, data CC BY 4.0, run records to Zenodo at
  release.

## Results

- **Cost reading (bare pair).** Premiums, reading A / B: Crc32 1.1610 /
  1.1610, FreeList 2.8070 / 2.8879, LZW 1.3749 / 1.3749, Paxos 2.4306 /
  2.2437, LRU 1.2826 / 1.1521. **5 of 5 above 1, p = 0.0312 under both
  readings.** Only FreeList and Paxos clear the resolvable floor
  (2.0072× at n = 3, pooled sd(ln cost) 0.30458). Within-condition noise
  exceeds the smallest premium: FreeList plain spans $13.01–$23.38 (1.80×).
- **Gold pair (statement amendment).** Crc32 **0.9384×**, FreeList 1.5246×,
  LRU 1.5214×, LZW 1.8105×. **3 of 4, p = 0.3125, no verdict**, and every
  premium is below the floor. At k = 4 no outcome can reach .05. "Set beside
  the bare arms … the two arms do not tell the same story."
- **A registered prediction failed.** Two diet-with-statement conditions were
  predicted to hit the cost cap. One never fired, and the other did not cap.
  The failure is recorded as a failure.
- **Placebo** returned unresolved on all five problems, inside
  [0.4982, 2.0072], as predicted from the floor. Reading it as support for the
  treatment was **forbidden by name in the registration**.
- **Post-hoc correctness pass** (43 cells, 246 s, no model tokens): pass 35,
  fail 1, cap-cost 3, failed-boot 4. Four of five problems tie. The one
  difference is FreeList treatment 0/1 landed at 6 of 7 tests. A first tally
  read **five** failures, because the runner returns non-zero for a cell
  that never booted; the frozen classes cut that to one. The 18
  statement-arm cells, registered before they ran, passed 18/18.
- **Every budget-capped cell was a treatment cell.** Control: 18 landed, 0
  capped. Treatment: 12 landed, 3 capped. **All three capped cells pass their
  suites completely** (7/7, 7/7, 17/17). The surviving treatment sample is
  therefore easier than the arm, and any treatment pass rate is biased upward
  by construction.
- **The fence had a hole the probes could not see.** The sandbox binds the
  shell tool and its children. Claude Code's own file-read tool executes in
  the harness process, and neither substrate carried a tool-permission rule,
  so "no scored episode of this campaign had the agent's tools fenced by
  path." The certifying probe read through the shell and an in-language
  spawn, both sandboxed. The audit layer is what rescues the S2 reads: 0
  unblocked fenced reads across 15 S2-Rust Opus episodes, and **0 file-tool
  calls at a fenced path, served or not, across 278 parsed S2-Lean
  episodes**. The repair derives a tool-permission deny list from the same
  path list (132 rules beside 11 paths). A canary reached the agent's final
  message without it and appeared nowhere with it.
- **Other fence findings.** The Verus referee's docstring claimed a sandbox
  and process group, but a grep found only the comment and the referee had
  run unfenced. A fifth exec point surfaced only in the red arm's panic text.
  A fence derived from the run root hid the agent's own episode directory, so
  it wrote proofs it could never check. Two passed, at **11–16× the cost** of
  a sighted episode.
- **Gates that penalize the treatment.** Three Verus grader gates in turn
  refused helper lemmas the treatment invites. The AST comparison classified
  a helper before its enclosing `impl` as `STATEMENT_ALTERED`, a cheating
  class (47 of 207 views are impl-enclosed). The screen enforced 29 refusals,
  of which the prompts named 12. The whitelist split a lemma by brace depth,
  a bug found on the third episode ever run after 29 screen, 18 fence and 11
  fixture arms had passed.
- **Screens hide their false positives.** 201 scored Lean episodes produced
  exactly one screen refusal, and it was a complete proof.
- **Guards whose predicate cannot hold.** A provenance guard took its
  fallback on 78 of 78 rows while no number moved. The token-ceiling watchdog
  crashed on the first path-carrying tool call and the shell turned the
  crash into zero, so **the token ceiling was blind on every episode where it
  could have mattered**.
- **Censored caps.** S2-Rust hard band at Opus 5: 9 of 10 pass at a 40-call
  cap, with uncensored passing call counts 23–39, so the p90 is the cap. At
  Sonnet 5 an episode read **0 verified at call 40** and passed at **call 79**
  under a 120-call cap: the same verdict as Opus at 8.7× the tokens and
  3.4× the calls. The median episode spends 81% of its tokens before its
  first clean referee run (range 64–91), so a cap cuts search, not polish.
- **Earlier reads.** SWE-bench Verified saturated: both control arms
  resolved the same 13 of 15 and failed the same 2, both at the cap, with
  contamination verified at the transcript (the agent typed lines of the
  upstream fix before any read could have shown them). S2-Lean: every arm
  proved 8/15 at Sonnet 5 (b = c = 0) and 10/15 at Opus 5 (placebo 9/15).
  Blind triage of 30 failed stage-B cells put **17 as human-loose** (the
  reference spec is weaker than the agent's), 5 agent-wrong and 8
  proof-hard. Problem 18's reference spec is machine-checked unsatisfiable.
  CLEVER's own checker certifies 15/15 when the theorem is replaced with
  `True`.
- **Dataset structure hidden by the scoreboard.** Three problems reached
  n = 3 on the plain arm only by counting a borrowed smoke cell; without them
  the test is 2 of 2, p = 0.25. The top-up cells restored the sign but moved
  magnitudes *down* (LRU 1.2826 → 1.1521). A scorer keyed on (task, arm)
  would have silently pooled the bare and statement arms. Subject-forked
  processes outlived their cell by 3 h 09 min, and the box-load bound written
  over the matrix root does not cover the three cells added later.

## Critique / open questions

- **The digest's "≤ 2.89×" framing is the claim the paper forbids.** The
  abstract does print "no premium exceeded 2.8879×," but Section 4 states that
  a summary "may not state a ratio, a range or a percentage as the cost of
  the method … not 2.7×, not one to three times." The quotable result is the
  sign: 5/5 at n = 3, diet rendering, cost only. The index line and any
  concept prose should carry the sign, not the bound.
- **The effect question is entirely open, and the design mostly shows why.**
  Each substrate closed for a different reason: SWE-bench saturated and was
  contaminated, S2-Lean's null belongs mostly to the oracle, S2-Rust sits at
  the plain arm's ceiling, and S3 measured price only. That is a useful map
  of where a method-effect benchmark can't live. It is not evidence that
  specify-and-verify helps or hurts.
- **Tiny n, authored population, and a treatment by the same author.** Five
  problems at n = 3, written by "the author's assistant heads under the
  author's direction," with the treatment also the author's. The paper states
  both limits. The FreeList within-arm spread (1.80×) exceeding the smallest
  premium means three of five signs are not individually trustworthy, which
  the paper also states.
- **The gold pair weakens rather than supports the bare reading.** At k = 4
  it cannot reach significance at any outcome, Crc32 came in below parity,
  and no premium clears the floor. Read plainly: once the spec is given, the
  verification overhead is not distinguishable from zero on this population.
  The paper refuses to read it that way too, since a sub-floor premium is not
  a direction.
- **The prose is exhausting, and that is partly the point.** Nearly every
  number travels with its registration status, qualifiers and forbidden
  readings. That makes this an unusually honest record and a hard paper to
  extract from. Some instrument findings (a guard taking its fallback 78/78,
  a watchdog crash read as zero) are harness bugs rather than design
  results. They are valuable as base rates for how quietly instruments fail,
  not as protocol.
- **The Claude Code fence finding is specific and checkable.** The Seatbelt
  sandbox binds Bash subprocesses. File tools are governed by permission
  rules instead. Any project here that enforces a path boundary (HCE's
  `test/`, `raw/` immutability) only through a Bash hook or sandbox has the
  same hole. The paper does not test Edit/Write tools separately. It
  describes the deny list as covering "tool-permission" rules generally.
- **Censoring cuts both ways for this repo's ceilings.** The paper treats
  halts as unresolved in a *measurement*. An autonomous chain that halts at
  `max_tokens` is doing *work*, so "unresolved" is the right label for its
  output, but the chain still has to act on it. The paper offers no rule for
  when to raise a cap beyond "until the censoring fraction is near zero,"
  which prices badly: one uncensored Sonnet episode priced a 26-episode read
  at roughly 218M tokens.
- **Disclosure.** The runs used Claude Code with Sonnet 5 and Opus 5 on a
  consumer subscription. Protocol documents and result files were written by
  "the author's assistant heads" under the author's direction and
  responsibility.
- **Open question.** No arm here is referee-scored *as it runs*. The
  brownfield (planted-defect) form, which is authored but not run, is the one
  that can attach a correctness verdict to a price, because a repair is a
  delta against a baseline measurable before the agent starts. That is the
  experiment that would make the cost premium interpretable.

## Trust signals

- **Credibility:** 3. A single-author arXiv preprint (cs.SE) with no
  affiliation in the author block; not peer-reviewed; no citations
  established. The author is an established formal-methods researcher, but
  the PDF states no institution, so that counts as a prior only. Raised by
  a **public repository with the protocol, harness and result records**
  (github.com/jyh/saltbench, confirmed to exist; the PDF itself names only
  the Salt method repo), every pin hashed in `harness/HASHES.txt`, a dated
  pre-registration with amendments, and unusually forthright negatives:
  failed predictions recorded with direction, the fence hole disclosed
  against all reported reads, the abstract corrected against its own table,
  and a placebo reading explicitly forbidden. Held at 3 because the
  population is tiny and authored by the treatment's author, there is no
  effect result, independent replication is absent, and several findings
  are harness bugs found in the author's own instruments.

## Follow-up

- **Relevance:** 4. No new concept, but it **corrects and extends
  [[concepts/budget-as-ceiling]]** with a new axis: the ceiling as a
  *censoring and sampling instrument* inside a measurement. Three mechanisms
  come with it: halt ≠ failure enforced at the verdict writer, arm-correlated
  caps biasing downstream samples, and a p90 from capped runs returning the
  cap. It adds the measured fence-probing failure to
  [[concepts/hce-evaluation]] on the exact harness this project runs, and
  supplies quiet-instrument-failure base rates plus a false-rejection data
  point to [[concepts/evidence-gated-completion]]. It is not a 5 because it
  anchors no concept canonically and its headline comparison is
  underpowered by its own account.
- **[[concepts/budget-as-ceiling]].** Guidance 3 ("hitting a ceiling is
  information … this search direction is exhausted") holds for the
  no-improvement counter but not for token or call caps. Under a verifier, a
  cap reading of "0 verified" is an absent measurement; the same episode had
  a complete proof 39 calls later. The "a halt says nothing about whether
  the work was correct" scope note is confirmed by measurement: 3/3 capped
  cells passed fully. The token-as-defensible-unit claim needs a tier
  caveat, since quota per token roughly doubled at the higher tier while
  token count fell.
- **[[concepts/hce-evaluation]].** This is lu2026meta's "make the wall
  structural" taken one step further: *verify the wall you built, in every
  tool's language, with a neutral probe*. It pairs with
  [[literature/papers/wang2026androids]] (red-team the evaluator) on the
  isolation side. The gate-penalizes-treatment finding belongs beside
  [[literature/papers/ray2026what]]'s point that an intervention visible to
  the agent changes the measured system.
- **[[concepts/evidence-gated-completion]].** The failed-boot class is a
  measured case for zhu2026claimreceipt's three-way verdict: 4 of 5 raw
  "failures" were never asked the question. Instrument finding 8 extends
  [[literature/papers/ning2026scores]]'s "a null needs a positive control":
  a positive control drawn from the same truncated population, here an
  unfetched remote, cannot detect the truncation, so an absence claim has to
  name the population it enumerated.
- **[[concepts/information-firewall]].** Two contributions. Authoring the
  population is a boundary against the weights, with a stated cost (no
  independent authorship). The staged release of ground truth (blind stage-A
  views, later views shipped only after every stage-A episode lands, verified
  by content set-hash) is a concrete example. The SWE-bench transcript is a
  third attestation of parametric contamination on a public software set. It
  does not close the concept's ML-research-task gap.
- **[[concepts/enforcement-boundary-placement]]** (optional, source-only).
  Rule 2 ("the constrained component cannot reach the constraining one")
  fails silently when the boundary covers one of several action paths. The
  Seatbelt-vs-file-tool split is the concrete case.
- **Companion.** arXiv:2608.21356, "AI with Authority, from Application to
  Silicon" (same author), is the source for the Salt method. Not ingested.
