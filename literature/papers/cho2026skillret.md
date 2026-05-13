---
kind: paper
title: "SkillRet: A Large-Scale Benchmark for Skill Retrieval in LLM Agents"
authors: ["Hongcheol Cho", "Ryangkyung Kang", "Youngeun Kim"]
year: 2026
venue: "arXiv 2605.05726"
url: "https://arxiv.org/abs/2605.05726"
source: "raw/papers/cho2026skillret.pdf"
added: "2026-05-11"
relevance: 4
status: read
related_experiments: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/web-grounded-literature]]"
tags: [skill-library, retrieval, benchmark, long-document, fine-tuning, claude-skills, knowledge-organization]
---

# SkillRet: A Large-Scale Benchmark for Skill Retrieval in LLM Agents

## TL;DR

SkillRet is a 17,810-skill benchmark (filtered from 22,795
community-contributed Claude skills on claude-plugins.dev) with
63,259 training and 4,997 evaluation queries, a 6×18 two-level
taxonomy, and an empirical finding that **domain-specific
fine-tuning dominates model scale** for skill retrieval: an 8B
fine-tuned retriever beats a 12B off-the-shelf retriever by ~28 pp
NDCG@10. The deeper finding is mechanistic: fine-tuned models learn
to focus attention on the small skill-relevant subset of sentences
within long noisy queries — general retrieval competence transfers
poorly because the signal locus is different.

## Claims

- **Skill retrieval is a distinct retrieval problem**, not a special
  case of tool retrieval or general semantic retrieval. Skills are
  long-form markdown documents (median 1,583 tokens, 95th percentile
  5,531) — an order of magnitude longer than tool descriptions.
  Encoding the full body, not just name+description, consistently
  outperforms shorter representations (1.5–11.4 pp NDCG@10).
- **MTEB Retrieval rank does not predict skill-retrieval rank**
  (Spearman ρ = 0.71 with many large inversions). General-purpose
  retrieval competence transfers poorly; specialized fine-tuning
  is the only reliable path to high skill-retrieval quality.
- **Domain-specific fine-tuning dominates scale.** Best off-the-shelf
  (harrier-oss-v1-0.6b): 66.55 NDCG@10. Best prior fine-tuned
  (SkillRouter-0.6B): 70.38. Their fine-tuned 0.6B: 78.03. Their
  fine-tuned 8B: 83.45 — +16.9 over best off-the-shelf, +13.1 over
  best prior fine-tuned.
- **Mechanistic finding: fine-tuned models concentrate attention on
  skill-relevant sentences in long noisy queries.** Sentence erasure
  ablation: masking the top-1 most important sentence drops the
  trained model's NDCG@10 by 29.2% vs. only 23.3% for the base model
  — i.e., the trained model's retrieval signal is *more
  concentrated* on a small fraction of the query.
- **Reranking helps only when the first-stage retriever has
  headroom.** Off-the-shelf rerankers actively *hurt* on top of
  already-specialized retrievers (domain mismatch). Once the
  embedding stage saturates, reranking adds 1–4 pp at best.

## Methods

- **Corpus construction:** 22,795 community-contributed Claude skills
  → 5-stage filter (description recovery, English-only,
  MIT/Apache-2.0 license, content dedup, name+desc dedup) → 17,810
  skills (78.1% retention). Two-level taxonomy (6 Major × 18 Sub)
  built by consensus k-means clustering over action × object tag
  pairs + iterative human refinement; Claude Sonnet 4.6 assigns
  taxonomy labels (95.5% major-cat accuracy on 200-skill human
  validation).
- **Query construction:** self-instruct-style. 165 GAIA validation
  seeds for style diversity; inverse-frequency-weighted skill
  sampling (k=1/2/3 skills per query, ensuring every skill is
  covered ≥1×). Training queries: Qwen3.5-122B. Evaluation queries:
  Claude Opus 4.6 (deliberate different generator to prevent
  stylistic leakage). 3-stage filter (3-gram leakage detection +
  three-perspective LLM review + human expert validation).
- **Models evaluated:** 18 embedding models from BM25 baseline through
  encoder-only (33M–335M) through decoder-only (80M–12B); 4 rerankers
  including SkillRouter and Qwen3-Reranker family.
- **SkillRet model family** (theirs): fine-tuned Qwen3-Embedding-0.6B
  and 8B with MultipleNegativesRankingLoss + in-batch negatives on
  127K positive query-skill pairs. SkillRet-Reranker-0.6B fine-tuned
  with hard-negative mining via the trained 0.6B retriever.

## Results

- **NDCG@10 ladder** (best models per category):
  | Model | Params | NDCG@10 |
  |---|---|---|
  | BM25 (sparse baseline) | – | 48.86 |
  | bge-large-en-v1.5 (encoder ceiling) | 335M | 55.82 |
  | KaLM-Gemma3-12B (largest, off-the-shelf) | 12B | 55.38 |
  | harrier-oss-v1-0.6b (best off-the-shelf) | 0.6B | 66.55 |
  | SkillRouter-Embedding-0.6B (best prior fine-tuned) | 0.6B | 70.38 |
  | **SkillRet-Embedding-0.6B** (theirs) | 0.6B | **78.03** |
  | **SkillRet-Embedding-8B** (theirs) | 8B | **83.45** |
- **Per-category difficulty is stable.** Hardest categories under
  both base and fine-tuned 8B: Information Retrieval and AI Agents
  (lowest NDCG@10); easiest: Data & ML. Fine-tuning improves every
  category, but the relative ordering is preserved — the
  category-level disparity (16 pp gap between easiest and hardest
  even after fine-tuning) is invisible to the aggregate score.
