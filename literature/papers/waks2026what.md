---
kind: paper
title: "What Output-Only Review Cannot Verify: Study Contracts for Research Agents"
authors: ["Eitan Waks", "Ben Glocker"]
institutions: ["Imperial College London (Department of Computing)"]
year: 2026
venue: "arXiv 2610.11754v1 (cs.AI), 2026-10-08; 'Preprint. Not peer reviewed.'; one of three companion manuscripts sharing a platform and audit records"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.11754"
code_url: null
citations: null
source: "raw/papers/waks2026what.pdf"
added: "2026-10-12"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/constraint-pinning]]"
tags: ["study-contract", "preregistration", "expected-run-manifest", "denominator", "survivorship", "claim-scope", "scope-ceiling", "waiver-policy", "human-authenticated-gate", "binding-drift", "information-asymmetry", "absence-faults", "llm-as-judge", "abstention", "self-authored-fixtures", "design-not-evidence", "objection-verdict-enforcement"]
---

# What Output-Only Review Cannot Verify: Study Contracts for Research Agents

## TL;DR

A threat model and a design for verifying research-agent output against an
**approved study** rather than against the write-up. The approved study is
an immutable, versioned object C_v = (Q, D, A, M, R, O, H, Π), bound by
digest to the compiled run configuration. The agent may propose revisions
and run work against it, but it cannot approve its own changes.

The empirical part is a small diagnostic on 8 self-authored clean/mutated
package pairs:

- A deterministic checker, given the approved contract and the registered
  rule for each fault, blocked 8/8 mutations with 0/8 false blocks.
- 18 LLM judge aliases saw one blinded package at a time, without the
  contract. They flagged 104/144 mutated cases and 1/144 clean controls.
  They never called a mutated package clean.
- Detection was bimodal by fault type. The six faults that leave a visible
  contradiction inside the package were flagged by 16–18 of 18 aliases. The
  two faults that consist of something *missing* (an undeclared split, a
  missing required metric artifact) were flagged by 2/18 and 1/18.

The paper reads **at** the digest framing, and it is an exception to this
graph's run of abstracts that overstate: the abstract carries its own
caveats, and the headline 104/144 is the pooled number, not a favourable
cut. The overreach is in the title, which says "cannot", where the paper
shows an analytic argument plus three flags on two fixtures. Ingest it as a
design. The schema is directly usable. The numbers are a sanity check on
the authors' own fixtures.

## Claims

- **Every current proxy is agent-authored or output-local.** Task
  completion, artifact presence, unit tests and LLM-as-judge on the output
  "need not encode the relation between the authorized study and what was
  executed". The claim is one-directional: completion does not entail
  validity. The authors say they have not measured the association.
- **Six silent-mutation channels (M1–M6)** are reachable "through
  ordinary, non-adversarial agent behaviour":
  - M1 design underspecification: split, comparator or seed policy left
    open and filled in after the data are seen;
  - M2 binding drift: plan v is approved, a later config edit runs
    something else, and a plan hash stored in the report does not help;
  - M3 semantic drift across translation: engineering edits and scientific
    edits travel through the same channel. Whether a batch-size change is
    in scope is "a declared property of the contract rather than a
    property of the parameter";
  - M4 survivorship drift: failed, timed-out or never-started units leave
    the denominator, and retries re-enter as fresh successes;
  - M5 scope inflation: a benchmark result becomes a population, transfer
    or clinical claim;
  - M6 reconstruction drift: the study cannot be re-derived without private
    chat or memory state.
  Whether a loop actually drifts into these is "an empirical question that
  this paper does not answer".
- **Four checkable properties.**
  - P1, study-level binding (M1–M3): approval records a digest over the
    spec, compiler identity and capability identity, and any post-approval
    change invalidates it.
  - P2, expected-run denominators (M4): a manifest is emitted before
    execution, and the ledger must account for every intended unit.
  - P3, typed claim promotion (M5): claims carry typed scope and
    obligations carry discharge and waiver types.
  - P4, human-authenticated gates: approval, waivers, data governance and
    promotion need out-of-band human identity. "Chat confirmation is not
    approval."
- **Some faults are unreachable from the output by construction (§3.2).**
  One package X lacks artifact A. Under contract C1, A is required; under
  C2 it is optional. A deterministic output-only rule must return the same
  verdict in both worlds. This is an argument, and the authors keep it
  separate from the diagnostic.
- **Freeze the adaptation policy Π, not the trajectory.** Leaving the
  envelope is an explicit amendment, or an exploratory fork with a
  downgraded claim status.
- **Export a study object, not only a paper object.** A meta-agent's keep
  rule can then be defined over contract-relative quantities (binding
  integrity, denominator completeness, admissible-claim yield at fixed
  scope) rather than judge scores on prose. The authors flag that any such
  signal becomes a new optimization target.
