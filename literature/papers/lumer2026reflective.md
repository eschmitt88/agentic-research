---
kind: paper
title: "Reflective Prompt Optimization of a Deep-Research Agent: A Topic-Disjoint, Multi-Seed Protocol for Measuring Transferable Gains and Prompt-Level Overfitting"
authors: ["Elias Lumer", "Sahil Sen", "Vamse Kumar Subbiah", "Kevin Paul"]
institutions: []    # not stated in the PDF author block or on the arXiv abs page; the group's companion papers (Onweller, Leung, Sen et al. 2026) are self-cited throughout
year: 2026
venue: "arXiv 2610.12601 (cs.CL), v1 2026-10-08, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.12601"
code_url: null      # artifacts (prompts, lineage diffs, analysis) are said to be released, but neither the PDF nor the abs page gives a link
citations: null
source: "raw/papers/lumer2026reflective.pdf"
added: "2026-10-13"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/skill-library-lifecycle]]"
tags: ["prompt-optimization", "gepa", "deep-research-agent", "topic-disjoint-split", "held-out-audit", "prompt-level-overfitting", "memorized-answer-key", "winners-curse", "decomposed-feedback", "scalar-reward", "llm-judge", "citation-reward", "verifier-ablation", "multi-seed", "prompt-bloat", "recipe-baseline"]
---

# Reflective Prompt Optimization of a Deep-Research Agent: A Topic-Disjoint, Multi-Seed Protocol for Measuring Transferable Gains and Prompt-Level Overfitting

## TL;DR

The authors run GEPA (reflective prompt evolution with a Pareto pool) on
the two system prompts of an orchestrator + researcher deep-research
agent (GPT-5.4 policy, GPT-5.5 reflector). The reward is an LLM-judged
citation-quality composite behind a correctness gate. Train and
validation are 24 tasks each, and the 500-task, 254-topic test set
shares no article topic with either. Three results matter for this graph:

1. **The gain transfers, and the headline is the conservative seed.**
   Held-out reward rises 0.702 → 0.781 (+0.079, bootstrap 95% CI
   [0.051, 0.107], 287/66/147 win/tie/loss). Two more seeds give +0.130
   and +0.112. The validation score claimed +0.202, so validation
   overstated the transferable gain about 2.6×.
2. **The optimizer writes a literal answer key into the prompt, and it is
   inert.** By word count, 52% of the winning orchestrator prompt and 39%
   of the researcher prompt are "Known Benchmark Facts" blocks that pair
   verbatim training questions with answers and URLs. Re-scored on those
   exact 24 training questions, the best prompt gains +0.090, the same as
   on unseen topics.
3. **Per-dimension textual feedback is what the reflector needs.** With
   the reward collapsed to a scalar, held-out transfer is −0.021, against
   +0.075 with decomposed feedback. Coarser or 3×-repeated scoring does
   not change transfer. The evidence is 2 seeds per arm on 120 tasks.

## Claims

- A topic-disjoint split plus a large held-out set separates transferable
  behaviour from memorized lookup. The validation-minus-held-out gap then
  tests whether the apparent gain is memorization.
- That gap "is therefore small-sample selection under a noisy judge, not
  memorization" (abstract). The evidence offered is that the gap swings
  0.034–0.160 across seeds and that the answer key earns nothing.
- "Decomposed per-dimension feedback is the causal ingredient." A scalar
  gives the reflector "nothing to diagnose".
- "Reward gaming tracks the weakest verifiers rather than the strongest."
  The gaming gap is 0.071 coarse and 0.061 scalar, against 0.043 for the
  fine decomposed reference.
- A "hand-written version of the discovered recipe" recovers most of the
  gain (57%). The rules are answer-first structure, one claim per
  citation, verified page content and corpus-only URLs.
- The gain is not a same-family judge artefact. An out-of-family judge
  (Claude Opus 4.8) agrees with a human at least as well, and it scores the
  gain larger.
- Pareto selection rejected the two longest candidates (3,261-word
  orchestrator prompt). This is in-run evidence of prompt bloat, but only
  a validation-time guard.

