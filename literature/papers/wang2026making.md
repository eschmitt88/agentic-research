---
kind: paper
title: "Making AI Scientists Auditable from Evidence to Claim"
authors: ["Zijian Wang", "Hanqi Li", "Ziyue Yang", "Zijian Hu", "Shenghan Zuo", "Yunzhe Zhang", "Da Ma", "Danyu Luo", "Chenrun Wang", "Jing Peng", "Tiancheng Huang", "Sijia Guo", "Huayang Wang", "Zichen Zhu", "Senyu Han", "Yilu Cao", "Bo Chen", "Xin Chen", "Kai Yu", "Lu Chen"]
institutions: ["X-LANCE Lab, School of Computer Science, Shanghai Jiao Tong University", "Shanghai Innovation Institution", "Suzhou Laboratory"]
year: 2026
venue: "arXiv 2606.18874v4 (cs.AI), revised 2026-09-27; self-described technical report"
peer_reviewed: false
url: "https://arxiv.org/abs/2606.18874"
code_url: "https://github.com/OpenDFM/Xcientist"   # harness code released (57 stars at ingest); no path in the repo tree matches audit/drift/claim, so the CDR audit pipeline and audit records appear unreleased (path-name check only)
citations: null
source: "raw/papers/wang2026making.pdf"
added: "2026-10-06"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/evidence-gated-completion]]"
tags: ["claim-drift", "auditability", "evidence-graph", "validation-contract", "worker-validator", "component-attribution", "ablation-coverage", "unverifiable-vs-drifted", "retrospective-audit", "llm-judge-audit", "ai-scientist", "provenance", "claim-boundary"]
---

# Making AI Scientists Auditable from Evidence to Claim

## TL;DR

XCIENTIST is an end-to-end AI-scientist harness: literature review, MCTS
idea search over editable components, contract-gated experiment
validation, and report writing. It is built on a ~50K-paper evidence graph
whose records carry verbatim source quotes. The paper's contribution for
this graph is the **audit vocabulary**, more than the harness:

- **Claim drift**: an accepted claim that is still presented as supported
  while a required relation to its basis fails.
- **Three relations**: evidence-to-idea, idea-to-implementation and
  experiment-to-claim.
- **Three outcomes**: supported, drifted (a confirmed mismatch) and
  unverifiable (the evidence cannot settle it).

The headline "audit CDR" of 3.6–16.7% is (drifted + unverifiable) / N, and
the abstract itself says most of the gap to the three comparison systems
comes from **fewer unverifiable records**. On confirmed drift alone, the
fixed-budget audit does not separate XCIENTIST from EvoScientist: 3/362
(0.8%) vs 1/111 (0.9%). The supported reading is "XCIENTIST's records are
easier for the authors' own audit pipeline to resolve." "XCIENTIST drifts
less" rests on a single-LLM expanded review that a 20-record human check
found erring only against the comparators.

## Claims

- **Definition.** "We define claim drift as an unresolved mismatch between
  a scientific claim and its attributed evidential or experimental basis
  during research." "A new hypothesis, a negative result or an explicitly
  documented revision is not itself drift." Formally, c drifts at stage t
  iff c is accepted and some required relation r(c, B_t(c)) = 0.
- **Missing evidence is separated from mismatch.** "A confirmed mismatch
  takes precedence over a missing relation." "Missing, unread or truncated
  material was not treated as proof that an experiment had not occurred."
- **Headline.** Audit CDR is lower than EvoScientist, InternAgent-1.5 and
  AI-Scientist-v2 "in all nine task-by-relation comparisons". "Most of
  this difference arose from fewer unverifiable claims. The main observed
  advantage was therefore that the retained records more often allowed
  readers to recover and check the support for a claim."
- **Contracts, not narration, gate stages.** "A textual progress summary
  alone cannot satisfy a completion condition." Convergence requires a
  validator PASS for every phase, no unresolved blockers, and "all
  canonical components must have corresponding ablation evidence." "A
  phase marked partial or blocked is never accepted as sufficient,
  regardless of textual justification."
- **A switch is not a test.** "A component-disable switch makes a
  comparison possible; an executed, matched comparison is still required
  to support a contribution claim."
- **The authors hedge their own claims.** The evaluation "does not isolate
  the contribution of the evidence graph, component representation or
  validation contracts to the observed CDR differences." The findings "do
  not estimate claim-drift incidence across independent research runs."

