---
kind: paper
title: "Position, Role or Label? A Controlled Study of Instruction Conflicts in LLM Agents"
authors: ["Anish Laddha", "Nitesh Pradhan", "Gaurav Srivastava"]
institutions: ["The LNM Institute of Information Technology (LNMIIT), Jaipur", "Virginia Tech"]
year: 2026
venue: "arXiv 2610.33201 (cs.CL), v1 2026-10-23, preprint (55 pages)"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.33201"
code_url: "https://github.com/anishh15/WhoWins"
citations: null
source: "raw/papers/laddha2026position.pdf"
added: "2026-11-03"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/shared-skill-namespace]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/information-firewall]]"
tags: ["instruction-hierarchy", "instruction-conflict", "position-bias", "primacy", "recency", "message-boundary", "chat-role", "system-prompt", "tool-output", "memory-note", "declared-priority", "conflict-flagging", "luce-choice", "open-weight", "qwen3", "qwen3.5", "gpt-oss", "code-scored", "held-out-split"]
---

# Position, Role or Label? A Controlled Study of Instruction Conflicts in LLM Agents

## TL;DR

WhoWins is 1,080 code-scored scenarios in five task types: which tool to
call first, where to write a file, which output format, the order of
three steps, and whether to refuse a gated action. Each scenario has up to
five mutually exclusive one-line instructions. Each instruction is placed
in a "seat": operator (S1) and repository configuration (S2), both in the
system message (developer message on gpt-oss); user (S3); a tool result
returned by a `config_read` call (S4); and a saved memory note (S5). Seven
experiments keep the text fixed and change only how it is delivered. The
readers are 14 open-weight models: Qwen 3 (0.6B–14B), Qwen 3.5
(0.8B–122B-A10B) and gpt-oss (20B, 120B). Qwen runs with thinking off and
on, giving 26 model-modes. Every trial is sent three times, for 1,887,356
scored replies. No LLM judge is used.

Four results matter for this graph:

1. **Message boundaries flip the sign of order.** Inside one message the
   first of two conflicting lines wins 72.42–94.66% of decided replies.
   Split into separate messages, the first keeps only 22.95–62.65%, and on
   Qwen the later message wins most conflicts (59.77–89.05%). This holds
   in all 26 model-modes on both scenario halves.
2. **The system role does not protect a policy on Qwen.** A policy sent
   first in the system message wins only 6.07–17.35% of its conflicts on
   Qwen 3 and 11.66–41.50% on Qwen 3.5. On gpt-oss it wins 62.94–75.73%
   (of replies that act). So role rank is a property of the model family,
   not of the chat format.
3. **In tool and file-write tasks, the real system role is worse than the
   same text as a labelled user message**, by 6.41–48.75 points, in all
   26 model-modes. In format tasks there is no such drop (0 of 26). The
   paper says tool definitions share the system turn, so this cannot be
   separated from sitting next to them.
4. **Stated priorities are not reliable and conflicts go unreported.** A
   declared tier rule ("lower number wins") is followed in 50.78–98.51%
   of decided replies. No model-mode keeps even two tiers in order at 95%
   reliability. At most 2.92% of decided pair replies flag or hedge the
   conflict.

**Transfer check: no Claude or other hosted model is a reader.** All 14
readers are open-weight and served locally. Claude Opus 5 agents only
*wrote* the scenarios. The paper says its conclusions cover these three
families only (Limitation 1). Nothing here measures Claude Code.

## Claims

- An instruction's source changes three things at once: its position,
  the chat role that carries it, and the label that names it. A benchmark
  that moves a whole source cannot say which of the three caused a win.
- Constraint benchmarks put both lines in one message. That choice
  changes the winner, because order works in opposite directions inside
  and across messages.
- The hierarchy that puts the system message on top (Wallace et al.
  2024) does not describe Qwen. It roughly describes gpt-oss, which was
  trained on a system > developer > user order.