## Methods

- **Agent.** An orchestrator plans and delegates to a research sub-agent
  that searches a frozen news corpus and writes a cited report. Both
  system prompts are optimized. Each GEPA candidate mutates one component.
- **Reward (Eq. 1, App. C).** R = answer_gate × (0.25 link_works + 0.30
  relevant_content + 0.30 fact_check + 0.15 citation_numbering).
  - The answer gate is a three-tier cascade: canonicalized substring,
    then token coverage, then a GPT-5-mini verdict in the near-miss band.
  - link_works and numbering are deterministic.
  - Relevance (GPT-5-mini) and factual support (GPT-5.4-mini) are binary
    per-citation judges that see corpus content truncated to 5,000 chars.
  - The verbatim judge prompts are in the appendix.
- **GEPA config.** Pareto candidate selection, reflection minibatch 3,
  240 metric calls. The feedback names each lever and surfaces the
  weakest citations' judge reasons. Seven candidates were produced, and
  the best was found at rollout 156.
- **Split.** Tasks are grouped by source-article slug. Whole groups go
  to test first, then validation, then train: 24/24/500 tasks over
  12/12/254 topics, zero topic overlap.
- **Statistics.** Paired mean delta with a bootstrap CI over tasks,
  Wilcoxon, paired t, d_z = 0.247. Per-lever decomposition on a 200-task
  held-out subset.
- **Ablations.**
  1. 3 optimization seeds, each scored on n = 500.
  2. Topic-shared re-score of the fixed seed and best prompts: the 24
     exact training questions plus 96 fresh questions from the 12
     training families, split into memorized and non-memorized families.
  3. Verifier variants (fine/coarse, 1×/3× scoring, decomposed/scalar),
     2 seeds each, all re-scored on one canonical verifier (n = 120).
  4. A "recipe" prompt: the best prompt with the answer-key blocks and
     two answer-bearing examples deleted, scored on 100 tasks.
  5. A judge check: one author labels 48 held-out (attribution, citation)
     pairs blind (Table 10 says n = 45), plus an out-of-family re-score
     on 12 tasks.

## Results

- **Lineage (Table 3).** Validation reward goes 0.672 → 0.725 → 0.795 →
  0.867 → **0.874**, then 0.805 and 0.840 for the over-grown candidates.
  The answer-key share enters at the first accepted edit and rises with
  validation reward (Pearson r = 0.83).
- **Held-out per lever (Table 5, n = 200).** link_works +0.086,
  relevant_content +0.094, fact_check +0.095 (0.527 → 0.622, still the
  binding lever), answer_gate +0.030, numbering at ceiling.
- **Seeds (Table 6).**

  | Seed | Val Δ | Test Δ | 95% CI | Val − test |
  |---|---|---|---|---|
  | 0 | +0.202 | +0.079 | [0.051, 0.107] | 0.123 |
  | 1 | +0.290 | +0.130 | [0.104, 0.156] | 0.160 |
  | 2 | +0.147 | +0.112 | [0.086, 0.140] | 0.034 |

- **Topic-shared (Table 7, one rollout per task).**
  - Exact training questions: +0.090 overall, +0.089 on memorized
    families and +0.090 on the others. The answer gate is flat at 0.92
    for both prompts.
  - Fresh same-family questions: +0.112 overall, +0.071 memorized vs
    +0.154 non-memorized.
- **Verifier ablation (Table 8).** Validation Δ / transfer Δ / seed spread:

  | Verifier | Val Δ | Transfer Δ | Seed spread |
  |---|---|---|---|
  | fine, 1×, decomposed | +0.118 | +0.075 | 0.037 |
  | coarse, 1×, decomposed | +0.150 | +0.079 | 0.079 |
  | fine, 3×, decomposed | +0.109 | +0.059 | 0.018 |
  | fine, 1×, scalar | +0.040 | −0.021 | 0.030 |

  Per the text, one scalar seed found no accepted improvement at all.