## Methods

- **Evidence graph.** About 50K Semantic Scholar CS papers are parsed in
  full text with MinerU. Schema-bound extraction runs in three passes.
  Each field is a (keywords, summary, insight, **quote**) tuple, and the
  quote "anchors the record to a verbatim passage". Baseline and dataset
  entities are merged only when they resolve to the same Semantic Scholar
  Paper ID. Result: 72K core, 250K baseline and 63K dataset nodes, and
  about 1.15M typed edges.
- **Idea generation.** MCTS over component edits (add, remove, replace,
  rewire, add a validation protocol), run under several "Idea Taste
  Modes". It uses a vector memory of edit patterns and a symbolic memory
  derived from earlier ablation results. Fusion is host-led: one core
  mechanism, with other components kept only as support, protocol or
  guardrail. A repair step is accepted only if the composite score rises.
- **Experiment validation.** Stages are prepare (5 sub-stages), code,
  standard science and ablation science. Each stage has a contract (goals,
  input paths, permitted write roots, required outputs, done condition). A
  worker acts and a separate validator judges the artifacts. A master
  scheduler checks four gates in priority order. The first is a static
  self-containment scanner that forces code repair if any import reaches
  outside `project/`. The number of ablation steps must equal the number
  of canonical components.
- **Audit protocol (§5.6–5.7).**
  - An unnamed language model extracts deduplicated "accepted core claims"
    from each system's designated research endpoints.
  - BM25 plus explicit file references build an evidence bundle for each
    claim.
  - An unnamed LLM judge assigns the relation status and supporting
    quotes.
  - Programmatic checks verify evidence IDs and quotes, reject support that
    is "solely … a report repeating its own assertion", require execution
    evidence (not only code) for design identity, and recompute numeric
    claims.
  - The CDR judge prompt is not among the Appendix H prompt contracts.
- **Expanded review.** gpt-6-astra with web access re-assessed all 198
  initially unverifiable records.
- **Human review.** 20 records were "selected" (5 / 7 / 8 by relation),
  with no stated selection rule. Reviewers could retrieve evidence beyond
  the automated bundle.
- **Comparison runs.** One retained trajectory per system per task, on
  three tasks: LoCoMo memory starting from A-Mem, PEMS-BAY forecasting
  starting from Graph WaveNet, and multiscale-heat PINNs. The paper does
  not say how the comparison systems were run (versions, backbone models,
  budgets). The backbone LLM of XCIENTIST itself is also unnamed.

## Results

- **Table 3, audit CDR (D, U, N):**

  | Task | Relation | XCIENTIST | Comparators (range) |
  |---|---|---|---|
  | Memory | evidence→idea | 11.6% (1,4,43) | 50.0–87.5% |
  | Memory | idea→impl | 4.1% (0,2,49) | 41.7–100% |
  | Memory | exp→claim | 5.8% (0,4,69) | 10.0–47.6% |
  | Graph-TS | evidence→idea | 16.7% (0,6,36) | 57.1–87.5% |
  | Graph-TS | idea→impl | 8.6% (0,3,35) | 81.3–100% |
  | Graph-TS | exp→claim | 3.6% (1,0,28) | 11.1–38.1% |
  | PINN | evidence→idea | 8.3% (0,2,24) | 37.5–88.9% |
  | PINN | idea→impl | 7.9% (1,2,38) | 37.5–77.3% |
  | PINN | exp→claim | 7.5% (0,3,40) | 36.8–50.0% |

- **Pooled, our arithmetic from Table 3:**

  | System | Records | Confirmed drift D/N | Unverifiable U/N |
  |---|---|---|---|
  | XCIENTIST | 362 | 3 (0.8%) | 26 (7.2%) |
  | EvoScientist | 111 | 1 (0.9%) | 56 (50.4%) |
  | InternAgent-1.5 | 111 | 6 (5.4%) | 65 (58.6%) |
  | AI-Scientist-v2 | 122 | 7 (5.7%) | 51 (41.8%) |

- **Expanded review (Table 4).** Of XCIENTIST's 26 unverifiable records, 1
  became drifted, 22 supported and 3 stayed unresolved. For the comparators
  combined, 34 of 172 (19.8%) became drifted, with a per-system range of
  14.3–31.4%. Our arithmetic for drift after expansion: XCIENTIST 4/362
  (1.1%), EvoScientist 9/111 (8.1%), InternAgent 16/111 (14.4%) and
  AI-Scientist-v2 23/122 (18.9%).