- **Distribution:** Software Engineering = 62.2% of skills, Info
  Retrieval = 3.1% — a 20× imbalance reflecting the natural
  composition of public agent-skill ecosystems.
- **Skill length distribution:** median 1,583 tokens, mean 2,083,
  95th percentile 5,531, max 47,412 — ~37.1M tokens across the
  corpus. Data & ML skills are longest (median 1,795); Information
  Retrieval skills shortest.
- **Query length asymmetry:** evaluation queries (Opus 4.6 generated)
  have median 170 words; training queries (Qwen3.5 generated) have
  median 72 words. The longer evaluation queries are harder for
  lexical matching — intentional, to surface the
  signal-in-noise problem.

## Critique / open questions

- **Synthetically generated queries.** Limitation acknowledged in §6:
  the eval set is generated by Opus 4.6, not collected from real
  agent interactions. Under-represents terse, conversational,
  user-context-dependent requests that probably dominate real
  deployment. The leakage filter (3-gram overlap < 10%) prevents
  trivial lexical-overlap shortcuts but doesn't prevent stylistic
  drift between generator and real user voice.
- **End-to-end gap.** SkillRet measures retrieval in isolation;
  higher NDCG@10 doesn't necessarily translate to better agent
  task completion. The paper flags this; SkillsBench and
  SWE-Skills-Bench actually find that public skills often don't
  improve downstream task performance at all. Retrieving the right
  skill is necessary but not sufficient.
- **Single-author benchmark, small lab.** ThakiCloud, three authors.
  Public dataset + checkpoints released on HuggingFace (which is
  the right move for reproducibility), but the benchmark hasn't
  yet been independently re-run. Worth flagging as
  not-yet-validated.
- **The "skills" here are community Claude skills**, which skews
  heavily toward software engineering (62.2%) and reflects the
  current Claude-skills ecosystem composition specifically. Whether
  the retrieval findings transfer to other skill formats
  (Anthropic's full SKILL.md spec with scripts and sub-skills,
  SkillOS's curated/RL-trained skills, ExpWeaver-style insights)
  is open. The benchmark *is* the ecosystem, not a general
  skill-retrieval test.
- **The "fine-tuning beats scale" finding is empirically strong**
  but inherits the usual confounds — the training data is
  in-distribution by construction (eval skills are held out, but
  the query distribution is shared style). The argument that this
  is a *general* skill-retrieval finding rather than a
  domain-overfit-helps finding would benefit from cross-corpus
  evaluation.

## Follow-up

**Relevance: 4** — strengthens [[concepts/skill-library-lifecycle]]
with the strongest empirical anchor available on what skill
libraries actually look like at scale (17.8K skills, median 1.6K
tokens each, 6×18 taxonomy, 20× category imbalance). Also
materially extends the read-side picture by showing that retrieval
quality on long skill documents requires *focused attention on the
small skill-relevant subset of sentences* — a within-retrieval
analog to ExpWeaver's between-retrieval gating decision.

**Connections to our own implementation.** Three concrete takeaways
for this project's knowledge graph:

1. **Body carries the retrieval signal, not frontmatter.** SkillRet
   shows encoding the full markdown body gains 1.5–11.4 pp NDCG@10
   over name+description only. This project's `concepts/` and
   `literature/` notes follow this pattern (extensive body text
   beyond frontmatter); the empirical result validates the
   structural choice. Frontmatter-only retrieval (e.g., grep on
   tags) would miss the load-bearing signal.

2. **Long-form notes are the right unit.** Median skill = 1.6K
   tokens. The project's literature notes (GSAR, ExpWeaver, SkillOS
   ingested in this batch) run ~1–2K tokens of body. SkillRet's
   evidence: this length range is where the retrieval problem
   actually lives. Shorter notes would lose retrievability; much
   longer notes start exceeding non-decoder retriever context
   windows. The project's current length norms are well-calibrated.

3. **Domain-specific retrieval > general retrieval.** SkillRet's
   strongest finding: a fine-tuned 0.6B beats a 12B general-purpose
   model by ~23 pp. For a research-memory project, this points to
   a future where the project's own retriever (over its own
   concept/literature corpus) might be trained on its own query
   patterns rather than relying on general-purpose embedding
   models. Speculative but on-trend.

**Pairing with the batch:**
- **SkillRet measures, SkillOS curates, ExpWeaver gates.** The three
  papers cover three distinct layers of skill-library memory: how
  the library is *built* and *evaluated* (SkillRet, this paper),
  how it is *grown over time* (SkillOS, [[literature/papers/ouyang2026skillos]]),
  and how it is *selectively read* at decision time (ExpWeaver,
  [[literature/papers/zhao2026expweaver]]). None of the three does
  all three; combining them is the natural research direction.
- **GSAR ([[literature/papers/kamelhar2026gsar]]) is at a different
  layer.** GSAR is about *output* knowledge organization (typed
  claim partition for the agent's writes); SkillRet/SkillOS/ExpWeaver
  are about *input* knowledge organization (skill libraries the
  agent reads). The four-paper batch together spans the read and
  write axes of agent knowledge organization.

**Suggested next read** from this batch:
- AutoResearchBench (`xiong2026autoresearchbench`, pending) — the
  remaining ingest. Different angle entirely: evaluating agents on
  *literature discovery* rather than skill retrieval, which is the
  most direct external yardstick for this project's `/discover`
  and `/digest` skills.