- **Recipe (Table 9, n = 100).**
  - Recipe − seed: +0.047, CI [−0.011, +0.106].
  - Best − recipe: +0.036, CI [−0.021, +0.096].
  - Best − seed: +0.083, CI [0.019, 0.148].
- **Judges (Table 10).** In-family vs human: relevance 77.8% (κ 0.23),
  factual support 80.0% (κ 0.60). Out-of-family: 91.1% (κ 0.46) and
  80.0% (κ 0.61). On 12 tasks the out-of-family gain is +0.129, against
  +0.069 in-family.
- **Error analysis (Table 12).** The remaining weak citations are mostly
  "source does not support the specific claim" (173) and "topically
  related but not the exact fact" (165). Regressions are mostly
  answer-gate flips from a more assertive answer-first style (gate gained
  12 tasks and lost 6 on the 200-task subset).
- **Cost.** About 40M policy tokens for the search. Per-query tokens rise
  ~32% (146k → 193k, almost all input), and latency does not rise (6
  queries per prompt).

## Critique / open questions

- **House-pattern check: the paper reads BELOW the digest framing.** The
  digest repeats the abstract. The body supports the main transfer claim
  as stated and is weaker on three of the digest's four other claims.
  - **Most-favourable-cut check: the headline passes.** The +0.079 in the
    abstract is seed 0, the *smallest* of three seeds. It is a paired test
    on 500 never-touched tasks, and the abstract also gives the mean
    (+0.107). This is the rare headline that is the least favourable cut.
    One soft spot: the CI bootstraps over tasks, not topics (~2 tasks per
    topic), so it is slightly too narrow.
  - **"Decomposed feedback is causal" is a 2-seed direction.** Each
    verifier arm is 2 seeds on 120 tasks with no CIs, and Section 6 calls
    the cells "directional". The scalar arm's −0.021 is one seed that
    found no accepted edit (≈ 0) plus one at about −0.035. The robust
    reading: a scalar gave about no transfer, decomposed feedback gave
    +0.06–0.08. "Transfer turns negative" overstates it. The gaming-gap
    ordering (0.071/0.061 vs 0.043) is inside the seed spreads (up to
    0.079).
  - **The "hand-written recipe" is neither hand-written nor
    significant.** It is the optimized prompt with the answer-key blocks
    deleted. A person did not write it without seeing the search output.
    Its own gain over the seed, +0.047 [−0.011, +0.106], is not
    significant at n = 100. So "largely reproduced" is a 57% point
    estimate whose CI includes zero. It shows the deleted content adds
    little. It does not show that the search was unnecessary, which would
    need a recipe written before the search. The digest's link to the
    08-23 `/elevate` paired-control proposal is loose. That proposal
    re-scores a known-good *accepted* proposal so a broken gate shows
    itself. It is a positive control on the gate, not a hand-made rival
    to the optimizer.
  - **The memorization-vs-selection contrast is settled by the split, not
    by the ablation.** Table 1 gives train and validation zero shared
    topics. The answer key holds *training* answers. So it could not have
    raised the *validation* score, unless the reflector also saw
    validation traces, which the paper does not say. The +0.202 → +0.090
    comparison ("would approach 0.202") is therefore not the right
    counterfactual. The ablation's real finding is narrower and still
    useful: the key does not help even on the questions it encodes.
    Selection noise is then the explanation left by elimination. It is
    consistent with the 24-item validation set and with
    [[literature/papers/hu2026winners]]'s GEPA numbers (+38.8 claimed vs
    +19.2 real at 16 items). But no larger validation set or repeated
    scoring was run to test it, as the authors say in Section 6. "Is
    small-sample selection" should read "is most likely".
- **Why the key is inert may be task-specific.** On the exact training
  questions the seed already passes the answer gate 92% of the time. The
  reward is dominated by per-citation levers that a memorized answer does
  not improve, so there is little headroom for a lookup to pay off. The
  authors expect memorization to pay more in narrow, entity-recurrent
  domains and do not test this. The detection protocol transfers. "The
  key is harmless" does not.
