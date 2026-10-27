---
kind: paper
title: "One Skill Takes Over: Clashing Agent Skills and a One-File Fix"
authors: ["Parsa Razmara", "Seyedarmin Azizi", "Ardalan Aryashad"]
institutions: ["University of Southern California"]
year: 2026
venue: "arXiv 2610.25064 (cs.AI), v1 2026-10-18, preprint (16 pages)"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.25064"
code_url: "https://github.com/prazmara/skillclash"
citations: null
source: "raw/papers/razmara2026one.pdf"
added: "2026-10-27"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/shared-skill-namespace]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/constraint-pinning]]"
tags: ["agent-skills", "skill-md", "skill-conflict", "co-installed-skills", "takeover", "worked-example", "few-shot-dominance", "meta-skill", "skill-merge", "priority-instruction", "instruction-hierarchy", "load-order", "rule-checkers", "ifeval-style", "activation-steering", "linear-probe", "synthetic-skills", "open-weight", "gpt-4.1", "claude-opus-5.5-as-merger"]
---

# One Skill Takes Over: Clashing Agent Skills and a One-File Fix

## TL;DR

SkillClash is 16 jobs (commit message, PR description, SQL query, Dockerfile,
…). Each job has two author-built skills, A and B. Each skill is a SKILL.md
with six code-checked rules and one worked example. Relative to its rival,
each skill has one contradictory rule, one shared, one entangled (both can
hold only with careful arrangement) and three exclusive rules. A stated
*primary* decides the contradiction, so a correct output follows ten
achievable rules. Both skill files are put in the system prompt with an
instruction to follow every rule of every loaded skill. Seven readers are
used: Qwen3-4B, Qwen2.5-7B, Mistral-7B, Llama-3.1-8B, Phi-4-mini,
Qwen3-235B and GPT-4.1. Decoding is greedy, with 8 requests per job (4 for
GPT-4.1). All intervals are 95% bootstraps over the 16 jobs, paired by job.

Three results matter for this graph:

1. **A same-job rival costs a third of a skill's rules, and the loss is
   all-or-nothing.** Alone, 93–99% of a skill's rules are followed. An
   unrelated skill changes nothing. A same-job skill drops this to 61–62% on
   every reader, from 4B to GPT-4.1. On the five rules per skill that can
   always hold, the drop is still 28–35 points. In 45–78% of outputs one
   skill's three exclusive rules are all kept and the other's all dropped
   ("takeover"), although those rules conflict with nothing.
2. **The worked examples drive it, and one shared example nearly removes
   it.** Removing both skills' examples cuts takeover to 7–41%. When only
   one skill keeps its example, that skill wins 67–93% of takeovers, and
   moving the example moves the winner (Phi-4-mini excepted). When each file
   instead ends with the same example that follows both skills, takeover
   falls to 1–15% and 90–96% of achievable rules are followed, still as two
   files.
3. **Instructions do not fix it; a strong-model merge does.** A priority
   line leaves achievable rules at 62–71% and takeover at 36–82%. "Apply
   both" works only on the two strong readers (88–89%, no takeover), and
   even there entangled rules break (fully correct 22% and 21%). A single
   merged file written by Claude Opus 5.5 reaches 96–99% on all seven
   readers with no takeover. Most of its gain over a plain rule list comes
   from its one correct worked example.

**Ingest check (digest): no frontier Claude model is among the seven
readers.** Claude Opus 5.5 appears only as the *merger*. It writes files
that the seven models read and never executes a skill itself. The only
Claude-as-reader evidence is App. F, an observational reanalysis of Yan et
al.'s public Claude Code run summaries. There, runs that used both skills
met the first skill's own requirements about 8 points less often than the
paired A-only run. The Claude model behind those runs is not named in this
paper. The strongest readers tested, Qwen3-235B and GPT-4.1, are 2025
models.

## Claims

