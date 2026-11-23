---
kind: paper
title: "How Feedback-Free Self-Evolution Reshapes Agent Skills: A Systematic Study"
authors: ["Litao Guo", "Man Chen", "Xinli Xu", "Jinsong Zhou", "Zixin Zhang", "Ying-Cong Chen"]
institutions: ["The Hong Kong University of Science and Technology (Guangzhou)", "The Hong Kong University of Science and Technology", "Knowin"]
year: 2026
venue: "arXiv 2611.25534 (cs.AI), v1 2026-11-19, preprint (9-page main text + 14-page appendix)"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.25534"
code_url: "https://github.com/LitaoGuo/How-Self-Evolution-Reshapes-Skills"    # named in the abstract, not opened at ingest
citations: null
source: "raw/papers/guo2026how.pdf"
added: "2026-11-23"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/intent-level-briefing]]"
  - "[[concepts/constraint-pinning]]"
tags: ["feedback-free-rewriting", "skill-evolution", "model-prior", "self-written-skills", "item-level-retention", "restoration-test", "deletion-test", "skilldiff", "skillsbench", "logicbench", "medcalc-bench", "tau2-bench", "iterated-learning", "instruction-compression", "llm-judge", "negative-result"]
---

# How Feedback-Free Self-Evolution Reshapes Agent Skills: A Systematic Study

## TL;DR

Six models each rewrite expert-written skills 30 times in a row, with
the prompt "identify problems and improve the skill". They see only the
skill and a one-sentence topic: no task, no run, no judge. The same
model then runs as an agent with the rewritten skill. Each model's
**prior** for a skill is sampled as ten skills it writes from the topic
alone. An LLM judge splits every original skill into items and counts
how many of the ten self-written skills state each item: 0/10 means the
item is outside the prior.

Four results matter for this graph:

1. **Rewriting costs pass rate, and the first round does half the
   damage.** On SkillsBench (87 tasks) the mean loss is 7.1 points after
   round 1, 10.9 after round 5 and 14.6 after round 30 (40.2% → 25.7%).
   All 18 model × benchmark cells are negative at round 30. After 30
   rounds, LogicBench's skills are worth what no skill is (71.4% vs
   71.4%). MedCalc-Bench's fall *below* no skill (78.7% vs 81.1%).
2. **The loss is selective.** After 30 rounds, 23.1% of 0/10 items
   survive against 84.2% of 10/10 items. The gap shows up in round 1:
   on SkillsBench, 49.5% of 0/10 items are lost against 23.0% of the
   rest. It holds within every kind of item (list entry, paragraph, code
   block, heading), under three judges and across repeated chains. The
   lost outside-prior items rarely come back: 6% are back by round 30,
   against 30% of the others.
3. **The outside-prior content carries the value.** Three tests point the
   same way:
   - *Restoring to round-30 skills.* Putting only the 0/10 items back
     recovers 8.8 of SkillsBench's 14.6 lost points; the other lost
     items recover 4.4, and all lost items 10.2.
   - *Deleting from the original skills, word-matched* (two models, App.
     B.6). Deleting ~23% of the body from 0/10 items costs 4.6 points;
     deleting the same number of words from ≥1/10 items changes the pass
     rate by +0.6.
   - *Repeated runs.* The same pattern holds over 16 full evaluations.
4. **SkillDiff** tags the 0/10 items in place ("[FOCUS]") and adds a
   short note asking the agent to carry them out as written. The skill's
   text is otherwise unchanged. It gains +4.8 points on SkillsBench on
   average, but +1.1 to +3.4 for four of six models, with one run and no
   neutral-note control.

## Claims

- Without feedback, a rewrite can draw only on the skill and on the
  model's prior. So rewriting pulls a skill toward what the model would
  write by itself, and gives up first what only the expert supplied.
- An expert skill's value lies largely in knowledge outside the model's
  prior. Losing it causes much of the decline (a causal claim, resting on
  restoration and deletion).