- **Objection, verdict and enforcement are different acts.** ClaimReceipt
  and output-only judges work at the first two. The contract adds the third
  and supplies the reference (approved denominator and claim ceiling) that
  makes a coverage question answerable at all.
- **What the paper says it does not show (§4.3).** It does not isolate
  verifier quality, compare agents, calibrate reviewers, demonstrate reward
  hacking, or test whether a stronger output-only model would do better.

## Methods

- **The contract schema.** C_v = (Q, D, A, M, R, O, H, Π):
  - Q: question and claim scope;
  - D: data identity;
  - A: arms;
  - M: metrics and decision rules;
  - R: expected-run manifest;
  - O: typed evidence obligations;
  - H: authority and approval state;
  - Π: authorized adaptation policy.

  Execution state references C_v through authenticated history and is never
  edited into it. Per Appendix D, each spec field carries provenance
  (`explicit`, `inferred`, `missing`).
- **Claim scope dimensions:** population, task and endpoint, data version
  and split, comparator, metric, time, site and domain.
- **Obligation fields.**
  - `discharge_method`: `mechanical` | `human_semantic`.
  - `waiver_policy`: `forbidden` | `scope_reducing` |
    `permitted_under_conditions`. The third has "no general condition
    language behind it", so an obligation declared under it stays
    unresolved.
- **Trust boundary (§1.1).**
  - Protected: the approved contract and its digest chain, gold labels,
    the authorization record and the checker predicate.
  - Assumed: observations are collected through a path the agent cannot
    rewrite.
  - Untrusted: the transcript, narrative, proposed edits and
    self-reported completion.

  "If an agent can edit the approved contract, the gold labels, the
  observations or the authorization record, the relation can be gamed."
- **Development corpus.** 8 clean/mutated study-package pairs (seed
  20260815), each mutation "a registered methodological fault", all
  authored by the paper's authors. The fault set was "not independently
  frozen" (Table 4). The fixtures are:

  | Fixture | Mutation | Flagged by (of 18) |
  |---|---|---|
  | compilation_drift_dev_001 | approval/compiler digest drift | 18 |
  | comparison_budget_001 | mismatched seed/budget/stopping | 17 |
  | external_validation_001 | overbroad external generalization | 17 |
  | reconstruction_dev_001 | reconstruction manifest incomplete | 17 |
  | runtime_contract_001 | material workload mutation | 16 |
  | integrity_digest_dev_001 | artifact digest mismatch | 16 |
  | split_policy_001 | missing split and grouping | 2 |
  | evidence_absence_dev_001 | missing required metric artifact | 1 |

- **Deterministic checker.** It received the approved study object and the
  executed package, and it applied "the registered, fault-specific rule for
  the mutation under test".
- **Judges.** 18 provider aliases: six OpenAI (gpt-4.1 to gpt-5.6-sol), six
  Gemini (3.1 to 3.7 flash/pro) and six Anthropic (haiku-4-5 to
  claude-fable-5).
  - Aliases were unpinned. Temperature was left at default for reasoning
    models and set to 0.0 otherwise. The evaluation window was about
    2026-08-15.
  - Each judge got one metadata-filtered package per case, never as a
    pair, without the contract or the fault label.
  - Some packages "retained approval and execution fields, including
    digests".
  - The prompt is frozen (SHA-256 given) and reproduced verbatim in
    Appendix B. Allowed outputs were `mutation_detected` | `clean` |
    `abstain`, plus confidence and issue ids. The prompt said: "Absence of
    evidence is not evidence of a defect … choose 'abstain'."
  - Scale: 16 cases per alias, 288 judgments, 309 attempt entries.
- **Historical audits.** Three shared fixtures, run locally at git
  `648338e` and human-approved "according to the retained records", which
  the paper "does not independently authenticate":
  - refusal audit Y2: 11/14 unauthorized transitions refused, three MCP
    cells `not_reached`, overall not passed;
  - denominator audit X3≡Y3: denominator 4, retries 2, passed;
  - scope-ceiling audit X4≡Y4: over-scope promotion blocked, largest
    admissible scope returned, overall not passed.

  These are shared with the companion papers, not replications.

## Results

- **Table 1** (288 cases):

  | Gold class | Cases | Defect flag | Explicit clean | Abstain | Terminal failure |
  |---|---|---|---|---|---|
  | Registered mutation | 144 | 104 | **0** | 32 | 8 |
  | Paired clean control | 144 | 1 | 74 | 63 | 6 |

- **The judges were specific but abstained a lot.** Decisions were 178/179
  correct. The one false flag came from gemini-3.1-flash-lite. Decision
  coverage was 179/288: 72% on mutated packages and **52% on clean ones**.
  Nearly half of all clean packages drew an abstention or a failure.
