---
kind: paper
title: "Do Senior Agents Do Their Job? Measuring Role Integrity in How LLM Agents Use Subordinate Material"
authors: ["Jinxiang Xie", "Xinyi Zhang", "Changjiang Jiang", "Kexu Liu", "Qi Yang", "Yuxin Su"]
institutions: ["Nanjing University", "Sun Yat-sen University", "Wuhan University", "Independent Researcher", "The University of Hong Kong"]
year: 2026
venue: "arXiv 2611.27139 (cs.CL), v1 2026-11-20, preprint (10-page main text + 8-page appendix)"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.27139"
code_url: "https://github.com/jxtse/ri-bench"    # data at huggingface.co/datasets/jxtse/RI-Bench; neither opened at ingest
citations: null
source: "raw/papers/xie2026do.pdf"
added: "2026-11-24"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/intent-level-briefing]]"
  - "[[concepts/typed-claim-partition]]"
tags: ["role-integrity", "ri-bench", "senior-agent", "subordinate-reports", "majority-following", "limitation-loss", "relay", "conformity", "obligation-prompt", "intervention-evaluation", "two-tier-ratification", "harness-variance", "llm-judge", "human-calibration"]
---

# Do Senior Agents Do Their Job? Measuring Role Integrity in How LLM Agents Use Subordinate Material

## TL;DR

RI-Bench tests one step of a hierarchy: a senior agent turns fixed,
pre-written subordinate material into a decision, ruling or report. Each
scenario changes one property of that material, holds the case fixed, and
reads the senior's output. It does not score correctness. The three
scenarios are an ICLR area chair and program chair (*weigh* reviews), a US
Supreme Court judge (*verify* counsel against the record), and a UK
select-committee chair (*relay* witness evidence with its limits). Eight
models run in a read-only `pi` harness, under a Vanilla prompt (role and
task) and an Obligation prompt (role plus a 3–7 item duty list): 12,392
episodes.

Five results matter for this graph:

1. **Relaying loses qualifiers, and Opus 5 is not exempt.** A cited
   witness claim keeps neither of its two stated limitations in **0.47**
   of citations pooled under Vanilla and **0.33** under Obligation. Per
   model under Vanilla, Claude Opus 5 is lowest at **0.20** and
   GLM-5.3-Flash highest at 0.65. The absolute rate depends on who reads
   the label: **0.14 to 0.83** across three human annotators. The paper
   uses it only to compare models and prompts. Agents opened every file
   (median), so the loss is not a failure to read.
2. **Majority pull is real but mostly below the headline, and zero for
   Opus.** A generic 3:1 majority against the historical decision carries
   the program chair in 0.48 of papers pooled (Vanilla). But the same
   model, shown only the paper, would disagree with the historical
   decision 0.31 of the time. The excess pull is **+0.17**, from **0.00
   for Opus 5** to +0.47 for GLM-5.3. Under Obligation the pooled excess
   is −0.06. Removing every numeric score barely changes following
   (−0.03), so the pull comes from the reviews' stance, not their scores.
   Following rises with the size of the majority: 0.28, 0.53 and 0.62 at
   2:1, 3:1 and 4:1.
3. **A second tier mostly ratifies the first.** The program chair matched
   the area chair's recommendation in **933 of 944** adversarial episodes
   under Vanilla. Under Obligation it matched in 891 of 939, although it
   was told to "override an Area Chair recommendation when its reasoning
   … merely tallies votes".
4. **A checkable planted error rarely reaches the ruling, and this does
   not show verification.** 111 of 3,832 rulings (**2.9%**) adopted it.
   The human majority confirmed 5 of 15 sampled machine adoptions, so
   the real rate is likely lower. The cases are real Supreme Court cases,
   and the authors note that recall of the case cannot be told apart
   from checking the record.
5. **Duty lists reduce failures partly, unevenly, and with side
   effects.** Obligation lowers pooled majority following by 0.21 and
   limitation loss by 0.14. For Opus 5 it changes neither (−0.01, −0.00)
   but raises explicit correction of counsel's error from **0.53 to
   0.84**. It also lowers control-arm agreement with a correct specific
   majority, 0.99 → 0.91 pooled and 0.99 → 0.84 for GPT-5.6 Sol. Under
   human relabelling the relay gain shrinks from 0.14 to 0.08 (majority
   reading), and to 0.00 for one annotator.