- About half of an expert skill is outside a given model's prior (0/10
  items: 2,757–3,419 of 6,605 SkillsBench items per model, 42–52%). That
  set is largely shared across models: 82% of one model's 0/10 items are
  also 0/10 for another, against 48% expected by chance. 1,866 items lie
  outside every model's prior.
- The agent may pass over outside-prior items when it follows a skill
  and fall back on its default procedure. SkillDiff targets this. The
  claim is asserted, not measured: no trajectory analysis.
- Future work (App. D), explicitly untested: a rewrite that leaves the
  0/10 items untouched and edits only the rest. The self-written skills
  locate those items without a run, so such a guard needs no feedback.

## Methods

- **Models and harnesses.** GPT-5.6-Luna and GPT-6.1-Sol in Codex
  (codex-acp 1.6.2 / 2.1.1); MiMo-V2.5-Pro, Qwen3.7-Plus and
  Qwen3.8-Flash in OpenHands; Claude Haiku 5.5 in Claude Code
  (claude-agent-acp 0.40.0). Agents run at the lowest reasoning setting.
  No Opus- or Sonnet-class Claude model is tested.
- **Rewriting.** Each round gets the skill text, a one-sentence topic, a
  length target (not stated; self-written skills average 1,031 words vs
  965 for the originals, so it is near original length) and "identify
  problems and improve the skill". Thirty recursive rounds.
  - The GPT models rewrite in Codex at high effort with their default
    tools, which include **web search**.
  - MiMo and the Qwen models rewrite in one API call with no system
    prompt.
  - Haiku rewrites in Claude Code with all tools off and an empty system
    prompt.
  - Only the skill document is rewritten. Scripts and reference files
    are unchanged.
- **Benchmarks.**
  - **SkillsBench:** 87 tasks, 8 domains, containerized, deterministic
    verifiers, skill loaded on demand. For the 63 tasks with several
    curated skills, a model picks one skill before any run, so each task
    tests one skill. The paper names the picker "GPT-5.6-Sol", which may
    be a typo.
  - **LogicBench** (760 questions, 19 skills) and **MedCalc-Bench**
    (1,100 questions, 55 skills): from SRA-Bench, gold skill in the
    prompt, one call per question.
  - **τ²-bench**, appendix only: 164 airline and retail tasks, the
    domain policy rewritten as the skill, two GPT models.
  - Evaluated rounds: 0, 1, 5, 10 and 30. **One run per task per
    condition** in the main text.
- **Prior and distance.** Ten self-written skills per skill and model,
  from the topic and the same length target. Distance is the mean
  TF-IDF cosine distance to the ten, normalised by their mean pairwise
  distance; a held-out self-written skill scores 1.00. The movement is
  repeated with Qwen3-Embedding and raw word frequencies.
- **Items and judge.** Mechanical split: every non-blank line belongs to
  one item. SkillsBench has 6,605 items in 83,950 words; LogicBench and
  MedCalc-Bench 2,376.
  - GPT-5.6-Luna judges every model's items, in one call per skill and
    candidate document.
  - "Stated" requires the same content; shared nouns, paths or numbers
    alone don't count.
  - Kept = KEPT or REWORDED. Lost = WEAKENED, DROPPED or CHANGED.
    UNCLEAR items are excluded.
- **Restoration.** GPT-5.6-Luna finds and removes the passage carrying
  each lost item's altered version. That gives a common base. The three
  versions then add back the 0/10 lost items, the ≥1/10 lost items, or
  all lost items, in original order. Word counts of the first two are
  **not matched**.
- **Deletion (App. B.6).** Delete ~quarter of each original SkillsBench
  body, either only 0/10 items or only ≥1/10 items, with **matched word
  counts** (23.4% / 23.1% of the body for Luna, 22.5% / 22.2% for Sol).
  Run once per task, in the same batch as the original.
