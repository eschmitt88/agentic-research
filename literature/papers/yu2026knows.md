---
kind: paper
title: "Knows: Agent-Native Structured Research Representations"
authors: ["Guangsheng Yu", "Xu Wang"]
institutions: ["Independent Researcher"]
year: 2026
venue: "arXiv preprint (cs.AI), technical report"
peer_reviewed: false
url: "https://arxiv.org/abs/2604.17309"
code_url: "https://knows.academy/"
citations: null
source: "raw/papers/yu2026knows.pdf"
added: "2026-07-20"
relevance: 4
credibility: 2
status: read
related_experiments: []
related_concepts:
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/file-as-bus]]"
  - "[[concepts/structured-world-model]]"
  - "[[concepts/citation-anchoring]]"
tags: [structured-metadata, agent-native, sidecar, provenance, claim-typology, knowledge-organization, tooling]
---

# Knows: Agent-Native Structured Research Representations

## TL;DR

Proposes Knows, a schema-validated YAML sidecar (`paper.knows.yaml`) that
coexists with a paper's PDF and binds typed claims, evidence, provenance,
and relations in a form LLM agents can consume directly instead of
re-parsing prose; across 140 comprehension questions on 20 papers, weak
LLMs (0.8B–2B) gain +29 to +42 accuracy points reading the sidecar
instead of the PDF while consuming 29–86% fewer tokens, and a real
community hub (knows.academy) has already indexed 10,000+ papers.

## Claims

- Research artifacts are distributed as reader-oriented PDFs; every
  downstream agent that wants fine-grained structure (claims, methods,
  results) must independently re-parse the same prose, which is
  expensive, lossy, and inconsistent across agents. Human-oriented prose
  is "no longer the only target format the scholarly record has to
  serve" now that agents author, review, and consume research
  (Agents4Science is cited as institutional evidence this is not
  hypothetical).
- A KnowsRecord is a YAML document (JSON Schema v0.9, 30 root fields, 23
  entity definitions) with five primary entity collections: *Artifacts*
  (typed identifiers for the paper and cited works), *Statements*
  (claims/assumptions/limitations/methods/questions/definitions, each
  with a modality and a two-dimensional confidence score —
  claim_strength and extraction_fidelity), *Evidence* (numeric or
  qualitative observations linked to statements), *Relations* (a typed
  directed graph over predicates like `supported_by`, `challenged_by`,
  `depends_on`, `cites`), and *Actions* (optional executable hooks with
  mandatory safety policies).
- Positions itself deliberately as a thin composing layer, not a
  replacement: it consumes identifier schemes (DOI, arXiv, ISBN, ORCID)
  directly and references artefact-level metadata standards
  (CodeMeta, CITATION.cff, Croissant, Model Card, RO-Crate) via a typed
  pointer, rather than absorbing them. The claim/evidence/relation
  layer has no direct counterpart in Nanopub, ORKG, Schema.org, DataCite,
  Paper2Agent, or Agentic Publication (Table I/II comparison) — this is
  the paper's central "what's actually new here" argument.
- Two-layer trust model: a deterministic linter (`knows-lint`, 7 checks)
  catches *structural* corruption (malformed records, dangling
  cross-references, invented field names) and fails fast; it explicitly
  does **not** catch *semantic* corruption (inflated confidence,
  fabricated observations, misattributed citations) — a KnowsRecord is
  "an author assertion, not a certification." This mirrors the
  structural/semantic split this project's own citation-anchoring
  discipline has to reckon with (an anchor can be present and still
  wrong).
- Same specification supports "review-as-sidecar": a peer review is
  itself a KnowsRecord (`profile: review@1`) whose weakness statements
  link to the original paper's specific claims via cross-record
  relations (`record_id#local_id`), making review machine-traversable
  and per-weakness auditable.

## Methods

- Benchmark: 140 L1-comprehension questions across 20 classic papers
  (579 words–198K words) spanning 14 disciplines, stratified into 4
  length buckets. 6 LLM agents across 3 capacity tiers (weak: Qwen3.5-0.8B/2B;
  medium: MiMO-V2-Flash/Qwen3.5-27B; strong: MiMO-V2-Pro/Kimi-K2.5), 3
  conditions (PDF-only, sidecar-only, sidecar+PDF fallback). Scoring:
  keyword overlap (40% phrase-match threshold) cross-validated against
  an LLM-judge (Claude Sonnet 4) on the full 1,200-answer corpus.
- Ten experiments (E1–E10) covering accuracy-by-length/discipline,
  token/latency efficiency, review traceability, lint-based consistency
  validation (structural-corruption injection), cross-record
  traceability, generator quality (which LLMs can *author* valid
  sidecars), and a granularity ablation (dense vs. uniform ~7-statement
  sidecars). A pre-registered release-rules document locks thresholds
  and the outcome-action decision tree before the run — an unusual
  amount of methodological discipline for a two-person independent
  report.
- E10 matched-output protocol: both PDF and Knows conditions emit a
  single JSON object with verbatim quotes and page numbers, specifically
  to separate the metadata-format effect from a prompt-design asymmetry
  they identify in E4 (the PDF condition was prompted for free-form
  review; the Knows condition was prompted to cite statement IDs).

## Results

