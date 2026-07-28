---
kind: paper
title: "EvoBrowseComp: Benchmarking Search Agents on Evolving Knowledge"
authors: ["Yunhan Wang", "Jiaan Wang", "Lianzhe Huang", "Xianfeng Zeng", "Fandong Meng"]
institutions: ["Northeastern University, China", "Weixin AI, Tencent Inc"]
year: 2026
venue: "arXiv preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2606.13120"
code_url: null
citations: null
source: "raw/papers/wang2026evobrowsecomp.pdf"
added: "2026-07-28"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/information-firewall]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/web-grounded-literature]]"
  - "[[concepts/programmable-evaluator-oracle]]"
tags: [evaluation, contamination, benchmark, retrieval, search-agents, freshness]
---

# EvoBrowseComp: Benchmarking Search Agents on Evolving Knowledge

## TL;DR

Contamination resistance **by construction**: 800 multi-hop questions
(400 EN / 400 ZH) synthesized automatically from *freshly emerged* web
knowledge, so parametric memorization cannot answer them. Because the
synthesis pipeline is fully automated, the benchmark can be refreshed
continuously — swapping in new facts and retiring over-exposed ones —
rather than aging into contamination the way a hand-curated static set does.

## Claims

- Static benchmarks are structurally doomed: BrowseComp, GAIA,
  BrowseCompPlus, DeepSearchQA all fix their QA pairs (or a document
  snapshot) at one point in time, so as pre-training corpora grow the
  benchmark leaks into model parameters. The paper cites Anthropic's own
  report of BrowseComp answers appearing in public data as confirmation.
- Human curation is what makes refresh prohibitive. Automating synthesis
  removes the cost barrier and makes *continuous refresh* the contamination
  defense.
- Freshness alone is insufficient — **over-exposed** knowledge is a
  parametric shortcut even when recent. Popularity filtering is a separate,
  necessary filter.
- The tool-free ablation is the benchmark's own validity certificate: if a
  model scores well *without* retrieval, the benchmark is measuring recall.

## Methods

- **Three-agent synthesis pipeline.** (i) *QA Synthesis Agent* — traverses
  the live web from a seed entity and proposes candidate QA pairs; each
  reasoning hop carries an explicit `is_new_knowledge` flag. (ii)
  *Information Filtering Agent* — screens on **credibility** (source
  reputation, cross-source consistency) and **popularity** (blocks
  over-exposed facts that a model could answer parametrically). (iii)
  *High-level Guidance Agent* — structures each question as a reasoning
  graph over three operations (projection, intersection, complement),
  detecting structural redundancy and shortcut paths and steering synthesis
  away from them.
- Questions are deliberately obfuscated (entities described by properties
  rather than named) so the multi-hop chain cannot be short-circuited by a
  single lookup. Strategies include inverse projection (flip effect → cause).
- Quality gates: textual quality, answer uniqueness, question difficulty;
  domain-balanced down to the final 800.
- Evaluation under matched tool-based and tool-free settings across frontier
  models. Backbone for all three synthesis agents is DeepSeek-V3.2.

## Results

- Claude-Opus-4.6 with tools: **44.8%** — the questions resist retrieval
  even for a frontier agent.
- Same model without tools: **6.0%** — a ~39-point drop that is the
  paper's central evidence that the benchmark measures browsing rather
  than memorization.
- Some models show high "ER" (entity-recall-style) scores while answering
  poorly (DeepSeek-V4-Max 75.5% EN / 82.5% ZH), i.e. retrieving the right
  entities is not the bottleneck — composing them is.
- The time anchor is a parameter: the pipeline can be re-pointed at any
  timestamp (e.g. a newer training cutoff) to regenerate a fresh set.

## Critique / open questions

- No artifact URL in the paper; data is promised under CC-BY-NC-SA 4.0 but
  no link is given — hence `code_url: null`. Ironically, a benchmark whose
  whole premise is refresh needs a *living* distribution channel, and the
  paper does not describe one (who runs the refresh, on what cadence, and
  how versions are pinned for comparability).
- Refresh and comparability are in tension and the paper does not address
  it: if the question set changes each cycle, scores across cycles are not
  strictly comparable. This is the same tradeoff `hce-evaluation` makes
  when it forbids re-running the final pass — a fresh holdout buys
  unbiasedness at the cost of longitudinal continuity.
- Authors' own limitations: synthesis inherits DeepSeek-V3.2's biases; the
  judge scores only final answers, so a lucky guess is indistinguishable
  from correct reasoning.
- "Popularity" is doing heavy lifting but is not operationalized in enough
  detail to reimplement the threshold.

## Trust signals

- **Credibility:** 3 — reputable industrial group (Tencent Weixin AI) with a
  large, well-instrumented evaluation and a clean ablation design; but arXiv
  preprint, no peer review, no released artifact link, and the central
  filtering criterion (popularity) is under-specified.

## Follow-up

- **Relevance:** 4 — the **second attestation**
  [[concepts/information-firewall]] was flagged as needing in the 07-20
  NOTES, and it arrives from the construction side rather than the
  method-withholding side, which is what makes it worth having.
- Pairs directly with [[literature/papers/wang2026search]]: that paper
  *detects* retrieval-time contamination during evaluation; this one
  *prevents* it at benchmark-construction time. Together they cover both
  ends of the same channel, and the pairing is the strongest argument that
  `information-firewall` is a real boundary and not a restatement of
  `hce-evaluation`.
- Two mechanisms worth importing regardless of the benchmark:
  1. **Tool-free ablation as a contamination audit.** Cheap, general, and
     applicable to any retrieval-enabled evaluation: run the holdout with
     retrieval disabled: a high score means the split is already in the
     weights. This is a validity check a project can run on its *own*
     `test/` without spending the holdout on a scored result.
  2. **Retire over-exposed items, don't just add fresh ones.** Freshness
     decays. A holdout has a shelf life, and popularity — not age — is the
     right expiry signal.
- Relevant to this project's own `/digest` and `/discover`: the same
  live-web traversal that powers literature sweeps is the synthesis
  substrate here. See [[concepts/web-grounded-literature]].