- **Where the misses went (our arithmetic from Tables 1–2).** Of the 40
  unflagged mutated cases, 33 belong to the two absence fixtures and only 7
  to the six visible ones. On absence faults the judges abstained or failed;
  they did not accept. 101 of the 104 flags come from the six visible
  fixtures.
- **Per alias (Table 3).** 13 of 18 aliases flagged exactly 6/8 mutations.
  Two flagged 7/8 (claude-fable-5, gemini-3.1-flash-lite). The low end was
  gemini-3.5-flash at 3/8, with 3 decisions out of 16.
- **Scope inflation is visible from the output.** 17/18 flagged the
  overbroad external claim. The authors conclude that for M5 the contract
  adds "not detection but the authorized ceiling … and the authority to
  block promotion."
- **Surface-heuristic sanity check.** A deterministic heuristic rewarding
  length, inflation vocabulary and thinner seed lists scored the mutated
  package above its clean twin on 2/8 pairs and tied on 3/8. The authors
  say this "demonstrates nothing about any optimizer".
- **Implementation status (Table 4).**
  - Implemented: the versioned spec with provenance, capability preflight
    and locked compilation for named paths, expected-vs-observed run
    accounting, typed claim scope, and waivers requiring out-of-band
    human authority.
  - Not implemented: general adaptive enforcement (early stopping,
    asynchronous observation, multi-backend), compiler-wired revision
    transitions, and conditional waivers.
  - Promotion is "not an ordinary-path blocking guarantee".

## Critique / open questions

- **The 8/8 checker result is a unit test, not a detection rate.** The
  checker was handed the registered rule for the fault under test, on
  fixtures the same authors wrote. That it fires on the fault it was
  written for, and not on a clean twin, shows the rules are implemented.
  It says nothing about recall on faults nobody registered. The paper
  says as much ("sensitivity … not evidence of"), but "detected all eight"
  will travel without that qualifier.
- **104/144 mostly encodes the fixture mix.** The per-fixture rates are
  about 17/18 or about 1.5/18, and nothing in between. The pooled rate is
  therefore close to (number of visible fixtures) ÷ 8. With four absence
  fixtures instead of two, the same judges would score about 50%. The
  effective n is 8 fixtures, not 144 cases, which the authors concede
  ("repeated evaluations of eight pairs").
- **Answer to the digest's question: there are clean controls, and they
  help the judges.** 1/144 false flags and 0 explicit clean verdicts on
  mutated packages make the judges near-perfectly specific *when they
  decide*. The weak point is coverage: 63 abstentions on 144 clean
  packages. Used as a gate, an abstaining judge either blocks about 48% of
  clean work (abstain = block) or passes 33/36 absence faults (abstain =
  pass). Neither the abstract nor the digest reports the clean row.
- **The prompt pulls both ways on the absence faults.**
  - Its defect list has eight bullets that map one-to-one onto the eight
    fixtures. That includes "a successful run whose required metric or
    evidence artifact is absent" and "a data split that is undeclared". The
    judges were primed on the taxonomy, so 16–18/18 on visible faults is a
    best case for output-only review.
  - The same prompt says "Absence of evidence is not evidence of a defect …
    choose 'abstain'". That is a direct instruction to abstain on exactly
    the two fixtures where judges abstained.
  - So the absence-fault result is partly prompt-induced. The paper does
    say judges "may appropriately abstain", which concedes the point.
- **The §3.2 argument is right, and it is the real contribution.** If two
  contracts disagree on whether artifact X is required, no function of the
  package alone can tell which applies. That needs no experiment. The
  diagnostic adds only that frontier judges abstained, rather than
  hallucinating a verdict, when the requirement was missing. Per-fixture
  outcomes on the clean twins of the absence pairs are not reported. If
  judges abstained there too, they behaved exactly as §3.2 predicts.
- **"Output-only" judges were not fully output-only.** Some packages kept
  approval and execution fields, including digests. The 18/18 on
  compilation drift is a judge reading a digest mismatch that the
  contract machinery put in the package.
- **The equal-information comparison is "proposed and has not been run".**
  No advantage of contract checking over judging can be attributed yet:
  the two instruments had different information and different task specs.
  The closed-loop question (does contract scoring improve useful compliant
  completion under optimization, or does it reward narrow, trivially
  admissible studies?) is also unrun.
- **The authority boundary is assumed, not shown.**
  - P4 needs authenticated out-of-band human identity. Yet the historical
    records' human approvals are not authenticated in the paper.
  - The refusal audit failed overall (11/14, three cells `not_reached`).
  - On a single-user box where the agent runs as the user, H has no
    out-of-band channel to sit in.
