---
kind: paper
title: "When Passing Tests Hides Vulnerabilities: An Empirical Study of Silent Failures in Agentic Systems"
authors: ["Wenji Bai", "Muhammad Waseem", "Zeeshan Rasheed", "Jaakko Peltonen", "Pekka Abrahamsson"]
institutions: ["Tampere University (Faculty of Information Technology and Communication Sciences)"]
year: 2026
venue: "arXiv (cs.SE); preprint submitted to Elsevier"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.10548"
code_url: "https://doi.org/10.5281/zenodo.20831595"
citations: null
source: "raw/papers/bai2026when.pdf"
added: "2026-09-22"
relevance: 3
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/enforcement-boundary-placement]]"
tags: ["code-repair", "silent-failure", "security", "taxonomy", "qualitative-coding", "grounded-theory", "multi-agent", "observational", "construct-validity", "no-intervention", "missing-denominator", "software-engineering-agents"]
---

# When Passing Tests Hides Vulnerabilities: An Empirical Study of Silent Failures in Agentic Systems

## TL;DR

Seven agent frameworks (four multi-agent, three single-agent), all driven by
GPT-4o-mini at temperature 0.2, were run on 150 purposively sampled Python
security tasks from SecurityEval and CVEfixes. Of 1,050 target traces,
**1,030** were valid; a four-level pipeline (L0 parse, L1 "available
functional checks", L2 Bandit, L3 a hand-weighted exploitability heuristic)
flagged **252** patches that passed L0+L1 and failed L2 or L3; one author's
manual review confirmed **170**, discarding 82. Those 170 split
**Omission 82 (48.2%) / Introduction 52 (30.6%) / Inadequacy 36 (21.2%)**,
and Inadequacy — a defense visibly attempted but bypassable — carries the
highest mean L3 score (**17.7**, against Omission 10.7 and Introduction 8.0,
corpus mean 11.4).

Those three numbers are real and grep-verified. The title is not measured.
The paper **never reports how many of the 1,030 traces passed L0+L1**, so
there is no denominator and no silent-failure *rate* anywhere in it. "Passing
tests" is operationalized as "passes available functional checks, including
import resolution, structural correctness verification, and execution of
associated test cases" — and the paper never says how many tasks had
executable test cases at all versus only the import/structure checks. And
"confirmed" means a human read Bandit's output and the diff; **no exploit was
ever run**, L3 is regex pattern-matching over source/sink tokens with weights
the authors tuned on their own Iteration-1 candidates. The study is **purely
observational**: no gate, no intervention arm, no before/after.

## Claims

- **Silent failures are a structurally distinct defect class.** "In an
  ordinary bug, the code fails to produce expected behavior, and this failure
  is in principle detectable through functional testing. In a silent failure,
  by contrast, the code not only runs but produces the expected result; the
  deficiency lies in a non-functional dimension, namely the security property
  of the output, that the evaluation oracle was not designed to assess." The
  authors name the overarching theme **security-compliance decoupling**.
- **A three-part taxonomy with ten open codes**, derived by open → axial →
  selective coding over the 170 confirmed cases: Omission (a required control
  is absent), Inadequacy (a control is attempted but incomplete/bypassable),
  Introduction (repair injects a vulnerability that was not there).
- **Attempted-but-incomplete defense is the riskiest class** *by the paper's
  own L3 instrument*: Weak Mitigation's 17.7 "challenges the intuition that
  any defense is better than none," and "a bypassable defense can lead
  downstream reviewers to assume that the issue has been closed and may
  therefore carry greater practical risk than an absent one."
- **Test-passing evaluation would have scored every one of these as a
  success.** "Industry-standard benchmarks such as SWE-bench and HumanEval
  treat test-passing as the primary success criterion. Under these criteria,
  every confirmed case in our corpus would have been classified as a
  successful repair." **This is a definitional restatement, not a
  measurement** — see the critique.
- **Reviewer roles do not intercept.** "In the confirmed cases we reviewed,
  the Reviewer rarely flagged the relevant security deficiency before it
  reached the final output. This pattern appeared across the multi-agent
  frameworks in our corpus, despite their differing coordination
  architectures." The three most frequent multi-agent propagation patterns
  all end in reviewer non-detection.
- **Defects originate upstream and survive handoffs.** "A plausible
  interpretation is that agent handoffs preserve functional intent more
  reliably than security intent: once a security control is omitted or
  weakened at design or implementation time, subsequent roles rarely
  reconstruct the missing requirement in our cases."
- **Architecture shapes the failure profile, but not its existence.**
  Multi-agent systems skew toward Introduction (41/115, 35.7% vs 11/55,
  20.0%), single-agent toward Inadequacy (16/55, 29.1% vs 20/115, 17.4%);
  Omission is essentially flat (47.0% vs 50.9%).
- **Cross-framework convergence is reported and then explicitly
  disclaimed.** 107/170 (62.9%) of confirmed failures are "convergent" —
  same task, same confirmed CWE, three or more frameworks — but "because all
  experiments used the same base model and related task prompts, this
  convergence should not be attributed to framework architecture alone."
- **The verification boundary is too narrow, not just too shallow.** Multi-
  agent pipelines emit config files, build scripts and test scaffolding
  outside the prompted target; Introduction concentrates there (32/52 of
  Introduction cases sit in Configuration). "Vulnerability analysis of agent
  output should enumerate and scan every artifact the pipeline produces."

## Methods

- **Tasks.** SecurityEval (130 prompts → 121 Python retained) and CVEfixes
  (5,365 fixes → 1,115 Python single-file fixes with CWE annotations); pool
  1,236. **150 tasks purposively sampled** in three iterations of 50 (85 from
  SecurityEval, 65 from CVEfixes), each iteration steering the next. The
  authors state the aim plainly: the strategy "was intended to support
  progressive coverage of silent-failure patterns rather than statistical
  representativeness."
- **Frameworks.** Multi-agent: MetaGPT, ChatDev, AutoGen, AgentCoder.
  Single-agent: Mini SWE-Agent, Aider, OpenHands. All on **GPT-4o-mini,
  temperature 0.2**, 600 s timeout, on a 5 GB A100 MIG slice, 8 CPU cores,
  17.5 GB RAM. Prompts held identical in content, adapted per input schema.
  "No additional security prompts were injected unless the task itself
  explicitly implied a security requirement."
- **Four-level verification.** L0 parses; **L1 = "passes available functional
  checks, including import resolution, structural correctness verification,
  and execution of associated test cases"**; L2 = Bandit, failing on any
  HIGH-severity finding mapped to a CWE via a lookup table; L3 = a scalar
  heuristic `R` summing weighted user-input-source tokens and dangerous-sink
  tokens, ×1.5 when a source co-occurs with a sink, ×1.3 when the sink's CWE
  matches the task's target CWE, +10 if the original vulnerable pattern is
  still present, minus a bounded discount of up to 10 for detected
  safe-practice categories, clipped to [0, 100]. **Threshold 15**, with
  weights and threshold "based on calibration with Iteration 1 candidates."
- **Candidate definition.** `SF_candidate = (L0 pass ∧ L1 pass) ∧ (L2 fail ∨
  L3 fail)`. 20 traces excluded for timeout/crash/incomplete output → 1,030
  valid; **252 candidates; 170 confirmed after manual review**.
- **Confirmation.** Three labels — True Positive (clear instance of the
  target vulnerability), **Partial** (a relevant mitigation is visibly
  attempted but incomplete, with "a plausible exploit path still present in
  the code"), False Positive. TP and Partial are both retained. **All manual
  review and all three coding passes were done by the first author.**
- **Reliability.** A random 35 cases (about 20% of the corpus) were
  independently recoded by a second author. Composite D0×D1 Cohen's
  **κ = 0.71**, with D0 (triage verdict) **0.63** and D1 (axial category)
  **0.66**. All disagreements resolved by discussion; the codebook was
  updated and earlier judgments retrospectively revised.
- **Artifacts.** The execution traces, generated patches, verification
  results and taxonomy codes are deposited on Zenodo
  (doi:10.5281/zenodo.20831595). No code URL for the pipeline itself is given
  in the text beyond that deposit.

## Results

### The taxonomy and its distribution (RQ1)

- **Omission 82/170 (48.2%)**, mean L3 10.7. Open codes: Insecure API
  Preserved **42 (24.7%, L3 16.7)** — the single largest code in the corpus,
  where "agents often applied cosmetic adjustments such as renaming variables
  or adding length checks, while leaving the vulnerable call intact"; Missing
  Mitigation 22 (12.9%, L3 5.8); Deployment Context Risk 9 (5.3%); Missing
  Sanitization 4 (2.4%); Unsafe Default Retained 4 (2.4%); Ambiguous Repair
  Intent 1 (0.6%).
- **Inadequacy 36/170 (21.2%)**, a single open code, **Weak Mitigation**,
  mean L3 **17.7**, max L3 **100.0**. Examples: path-traversal prevention via
  string comparison rather than a path-aware API; a command allowlist beside
  an injection-prone execution mode; a sandbox whose filtering rules were "too
  permissive to prevent known escape techniques."
- **Introduction 52/170 (30.6%)**, mean L3 8.0. Introduced New Vulnerability
  41 (24.1%, L3 6.9); Hardcoded Secret 10 (5.9%, L3 10.4); Vulnerable Test
  Code 1 (0.6%, L3 **29.0** — the highest single-code mean, from n = 1).
- Two codes carry nearly half the corpus: Insecure API Preserved + Weak
  Mitigation = **78/170 (45.9%)**.

### "Riskiest class" depends on which instrument you read

- By **mean L3 heuristic**: Inadequacy 17.7 > Omission 10.7 > Introduction
  8.0 (corpus 11.4). Max L3: Inadequacy 100.0 > Omission 60.8 > Introduction
  58.0.
- By **coder-assigned severity** (Table 9, a subjective rating from the
  coding procedure, not CVSS): Omission holds **17 of the 22** Critical cases
  and 31 of 53 High; Inadequacy holds 5 Critical and 17 High; Introduction
  holds **zero** Critical and is 88.5% Medium (46/52). Total severities:
  22 Critical / 53 High / 94 Medium / 1 Low.
- So "Inadequacy is the riskiest class" is true of the *mean exploitability
  heuristic on the smallest category*, and false of the critical-severity
  count. The paper states both and does not reconcile them; it is careful to
  say "small differences between adjacent categories, frameworks, or open
  codes should not be interpreted as stable rankings."

### Propagation and architecture (RQ2)

- Per-framework confirmed counts (Om/Inad/Intro): AutoGen 39 (25/9/5),
  MetaGPT 30 (11/4/15), ChatDev 27 (12/2/13), AgentCoder 19 (6/5/8), MiniSWE
  21 (8/4/9), Aider 19 (12/6/1), OpenHands 15 (8/6/1). Multi-agent 115,
  single-agent 55.
- Propagation patterns (Om/Inad/Intro): Planning-Stage Omission Cascade
  22/9/0; Design-Originated Omission 22/6/1; Design-Originated Introduction
  0/0/27 (**all** of it ChatDev and MetaGPT); Direct Implementation, no review
  10/4/6; QA-Involved 0/1/1; Implementation-Originated 0/0/6; Single-Agent
  Direct Generation 28/16/11.
- Convergence: **26 tasks** produced a same-CWE silent failure in three or
  more frameworks, accounting for **107/170 (62.9%)** of confirmed cases; on
  **four** tasks, six of seven frameworks converged.

### Location and family (RQ3)

- Code location: Input Handling 61 (35.9%), Configuration 37 (21.8%), API
  Call 35 (20.6%), Business Logic 19, Authentication 9, Other 9. The alignment
  is sharp: **Inadequacy is 33/36 in input handling; the API Call row is
  100% Omission (35 Om / 0 Inad / 0 Intro); Introduction is 32/52 in
  configuration and 0 in API calls.**
- CWE family: Omission dominates API-substitution families (weak crypto 8/9,
  deserialization 7/8, XXE 7/8, command injection 5/6); Inadequacy clusters in
  logic-intensive families (XSS, path traversal); Introduction owns info
  disclosure (100%) and open redirect.

### The paper's own nulls, caveats and non-measurements

- **No L1 pass rate, anywhere.** Patches failing L0/L1 are defined as
  "explicit failures" and then never counted. There is no base rate against
  which 170 can be read.
- **The corpus is explicitly a lower bound, not an estimate.** "False
  negatives are inherently difficult to quantify; the confirmed corpus should
  be interpreted as a conservative estimate of the silent failures detectable
  by our multi-level verification framework, not as an estimate of all
  possible silent failures in agentic code repair."
- **A Partial-only sensitivity check is run and reported.** 39/170 (22.9%)
  are Partial, 131 (77.1%) TP. Re-deriving the taxonomy on the 131 TPs alone
  keeps all three axial categories and seven of ten open codes; the three lost
  codes (Deployment Context Risk, Ambiguous Repair Intent, Vulnerable Test
  Code) "occurred only as Partial cases," "since their exploitability could
  not be confirmed from static evidence alone."
- **The L3 threshold is acknowledged as a screen that can miss.** "A true
  silent failure scoring below the threshold would not enter manual review and
  would be missed."
- **Model effects are explicitly unseparated.** "Because our design fixes the
  base model and varies the frameworks, it cannot separate model-level effects
  from shared factors such as prompt content, dataset characteristics, and
  recurring vulnerability patterns."
- **Reproduction is not guaranteed.** "Framework execution involves
  stochastic LLM inference, meaning that exact trace reproduction is not
  guaranteed." No repeats or seeds are reported; each task–framework pair is
  run once.
- **The study is descriptive by design.** "The analysis is primarily
  qualitative and descriptive... we avoid drawing causal or population-level
  conclusions from descriptive differences alone."
- **An ablation named and not run.** AgentCoder is selected specifically to
  "test whether agent-generated tests mask vulnerabilities" (Table 2), but no
  analysis anywhere addresses that question. AgentCoder appears once more, as
  a bar in Figure 5.
- **The remedy is deferred to future work.** "Future work should... test
  whether targeted verification strategies meaningfully reduce silent failure
  rates in deployment settings."

## Critique / open questions

- **The title's mechanism is asserted by construction, not measured.** A case
  enters the corpus only if it passed L0+L1 and was flagged at L2/L3. So "a
  test-passing patch hides a vulnerability" is the *selection criterion*, and
  "every confirmed case... would have been classified as a successful repair"
  under SWE-bench-style scoring is a restatement of that criterion. The
  sentence reads as a finding and is a tautology over a
  selected-on-the-outcome sample. What would make it a finding is a rate —
  of the patches that passed L1, what fraction were flagged and confirmed? —
  and that number is not in the paper.
- **"Passing tests" may be much weaker than the title implies.** L1 is
  "available functional checks, including import resolution, structural
  correctness verification, and execution of associated test cases." Neither
  SecurityEval (described here as "130 predefined code generation prompts")
  nor the CVEfixes single-file-fix filter is described as shipping an
  executable test suite, and **the paper never reports how many of the 150
  tasks had runnable tests versus only import/structure checks**. The one
  worked example does mention tests ("the available tests used only benign
  inputs"), so tests existed somewhere; the distribution is unknown. Until
  that split is published, an unknown share of the 170 may be "imports
  resolve and the file parses hides a vulnerability" rather than "a passing
  test suite hides a vulnerability."
- **"Confirmed" is one reader plus a regex score, with no exploit.** L3 is
  pattern matching over source/sink tokens with hand-set weights (sinks 6–12,
  sources 2–4, a 1.5 co-occurrence multiplier, a 1.3 CWE-match multiplier,
  +10, −min(2·n_safe,10)) and a threshold of 15 calibrated on the authors'
  own first-iteration candidates. It estimates exploitability; it never
  demonstrates it. No proof-of-concept, no dynamic analysis, no runtime
  check — and the threats section concedes that "security properties
  requiring dynamic analysis, runtime monitoring, environmental
  configuration, or domain-specific knowledge may not be captured." All 170
  judgments were made by the first author, with 35 cases (20%) double-coded
  at κ = 0.71 composite / **0.63 on the triage verdict itself**. A κ of 0.63
  on TP-vs-Partial-vs-FP means the second coder disagreed on roughly a third
  of triage decisions before discussion. The `170` is therefore a
  *static-analysis-flagged, single-reader-endorsed* count, not an exploited
  one, and 39 of them are `Partial` by the authors' own admission of weaker
  evidence.
- **No denominator, purposive sampling, one model, one run.** The sampling is
  explicitly non-representative, the model is a single cheap one
  (GPT-4o-mini — arguably the weakest plausible choice for security
  reasoning, which inflates all failure counts relative to a frontier model),
  and each task–framework cell is executed once with no seeds. Nothing here
  supports a prevalence claim, and the authors mostly avoid making one —
  though the abstract's framing invites the reader to.
- **The abstract's claim (iii) is scoped by a clause that removes its
  force.** "Current test-passing evaluation and LLM-based reviewer roles were
  insufficient to expose or intercept these failures **in the confirmed
  cases**." Since the confirmed cases are defined as the ones that got past
  those checks, this cannot be otherwise. The reviewer-role result would need
  the complement — how many security deficiencies a reviewer role *did*
  catch, as a fraction of those present — and no such number exists here.
- **Introduction is a different phenomenon wearing the same label.** 52/170
  are vulnerabilities the agent *added*, mostly in config and scaffolding that
  the task never named, 88.5% rated Medium and none Critical. Only
  Omission + Inadequacy = **118/170 (69.4%)** are cases where a checkable
  pass-signal coexists with an *unmet requirement*, which is the pattern this
  repo cares about. Quoting 170 for that pattern overstates by ~44%.
- **A within-paper inconsistency worth naming.** The Weak-Mitigation risk
  ordering rests on mean L3 across a 36-case group whose max is 100.0, i.e. a
  mean plausibly driven by a tail, and the paper reports no dispersion (no
  SD, no median, no CI) for any L3 mean. The same paper cautions that "small
  differences between adjacent categories... should not be interpreted as
  stable rankings" — a caution that applies to 17.7 vs 10.7 as much as
  anywhere.
- **Positives, stated fairly.** The construct is clearly operationalized and
  its limits are drawn in unusual detail; the Partial-only sensitivity
  re-derivation is exactly the ablation a skeptical reader wants and it was
  run; the convergence result is reported *and* then explicitly refused as
  evidence for an architectural cause; the location×category separation
  (Inadequacy 33/36 in input handling, API Call 35/35 Omission,
  Introduction 32/52 in configuration) is a genuinely clean structural signal
  that a purely narrative taxonomy would not produce; and the full corpus is
  deposited publicly so the 170 judgments can be re-audited by anyone who
  disagrees with them.
- **Open question the paper names and does not answer.** "How the set of
  at-risk artifacts scales with pipeline complexity remains unquantified."

## Trust signals

- **Credibility:** 3. A competent, self-aware empirical-SE preprint from a
  known group (Tampere; Abrahamsson's lab), submitted to an Elsevier journal
  but **not peer-reviewed**, with the full corpus — traces, patches,
  verification results, taxonomy codes — deposited on Zenodo under a DOI, a
  threats-to-validity section that concedes most of what a critic would
  raise, and a Partial-only sensitivity re-derivation actually run. Held at 3
  rather than 4 by the instrument and the staffing: exploitability is a
  hand-weighted regex heuristic calibrated on the authors' own first
  iteration with **no exploit ever executed**; all 170 confirmations come
  from a single coder with 20% double-coding at κ = 0.63 on the triage
  dimension; one cheap model, one run per cell, purposive non-representative
  sampling, and no L1 pass rate reported anywhere, so the central count has
  no denominator. Not lower than 3 because the artifact release and the
  candour about scope are real and checkable.

## Follow-up

- **Relevance:** 3. Cite-worthy prior art on an active thread, not a concept
  mover. Domain-wise it sits in the part of software-engineering-agent work
  this project keeps at arm's length (ChatDev/MetaGPT/AutoGen doing code
  repair), and it earns its place only through the mechanism: it is the
  code-execution analogue of [[literature/papers/nepal2026faithful]]'s
  checkable/unverifiable split, with the *checkable* signal being a pass
  result rather than a countable prompt rule. It does not score 4 because it
  supplies no denominator, no intervention and no behavioural delta — exactly
  the three things [[concepts/evidence-gated-completion]] has been waiting
  for — and because its confirmation instrument stops at static analysis.
- **[[concepts/evidence-gated-completion]] — record, and explicitly do NOT
  lift the hold.** This paper is **purely observational**: no gate is built,
  no arm is withheld, nothing is measured before and after. Its own future
  work asks someone else to "test whether targeted verification strategies
  meaningfully reduce silent failure rates in deployment settings." So the
  hold stands, for the fifth cycle, on the same grounds as
  nepal2026faithful. What it *does* add, and the only thing worth appending:
  a second domain instance of the **hiding-proxy** row, in the form the
  concept's table already uses — failure mode *unmet security requirement*,
  hidden by *a functional pass signal*, minimum evidence *a security-semantic
  check over every generated artifact, not just the prompted file*. And one
  sharper sub-claim the concept does not currently hold: **a partially
  satisfied requirement is worse than an unsatisfied one at the gate**,
  because the visible mitigation is itself a soft signal that a downstream
  reviewer reads as the requirement being met (Weak Mitigation, 36 cases,
  highest mean L3 at 17.7). That is a mechanism for why `PASS` is the
  dangerous default resolution of missing evidence — the argument
  [[literature/papers/zhu2026claimreceipt]]'s three-way verdict makes
  abstractly, here with cases attached. Caveat the append heavily: the
  denominator is absent, "confirmed" stops at Bandit plus one reader, and
  only 118 of the 170 are unmet-requirement cases at all.
- **[[concepts/programmable-evaluator-oracle]] — record and decline to
  edit.** The paper's actual thesis is an oracle-contract point the concept
  already holds: the deficiency "lies in a non-functional dimension... that
  the evaluation oracle was not designed to assess." The concept's tail
  already covers what the evaluator returns and what it rewards. The one
  genuinely new angle is **oracle scope over the artifact set** — multi-agent
  pipelines emit config, build files and scaffolding outside the prompted
  target, and Introduction concentrates there at 32/52 in Configuration, so
  an oracle scoped to the named file is blind by construction. But the
  supporting evidence is a single-model, single-run, purposively sampled
  corpus with no denominator, and the finding is confounded with "ChatDev and
  MetaGPT emit more files." Worth revisiting if a paper measures
  scanned-artifact coverage against pipeline complexity, which this one
  explicitly leaves "unquantified."
- **[[concepts/hce-evaluation]] — record and decline to edit.** Tempting,
  because "the same scorer measures something other than what the headline
  implies" is this concept's territory. But the paper's construct-validity
  lesson is undercut by its own construct: it cannot say how much of its L1
  was a real test suite versus an import check, so it is not a clean
  demonstration that a *passing test suite* misreads security — it is a
  demonstration that a pipeline whose strongest check is Bandit finds things
  Bandit finds. [[literature/papers/shao2026language]]'s 24.0→57.0 spread on
  one fixed corpus remains the cleaner artifact for this point. Revisit if
  the Zenodo deposit turns out to record per-task L1 provenance.
- **[[concepts/enforcement-boundary-placement]] — record, weak support, do
  not edit yet.** The paper's design implication is a boundary-placement
  claim: "assurance mechanisms should move upstream: because silent failures
  can originate during planning, design, or implementation and persist
  through subsequent handoffs, tool-supported checks should be deployed where
  defects first arise rather than only at the final review stage." The
  supporting datum is that the two largest multi-agent propagation patterns
  originate at Planner (31 cases) or Architect (29 cases) and terminate in
  reviewer non-detection. But originator attribution is qualitative
  role-reading by one coder, the paper itself lists "alternative
  interpretations of complex agent interactions could lead to different role
  assignments" as an internal-validity threat, and no upstream check was
  placed or tested. This is a hypothesis with a case series, not a placement
  result.
- **Digest candidates from its bibliography.** Three are directly on this
  thread and none is in the graph: **Chen, He, Jana & Ray, "Red teaming
  program repair agents: When correct patches can hide vulnerabilities"**
  (arXiv 2509.25894) — the adversarial counterpart, and the closest thing to
  the exploit-based confirmation this paper lacks; **Wang et al.,
  "VulnRepairEval: An exploit-based evaluation framework for assessing LLM
  vulnerability repair capabilities"** (arXiv 2509.03331) — an
  exploit-gated oracle, i.e. the measured version of L3 and the strongest
  single candidate; and **Mahmud, Rawajfih & Wu, "A systematic evaluation of
  AI-generated security patches"** (IEEE AITest 2025), which reports
  AI-generated patches relying on minimalistic input validation and remaining
  exploitable under systematic bypass testing — the Inadequacy finding with
  actual exploits behind it. Secondary: Zhang et al., "Which agent causes task
  failures and when?" (ICML 2025) and AgenTracer (ICLR 2026) are the
  attribution frameworks this paper positions against.
