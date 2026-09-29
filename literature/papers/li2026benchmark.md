---
kind: paper
title: "A Benchmark and Diagnostic Study of Epistemic Admission in Shared Agent Memory"
authors: ["Xiaoyang Li", "Yiqi Wang", "Chencheng Zhu", "Ke Xu", "Wencheng Yang", "Zequn Sun", "Pingan Song", "Yiqun Duan", "Taotao Cai"]
institutions: ["University of Southern Queensland", "University of New South Wales", "Aikaier", "Nanjing University", "Facebook"]
year: 2026
venue: "arXiv (cs.AI); code repo named iclr_2027, so likely under review"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.30813"
code_url: "https://github.com/lxy1134/iclr_2027"
citations: null
source: "raw/papers/li2026benchmark.pdf"
added: "2026-09-29"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/shared-substrate-contagion]]"
tags: ["shared-memory", "memory-admission", "source-lineage", "copy-detection", "truth-discovery", "multi-agent", "benchmark", "false-belief-propagation", "no-channel-arm", "mem0", "a-memguard", "pre-registration"]
---

# A Benchmark and Diagnostic Study of Epistemic Admission in Shared Agent Memory

## TL;DR

The paper introduces the Correlated Promotion Benchmark (CPB). It asks
whether a candidate claim should be admitted to a memory store that a
team of agents shares, when repeated support may be copies rather than
independent evidence. There are two halves. **CPB-Static** is 1,200
six-agent episodes built from public claim-verification data, scored
against a fixed gold rule ("two independent sources"). **CPB-Live** runs
six agents for six rounds over a logged shared store, with lineage fixed
by authored-fiction scenarios, and a consumer answers from the store
alone. Eight policies are tested on four backbones (Qwen3.5-9B,
Gemma-3-12B, Llama-3.1-8B, Claude Opus 5), plus Mem0's write path and
A-MemGuard's consistency check. Each mechanism class leaves an entry
route. Copy collapse rejects true claims *more* firmly than false ones.
Voting and judges admit most of what share-all admits. A gate on the
*declared source type* keeps adoption at 0.06–0.09, but only because the
scenarios supply that field, and it admits a reworded copy once the copy
is typed authoritative. Even a perfect lineage oracle admits a false
claim whose second source is a genuinely independent restatement. Its
cleanest result is a **no-channel arm**: switching off the agents'
retrieval of shared memory removes three quarters of what the Claude
agents say but changes false writes only from 28 to 27.

## Claims

- **F1.** "only governance keeps damage low while still answering in
  every family, and the judges that filter halve damage at best."
  Governance damage share is 0.112–0.152, adoption 0.06–0.09, correctness
  0.58–0.65. Its correctness is "0.90 on concurrent conflict, 0.90 on
  correlated agreement and 0.00 on mixed scenarios": it works where the
  scenario supplies the authoritative source type it gates on.