- **Robustness (App. A.3–B.7).**
  - Judge checks on 20 SkillsBench tasks (1,800 items) with a fixed
    seed: second judge GPT-6.1-Sol, third judge Claude Haiku 5.5, and a
    second run of the main judge. Plus two known-answer controls.
  - Prior stability: 30 more self-written skills (Luna, SkillsBench).
  - Web search off during rewriting (Luna, MedCalc-Bench).
  - Repeated agent runs: Luna ≥16 runs per task, Sol ≥4.
  - Repeated rewriting chains: 3 per skill on LogicBench and
    MedCalc-Bench; 10 per policy on τ².
  - Cross-model swap of round-30 skills.

## Results

- **Pass rate by round (Table 4, Fig. 1).** Mean change from the
  original skill:

  | | R1 | R30 |
  |---|---|---|
  | SkillsBench | −7.1 | −14.6 |
  | LogicBench | −5.2 | −14.7 |
  | MedCalc-Bench | −4.9 | −11.1 |

  - Per-model SkillsBench R30 ranges from −10.3 (Qwen3.8-Flash) to
    −23.0 (GPT-5.6-Luna); Haiku 5.5 is −12.6.
  - Not every round-1 cell is negative: MiMo +2.3 on SkillsBench.
    Qwen3.8-Flash loses less by R30 (−10.3) than at R1 (−13.8).
- **Distance (Table 5, Fig. 2).** Mean TF-IDF distance to the prior:
  1.37 (original) → 1.19 (R5) → 1.22 (R30). Embeddings: 2.24 → 1.78 →
  1.93. Word frequency: 2.42 → 1.62 → 1.77.
  - By R5 the rewrite is closer than the original in 18/18
    model × benchmark pairs (TF-IDF and word frequency) and 17/18
    (embeddings).
  - The distance flattens after R5 while pass rate keeps falling, so
    whole-text distance does not predict the loss.
- **Item retention (Tables 1, 6, 7).**
  - After 30 rounds, 0/10 vs 10/10 items kept: SkillsBench 19 vs 83,
    LogicBench 25 vs 84, MedCalc-Bench 25 vs 85.
  - Lost after round 1, 0/10 vs ≥1/10: SkillsBench 49.5 vs 23.0,
    LogicBench 33.4 vs 11.0, MedCalc-Bench 36.0 vs 14.1. After round 30:
    81.3 vs 41.2, 75.7 vs 31.5, 73.6 vs 33.7.
  - Within item kinds on SkillsBench, the gap is +31.6 (code blocks) to
    +39.4 (paragraphs) points.
  - Of the lost items, 0/10 ones are more often dropped outright (53% vs
    40%) and less often weakened (29% vs 42%) (Table 12).
- **Judge (Tables 8–9).**
  - Agreement with the main judge: second judge 89.7% on self-written
    labels (κ 0.78) and 87.2% on rewrite labels (κ 0.74). Rerun κ 0.84 /
    0.79. Haiku κ 0.72 / 0.64.
  - The 0/10 − ≥1/10 loss gap is 49.5, 52.5, 51.4 and 40.7 points under
    the four judgments.
  - Known-answer controls: an identical text loses nothing. Against an
    unrelated skill, 1.3% and 1.5% of items are labelled kept.
- **Prior stability (Tables 10–11).** Further samples of ten agree with
  the original classification on 88% of items (κ 0.76). An original 0/10
  item appears in 1.1% of 30 further self-written skills, a 10/10 item
  in 95.1%.
- **Restoration (Table 2, Fig. 4).** Pass rate (original / R30 / 0/10
  restored / ≥1/10 restored / all restored):
  - SkillsBench: 40.2 / 25.7 / 34.5 / 30.1 / 35.8.
  - LogicBench: 86.2 / 71.4 / 80.4 / 75.0 / 80.6.
  - MedCalc-Bench: 89.7 / 78.7 / 83.1 / 80.0 / 84.4.

  Our arithmetic: restoring 0/10 items recovers ~61%, 61% and 40% of the
  loss. Adding everything else back gains at most 1.3 more points.