- **The out-of-family and human checks are tiny.** The gain under Claude
  Opus 4.8 is measured on 12 tasks. The human check is 45–48 pairs from
  one author. Relevance-judge κ is 0.23, so the relevance lever, worth
  0.30 of the reward, is weakly validated. All four citation judges and
  the policy are one vendor.
- **The per-lever gains are not independent.** link_works,
  relevant_content and fact_check each move about +0.09, and all three
  are multiplied by the same answer gate.
- **Tension with [[literature/papers/park2026when]] is mostly apparent.**
  Park's "sign-only ≈ full feedback" compares a bare accept/reject with
  the gate's *numbers*, under a noise-free oracle. Here the comparison is
  a scalar against *named, per-citation diagnostic reasons* fed to the
  reflector. Both results fit one reading: a number adds little over a
  sign, and a diagnosis of *why* a candidate failed is what carries
  information. The open contrast is diagnostic text vs scalar under a
  *noisy* judge. Only this paper tests it, and only at 2 seeds.
- **Domain fit.** One agent, one citation task, one frozen news corpus.
  The artifacts are system prompts, not code or experiments. It is the
  closest graph source to optimizing a research agent's own instructions,
  but the reward is answer-plus-citation quality, not research progress.

## Trust signals

- **Credibility:** 3.
  - For: the design is careful well above the median preprint here. There
    is a 500-task topic-disjoint held-out set, paired statistics, three
    seeds with the conservative one as headline, single-knob verifier
    ablations, verbatim judge prompts and weights, a judge-vs-human check
    with an out-of-family judge, and candid limitations. Section 6 itself
    flags the 2-seed ablation and the small judge check.
  - Against: no affiliations anywhere in the PDF or on the abs page. No
    locatable code or artifact link, although release is claimed. Not
    peer reviewed. Four authors with heavy self-citation of companion
    papers. Small cells (12-task judge check, 100-task recipe test,
    2-seed ablation). There is a reporting slip (48 vs 45 pairs).
  - By the letter of the rubric (unknown group, no code) this is a 2. It
    is lifted to 3 for disclosure and design. It is not 4 because nothing
    can be reproduced from a public artifact.

## Follow-up

- **Relevance:** 4. It is the first graph source to run a held-out audit
  on prompt optimization of a *research agent*, not a classifier. It
  confirms [[literature/papers/hu2026winners]]'s validation overstatement
  at 24 items (2.6×) in an agentic, LLM-judged setting. It adds a
  single-knob test of what the evaluator should return (decomposed
  diagnosis vs scalar), and a prompt autopsy that catches literal
  answer-key memorization. Not 5: one task, 2-seed mechanism evidence,
  and no ML-research loop.
- **For [[concepts/hce-evaluation]]:** use a topic- or entity-disjoint
  split, not item-disjoint, whenever the optimizer reads training traces.
  Report the least favourable seed with the mean. Run a prompt autopsy
  for verbatim training content alongside the held-out number.
- **For [[concepts/programmable-evaluator-oracle]]:** decomposed,
  reason-bearing feedback joins li2026discover's Audit arm and
  xu2026conflictguide's probes. Here the effect is on transfer, not only
  on the validation score.
- **For [[concepts/skill-library-lifecycle]]:** the answer key is the
  hard-coding that [[literature/papers/yang2026skillopt]]'s update rule
  forbids. The authors name SkillOpt's validation-gated, no-hardcode rule
  as the fix they did not run. A token-overlap novelty penalty between
  candidate prompt and training questions is their cheaper in-run guard.
- **Candidates** (cited, not in the graph):
  - Onweller et al. 2026, citations present but unverified in
    deep-research agents.
  - Leung et al. 2026, LLM rubric verifiers vs human labels for
    deep-research source attribution.
  - Fu et al. 2026, prompt-level overfitting.
  - Bai & Shi 2026, MAS-PromptBench (arXiv 2606.23664).
  - Decagon 2026, "Optimizing GEPA for production" (blog).