- **Human review.** 14/20 automated labels were retained. **All six
  changes went from a worse label to "supported", and all six were
  comparator records:** Cases 12, 17, 18, 19 and 20 (unverifiable →
  supported) and Case 02 (EvoScientist, drifted → supported). Every
  XCIENTIST record kept its automated label.
- **Task results** (single runs, final version chosen from the version
  history on the same evaluation set):
  - Memory: LoCoMo subset overall F1 rose from 0.306 to 0.391 over 13
    versions (v3.2 reached 0.388). Tokens per query fell from 2844 to 1017
    (−64.2%). Matched ablations were missing for some components.
  - Forecasting: PEMS-BAY average MAE went from 1.588 to 1.556 (−2.0%) and
    60-min MAE from 1.972 to 1.908. The hard orthogonal projection, which
    search had selected, moved MAE by only 0.0019 and was replaced by
    "innovation coverage" (removing it costs +0.0042 MAE).
  - PINN: best mean rel-L2 on heat-1D (0.067) and PINNACLE heat (0.431).
    On heat-2D it is worse than MMPINN (0.007), Multiscale PINNs (0.026)
    and its own earlier v5 (0.107). Average rank went from 6.33 to 2.33.
- **XCIENTIST's own drifts, all in human review.** All are of the
  inference kind:
  - Case 08: a non-clean "removal" of the coarse backbone was claimed as
    proof that it is essential.
  - Case 15: a range reported as 1.06–1.75% recomputes to 1.05–1.76%.
  - Case 16: a better endpoint error was claimed as a training-stability
    benefit.

## Critique / open questions

- **The digest framing is narrower than the paper's own abstract.** The
  digest says "claim drift of 3.6–16.7%". The paper says "audit claim
  drift rates", adds that the measure "includes both confirmed mismatches
  and unverifiable claims", and says "most of the difference arose from
  fewer unverifiable records". On **confirmed drift** (D/N) per cell,
  XCIENTIST is:
  - strictly lowest in **1 of 9** cells (Graph-TS evidence→idea);
  - tied at zero in 5;
  - **higher** than at least one comparator in 3 (Memory e→i 1/43 vs 0/6
    and 0/4; Graph-TS exp→claim 1/28 vs 0/18 and 0/21; PINN i→i 1/38 vs
    0/8 and 0/11).

  The counts are tiny, so this is no evidence that XCIENTIST drifts more.
  It does mean "lower in all nine cells" holds only for the composite.
  This matches the running pattern (now 32/32): the headline is the most
  favourable cut, here the metric component the system was designed to
  minimise. Unlike most cases, the paper discloses this itself.