- Pairwise results cannot fix the winner of a three- or five-way
  conflict without an assumption like Luce's axiom (proved by example in
  App. G). In the data, forecasts from pairs improve in every model-mode
  once they know where the task line sits.
- Six practices (Sec. 4.6): test where a policy is placed rather than
  assume the system prompt protects it; on Qwen 3 and gpt-oss, try
  sending the policy last; test conflicts across messages; model the task
  line's position; test each conflict on many scenarios and replies; do
  not count on declared tiers or on a warning.

## Methods

- **Scenarios.** 240 each of tool, file-write, format and ordering, and
  120 refusal (two options, pairs only). Claude Opus 5 agents wrote them
  one at a time from a written guide, in batches of at most 50. Two
  automatic validators check exclusivity and wording that gives an option
  away. The authors reviewed every batch. The 240 format classifiers are
  tested on 1,382 written replies, including adversarial near-misses.
  English only. One label wording per seat (Table 5).
- **Seats as sent.** The operator and repository lines share the first
  system message, each under its label ("Operator policy for this
  deployment." / "Repository configuration for this workspace."), in
  either order half the time. The user line has no label. The tool line
  answers an assistant `config_read` call already in the transcript. The
  memory line is a user message labelled "Your saved note from an earlier
  session." The task is its own user message starting "Task:". No other
  system prompt is added. Qwen 3.5's template rejects a system message
  that is not first, so it has no "real, last" layout.
- **Seven experiments (Table 2, per model-mode).**
  - *Ladder*: all 10 pairs, 10 triples and the five-way set, with each
    seat carrying each option equally often; each side also asked alone
    (12,840 trials).
  - *Operator*: S1 or S2 against S3, S4 or S5 in four layouts (real
    system role or labelled user text × first or last); 180 scenarios
    per pair (4,320 trials; 3,240 on Qwen 3.5).
  - *Boundary*: user vs memory note in separate messages or inside the
    task's message, both orders (270 scenarios, 1,080 trials).
  - *Context*: 50 tool and file-write items, each line bare or inside
    ~200-word realistic text (1,200 trials; 900 on Qwen 3.5).
  - *Agreement*: 80 groupings where 2–4 seats share an option (3,240).
  - *Load*: 2–12 instructions and 0–6 conflict groups in one operator
    message, 10 bundles (100 trials).
  - *Tiers*: two options tagged `[priority: t]`, t in 1–6, under a system
    rule that the lower number wins (6,480 trials, sent once).
- **Decoding.** vLLM; Qwen 3 on 0.8.5.post1 in float16, the rest on 0.28
  in bfloat16 (gpt-oss experts in MXFP4). Qwen thinking off: T 0.7,
  top-p 0.8. Thinking on and gpt-oss: T 0.6, top-p 0.95. gpt-oss at high
  reasoning effort. Max 32,768 tokens, up to six tool turns. One seed per
  draw.
- **Scoring.** Every reply is labelled chose / unresolved / off-menu /
  multiple / no action by code. Rates are over *decided* replies (chose
  an option exactly one seat asked for). Flagging is a fixed keyword
  pattern over the answer text; reasoning is never scored.
- **Statistics.** 95% percentile bootstrap, 2,000 resamples of
  scenarios; contrasts paired within scenario. Qwen 3 exploratory
  findings were fixed in writing before Qwen 3.5 and gpt-oss were
  analysed. Other patterns were found on a hash-fixed discovery half and
  tested on the other half (Table 9). 2,103 tests: 1,299 pass
  Benjamini–Hochberg at q = 0.05, 892 pass Holm. Model-mode counts in the
  text ("25 of 26") are uncorrected.
- **Forecasting.** Conditional-logit fits on Ladder pairs only (position;
  task-aware position; position + one strength per seat), scored by log
  score on the triples and five-way sets, as share of the gap from a
  uniform guess to an empirical reference.

## Results

- **Order and boundaries (Fig. 2, Table 3).** First-listed inside one
  message: 72.42–94.66%. Drop when split into two messages: 19.96–60.68
  points. In free Ladder pairs (user, tool, memory), the later message
  wins 69.36–89.05% on Qwen 3, 59.77–80.52% on Qwen 3.5 and
  51.37–57.86% on gpt-oss. Inside the system message the second line
  wins only 10.80–36.21% (26 of 26 below half). A later line wins less
  when both lines come before the task line, by 16.80–29.29 points on
  Qwen 3. The authors read a line after the task as a correction to it.
- **Operator pinned first (Fig. 3).** Qwen 3 6.07–17.35%, Qwen 3.5
  11.66–41.50%, gpt-oss 62.94–75.73%. gpt-oss takes no action in
  17.64–23.40% of usable replies, mostly asking for missing content.
  Counting every usable reply, gpt-oss-120b keeps 51.03%, gpt-oss-20b
  34.57%, and Qwen3.5-122B-A10B 29.64% / 30.51%. On Qwen this rate also
  carries the cost of an early position.
- **Role vs label (Fig. 4, App. H.3).** In tool and file-write tasks the
  real role minus labelled text, both first: −6.41 to −48.75 points on
  Qwen 3, −8.70 to −29.59 on Qwen 3.5, −10.96 to −14.15 on gpt-oss. In
  format tasks the drop is absent (0 of 26), and gpt-oss *gains*
  +11.29 to +13.86. In order and refusal tasks the real role loses on
  Qwen but gains +19.27 to +25.52 on gpt-oss. Sent last, the real role
  gains 19.79–48.22 points on Qwen 3 and 12.49–20.41 on gpt-oss. On
  gpt-oss the late developer message beats the late label by
  25.09–32.91 points, so there the role itself has an effect. The
  tool/file-write restriction was found on the discovery half and held on
  the held-out half.
- **Seats (Fig. 7).** User beats tool result by point estimate in every
  model-mode, interval above half in 20 of 26. Part of this exists with no
  conflict: asked alone, a user line is carried out more often than the
  same line from a tool, by 0.07–25.44 points (23 of 26). User vs memory
  note is close to even.
- **Realistic context (App. I).** Inside ~200-word text the gpt-oss
  operator falls 15.12–29.77 points (75.71–85.03% bare to 45.79–70.05%).
  Qwen3.5-122B-A10B (thinking off) falls from 47.39% to 15.38%. Where the
  bare operator is weak (Qwen 3) context sometimes helps. The later line
  still wins in context in 25 of 26.
- **Forecasting (Fig. 5, App. J).** The task-aware fit beats the
  position fit in all 26 model-modes, by 0.012–0.113 nats per reply. Seat
  strengths add to position in 10 of 14 Qwen 3.5, 2 of 2 gpt-oss and only
  3 of 10 Qwen 3 model-modes. Even the empirical reference names the
  winner in only 44.44–69.70% of replies.
- **Tiers (App. M).** Lower tier wins 50.78–98.51% of decided replies
  (above half in 24 of 26); 24.29–87.08% of usable replies. Thinking
  raises it by up to +25.83 on Qwen 3.5; Qwen3.5-122B-A10B goes from
  75.67% to 95.44%. Upper bound on tiers kept in order at 0.95
  reliability: 1, in every model-mode. The tier gap matters little once
  the winning tier is fixed.
- **Flagging (App. N).** 0.14–2.92% of decided pair replies flag or hedge
  (gpt-oss 2.14–2.92%). In Qwen, flagged replies favour the earlier line
  more than silent ones (+20.03 points on Qwen 3, +15.65 on Qwen 3.5,
  unpaired).
- **Load (App. O).** More conflicts at fixed length have no steady cost
  (all pooled intervals span zero). Instruction count has no common
  direction across families.
- **Agreement (App. K).** When several seats share an option, it wins at
  most 6.85 points more than the pair-fitted seats model predicts.

## Critique / open questions

- **Against the digest framing: below framing (moderate).** Every number
  in the digest summary is in the paper and quoted correctly (72–95%,
  23–63%, 1.89M, ≤2.92%). The "Why" line is the overstatement.
  1. **"Position and label … decide."** The label is never varied. Each
     seat has one label wording, and the "label" layout keeps the label
     and removes the system role. What is tested is *role vs no role*,
     with the label held fixed. Whether a label like "Repository
     configuration" itself moves outcomes is untested.
  2. **"Not source rank."** The paper's own conclusion is that the winner
     depends on position and boundaries "as well as on the role". Role
     matters on gpt-oss: the pinned developer message wins 63–76%, and a
     late developer message beats the late label by 25–33 points. Seat
     strengths improve forecasts in 12 of 16 Qwen 3.5 and gpt-oss
     model-modes. Rank is weak on Qwen 3, not absent everywhere.
  3. **"Wins less often than a labelled user message"** holds in tool and
     file-write tasks only. In format tasks there is no drop, and on
     gpt-oss the real role helps in format, order and refusal. The task
     restriction was found post hoc on the discovery half (then held on
     the other half), and the role contrast also moves the policy next to
     the tool definitions, which the paper says it cannot separate.
  4. **"Code and data."** Code, scenarios and run configs are public. The
     1.89M scored replies "will be released separately", and the
     aggregation scripts behind the paper's tables are not public
     (App. P.1).
- **Mapping to Claude Code is not one-to-one.** The paper's "repository
  configuration" seat sits in the system message. On this box, CLAUDE.md
  and `.claude/rules/` arrive as labelled text in a user-turn
  `system-reminder` after the system prompt (as session context shows).
  That is closer to the paper's "label, first" layout than to its S2
  seat. A loaded SKILL.md and later user turns arrive in later messages.
  So the relevant contrasts here are within-message order (inside the one
  reminder block) and across-message recency (skills and chat after
  CLAUDE.md). The paper measures both, but only on open-weight models.
- **Transfer to Claude is untested, and one measure already disagrees.**
  This paper finds ≤2.92% of replies flag a conflict. In
  [[literature/papers/xu2026stale]], Claude Code named the file–code
  conflict in 79 of 80 replies. In [[literature/papers/zhu2026precedent]],
  Sonnet discussed the rival format in 22 of 45 transcripts. The settings
  differ (keyword patterns on one-line conflicts vs multi-step coding
  tasks), but at least on reporting, Claude does not behave like these
  readers. The position and role numbers should not be read as Claude
  numbers.
- **What it adds to "channel rank".** The graph's working reading, from
  xu2026stale, is that an instruction beats tool output because of its
  channel. This paper supports the weak form: user beats tool result in
  20 of 26 model-modes, partly even without a conflict. It undercuts the
  strong form that higher role always wins: on Qwen a system-message
  policy loses to later user, tool and memory lines most of the time.
- **Fit with [[literature/papers/razmara2026one]].** Razmara force-loads
  two skill files into one system prompt and finds load order picks the
  winner inconsistently (first-listed wins 74–77% of takeovers on
  Llama-3.1-8B and GPT-4.1, 3–24% on Phi-4-mini and Qwen2.5-7B). Here,
  one-line rivals in one message give first-listed 72–95% on every model.
  The two disagree for Qwen-family readers (a different generation,
  Qwen2.5, in razmara). Our inference: a worked
  example (razmara's mechanism) can overrule the position effect that
  one-liners show. Razmara's failed priority line also fits this paper's
  tiers result: no model keeps two declared tiers at 0.95.
- **Scale and setting.** One-line synthetic instructions written by one
  model family, one label per seat, English only, one task of at most six
  tool turns. Only 50 items are tested in realistic text, and there the
  strong operator weakens most, so the bare numbers may overstate how
  much a policy holds in a long CLAUDE.md. Every option is harmless and
  equally valid, so no option is ever correct on the merits. Family
  contrasts mix training, chat format, vLLM release and precision
  (Limitation 3).
- **Decided-reply denominator.** Rates exclude no-action, off-menu and
  unresolved replies. For gpt-oss, which asks back in about a fifth of
  replies, "operator wins" describes behaviour when it acts. The paper
  reports both readings for the headline cells.
- **No arm for what a harness can do.** Nothing tests re-injection, a
  conflict-detection prompt, or a checker. The six practices are all
  "test placement"; none says which placement is safe across families.
  The one consistent pattern is "test it on your model".

## Trust signals

- **Credibility:** 3.
  - For: a clean one-factor-at-a-time design with fixed text; code-only
    scoring with tested classifiers; three draws per trial; scenario-level
    bootstrap with paired contrasts; Qwen 3 findings fixed in writing
    before the other families were analysed; a held-out half for later
    patterns; Benjamini–Hochberg and Holm over all 2,103 tests; both
    decided and all-usable rates for the headline cells; an honest
    limitations section on confounds (tool definitions, serving release,
    family differences). Code, scenarios, run configs and a PyPI package
    are public.
  - Against: not peer reviewed, 11 days old, no citations; a small academic
    group (LNMIIT, with one Virginia Tech author); scored replies and
    aggregation scripts not yet released; scenarios written by a single
    LLM family; model-mode counts in the text uncorrected; open-weight
    models only.
  - Not 4 because there is no peer review or major-lab backing and the
    replies behind the tables are not yet public. Not 2 because the code
    is released and the design and statistics are unusually careful.

## Follow-up

- **Relevance:** 4. It is the first controlled split of position, role
  and message boundary in this graph, and it qualifies the "channel rank"
  reading the graph built from xu2026stale. It adds material evidence to:
  - [[concepts/enforcement-boundary-placement]]: role rank is
    family-dependent; position and boundary are levers of similar size.
  - [[concepts/constraint-pinning]]: presence at the top of the context
    does not confer authority on Qwen; later lines win.
  - [[concepts/shared-skill-namespace]]: a skill loaded later arrives in
    its own message, where recency, not load order, applies.
  - [[concepts/typed-enforcement]]: declared tiers fail at 0.95 and
    conflicts go unreported, so precedence has to be checked, not stated.
  Not 5 because no Claude or hosted model is tested, the instructions are
  one-line and synthetic, and it gives no mechanism a harness can adopt.
- **Bearing on [[concepts/information-firewall]].** A tool-result line
  still wins about 21–47% of its conflicts with a user line (user wins
  52.57–78.51%), with no adversarial wording. Plain instructions in tool
  output are not inert.
- **Cheap local test (Claude).** The package runs any chat API in
  principle (Limitation 1). Run the Boundary and Operator tests on Claude
  with the `system-reminder` user-turn layout this box uses: one
  CLAUDE.md-style line vs a later user line and vs a SKILL.md-style line
  in a later message, both orders, 50 tool and file-write scenarios × 3
  draws. That is a few hundred API calls, no GPU. It decides whether
  within-block primacy and cross-message recency hold for this box's
  model.
- **Candidates** (cited, not yet in the graph):
  - Zhang et al. 2026, "Many-tier instruction hierarchy in LLM agents"
    (ManyIH, arXiv 2604.09443). Up to twelve tiers as tags.
  - McCauley et al. 2026, "IH-Benchmark" (arXiv 2607.25987). Conflicts
    inside tool outputs, 37 model variants including frontier.
  - Geng et al. 2026, "Control Illusion: The failure of instruction
    hierarchies in LLMs" (AAAI 2026).
  - Kumar and Chopra 2026, "Authority bias in language models"
    (arXiv 2609.37616). Includes three closed APIs.
  - Kariyappa and Suh 2026, "Where instruction hierarchy breaks"
    (arXiv 2606.07808). Benign turns between system and user conflict.
  - Liao 2026, "Auditing provenance sensitivity in LLM agent action
    selection" (arXiv 2607.20827). Changes only a textual source marker.