## Claims

- *Role integrity* is whether an agent keeps the functional duties of its
  role when it acts on subordinate material. It cannot be read from the
  final answer, because "an agent that weighed the arguments may reach the
  same decision as one that merely counted votes". It has to be tested by
  intervening on the input.
- Across eight models, senior agents rarely adopt a planted procedural
  error but often follow a generic majority or drop witnesses'
  limitations. Every model shows each failure to some degree.
- Obligation prompts mitigate these failures without removing them, and
  also change control behaviour and which claims get cited.
- General capability does not order the results. The authors also concede
  that "differing rankings do not show independence from capability"
  (8 models, index not matched to reasoning setting).
- RI, the unweighted mean of the three complements, is a descriptive
  summary of avoided failures. It does not certify that a role was
  fulfilled.
- Hierarchy is not isolated as a cause. There are no matched
  non-hierarchical controls, and Court and Parliament have no interacting
  agents (authors' Limitations).

## Methods

- **Harness.** The senior role is a `pi` coding agent with only `read`,
  `ls` and `grep`, no memory or skills, and a fresh case directory with no
  outcome metadata. Nothing is preloaded, so what was opened shows in the
  trace. Reasoning effort is medium for every model; sampling is at
  provider defaults.
- **Models.** Claude Opus 5, GPT-5.6 Sol, Terra and Luna, GLM-5.3,
  GLM-5.3-Flash, Qwen3.5-35B-A3B and DeepSeek-V4.1-Flash.
- **Prompts (App. B, quoted in full).** Vanilla is the role sentence and
  the task. Obligation adds a numbered duty list. Examples: the area
  chair gets "Synthesize all reviewer arguments by evidential quality,
  not by counting votes". The committee chair gets "Distinguish contested
  evidence and uncertainty from established findings". There is no
  length-matched neutral addition.
- **OpenReview (weigh).** Real ICLR 2024–2026 submissions with full text
  and rebuttal. An area-chair agent writes a meta-review, then a
  program-chair agent decides with the same materials plus the
  meta-review. Claude Sonnet 5 regenerates the review texts in two
  styles: *specific* (cites paper content) and *generic* (templated). In
  a blind rating these score 5.00 vs 2.83 on specificity at 226 vs 233
  words.
  - *Real-vote design*, 119 papers. The real 3:1 split is kept, and the
    specific reviews sit with the majority or the minority.
  - *Constructed-vote design*, 118 papers (58 accepted, 60 rejected).
    The control arm has a specific 3:1 majority toward the historical
    decision. The adversarial arm has a generic 3:1 majority against it
    and a lone specific dissenter toward it.
  - Four controls: paper only (no reviews), no numeric scores, 2:2
    specific vs generic, and majority ratio and order (30 papers, four
    models).
- **Court (verify).** 120 Supreme Court cases with facts, question and
  both sides' argument excerpts. GPT-5.6 Sol plants one procedural-fact
  error (about 90% concern the lower court's holding or posture) in one
  side's excerpts. The record given to the judge stays correct. Rulings
  are labelled ADOPTED, CORRECTED, TRUTH STATED or SILENT.
- **Parliament (relay).** 120 inquiries (from 469 processed) with full
  written evidence. One headline claim from each of two opposed witnesses
  is rewritten by GPT-5.6 Sol to carry two limitations (time period,
  scope, certainty, source, sample size). Reports are labelled CITED WITH
  LIMITATIONS, CITED WITHOUT, or OMITTED. Loss counts cited claims only.
- **Scoring.** Per-case regex rules give a first label. Gemini 3.8 Flash
  (temperature 0) then classifies and must return a verbatim quote, which
  is verified. Only verified judgments override the regex. Verified:
  99.5% (Court) and 93.0% (Parliament). The two layers agree on only 4.5%
  of Court episodes, so the endpoint rests on the LLM layer.
- **Human calibration.** Three annotators (not authors, paid, ethics
  approved), blind, on stratified samples: 100 rulings, 100 reports and 30
  reviews, drawn from seven models before the DeepSeek run. A
  decomposed-question relabel pilot is in App. D.
- **Statistics.** Case-cluster bootstrap, 10,000 draws, fixed seed.
  Thresholds of 0.20 for a "substantive failure" were set in advance. No
  multiplicity correction, and model contrasts are labelled descriptive.
- **Harness check.** 800 extra episodes on a hard subset: the 20 cases
  per scenario where the other six models failed most. Sol runs under pi
  and Codex CLI, DeepSeek under pi and its own harness (dsh).
- **Cost.** About 1,300M input and 68M output tokens for the senior
  roles, about $3,000 at list prices, plus about $150 for the harness
  check and the judge.

## Results

- **OpenReview, constructed votes (Tables 1, 2, 13).** Adversarial-arm
  following, Vanilla → Obligation:

  | Model | Vanilla | Obligation | Excess over paper-only (V) |
  |---|---|---|---|
  | Claude Opus 5 | 0.20 | 0.19 | 0.00 [−0.07, 0.07] |
  | GPT-5.6 Sol | 0.51 | 0.25 | +0.16 |
  | GPT-5.6 Terra | 0.47 | 0.30 | +0.08 |
  | GPT-5.6 Luna | 0.50 | 0.36 | +0.14 |
  | GLM-5.3 | **0.71** | 0.33 | +0.47 |
  | GLM-5.3-Flash | 0.43 | 0.23 | +0.14 |
  | Qwen3.5-35B-A3B | 0.66 | 0.27 | +0.24 |
  | DeepSeek-V4.1-Flash | 0.38 | 0.29 | +0.17 |
  | **Pooled** | **0.48** | **0.28** | **+0.17** |

  - Control-arm agreement: 0.99 Vanilla, 0.91 Obligation. Sol drops to
    0.84, Luna to 0.83.
  - Under Obligation the paper-only reference is 0.34, so pooled
    following is 0.06 *below* what the paper alone would give. Sol (−0.19)
    and Qwen (−0.18) now side with the lone dissenter more than the
    paper alone would.
  - Shifts: 441 reference-consistent control decisions flipped to the
    majority under adversarial reviews, and 0 flipped the other way
    (Vanilla).
  - Paired across prompts (939 pairs): 234 follow under both, 222 only
    under Vanilla, 26 only under Obligation, 457 under neither.
  - 2:2 design: agents follow the specific side above the paper-only
    reference whichever way it argues (0.86 vs 0.69 when it agrees with
    history, 0.51 vs 0.31 when it does not).
  - Accepted vs rejected papers are followed about equally (0.50 vs 0.46,
    Vanilla).
- **OpenReview, real votes (Table 12).** Swapping argument quality under
  the real 3:1 split barely moved decisions. **691 of 711** Vanilla
  pairs were identical (12 vs 8 flips). Obligation gave 49 vs 22. This is
  why the votes had to be constructed.
- **Two-tier pass-through.** Program chair = area chair in 933/944
  (Vanilla) and 891/939 (Obligation). Qwen accounts for 28 of the 48
  Obligation disagreements.
- **Court (Table 14).** Adopted 111/3,832 (2.9%, CI 2.0–3.9). GLM-5.3,
  GLM-5.3-Flash and Qwen account for 81 of them. Most rulings state the
  true fact without naming counsel's error (TRUTH 0.86 Vanilla).
  Corrections, Vanilla → Obligation: pooled 0.09 → 0.25. Opus 0.53 →
  0.84, Sol 0.01 → 0.28, DeepSeek 0.10 → 0.47. Luna and Qwen stay at 0.00.
- **Parliament (Table 3).** Loss of both limitations among cited claims:

  | Model | Vanilla | Obligation |
  |---|---|---|
  | Claude Opus 5 | **0.20** | 0.20 |
  | GPT-5.6 Sol | 0.34 | 0.25 |
  | GPT-5.6 Terra | 0.44 | 0.26 |
  | GPT-5.6 Luna | 0.45 | 0.33 |
  | GLM-5.3 | 0.57 | 0.34 |
  | GLM-5.3-Flash | **0.65** | 0.42 |
  | Qwen3.5-35B-A3B | 0.60 | 0.61 |
  | DeepSeek-V4.1-Flash | 0.48 | 0.29 |
  | **Pooled** | **0.47** | **0.33** |

  - Citation coverage rises under Obligation (0.46 → 0.55). On the 723
    claims cited under both prompts, loss falls 0.46 → 0.33:
    limitations restored on 139, newly dropped on 51.
  - Selective citation: under Vanilla, 251 reports cite only the aligned
    witness and 164 only the opposed one (difference 0.091, CI
    0.016–0.164). Loss is not directional (0.49 vs 0.44, CI spans zero).
- **RI (Tables 4, 15).** Vanilla RI ranges from 0.55 (GLM-5.3) to **0.86
  (Opus 5)**. Obligation raises it for every model; Opus moves 0.86 →
  0.87. The spread comes from weighing and relaying, since every model
  keeps verification at ≥ 0.92.
- **Human calibration (§4.4, Table 9).** Semantic layer vs annotator
  majority: κ = 0.63 (Court) and 0.68 (Parliament). Reweighted to each
  reader:
  - Court adoption: 0.03 machine, 0.01 majority, 0.00–0.13 across
    annotators.
  - Parliament loss: 0.39 machine, 0.43 majority, **0.14–0.83** across
    annotators.
  - Obligation relay gain: 0.14 machine, 0.08 majority, 0.00–0.08 across
    annotators.
  - Obligation correction gain: 0.16 machine, 0.15 majority, 0.01–0.14.
  - One annotator departed from the guideline. Under that reading both
    Obligation effects vanish.
- **Pilot relabel of the reviews (App. D).** Two ML annotators checked
  the lone specific dissenter and one generic majority reviewer on 4
  papers. Point by point, the dissenter's claims were rated accurate more
  often. But taken as a whole, both annotators mostly judged the majority
  side better grounded (4/4 and 3/4). The authors conclude that "the
  specific dissenter is therefore not shown to be the better-supported
  side".
- **Harness (Table 5).** Sol pi vs Codex: RI changes +0.03 and −0.03,
  smaller than Sol's own same-case rerun drift (+0.08, +0.12). DeepSeek
  dsh vs pi: −0.08 under Vanilla, **+0.16 under Obligation** (CI
  0.07–0.25), almost all of it in relaying. The Obligation effect for
  DeepSeek is +0.26 in dsh but +0.02 in pi.
- **Reading.** The median episode opens every file in the case
  directory. Median wall time per role runs from 10 s (Luna program
  chair) to 240 s (Opus 5 in Parliament).

## Critique / open questions

- **Against the digest entry: below.** The numbers are in the PDF and
  quoted correctly, but the two headlines are worst cells and need their
  references.
  - **"Follow a majority of generic reviews up to 71%"** is GLM-5.3 under
    Vanilla. The raw following rate includes cases where the model would
    have disagreed with the historical decision from the paper alone.
    The cleaner measure, excess over paper-only, is +0.17 pooled, **0.00
    for Opus 5**, and −0.06 pooled under Obligation. The pilot relabel
    also does not show that the specific dissenter was right. So
    "following" is defined against a historical decision, which is a
    reference direction, not a gold label (the authors' own wording).
  - **"Drop witnesses' stated limitations from up to 65%"** is
    GLM-5.3-Flash under Vanilla. The absolute rate is not stable across
    human readers (0.14–0.83), and the paper says so. Opus 5 is at 0.20.
  - **"Rarely adopt a planted procedural error (2.9%)"** is correct, but
    it is not evidence of verification. Recall of real Supreme Court
    cases is a live alternative, and the authors say so.
  - **"Obligation prompts reduce these failures"** holds pooled under the
    machine label. Under the human majority the relay gain is about half
    (0.08), and it does nothing for Opus on weighing or relaying.
- **The fixed-material design is a strength and a limit.** It makes the
  input a controlled variable. But no subordinate is an agent, and the
  senior cannot ask a question. The constructed reviews, errors and
  limitations are all model-written.
- **Generator and subject families overlap.** Claude Sonnet 5 wrote all
  OpenReview reviews, and Opus 5 is the best subject there. GPT-5.6 Sol
  wrote the Court and Parliament interventions and is also a subject.
  Whether a model reads its own family's generic text differently is
  untested.
- **The weigh scenario's failure is not cleanly defined.** Constructed
  votes change scores, stance and specificity together. The controls
  separate them, but the new control reviews passed mechanical checks
  only, not the blind rating. And under real votes argument quality
  moved almost nothing (691/711), which the authors cannot interpret.
- **The relay endpoint is narrow.** It counts loss of *both* limitations
  among *cited* claims. Dropping one limitation, or omitting the claim,
  is not a loss under this metric. An agent that cites less can look
  better. Obligation raised citation by 9 points, and the paired
  analysis handles this only on the 723 claims cited under both.
- **The Parliament reference is weak.** 80 of 120 references are an LLM
  summary, and 76 cases are Special Reports. That matters only for the
  aligned/opposed split, not for limitation loss.
- **Human agreement is low.** Annotators agree "only slightly in Court
  and fairly in Parliament". The stratified samples over-represent rare
  labels, so κ is not a population accuracy.
- **No neutral prompt control.** Obligation adds 3–7 lines. A same-length
  addition with no duties would show how much is the content.
- **The two-tier result is under-analysed.** 933/944 pass-through is the
  paper's sharpest finding for hierarchies. But the program chair sees
  the same paper, reviews and rebuttal as the area chair, so agreement
  may be shared reading, not deference. There is no arm where the
  program chair sees no meta-review.
- **The harness result is mostly noise for Sol** and one large,
  prompt-dependent effect for DeepSeek. "RI belongs partly to the
  harness" is supported only for DeepSeek under Obligation.
- **Consistent with neighbours in the graph.**
  - [[literature/papers/singharoy2026quantifying]]: conclusions drop
    qualifications present in the cited set (*smoothing*). This paper's
    relay loss is the same failure at the sentence level, with the
    source opened.
  - [[literature/papers/chen2026signed]]: the parent's reading is the
    residual risk in a checked handoff. Here the senior's synthesis is
    where the failure sits.
  - [[literature/papers/sharma2026smsr]]: count-based aggregation fails
    under a repeated minority. Here an LLM aggregator follows counts as
    the majority grows, with scores removed.

## Trust signals

- **Credibility:** 3.
  - For:
    - Code, frozen per-episode results and a dataset are named (not
      opened at ingest).
    - Thresholds set in advance; case-cluster bootstrap with stated seed;
      no p-value claims beyond what the design supports.
    - Three blind human annotators, ethics-approved, with per-reader
      reweighting that the authors report even where it weakens their
      results.
    - A judge that must quote the output verbatim, from a family not
      among the subjects.
    - Paper-only, score-free, 2:2 and ratio controls on the weigh
      scenario; a harness rerun.
    - Unusually candid limitations and an abstract that matches the body.
  - Against:
    - Not peer reviewed, four days old, no citations.
    - Low inter-annotator agreement; the relay rate is reader-dependent.
    - Stimuli generated by subject families; no neutral prompt control.
    - Mid-tier academic groups; one Independent Researcher author.
  - Not 4: unreviewed, and two of three endpoints are only as reliable
    as a contested label. Not 2: the design and the honesty of the
    reporting are well above most preprints in the graph.

## Follow-up

- **Relevance:** 4.
  - First controlled test in the graph of what a *parent* does with its
    children's reports, with the input as the manipulated variable.
    Material new evidence for [[concepts/hierarchical-delegation]] (the
    synthesis step and two-tier ratification) and
    [[concepts/citation-anchoring]] (a resolved citation that drops the
    source's limits).
  - Tests a duty list on Opus 5, the closest model to this box's main
    agent. That bears on [[concepts/intent-level-briefing]].
  - Not 5: not an ML-research agent, one fixed step, no interacting
    subordinates.
- **The near miss 2611.22245** (*Multi-Agent Memory Inconsistency*; digest
  2026-11-23) reports that conflict detection falls at worker-to-lead
  handoffs. If ingested, it would be a second source on the parent's
  synthesis step.
- **Candidates** (cited, not yet in the graph):
  - Qu, Fu and Hu 2026, "Easier to mislead than to correct" (arXiv
    2606.01637), already cited by zhang2026review.
  - Hu and Qu 2026, "Social pressure breaks majority voting in LLM
    safety panels" (arXiv 2608.04415).
  - He et al. 2026, "Minority sentinel: when to overturn majority voting
    in multi-agent LLM debates" (arXiv 2606.29270).
  - Kasprova et al. 2026, sycophancy propagation in multi-agent systems
    (arXiv 2604.02668).
  - Cemri et al. 2025, MAST, "Why do multi-agent LLM systems fail?"
    (arXiv 2503.13657).
  - Han et al. 2026, conformity dynamics and topology (arXiv
    2601.05606).
  - Anthropic 2026, "Patterns and problems in emerging multiagent
    systems" (research note, 2026-08-13): hidden-profile failures, and a
    "CEO hierarchy" prompt that "did not make much difference".