- **Deletion from the original (Table 17).** 0/10 deleted: −4.6 (Luna
  −5.7, Sol −3.4). ≥1/10 deleted: +0.6 (Luna +2.3, Sol −1.1).
  Difference −5.2 (−8.0, −2.3). One run per task, two models.
- **Repeated runs (Figs. 10–11).**
  - Luna: original 39.9% vs R30 26.7% over 16 full evaluations. The R30
    skill is lower in 16 of 16 (Sol: 4 of 4, 52.9% vs 40.2%).
  - **Full-evaluation SD is 3.9 (Luna) and 4.6 (Sol) points.**
  - Restoring 0/10 items: +10.4 / +7.9; ≥1/10 items: +3.2 / +2.9.
  - pass@16: 75.3% vs 63.2%. 14 tasks are solvable only with the
    original skill, 2 only with the rewrite.
- **τ² (Fig. 8).**
  - Luna: 64.0 → 25.6. Sol: 72.6 → 18.9.
  - Restoring 0/10 policy items: 48.2 and 37.8. Restoring ≥1/10 items:
    23.2 and 19.5.
- **Web search off (Fig. 9).** The round-30 loss is 7.0 points vs 4.3
  with search on (Luna, MedCalc-Bench). Search returns part of what
  rewriting drops.
- **Cross-model swap (Table 18).** Luna: original 36.8, own rewrite
  31.0, Sol's 27.6. Sol: original 44.8, own 37.9, Luna's 42.5. Every
  cell is below the original (mean −6.0).
- **SkillDiff (Table 3).** Gain over the original in the same batch:

  | Model | Δ pass rate (pp) |
  |---|---|
  | GPT-5.6-Luna | +12.6 |
  | GPT-6.1-Sol | +5.7 |
  | MiMo-V2.5-Pro | +3.4 |
  | Qwen3.7-Plus | +2.3 |
  | Claude Haiku 5.5 | +3.4 |
  | Qwen3.8-Flash | +1.1 |
  | **Mean** | **+4.8** |

  - Cost per skill: 1 topic call, 10 generation calls and 10 judge calls.
  - It adds 31 tags and a note on average, 129 words in all.

## Critique / open questions

- **Against the digest entry: at on the summary, with one overstatement;
  the *Why* line misreads the box's practice.**
  - The summary's numbers are in the PDF and quoted correctly.
  - **"SkillDiff ... adds +4.8 points"** is stated without its caveats.
    - It is one run per task, SkillsBench only, with no interval, while
      a full evaluation's own SD is ~4 points (Luna, Sol).
    - Luna's +12.6 drives the mean. Without it the mean is +3.2 (our
      arithmetic), and four of six models gain ≤ +3.4.
    - There is no neutral control: the note alone, or tags on random
      items. [[literature/papers/mitra2026agent]] found a fixed neutral
      note worth +4.5 on held-out BFCL. So +4.8 cannot yet be credited
      to *which* items are marked.
  - **The *Why* line: "The 11-14 trim and `/elevate`'s text budget
    rewrite instruction prose with model help ... an unguided rewrite
    erodes exactly that."** Three problems:
    1. **The trim was not this paper's condition.** Per
       [[docs/system-proposals/2026-11-14-system-review-intent-briefing]],
       it was one user-directed pass with a stated method:
       - Machine-read contracts were found first, by reading every
         consumer.
       - Per file it *kept* "the contracts, the behaviour a strong model
         would not choose unprompted, and the reason for each
         constraint". It *cut* procedure, history, restated rules and
         "does NOT do" lists.
       - Six trimmed skills were behaviour-tested on Sonnet.
       - The 11-15 `/elevate` run audited that every accepted contract
         survived.

       The paper's rewriter gets "identify problems and improve", has no
       keep list, no human and no test, keeps the length near the
       original, and recurses 30 times. The paper never tests a goal-
       directed or compressing rewrite. Its App. D names the
       preserve-outside-prior rewrite as future work.
    2. **The text budget is not a rewriter.** It makes a proposal state
       its net word change and name an offsetting cut. A human then
       accepts or rejects the proposal. The nearest analogue to the
       paper's iterated rounds is the offsets: small model-chosen cuts
       to the same files, cycle after cycle, with no task run. That
       drift is real but slow, and a reviewer is in the loop.
    3. **The best-matched evidence in the paper *supports* the trim's
       criterion.** The digest missed it.
       - App. B.6 deletes ~23% of each skill. Deleting within-prior
         (≥1/10) items costs nothing (+0.6); deleting outside-prior
         items costs 4.6. That is P2's "leave out what it will" tested
         on agent outcomes, and it is what the trim said it did.
       - The paper also finds that about half of an expert skill is
         outside the prior, which is the scale of cut the trim made
         (−47%).
  - What the paper does rule in for the trim is a **risk to its
    execution, not its criterion**:
    - When a model re-emits a text, outside-prior items are lost at
      about twice the rate of the rest from the first round (49.5% vs
      23.0% on SkillsBench).
    - The trim classified "what a strong model would not choose" by
      model judgment, not by sampling.
    - It cut 47%, twice B.6's 23%.
    - Its tests checked contract conformance on six skills. They did not
      check task outcomes, and eight skills were not run at all.
  - Whether the trim kept its outside-prior items is **untested and
    testable** (see Follow-up).