- Takeover is not a context-length effect, since an unrelated skill
  changes nothing, and it is not a selection effect, since both files are
  in context. So "better skill selection would not fix it", and blocking
  one skill (Yan et al.'s hook) "gives up that skill's rules".
- The worked example is the mechanism: "an example that follows one skill
  causes takeover, and an example that follows both skills is the main part
  of a good merge."
- The winner "depends on the pair and the load order, not on the request".
  The first-listed skill wins 74% and 77% of takeovers on Llama-3.1-8B and
  GPT-4.1, but only 24% on Qwen2.5-7B and 3% on Phi-4-mini. "A user who does
  not control the load order cannot know which skill will be followed."
- A priority instruction fails on every model. This echoes Geng et al.
  (models often do not enforce stated priorities) and Camassa & Shiller (an
  in-context pattern overrides an explicit instruction).
- Merger quality follows merger strength, and the step-by-step merge "is
  only as good as its labels". Only Claude Opus 5.5 labelled the rule pairs
  well enough.
- Takeover "behaves like a choice between two alternatives". Activation
  steering can flip the winner but does not make the model follow both.
- Recommendation: merge the two skills once with the strongest available
  model, end the merged file with one correct example that follows both,
  and do not copy the source examples. Where merging is impossible, give
  each file such an example.

## Methods

- **Benchmark.** 16 jobs × 2 skills × 6 rules + 1 example. The authors
  constructed the skills rather than collecting them, "so that we control
  how their rules interact". Every rule has a program checker in the IFEval
  style. A self-test confirms that each example follows its own rules, that
  the contradiction and the entangled pair are real, and that a reference
  output exists for each choice of primary.
- **Label check.** One co-author who did not write the skills and had not
  seen the labels listed every interacting rule pair. They found all 16
  contradictory and all 16 shared pairs and listed no unplanned pair. Of the
  16 entangled pairs they labelled 8 as entangled and 4 as contradictory,
  and missed 4.
- **Measures.** *Rules followed* is the share of one skill's six.
  *Takeover* means one skill's three exclusive rules are all met and the
  other's are all missed. *Achievable* is the share of the ten achievable
  rules; following the primary alone gives 60%. *Fully correct* means all
  ten.
- **Placement.** Both files go in the system prompt inside `<skill>` tags,
  under a prompt that says every rule of every loaded skill must be
  followed. Putting both in the user message gives about the same drop
  (32–37 points on six readers). Each pair runs in both load orders and
  with each skill as primary.
- **Fixes.**
  - Two-file fixes: a priority line, and an "apply both" line ("complementary,
    not alternatives").
  - One-file meta-skills:
    - a rule union by code, with no model and no examples, plus a line
      naming the conflict winner;
    - a GEPA/SkillOpt-style feedback loop: 6 rounds of 3 practice requests,
      with checker feedback, run by Qwen3-4B;
    - a plain merge and a step-by-step merge. The step-by-step merge lists
      rules, labels interacting pairs, writes one rule per entangled pair and
      copies the rest. Each reader merged for itself, and Claude Opus 5.5 was
      an eighth merger.
- **Mechanism probes (Qwen3-4B, Qwen2.5-7B, Llama-3.1-8B).**
  - A linear probe at the last prompt token predicts the winner.
  - Single-layer activation steering toward the losing skill, with the
    layer and strength tuned on 4 requests per job and reported on the other
    4. Each direction has a same-norm random direction as its control.
- **Runs.** Qwen3-235B and GPT-4.1 ran via OpenRouter at temperature 0.
  The others ran on a cloud GPU with pinned revisions. All answers were
  re-scored from the saved generations with no difference. Code and data
  are released (link named in the PDF; not opened at ingest).

## Results

- **Single vs paired (Fig. 2a).** Rules followed: alone 93–99%, + unrelated
  skill unchanged, + same-job skill 61–62% on all seven readers.
- **Takeover with full skills (Tables 1–3):**

  | Reader | Both loaded, achievable | Takeover |
  |---|---|---|
  | Qwen3-4B | 58.1 | 78% |
  | Qwen2.5-7B | 58.6 | 69% |
  | Mistral-7B | 60.3 | 45% |
  | Llama-3.1-8B | 59.1 | 66% |
  | Phi-4-mini | 59.9 | 49% |
  | Qwen3-235B | 59.6 | 73% |
  | GPT-4.1 | 58.9 | 77% |

- **What triggers it (Table 6, five open readers + Qwen3-235B).**
  - Removing examples: takeover 7–41%.
  - Removing the contradictory and entangled rules as well, leaving fully
    compatible skills: takeover still 4–23%. Four of six readers follow both
    skills' exclusive rules in full 24–33 points less often than their
    single-skill runs predict, with CIs excluding zero.
  - Removing examples also lowers single-skill compliance on the weaker
    readers (Phi-4-mini 95.2 → 70.4%, Mistral-7B 92.4 → 81.4%).
- **File vs example (Fig. 3a).** Putting both full skills in one file,
  examples kept, still gives takeover 34–73%. Two files each ending in
  Claude's shared example give takeover 1–15%. On the strong readers that
  example lifts fully correct from 1% to 76% and 60%.
- **By job (Table 7).** Takeover is ≥75% in 5–12 of the 16 jobs, depending
  on the reader. For SQL it is ≤19% on every reader, where the two rule sets
  constrain different clauses and blend.
- **Fixes (Fig. 4; achievable rules, open readers / strong readers):**

  | Condition | Open (5) | Strong (2) |
  |---|---|---|
  | Both loaded | 58–60 | 59–60 |
  | + priority line | 62–65 | 63–71 |
  | + "apply both" | 69–78 | 88–89 |
  | Rule union (code) | 66–79 | 86–90 |
  | Rule union + feedback loop | 89–92 (Qwen3-4B's file) | not run |
  | Merged by the reader itself | 71–87 | 94–95 |
  | Merged by Claude Opus 5.5 | 96–99 | 99 |

  Claude's file gives 68–94% fully correct outputs, 11–29 points above
  "apply both". With the reader fixed at Qwen3-4B, merges by Qwen3-4B,
  Llama-3.1-8B, Qwen3-235B, GPT-4.1 and Claude Opus 5.5 score 79, 87, 94, 92
  and 98%.
- **The example is the active part (Fig. 3c, Fig. 5).**
  - Removing the example from Claude's file costs 9–25 points on the open
    readers and about 2 on the strong ones. It costs 14–52 points of fully
    correct outputs.
  - Adding Claude's example to the code-built union adds 9–24 points, to
    within 8 of the full file. On GPT-4.1, fully correct goes from 30% to
    89%.
  - Adding the *original* two examples to the union costs 0–14 points and
    brings takeover back to 32–55%.
  - Over all merged files, a correct example adds 12–17 points and an
    incorrect one adds roughly nothing. A correct example from another
    merger helps as much as Claude's (within 1 point).
  - The share of a merger's examples that meet every achievable rule ranges
    from Mistral-7B 0% to Claude 94%.
- **Labels (Table 4).** Claude labelled 48/48 interacting pairs right, with
  0 false pairs and 32/32 clean files. The seven other mergers labelled 2–39
  right, reported 8–61 false pairs and produced at most 14/32 clean files.
  Given the true labels, Qwen3-4B's step-by-step merge gains 8–10 points on
  every reader.
- **Feedback loop.** Qwen3-4B's union gains +13.8 [10.4, 17.5] after the
  loop. Practice requests chosen to provoke a clash do no better than plain
  ones (−0.2 [−2.1, 1.6]).
- **Steering (Fig. 6, Tables 8–9).**
  - A probe matches the trivial baseline on seen pairs (96–98%). On 16
    held-out pairs it does not beat "first-listed wins" on any reader.
  - A general steering direction flips 38–49% of takeovers; random
    directions flip 0–3%.
  - Both skills are followed in at most 13% of steered outputs. On
    Llama-3.1-8B a random direction does as well.
- **Real skills (Fig. 7).** In Yan et al.'s public Claude Code summaries,
  the both-skills runs show about −8 points on A's exclusive requirements
  against paired A-only runs, with a CI excluding zero. This is
  observational: which skills a run used was not randomized.

## Critique / open questions

- **Against the digest framing: at framing.** Every number in the digest
  entry is in the PDF and quoted correctly. The entry omits five things:
  1. No frontier Claude reader (see the ingest check above).
  2. Skill selection is removed by design. Both files are force-loaded,
     with an instruction to follow every rule.
  3. The skills are author-built, six rules and one example each.
  4. "Apply both" *does* work on the two strong readers, apart from the
     entangled rules.
  5. The 1–15% "one example satisfying both" condition uses the example
     from Claude's merged file. It is a strong-merger artifact, not a
     reader-side fix.

  None of these overturns a headline. The paper's own Discussion states
  most of them.
- **Who judged: code, not a model.** Every rule has a program checker, and
  achievability is proven by reference outputs. This is the strongest part
  of the design, and the reason the benchmark is clean where real skills
  are not. The authors say so: checking a merge of real skills "needs a
  judge model or hand-written checks". The rule-interaction labels that
  define "achievable" were validated by a co-author. The validation was
  blind to the labels but not independent of the team. It recovered only 8
  of 16 entangled pairs as entangled.
- **Effective n is small, and run-to-run noise is not measured.** Decoding
  is greedy, with one generation per prompt, and the winner is near-
  deterministic per pair and load order. The informative unit for takeover
  is therefore 16 jobs × 2 orders, not outputs. The authors resample over
  jobs, which is right. But no repeated-sampling arm exists. Kozyrev et al.
  (cited) found that repository-skill optimization gains could not be told
  apart from run-to-run noise. The ±1-point gaps here (step-by-step vs plain
  for Claude; another merger's correct example vs Claude's) should be read
  as ties. The large effects (−35, +29, takeover 78% → 1–15%) are far
  outside any plausible noise band.
- **Examples carry single-skill compliance too.** Removing examples cuts
  takeover, but on Phi-4-mini and Mistral-7B it also drops single-skill
  compliance by 11–25 points. So "remove examples" is not a free fix, which
  the paper's recommendation (one *correct* shared example) respects. The
  digest's "worked examples drive it" is right about takeover. It is not a
  case for example-free skills.
- **Ecological gap.** Real skills differ from these in several ways. They
  are longer, often have several examples or none, are loaded on demand
  (Claude Code puts only descriptions in context until the skill is
  opened), and rarely come with a planted contradiction. The no-contradiction
  rows (takeover 4–23% with fully compatible skills) partly answer the last
  point. Yan et al.'s 8-point observational gap is the only real-skill
  number, and it is smaller. Our inference: the mechanism (a worked example
  sets the output format wholesale) should transfer, but the 61–62% figure
  is an upper-end estimate for co-installed real skills.
- **Mechanism vs general few-shot dominance.** The paper does not test
  whether takeover is specific to *skill* files or is the familiar
  in-context-example-beats-instruction effect (Camassa & Shiller) applied
  to two documents. Swapping the worked example for a format template with
  no content would separate the two. So would an example that follows
  neither skill. Neither arm exists.
- **Fit with the graph.**
  - [[literature/papers/xu2026stale]] found that one instruction-tier
    sentence reversed a file-vs-code conflict (+87 points). Here a priority
    sentence barely moves a skill-vs-skill conflict (+2–11 points).
    Our inference: a sentence wins when its rival arrives at a lower rank
    (tool output), and loses when its rival is an equal-rank instruction
    with a worked example attached.
  - It also refines [[literature/papers/mcmillan2026instruction]]'s null.
    There, a contradicting AGENTS.md line did not move CLAUDE.md compliance
    (BF₁₀ 0.053). Bare contradicting rules are close to inert. A rival file
    with a worked example is not.
  - [[literature/papers/liu2026lost]] and
    [[literature/papers/giampieri2026cortex]] locate skill failure in
    *loading*. This paper isolates a failure that happens after loading
    succeeds.
  - [[literature/papers/zhang2026skillcomposer]] and
    [[literature/papers/pu2026skillops]] treat merge as a lifecycle
    operation and score it by task pass rate. This paper scores a merge
    rule-by-rule and shows that the merger and the example decide it.
- **The paper's proposed repairs are untested.** It proposes checking each
  combined rule and the example against the source rules before saving, and
  falling back to a plain merge when labels are in doubt. Both are "cheap
  and remain to be tested".

## Trust signals

- **Credibility:** 3.
  - For:
    - Program-checked rules with proven-achievable references; no LLM judge
      anywhere in scoring.
    - Paired, job-level bootstrap CIs, and a full condition table (App. B)
      with every interval.
    - Strong controls: an unrelated-skill arm, a user-message placement arm,
      ablations removing examples, contradictions and entangled rules, a
      blind label check, and same-norm random directions for steering.
    - The effect replicates across five families and a ~60× size range.
    - Generations were re-scored from saved outputs. Code and data are
      released (github.com/prazmara/skillclash, named in the PDF).
    - The authors state their own weak spots: 16 held-out pairs for the
      probe, the post-hoc best block, and the need for a judge on real
      skills.
  - Against:
    - Three authors from one USC group, not peer reviewed, 9 days old, no
      citations.
    - The steering citations are the authors' own (Azizi et al.,
      Saadatinia et al.).
    - The skills are synthetic and author-built. The label validator is a
      co-author. Greedy single samples.
    - GPT-4.1 ran on half the requests.
    - No frontier reader, and no agent harness: this is single-turn text
      generation, not a coding agent using tools.
  - Not 4 because there is no peer review or lab track record, and no
    in-harness or frontier-reader arm. Not 2 because the scoring is
    program-checked, the controls are well chosen, and the main effects are
    large and consistent across seven readers.

## Follow-up

- **Relevance:** 4. It is the first controlled measurement in the graph of
  what happens *after* two same-job skills are both in context, and the
  first to locate the cause in the worked example rather than the rules. It
  materially extends:
  - [[concepts/shared-skill-namespace]]: the open "conflict resolution"
    question now has a measured failure mode and fix.
  - [[concepts/skill-library-lifecycle]]: merge is an operation whose
    quality depends on who merges and on the example.
  - [[concepts/enforcement-boundary-placement]]: a priority sentence fails
    against an equal-rank rival.

  Not 5 because the readers are not frontier models, the setting is not an
  agent harness, and this box has no same-job skill pairs installed today.
- **For this box.** The user-level and project skill lists are the same
  set, so there are no rival same-job copies. Of the box's SKILL.md files,
  only `jobs` contains the word "example" at all. The exposure is therefore
  low now. It rises with plugin or third-party skills that overlap a house
  skill (a vendor review skill next to a house one), and with always-loaded
  instruction layers that both specify one artifact format with an example.
  Our inference: the `/curate` reader brief's "match these three exemplar
  notes" is the paper's mechanism used on purpose. The exemplars, not the
  template, will set the format. So exemplars must agree with the template
  and with each other.
- **Cheap local test.** Run the released SkillClash pairs with Claude Code
  and Opus or Sonnet as the reader, both force-loaded and loaded via the
  `Skill` tool. This answers the frontier-reader gap directly. It needs no
  GPU and costs 16 jobs × 8 requests × 2 orders × a few conditions of
  single-turn calls.
- **Candidates** (cited, not yet in the graph):
  - Yan et al. 2026, "One skill too many: how co-installed skills conflict
    in coding agents" (arXiv 2610.11647). It is the closest prior work: real
    public skills in Claude Code, about 1 run in 5 lost to the second skill,
    and a blocking-hook fix.
  - Huang et al. 2026, "SkillFusion: towards reliable multi-skill
    composition for LLM agents" (arXiv 2610.15481). It labels hard/soft
    conflicts and redundancy and compiles co-loaded skills.
  - Kozyrev et al. 2026, "Skill issue: lessons from optimizing repository
    SKILLs for coding agents" (arXiv 2609.12742). Gains within run-to-run
    noise. Already flagged from giampieri2026cortex.
  - Dong et al. 2026, "Agent skills can be harmful" (arXiv 2608.11888).
  - Song & Wei 2026, "More skills, worse agents? Skill shadowing"
    (arXiv 2605.24050).
  - Camassa & Shiller 2026, "Do as I say, not as I do: instruction-induction
    conflict in LLMs" (COLM Sci-FM workshop). This is the general
    example-beats-instruction effect the paper leans on.
  - Balp-Straffon et al. 2026, "How language models choose sides" (ICML
    MI workshop). Instruction-hierarchy winners read from activations.