- **The audit instrument favours the format of the system under test.**
  - The audit pipeline is the authors'.
  - XCIENTIST's records "also linked selected idea documents to
    experimental workspaces", and idea documents could "additionally
    supply" evidence-to-idea rationales.
  - XCIENTIST contributes 3–4× more claims (362 vs 111–122).
  - An unverifiable label measures whether *this* retriever found the
    evidence in *this* bundle.
  - The human check shows the bundle failing on comparator records: 6/20
    labels flipped, every one toward "supported" and every one on a
    comparator.

  Part of the U gap is therefore a measurement artefact of evidence
  access. The paper half-concedes this ("depended on the records available
  for inspection"). It never says how the comparators were configured.
- **The "drifts less" evidence rests on one LLM pass.**
  Drift after expansion (1.1% vs 8.1–18.9%) is the paper's strongest
  substantive result. It comes from one LLM (gpt-6-astra) applied to the
  initially unverifiable subset only, and its labels were not compared
  against humans. The judge behind the fixed-budget audit is unnamed. On
  a path-name check, the audit code and records are not in the public
  repo.
- **n = 1 trajectory per system per task.** The paper flags this itself.
  CDR here is a property of three retained runs, not an incidence rate.
- **The contract is stricter on paper than in the reported run.** The
  convergence rule requires ablation evidence for every canonical
  component, and a partial phase is "never accepted". Yet the headline
  memory result ships with "matched component-disabled results … not
  available for every component". The paper does not say whether that run
  hit the iteration limit or the contract was relaxed. The claim boundary
  was narrowed correctly in the report, which is the system working as
  intended at the reporting layer. It is not evidence that the
  convergence gate held.
- **The validators missed inference drift.** All of XCIENTIST's own drifts
  (Cases 08, 15, 16) are experiment→claim *inference* errors: an
  intervention that does not isolate its component, an endpoint metric
  read as a different property, and a rounding error. These got past the
  worker/validator contracts and were caught only by the audit. Contracts
  check that artifacts exist. They do not check that a conclusion follows
  from them.
- **The task gains are thin and chosen on the test set.**
  - Forecasting: −2.0% MAE from one run with no seeds, the best of 5
    versions on the evaluation set.
  - Memory: +0.085 F1 from one run, the best of 13 versions on the same
    LoCoMo subset.
  - PINN: wins 2/3 benchmarks, and the final version regresses against its
    own predecessor on heat-2D.

  The paper bounds these claims honestly. It does not claim
  generalization.
- **Table 1 is a self-scored capability matrix.** XCIENTIST is the only
  system with every cell filled. It is labelled "design support, not
  measured claim reliability" and should be read that way.
- **Not isolated.** There is no module ablation (graph, components or
  contracts) on CDR, as the authors say.

## Trust signals

- **Credibility:** 3. For the paper:
  - an established lab (SJTU X-LANCE, Kai Yu / Lu Chen);
  - harness code released;
  - unusually careful claim language (the abstract discloses the U/D
    composition, limits are stated in the paper's own words, and all 20
    human-review records are published in full);
  - exact D/U/N counts, so every rate can be recomputed.

  Against it:
  - not peer-reviewed;
  - the audit judge and XCIENTIST's backbone are unnamed;
  - the comparator configurations are unstated;
  - the CDR audit pipeline and records appear unreleased;
  - one run per system per task;
  - the audit is the authors' own and favours their record format.

## Follow-up

- **Relevance:** 4. It is the closest external analogue to this
  project's claim discipline and supplies three things:
  - a **relation-typed** audit (evidence→idea / idea→implementation /
    experiment→claim);
  - a **three-outcome** label set that keeps *unverifiable* apart from
    *drifted*;
  - a measured demonstration that most of the measurable difference
    between AI-scientist systems is auditability, not confirmed error.

  This strengthens [[concepts/citation-anchoring]] and
  [[concepts/typed-claim-partition]] and adds a contract-gate instance to
  [[concepts/evidence-gated-completion]]. It is not a 5: the comparative
  number is confounded by evidence access, and nothing is isolated.
- **Case 05 applies directly to this project.** An XCIENTIST claim about
  MACLA "could be traced to an internal evidence-graph summary, but the
  record lacked an independent abstract, full text or original retrieval
  return". It was labelled unverifiable: "Traceability to an internal
  summary established the origin of the wording, not the scientific
  validity of the attribution." Our concept notes cite
  `literature/papers/*.md`, which are derived summaries, not `raw/`
  passages. By this paper's standard, a concept claim anchored only to a
  literature note is traceable but unverifiable until the note carries
  the verbatim quote or a page reference into `raw/`. That argues for
  quoting the source in literature notes (house style already leans this
  way) and, possibly, for a `/lint` check on concept claims whose only
  anchor is a note with no quote for the claim.
- **Pairs with [[literature/papers/li2026discover]].** Li shows that a
  score can certify the number but not the mechanism claim. This paper
  shows that a CDR can certify auditability but not correctness. Both say
  the same thing: name what the verifier read.
- **Convergent three-outcome labelling.** Supported / drifted /
  unverifiable here matches PASS / INVALID / INCONCLUSIVE in
  [[literature/papers/zhu2026claimreceipt]], and implemented / contradicted
  / unresolved in li2026discover. These are three independent groups
  arriving at the same ternary for claim audits.
- **Instrument worth borrowing.** Report D/N and U/N separately and never
  only (D+U)/N. Any future `/lint` "unanchored claims" metric should report
  "anchor resolves but contradicts" apart from "anchor missing or does not
  resolve".
- **Candidates:**
  - EvoScientist (Lyu et al., ref [3]);
  - InternAgent-1.5 (ref [12]);
  - the ARIS and AI-Researcher systems from Table 1;
  - DeepSurvey (ref [27]), the literature-review module, whose
    paragraph-level citation verification with retry is close to
    `/ingest`.