- **Reproducibility.** The platform is unnamed, no code or repository is
  released, and only a git short hash is given. The frozen prompt hash and
  the per-alias tables let the arithmetic be rechecked, not the run.
  Aliases were unpinned. The acknowledgment says LLM tools, including
  Cursor, assisted with "drafting, and critical review". The very dense
  hedging reads like it.
- **Overlap with companion papers.** Two companion manuscripts share the
  diagnostic and audit records. The authors ask that they not be counted
  as independent evidence, and this graph should honour that if the
  companions are ingested.

## Trust signals

- **Credibility:** 3. For the paper:
  - Imperial College London Department of Computing, with an established
    PI (Glocker) as senior author;
  - the claims are bounded unusually tightly to the evidence;
  - the clean-control row is reported;
  - the frozen prompt is reproduced with a hash, and per-alias and
    per-fixture tables are given;
  - limits are listed in a dedicated "does not show" section.

  Against it, holding it at 3 rather than 4:
  - a preprint with no code or artifacts;
  - fixtures, fault rules and labels are all self-authored, and the fault
    set was not independently frozen;
  - n = 8 pairs;
  - unpinned judge aliases;
  - shared records with two companion papers;
  - an LLM-assisted draft.

  The trust is in the calibration of the claims, not in the strength of
  the evidence.

## Follow-up

- **Relevance:** 4. It is the most complete schema this graph has for the
  "declared vs enforced: who signs off on a result" front. It gives a named
  tuple, a threat model with six channels, four checkable properties and an
  explicit trust boundary. It strengthens:
  - [[concepts/evidence-gated-completion]]: the gate needs a prospectively
    approved reference, and absence faults are invisible without one;
  - [[concepts/typed-claim-partition]]: typed claim scope, plus discharge
    and waiver types on obligations;
  - [[concepts/hce-evaluation]]: preregistration that is bound to the run
    by digest, with a counted denominator;
  - [[concepts/enforcement-boundary-placement]]: objection vs verdict vs
    enforcement, and P4's out-of-band authority.

  It is not a 5 because nothing is measured in a loop. It provides a design
  and a fixture check, not evidence that contract checking changes outcomes.
- **Cheaply adoptable for this box's experiment loop? Partly, yes.** A
  lightweight version is:
  1. **Freeze.** At README-hypothesis time, write `contract.yaml` with Q
     (claim scope ceiling), D (dataset path and DVC hash), A (arms), M
     (metric and decision rule), R (arm × seed manifest) and O (required
     artifacts, each mechanical or human). Commit it, and record its hash
     in README frontmatter.
  2. **Bind.** Before each run, `config.yaml` must match the contract's
     declared fields, or the contract carries a committed amendment.
  3. **Count.** At the end, a checker reads the run ledger and fails if any
     R entry lacks a terminal status (success, failed or timeout). Retries
     must not add units.
  4. **Ceiling.** The `result:` line may not claim beyond Q; otherwise the
     claim is downgraded, not blocked.

  This is maybe 100 lines of Python, and it turns this project's "README
  hypothesis written before the run" rule from a declaration into a check.
  It closes M1, M2 and M4.

  What is **not** cheap:
  - H. There is no out-of-band approval channel when the agent commits as
    the user, so a git hash gives tamper *evidence*, not prevention.
  - Π. Adaptive-policy enforcement is unimplemented even by the authors.
  - Semantic scope containment. The authors list it as unsolved.

  Copy P2 first. Of all the channels, a missing denominator is the one
  output-only review almost never catches here (1/18), and it is the
  easiest to check mechanically.
- **Convergent sources.**
  - [[literature/papers/zhu2026claimreceipt]]: coverage against a
    committed universe, with PASS/INVALID/INCONCLUSIVE verdicts. Waks
    positions itself as the enforcement layer on top.
  - [[literature/papers/wang2026research]]: closure conditions frozen
    before work, and a tally computed at read time.
  - [[literature/papers/tiwari2026assay]]: a model-free merge gate whose
    floor rests on where the key sits.
  - [[literature/papers/zhang2026double]]: pre-registration does not
    validate the scorer it names, and this paper does not change that.

  Cited prior evidence for the M-channels:
  - [[literature/papers/moukpe2026deltaml]] (DeltaML-Bench) for
    specification gaming;
  - [[literature/papers/apodex2026frontierchallenge]] for completion
    language as an unreliable proxy.
- **Candidates:**
  - Chen 2025, EviBound (arXiv 2511.05524): a pre-execution approval gate
    plus post-execution artifact verification. It is the closest prior
    design.
  - Yin et al. 2026, "Artifact-centered claim-aware observability"
    (arXiv 2608.18312).
  - Waks & Glocker companions, "Who signed off on this? Research contracts
    for AI science" and "Spend it where it counts: Triage for AI science".
    They had no arXiv ids at preparation time. Treat them as the same
    evidence base, not as replications.