- E1: weak models improve +8 to +57pp across all four length buckets
  (largest gain on LONG papers, where PDF context windows are
  overwhelmed); medium models show a mixed length-dependent pattern
  (favor Knows short, favor PDF on STANDARD-length where fine detail is
  lost to the sidecar); strong models are near-parity (-9 to +17pp).
  LLM-judge scoring (vs. keyword scoring) shows weak-model sidecar
  accuracy (75–77%) approaching medium/strong PDF accuracy (78–83%).
- Token/latency: 29–86% token reduction depending on model tokenizer
  and PDF length; local weak-model latency drops 3.4–4.6×, API-served
  models only 0.9–1.3× (network overhead dominates compute there).
- E4 review traceability: PDF-condition reviews carry **zero**
  per-weakness ID references across all four models; Knows-condition
  reviews reach 64–91% traceability (2.6–4.3 IDs per weakness section)
  at comparable completeness (53–80%) — the format doesn't just add
  metadata, it changes what a downstream auditor can verify.
- E5 lint boundary: structural corruption is caught 15/15; semantic
  corruption (wrong numbers 0/10, inflated confidence 0/8) is caught
  0/18 — explicit empirical confirmation that the linter is a syntax
  gate, not a truth gate.
- E7 generation quality: only the three Claude-family generators
  achieve 100% lint pass; no non-Claude model (Kimi-K2.5, MiMO-Flash,
  MiMO-Pro, Qwen-27B) exceeds 20%. One-shot (no lint-feedback loop)
  generation from four non-Claude models is much worse (23–50%
  downstream accuracy even when consumed by a strong model) due to
  hallucinated values and shallow coverage — authoring a good sidecar is
  harder than consuming one.
- E8 Pareto/ablation: full sidecar matches PDF accuracy (59%) at 55%
  fewer tokens; a statements-only sidecar (dropping Evidence and
  Relations entirely) retains 88% of full-sidecar accuracy at 93% fewer
  tokens than PDF (12.7× token efficiency) — dropping Relations costs
  only 1pp, dropping Evidence costs 2pp. Most of the value is in typed
  Statements; Evidence/Relations are comparatively cheap ablations.

## Critique / open questions

- Self-published: both authors list "Independent Researcher," the
  benchmark and questions were authored by the same LLM (Claude Opus)
  that also generated the sidecars — the paper names this as circular
  evaluation bias itself. No third-party replication yet; the paper is
  three months old at ingest time.
- Keyword scoring is shown to underestimate weak-model accuracy by up
  to 32pp (partial credit / phrasing variance), which the authors
  disclose and correct for with an LLM-judge cross-validation — a
  genuinely careful move, but it also means the headline "+29 to +42pp"
  numbers are exactly the ones most sensitive to scoring-method choice.
- Semantic corruption (the failure mode that actually matters for
  trust — fabricated numbers, misattributed citations) is explicitly
  *out of scope* for v0.9; the spec's entire trust story currently
  rests on "structural validity," which the paper itself frames as a
  necessary-not-sufficient floor. LLM-based semantic verification is
  future work, not implemented.
- Generation is hard and vendor-narrow: only Claude-family models
  reliably author valid sidecars (100% lint pass vs. <20% for
  everything else tested). If authoring quality is this
  model-dependent, sidecar coverage at internet scale is bottlenecked on
  whichever labs' models are good at structured authoring, which cuts
  against the "adoption-ready, open format" framing.
- All 20 benchmark papers are "well-known classics" without
  supplementary materials, code repos, or multi-part structure — modern
  ML papers (the dominant target of this project's own corpus) with
  appendices, linked code, and figures/tables-as-evidence remain
  untested. The reported gains may not transfer to that harder case.

## Trust signals

- **Credibility:** 2 — two independent researchers, no institutional
  affiliation, arXiv-only, not peer-reviewed, no citations yet (paper
  is 3 months old). Raised above the credibility floor by unusually
  disciplined methodology (pre-registered release rules, ablations,
  cross-scoring validation, explicit bias disclosure) and by real
  deployment evidence — a live community hub (knows.academy) with
  10,000+ indexed papers and demonstrated downstream use (ICLR 2026
  trend reports) — but self-authored benchmark/scoring keeps this out
  of the "solid" band.

## Follow-up

- **Relevance:** 4 — this paper's entire premise (papers as structured
  research objects — typed claims, evidence, provenance, cross-record
  relations — passed between tools, agents, and memory systems, with
  the PDF becoming a projection rather than the primary artifact) is an
  independent, external convergence on the exact design this project's
  own `literature/*.md` notes already implement as a read interface for
  downstream `@import`-ing projects (see CLAUDE.md "Role"). It doesn't
  seed a new concept — the pieces (typed claim structure,
  file-as-interface) already exist here — but it's genuine, non-derived
  corroborating evidence for two under-attested concepts:
  [[concepts/typed-claim-partition]] (KnowsRecord's Statement/Evidence/Relation
  triad with typed predicates and modality/confidence scores is a second,
  independently-motivated instance of "type the claim structure so
  downstream consumers can act on the type, not just a scalar") and
  [[concepts/file-as-bus]] (the sidecar-as-durable-artifact-agents-read-
  instead-of-reconstructing-from-prose pattern, at document granularity
  rather than workspace granularity).
- Watch for: v1.0 (gated on the full 840-call matched-output rerun,
  currently in progress) and whether third-party adoption of
  knows.academy or the `paper@1`/`review@1` profiles produces
  independent evidence beyond the authors' own deployment numbers.