- **Copy collapse over-rejects truth.** The independence class (the
  independent-source vote, DEPEN and GovMem's review-all-correlated rule)
  "admits 0.119 to 0.306 of false candidates and 0.051 to 0.127 of true
  ones, so it rejects true claims more firmly than false ones on every
  family." Correctness is 0.13–0.17.
- **F2. Published components.** "Mem0 removes duplicates and not false
  adoption." Deduplication "absorbs 0.67 of Gemma's admissions and 0.86 of
  Llama's, mostly by folding false candidates into false beliefs already
  held." A-MemGuard's consistency check rejects "0.00, 0.01, 0.18 and 0.00
  of false candidates", because "each arrives with supporting source
  text". "The tested write and consistency checks leave falsehoods
  supported by their cited evidence admissible."
- **F3.** "With shared memory as the only source, an uncontested false
  belief is asserted in 0.97 to 0.99 of probes" (553 probes). With a
  competing true belief visible, false-only answers fall to 0.15 of 160.
  With 3+ false beliefs beside one true one, "false-only adoption returns
  to 0.42."
- **F4.** Across families "the rule's refusal moves by 0.04 and the vote's
  by 0.89." Governance refuses 0.84–0.88 of false candidates. Majority
  vote refuses 0.00 on Qwen and 0.89 on Llama, depending on what agents
  happen to cite.
- **F5.** "only the lineage oracle refuses all three copy regimes." The
  independence class refuses verbatim copies (0.00 adoption) and admits
  reworded ones (0.65–0.72 adoption). Governance holds 0.00 on web-typed
  copies and "0.70 once the reworded copy is typed as a register
  document, so its protection rests on source type and not on detecting
  the copy."
- **F6.** "Admission follows the form of the second source more than the
  truth of the claim." The lineage oracle admits a false independent
  restatement "in 0.95 when false and 0.90 when true": "Perfect copy
  detection removes the rewording failure of F5 but cannot establish the
  truth of independent support."
- **F7 (no-channel arm).** On Claude Opus 5, share-all with agent retrieval
  switched off: candidates per configuration "fall from 8.3 to 2.2", and
  candidates declaring a shared belief "from 189 to 0". Meanwhile "false
  writes at 28 and 27, configurations holding a false belief at 0.42 and
  0.45, damage at 115 and 122, and adoption at 0.27 and 0.23." Qwen:
  false writes 33 in both arms. "the loop adds agreement to falsehoods
  already supplied by the feeds while exposure changes little."
- **Static.** The frontier judge has the lowest false-promotion rate
  ("0.077 against 0.228 to 0.285") and the lowest accuracy (0.41), because
  it keeps 0.54 of candidates private: "it withholds more rather than
  separating better." Trained logistic regression and boosted trees reach
  "0.645 and 0.653 accuracy against the best rule's 0.368": "the rules
  leave most of the structure unused."

## Methods

- **Action space.** A policy maps ⟨claim, evidence graph, current store⟩
  to PROMOTE / REQUEST EVIDENCE / KEEP PRIVATE / ABSTAIN. Only PROMOTE
  writes, and the claim becomes retrievable by every agent the next
  round. Policies never see the retrieval log, the authored lineage or
  the truth.
- **Static.** Verdicts come from AVeriTeC/FEVER, supersession chains from
  MQuAKE, and lineage groups from the Wikipedia citation graph. The split
  is 600/200/400 episodes, and the test split was hashed and frozen before
  evaluation. An audit found two assembly defects, which were fixed, and
  the split was refrozen.
- **Live.** 100 configurations per family: 60 base configurations (four
  scenario families: correlated agreement, concurrent conflict, mixed,
  dual source, including order-swapped variants), 20 dual-source and 20
  reworded-copy. There is a 30-configuration truth×form cross on Qwen and
  Claude only. The open models are commit-pinned with greedy decoding.
  Claude uses provider-default sampling. Each family's backbone plays the
  agents, the consumer and the judge.
- **Metrics.** Damage = consumer reads made while a false belief was
  retrievable, reported as a share of share-all's. FA/TA = false and true
  admission rates over the candidates each policy itself saw. Adoption =
  probes asserting the false value.
- **Grading.** Claude Opus 5 labels candidate truth as an assertion grade
  over 6,443 pairs. On 120 pairs, blind human agreement is 0.867 (κ 0.791),
  and a second annotator gets κ 0.806 against the grader. Precision on
  "false" is 0.958 and recall 0.852, so false counts "lean low". The
  consumer-answer rule has recall 0.706 on "false", so adoption is
  undercounted wherever a value is rephrased. A normalizing matcher moves
  no adoption cell by more than 0.03.
- **Statistics.** Six Static comparisons are pre-registered and
  Holm-corrected, and the freeze files are hashed with timestamps. "every
  other analysis, Live included, is pre-registered as descriptive and
  reported without hypothesis tests."

## Results

See Claims. Additional numbers:

- The consumer "abstained in all 1,551 probes whose episode promoted
  nothing".
- "none of the 1,695 admitted false beliefs is corrected": the store
  offers contest/demote/supersede, and no policy uses them.
- **Order sensitivity.** Swapped arrival order flips share-all on 0.6 of
  Gemma pairs and every Llama pair, but on 0.00 of Qwen pairs.
- **Declared provenance is incomplete.** Qwen agents declared no
  dependency on any of their 78 base candidates. The share of false
  writes with a later declared dependency runs from 0.01 (Qwen) to 0.82
  (Llama), so "declaration-based amplification is a lower bound shaped by
  reporting behaviour."
- **Semantic collapse** (bidirectional entailment, not an oracle) lets
  through 0.10 of reworded false claims and 0.15 of authoritatively typed
  ones: close to the oracle's 0.00.

## Critique / open questions

- **The abstract's contrast is wider than the body's.** The abstract gives
  governance 0.06–0.09 adoption "compared with 0.22–0.47 for other
  answering policies". Table 5 has the Claude-family judge at **0.19**
  adoption (0.40 correctness) and the frontier judge over Qwen at 0.20.
  Those sit below the stated floor unless they are silently counted as
  non-answering. The accurate gap is 0.06–0.09 against about 0.19–0.47.
- **Governance's win is bought by the scenario.** The rule gates on a
  source-type field that authored scenarios supply and nothing
  authenticates. Correctness drops to 0.00 on mixed scenarios, and 0.70
  adoption follows once a copy is typed authoritative. It costs utility
  too: TA runs from 0.133 (Llama) to 0.481, and correctness is 0.58–0.65
  against share-all's 0.65–0.77. This is louck2026securing's point: a
  declared authority field is only as good as its binding to origin.
- **97–99% is the constrained ceiling, not the deployment rate.** The
  consumer has "no source of its own" by design. Under share-all, overall
  adoption is 0.32–0.47, because only 0.50–0.63 of configurations hold a
  false belief at probe time.
- **"No non-oracle policy survives" is narrower than it sounds.** Semantic
  collapse comes within 0.10–0.15 of the oracle on reworded copies. The
  deeper negative is F6: even the oracle admits a false *independent*
  restatement. Lineage fixes copy counting, not truth.
- **Limits the authors state (Appendix S).** Every Live scenario is
  authored fiction, so the results show how mechanism classes behave on
  this instrument, "not how often the failures arise in a deployment".
  Live is descriptive with no tests. Voting, collapse and judge policies
  are reimplementations. The protective effect of a competing truth is
  measured in one scenario family only. Correction is "untested rather
  than absent".
- The three independence-class rules share one surface-collapse predicate
  and decide identically on all three open families, so "three policies"
  is effectively one there.
- The small open backbones (8–12B) behave very differently from each
  other (Llama's 7.35 false candidates per configuration vs Qwen's 0.62),
  so family-pooled numbers hide a lot of variance.
- Claude Opus 5 is both an agent family and the grader. The authors check
  this: 3 of 4 disagreements on that family go against Claude.

## Trust signals

- **Credibility:** 3. Mid-tier academic group (USQ-led, with UNSW,
  Nanjing University, and one Facebook-affiliated author). Not peer
  reviewed; the repo name suggests an ICLR 2027 submission. The
  methodology is careful: hashed, timestamped pre-registration freezes; a
  frozen test split with a disclosed audit and refreeze; both graders
  validated against two blind annotators; limits stated plainly. The code
  repo resolves (Static recipe, Live scenarios, policies, table scripts),
  but transcripts and run summaries are held until acceptance. Held back
  by: no peer review, authored-fiction Live scenarios, descriptive-only
  Live statistics, reimplemented baselines, and an abstract range the body
  contradicts.

## Follow-up

- **Relevance:** 4. It is the first benchmark to execute admission to a
  *shared* store with fixed lineage and measured downstream adoption. It
  tests the admission policies verified-memory-writes collects, and it
  gives shared-substrate-contagion a no-channel arm on a written store. It
  sharpens both concepts without seeding a new one.
- **[[concepts/verified-memory-writes]].** Adds a section on admission
  to a shared tier: source counting is not verification; declared type is
  spoofable; faithfulness checks pass falsehoods their sources support;
  dedup removes redundancy, not error.
- **[[concepts/shared-substrate-contagion]].** The F7 no-channel arm:
  write-back adds agreement, not falsehood. It also bears on the
  provenance-depth open question: lineage stops copies but not false
  independent support, and writer-declared lineage is incomplete.
- **Repo read.** The governance rule's weakness maps onto this graph:
  `sources:` counts and `venue` fields are declared by the same agent pass
  that writes the concept, so they are a type field, not a lineage.
- **Candidates cited and not in this graph:** GovMem (Qi et al., 2026),
  MemTX (Li et al., 2026b), CAMA (Lin et al., 2026), A-MemGuard (Wei et
  al., 2025), Learning to Share (Fioresi et al., ICML 2026), and Becker et
  al. 2026 (arXiv 2606.16710) on majorities correcting misinformed agents.
  GovMem and MemTX are the closest fit to verified-memory-writes.