- **"Outside the prior" is partly confounded with "specific".** The
  judge counts an item as stated only when the content matches; shared
  paths or numbers don't count. So 0/10 items are disproportionately
  concrete commands, paths, parameters and thresholds, which anyone
  would expect to carry a skill's value. Table 7 controls for item
  *kind*, not for specificity. The paper's mechanism ("the prior") and
  a plainer one ("concrete details are what rewriting smooths away")
  make the same predictions here.
- **The prior is textual, the guide's default is behavioural.** "What
  the model writes from the topic" is not "what the agent does without
  being told". SkillDiff assumes they coincide (agents skip 0/10 items),
  but no trajectory shows it. The deletion test is the only behavioural
  bridge: two GPT models, one run each, and an effect (−5.2) about one
  full-evaluation SD.
- **≥1/10 is a loose definition of "within the prior".** It includes
  items written by only 1 of 10 samples. That makes the deletion result
  stronger, since even rarely-written items were safe to cut. But it
  means the clean "the model would write it anyway" set (10/10, 19% of
  Luna's items) was never deleted alone.
- **The starting skills are curated expert skills** (+16.6 points over
  no skill on SkillsBench, per the cited benchmark). The paper motivates
  rewriting with stale, misleading or over-long skills, but never tests
  rewriting a *bad* skill, where feedback-free editing could help.
- **No intervals in the main tables**, and single runs for four of six
  models. Some round-1 per-model losses (Haiku −2.3, Sol −8.0 on
  SkillsBench) are within ~2 SD of one evaluation. The direction holds
  under repeats; the per-model sizes are loose.
- **The judge is also a rewriter** (GPT-5.6-Luna judges all models).
  Cross-judge checks cover 20 tasks and only Luna's skills. The Haiku
  judge's κ (0.64 on rewrites) is the weakest.
- **Rewrite lengths are not reported**, so content loss and length change
  can't be separated round by round.
- **Weaker and cheaper models dominate.** The only Claude model is Haiku
  5.5. Whether an Opus-class rewriter has a broader prior, and so loses
  less, is open. The cross-model overlap (82%) suggests the prior moves
  slowly with model family.
- **Consistent with neighbours in the graph.**
  - [[literature/papers/so2026masked]]: proposers asked to make a harness
    cheaper mostly *rewrote* clauses, and 7 of 10 accepted rewrites
    relaxed a check.
  - [[literature/papers/zhou2026does]]: feedback-driven optimizers only
    add rules. This paper shows the feedback-free direction: rewriting
    erodes rules rather than adding them.

## Trust signals

- **Credibility:** 3.
  - For:
    - A reputable university group (HKUST-GZ / HKUST; corresponding
      author Ying-Cong Chen) and a named code repo.
    - Six models from four families, three benchmarks plus τ² in the
      appendix.
    - Three converging causal tests: restoration, word-matched deletion
      and repeated evaluations.
    - Three judges plus known-answer controls, prior stability checked
      with 30 more samples, repeated rewriting chains, a web-search
      ablation and a cross-model swap.
    - The abstract matches the body.
  - Against:
    - Not peer reviewed, four days old, no citations.
    - One run per cell in the main text, and no intervals outside the
      appendix.
    - The rewrite prompt and the length target are not quoted.
    - SkillDiff has no neutral control and no repeats.
    - The judge is one of the rewriters.
    - A possible model-name typo ("GPT-5.6-Sol").
    - Weaker models dominate.
  - Not 4: unreviewed, and the headline method result is
    under-controlled. Not 2: the causal tests and robustness checks go
    well beyond most preprints in the graph.

## Follow-up

- **Relevance:** 4.
  - It is the first evidence in the graph on what a feedback-free,
    model-written edit to instruction text loses, measured item by item,
    with a causal test.
  - Its word-matched deletion is the first outcome-level test of P2's
    "leave out what it will" half.
  - Material new evidence for [[concepts/skill-library-lifecycle]]
    (update and delete without feedback) and
    [[concepts/intent-level-briefing]] (P2, P6).
  - Not 5: not an ML-research agent, and it does not test a guided,
    goal-stated trim like the box's.
- **For `/elevate`'s evidence gate.**
  1. The gate's "removing, consolidating or clarifying can pass on a
     modest evidence base, because the downside is low" holds for
     *deleting* text the reader would supply itself (B.6, +0.6). It does
     not hold for model *rewording*. Rewording is where outside-prior
     items get weakened or changed: 29% and about 18% of the lost 0/10
     items (our arithmetic, Table 12).
  2. A text-budget offset should be a quoted deletion, not a rewording,
     so the reviewer sees exactly what goes.
  3. SkillDiff's +4.8 does not pass a gate as evidence for a
     "mark-the-important-items" mechanism. It lacks a neutral control
     and repeats, and Luna's +12.6 drives it.
  - One paper at credibility 3; any proposal should say so.
- **A cheap audit of the 11-14 trim (our inference, untested).**
  - For each trimmed skill or rule: have Opus write K self-written
    versions from the file's one-line description. Item-split the
    pre-trim text (claude-system@7656ad6^). Have a judge label which
    items no sample states.
  - Then compare the share of 0/10 items that survive in the trimmed
    text with the share of ≥1/10 items.
  - If the trim followed its criterion, 0/10 retention should be the
    *higher* one: the reverse of the paper's 23% vs 84%.
  - Cost: about 2K+1 calls per file; roughly 30 files.
  - It is a textual check. Outcome checks on the eight untested skills
    remain separate.
- **Candidates** (cited, not yet in the graph):
  - Xing et al. 2026, "What should a skill remember? Quality–cost
    trade-offs in cost-aware skill rewriting" (arXiv 2606.09421). Cited
    here for "rewrites that preserve the operational anchors keep quality
    under compression", which bears most directly on the trim. Also
    listed in zhou2026does's follow-ups.
  - Zhong et al. 2026, "SkillLearnBench" (arXiv 2604.20087). Accuracy
    rises, then falls over four rounds of feedback-guided revision.
    Declined from the 2026-07-27 digest.
  - Duan et al. 2026, "Repo2Skill-Evo: repository skills go stale in
    silence" (arXiv 2608.21964).
  - Li et al. 2026, "SkillsBench" (arXiv 2602.12670). The benchmark;
    cited in wang2026beyond and li2026who but has no note of its own.
  - Wu et al. 2026, downstream adaptation of agent skills (arXiv
    2607.03238). Yu et al. 2026, "Skill is not one-size-fits-all"
    (arXiv 2605.30723).
  - Shumailov et al. 2024 (Nature), model collapse; Kalish, Griffiths
    and Lewandowsky 2007, iterated learning. Background for the "drift
    to the prior" mechanism.
