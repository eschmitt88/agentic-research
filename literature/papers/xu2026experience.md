---
kind: paper
title: "Experience Rot: Experienced Agents Fail Where Novices Succeed"
authors: ["Zhen Xu", "Hang Yin", "Zitao Zhang", "Kevin Zhao", "Xuefeng Liu"]
institutions: ["University of Chicago", "eBay", "University of Florida"]
year: 2026
venue: "arXiv 2611.20922 (cs.CL), v1 2026-11-16, preprint (8-page main text + 8-page appendix)"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.20922"
code_url: null    # no code, data or preregistration link in the PDF
citations: null
source: "raw/papers/xu2026experience.pdf"
added: "2026-11-24"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/constraint-pinning]]"
tags: ["experience-following", "stale-memory", "silent-tool-change", "interface-drift", "read-back", "false-completion", "instruction-placement", "system-prompt-vs-task-message", "version-tag", "hand-written-history", "evaluation-fidelity", "preregistered", "mcp-style-tools", "negative-result"]
---

# Experience Rot: Experienced Agents Fail Where Novices Succeed

## TL;DR

Three small synthetic services (calendar, task list, mail) each come in
two versions, A and B, that differ in one interface detail: a renamed
field, a changed structure, a new set of enum values, or a number whose
range stays but whose meaning flips (priority direction, weekday
numbering). A changed request is **silently accepted**: unknown fields
are ignored and the response is `{"accepted": true}`. The agent does an
earlier task live on one version. A final task of the same kind then
runs on the same version (valid experience), on the other version
(out-of-date experience), or with no earlier task (no experience). The
current documentation is always one `docs_search` call away. Six
small/flash models, 12,624 episodes, cluster-bootstrap SEs over task
instances, most complete-documentation conditions preregistered.

Five results matter for this graph:

1. **The failure is a missing check, and reading back the write fixes
   it.** With out-of-date experience in the conversation and complete
   documentation, success is **0.46** (valid 1.00, none 0.88–1.00).
   Experienced agents look up the docs before their first write in **≤6%**
   of runs, against 70–100% for novices.
   - Of 184 runs that wrote the old usage and did not read back, **183
     failed**. Of 97 that wrote it and read back, **94 succeeded**.
   - **347 of 348** failed runs told the user the task was done.
   - One model, Qwen3.8-Flash, reads back after every write and scores
     **1.00** with out-of-date experience. MiniMax-M3 never reads back and
     scores 0.05.
2. **A read-back instruction paid heavily; announcing the change did
   not.** Out-of-date experience, complete docs (Table 6):
   - Version number and update notice: 0.46 → **0.50**.
   - "Before you reply that a request is complete, read back the stored
     state …" (system prompt): **0.91**, at 6.5 model calls against 4.0
     with nothing added and 2.9 with valid experience.
   - Writes that return the stored object (a tool change, no
     instruction): 0.45 → **0.79** at 4.6 calls. With valid experience
     it costs nothing (1.00, 2.7 calls). In 59 of the 77 remaining
     failures the agent saw a stored object with the wrong state and did
     not write again.
3. **Placement decided whether the same text was used.** The same
   documentation gives **0.99** in the task message, **0.54** in the
   system prompt and **0.55** in the tool description. The same "check
   the documentation before writing" instruction gives **0.99** appended
   to the final task message and **0.29** in the system prompt, below
   nothing added (−0.17 ± 0.02). Agents followed the system-prompt
   version in the earlier task (98%) and not in the final one (4%).
4. **Correct memory goes stale, and labels do not help much.** When the
   final task needs information from the earlier one, the cost of
   out-of-date experience is 0.53 (conversation), **0.27** (memory file),
   0.20 (top-3 retrieved notes) and 0.30 (self-written script plus note).
   60–73% of memory files, and 92% of skill notes, record the earlier
   version's usage. Tagging the memory with its service version moves the
   cost 0.27 → **0.20**. Putting the current documentation next to the
   memory moves it to **0.10**.
5. **Hand-written histories hide most of the cost.** A researcher-written
   earlier task that includes a read-back step gives 0.88 out-of-date
   success, against 0.46 after the agent's own earlier task. Agents copy
   the read-back with the call (84% read back vs 30%). Measured cost: 0.11
   against 0.54, about a fifth.

## Claims

- An agent's own successful tool use, correct when made, becomes wrong
  after a silent interface change, and the agent reuses it anyway
  ("experience rot"). The underlying behaviour is Xiong et al.'s
  experience-following.
- Experience stops the agent checking: lookups collapse, and agents
  refer to the earlier task as often when it is out of date as when it is
  valid. Typical: "I could search docs again, but I already know the
  format from the previous search" (GLM-5.3-Flash).
- With incomplete documentation (a number's range is documented, its
  meaning is not), experience is what makes agents succeed (0.63 → 1.00),
  and out-of-date experience leaves them far worse than none (0.23).
- Every memory form tested is affected, and more earlier tasks make it
  worse (one, three, ten earlier tasks: 0.46, 0.33, 0.26).
- Memory systems need a way to notice that a stored usage no longer
  holds. In these tests checking the stored result did this reliably,
  and version labels and documentation pointers did not.
- Evaluations of memory under change should use the agent's own
  experience and incomplete documentation, because hand-written
  histories and complete documentation both hide the problem.
- The authors explicitly do not claim a mechanism for why agents stop
  checking, and note the Einstellung parallel as resemblance only.

## Methods

- **Services and tools.** Calendar, task list and mail services with
  write tools, read tools (`calendar_get_event` etc., which show stored
  state in neutral terms such as "Highest", "Tuesday") and `docs_search`
  (returns the running version's docs). Tools use typed JSON-schema
  definitions "in the form used by function calling and by MCP". The
  services are synthetic. No real MCP server, and no
  `notifications/tools/list_changed` event, is exercised.
- **Changes (Table 1).** Field name (`attendees` → `guests`), structure
  (`end_time` → `duration_minutes`), value set (urgent… → p0…), priority
  direction, weekday numbering; month numbering and duration unit only
  with incomplete docs; a mail reply-default change reported separately.
  Each change runs in both directions. The tool definition always
  follows the running version. In the field cases it either lists the
  current fields (typed) or takes an untyped payload.
- **Documentation settings.** Complete: fields, values and meanings.
  Incomplete: the range only ("priority is an integer from 1 to 4"). The
  earlier request then states the meaning on its version, and the final
  request does not. Incomplete-docs conditions were **not
  preregistered**.
- **Episodes.** Earlier task done live (failed attempts discarded and
  retried up to ten times), then the final task as the next message.
  Variants: experience stored as conversation, memory file in the system
  prompt of a new session, top-3 retrieved note entries, or a
  self-written Python skill plus note; 1/3/10 earlier tasks; additions
  to the final task; hand-written earlier tasks with or without a
  read-back step.
- **Models.** GLM-5.3-Flash, Muse-Glimmer-30B, DeepSeek-V4-Flash (0731),
  MiniMax-M3, Qwen3.8-Flash and "Inkling" (no vendor or size given). One
  minimal loop, temperature 0.7, ≤16 model calls per task, native tool
  calling. No GPT, Gemini or Claude model.
- **Measures.** Programmatic state check for success. Process measures:
  lookup before first write, old usage in first write, read-back after
  it, model calls. "Reports done" and reasoning-text measures use
  keyword patterns (App. G.7, G.9).
- **Statistics.** Pooled over models, changes and directions. ± one SE
  from a cluster bootstrap over task instances (10,000 replicates),
  models and changes treated as fixed. Six instances per change ×
  direction × arm for complete docs, three for storage/amount/authorship,
  twelve for incomplete docs.
- **Preregistration (App. F, H).** Models, conditions, instances, seeds,
  exclusions, predictions and analysis script fixed with code hashes
  before the run, plus a second preregistration for the stored-object
  and task-message conditions. Every directional prediction held. Three
  "no prediction" contrasts came out with a sign (docs in system prompt
  +0.08, notice +0.04, read-first instruction −0.17). No registry link is
  given.
- **Exclusions.** 136 of 12,624 episodes: 9 failed requests, 127 earlier
  tasks that never succeeded (124 on the weekday change, mostly Inkling).

## Results

- **Main contrast (Table 4).** Success with no / valid / out-of-date
  experience:

  | Setting | None | Valid | Out-of-date | Old usage | Read-back |
  |---|---|---|---|---|---|
  | Incomplete docs, meaning of a number | 0.63 | 1.00 | 0.23 | 1.00 | 0.30 |
  | Complete, field, untyped tool | 1.00 | 1.00 | 0.26 | 0.99 | 0.27 |
  | Complete, field, listed in the typed definition | 1.00 | 1.00 | 0.49 | 0.69 | 0.22 |
  | Complete, meaning of a number | 0.88 | 1.00 | 0.42 | 0.99 | 0.43 |

  A typed definition that lists the new field halves the cost against
  docs-only (P2: −0.23 ± 0.02), but the agent still writes the old field
  in 69% of runs with `guests` in front of it.
- **Per model (Table 8, complete docs, typed).** Out-of-date success:
  GLM 0.70, Muse 0.25, DeepSeek 0.35, MiniMax 0.05, Qwen3.8 1.00, Inkling
  0.38. Read-back: 0.53, 0.23, 0.00, 0.00, 1.00, 0.00. Success follows
  the read-back rate across models (Fig. 3c).
- **What agents say (Table 11).** Before the first write, 69% of novices
  plan a lookup, against 6% (valid) and 9% (out-of-date) of experienced
  agents. 24–25% of experienced agents refer to the earlier task, the
  same rate whether it is valid or not.
- **Copying beyond the interface.** With untyped tools, experienced
  agents reproduce the earlier start-time format in every run, novices in
  77%. In 135 of 144 earlier tasks with a renamed field or structure, the
  agent made no lookup, and the first final write still repeated the old
  field in 61% of them.
- **Memory forms (Table 5).** Out-of-date success: conversation 0.47,
  memory file 0.72, retrieved 0.76, skill 0.61 (no experience 0.03–0.10).
  Content is carried over correctly in 95–100% of first writes. Only the
  interface fails. Note-based designs look up the docs more (24–34% vs
  2%). Version tag: cost 0.20 ± 0.02; current docs in the memory file:
  0.10 ± 0.01.
- **Amount (Table 7).** One, three, ten earlier tasks: out-of-date success
  0.46, 0.33, 0.26; valid 1.00, 0.99, 0.97. Lookups fall from 4% to 0%.
- **Additions to the final task (Table 6, complete docs).**

  | Added | Success | Old usage | Read-back | Calls |
  |---|---|---|---|---|
  | Nothing | 0.46 | 0.80 | 0.30 | 4.0 |
  | Docs in task message | 0.99 | 0.01 | 0.30 | 2.9 |
  | Docs in tool description | 0.55 | 0.62 | 0.30 | 3.6 |
  | Docs in system prompt | 0.54 | 0.63 | 0.29 | 3.5 |
  | Version number + notice | 0.50 | 0.76 | 0.30 | 3.9 |
  | Read docs first (system prompt) | 0.29 | 0.96 | 0.29 | 4.0 |
  | Verify and correct (system prompt) | 0.91 | 0.84 | 0.92 | 6.5 |

  Separately preregistered (Table 13): read-docs-first in the task
  message 0.99 (lookup 1.00); writes return the stored object 0.79, with
  read-back 0.48 and a second write in 0.70 of runs.
- **Cost of the instruction when experience is valid (App. I).** The
  task-message read-docs instruction keeps success at 1.00, raises
  lookups from 1% to 100%, calls 2.92 → 3.55 and input tokens 3,839 →
  5,361 (+40%). The verify instruction's cost on valid experience is
  **not measured**.
- **Incomplete docs (Tables 16–18).** Out-of-date 0.23. Read-docs
  instruction in the task message: 0.25, because the docs do not hold the
  answer. Verify instruction in the task message: 0.69 at 10.6 calls
  (vs 3.1 valid, 4.8 out-of-date). By model under verify: GLM 0.82, Muse
  0.21, DeepSeek 0.74, MiniMax 0.75, Qwen3.8 0.96, Inkling 0.67.
- **Hand-written histories (Table 7).** Out-of-date success: own earlier
  task 0.46, hand-written with read-back 0.88, hand-written without
  read-back 0.58. Cost 0.54 / 0.11 / 0.36. P5 contrasts hold (+0.25 ±
  0.04, +0.43 ± 0.02).
- **Completion reports (App. G.9).** 347 of 348 failed runs (complete
  docs) and 405 of 438 (incomplete) end with a done report. Only 12
  incomplete-docs runs told the user the write had not taken effect.

## Critique / open questions

- **Against the digest entry: at, with one scope overreach.** The entry
  says "After a silent MCP tool change, experienced agents fail where
  novices succeed, copying their old call and reporting success.
  Announcing the update barely helps." Every number behind it is in the
  PDF.
  - **"MCP" overstates the setting.** The tools are MCP-*style* JSON
    schemas on three toy services. No MCP server, client or
    `list_changed` notification is tested. The Todoist/MCP disagreement
    in Fig. 1a motivates the design; it is not an experiment.
  - **"Fail where novices succeed" holds only where the experience was
    not needed.** In the memory designs, where the final task needs the
    earlier one, novices succeed in 3–10% and out-of-date agents in
    **61–76%**. There, out-of-date experience is much better than none,
    and the cost is relative to valid experience (0.20–0.30). The
    abstract's "fail on most tasks" also does not hold there.
  - **It omits the three results that transfer**: read-back as the fix
    (183/184 vs 94/97; +0.45 from one sentence; +0.34 from a stored-object
    response at no cost on valid runs), the placement contrast (0.99 vs
    0.29 for one sentence), and the 5× understatement from hand-written
    histories.
  - **"Announcing barely helps" is right** (+0.04 ± 0.02), and the
    memory-file version tag is similar (−0.07 ± 0.03 on cost).
- **The cut reason does not hold.** The bullet carries no parenthetical.
  Of the header's three reasons: *overlap* would be with item 20
  (dip2026invariantaware, silent API failures). After the full read, the
  mechanism is different. There the API is faulty and the agent's
  query is right. Here the API is correct and the agent's own memory is
  wrong, so the remedy surfaces differ: the invariant check there, the
  read-back and placement here. The two papers are complementary.
  *Small n* holds for task diversity (three toy services, seven changes,
  3–12 instances per cell) but not for episodes (12,624), and the main
  contrasts are preregistered. It is not *method-first*. It would have
  been a reasonable ranked item.
- **The pooled numbers are a model mix.** Per-model out-of-date success
  runs from 0.05 to 1.00, and the ± 0.01 SEs treat models as fixed and
  resample instances only (the authors say this may be optimistic). The
  firm, model-independent finding is the path table: a run that reads
  back recovers, and one that doesn't fails. How often a model reads
  back by default is a property of the model.
- **No frontier model.** All six are flash/small or unidentified
  ("Inkling"). li2026where found frontier models read back after an
  *ambiguous* ack (lost ack, misleading 500) and almost never duplicate.
  Here the ack is a confident `accepted: true`, so there is no error
  cue. Whether a frontier model reads back after a clean ack is the open
  question for this box. It is untested here.
- **The system-prompt instruction's harm is unexplained.** The paper
  reports "followed in the earlier task (98%), not in the final (4%)".
  Our reading, untested: the instruction is discharged once per
  conversation. Following it in the earlier task also put version-A docs
  in the context, and old usage rose from 0.80 to 0.96. Either way, an
  always-present instruction keyed to a recurring event ("before your
  first tool call that changes data in a request") bound only at its
  first occurrence.
- **Placement is not uniform across instructions.** The verify
  instruction was in the system prompt and worked (0.91). The read-first
  instruction in the same place failed. So "text in the system prompt is
  ignored" is too strong. The two also differ in trigger (before
  replying vs before the first write) and in whether earlier compliance
  can be taken to satisfy them. Verify was never moved to the task
  message in the complete-docs setting, so the paper's Mechanism
  paragraph ("acted on when attached to the request") rests on the docs
  and the read-first sentence only.
- **"More than twice the model calls"** is against valid experience (6.5
  vs 2.9). Against the out-of-date baseline the same agent would face
  without it, verify costs 1.6× (6.5 vs 4.0, our arithmetic). Its cost
  when experience is valid, which is the common case in deployment, is
  unmeasured. The stored-object response is the only remedy priced on
  both arms, and it is free on the valid arm.
- **Experience here is one same-kind task, usually in the same
  conversation, with an explicit rule in the earlier request** (for
  incomplete docs). Experience found by trial, over many sessions, or
  mixed with unrelated tasks is untested. The authors list these limits
  themselves.
- **The memory forms are a single pipeline.** One note-writing prompt,
  top-3 retrieval from that same note, and "skill" = one script the
  agent wrote. Why note designs lose less (they look up more: 24–34% vs
  2%) is not tested.
- **The keyword classifiers** for "reports done" and "plans a lookup"
  are unvalidated against human labels. The 347/348 figure is likely
  robust (failure is programmatic, and the reply patterns are broad).
- **Rejecting unknown fields** (turning the silent change into an
  error) is named and not tested. It is the typed-enforcement remedy and
  the cheapest one at the service.
- **Consistent with neighbours in the graph.**
  - [[literature/papers/hu2026memory]]: agents prefer a stale stored
    value to an authoritative tool 0.92–1.00 of the time. This paper
    shows the same over-trust for interface usage, with the agent's own
    correct record.
  - [[literature/papers/shen2026revoked]] left "validity metadata in
    context" unmeasured. Here a version tag that made staleness
    inferable (memory "written at 2.2", tool "2.3, updated") cut the
    cost only 0.27 → 0.20.
  - [[literature/papers/dip2026invariantaware]] and
    [[literature/papers/zhang2026are]]: noticing without acting. Here 59
    of 77 remaining failures had the wrong stored state returned to them
    and did not rewrite.
  - [[literature/papers/lee2026unfinishedbench]] and
    [[literature/papers/laddha2026position]]: placement decides use. Here
    the same text moved 0.29 → 0.99 between system prompt and task
    message, with no conflicting rule in play.

## Trust signals

- **Credibility:** 3.
  - For:
    - University of Chicago, eBay and University of Florida authors.
    - Preregistered design, predictions, exclusions and analysis script
      for most complete-docs conditions, with a second preregistration
      for the late conditions. Every directional prediction held, and
      the unexpected signs are reported.
    - 12,624 episodes, both directions of every change, the same task
      instances across arms, programmatic success. A calibration run
      with fixed rules, and exclusions fully itemised.
    - Process measures (lookup, old usage, read-back) that explain the
      outcome, plus a clean path table.
    - The authors state their limits and disclaim a mechanism.
  - Against:
    - Not peer reviewed, no code, data or registry link.
    - Three synthetic services, single-detail changes, short tasks.
    - Six small or unidentified models, no frontier model. SEs treat
      models as fixed while per-model results range 0.05–1.00.
    - Incomplete-docs conditions, including the Fig. 1c headline, are
      not preregistered.
    - Keyword-based text measures.
  - Not 4: no artifacts, no frontier model, synthetic. Not 2: the
    preregistration, matched design and process measures are above most
    preprints in the graph.

## Follow-up

- **Relevance:** 4.
  - It is material new evidence for
    [[concepts/evidence-gated-completion]]: read-back as the active
    ingredient, near-universal false "done", and a harness-side state
    echo that is free on the valid arm.
  - It gives [[concepts/enforcement-boundary-placement]] a matched,
    preregistered placement contrast with no conflicting rule.
  - It adds to [[concepts/verified-memory-writes]] a case where every
    write is correct and the store still misleads, and prices the
    version-label mitigation.
  - Not 5: tool-use agents on toy services, not ML-research agents, and
    no frontier model.
- **Untested here, cheap to test on this box:** does a Claude model that
  has used a CLI once in a session re-check its usage after the CLI's
  flags change? `$J` (claude-coordinator-jobs) and the `kg_lint.py`
  interface are candidates. The paper predicts failure only for agents
  that don't read back.
- **Candidates** (cited, not yet in the graph):
  - Xiong et al. 2026, "How Memory Management Impacts LLM Agents: An
    Empirical Study of Experience-Following Behavior" (ACL; arXiv
    2505.16067). The named mechanism.
  - Li et al. 2026b, "SilentProbe" (arXiv 2609.00035): 44 of 61
    prose-only constraints fail silently against 0 of 111
    machine-checkable ones; four of five endpoints ignore undeclared
    parameters. Also cited in dip2026invariantaware.
  - Liu et al. 2026, "MCPEvol-Bench" (arXiv 2607.14642), mutation
    operators on 123 real MCP servers.
  - Li et al. 2026a, "ProEvolve / The World Won't Stay Still" (arXiv
    2603.05910): traces from earlier versions cut tool calls but not
    failures.
  - Chao et al. 2026, "STALE" (arXiv 2605.06527); Fan et al. 2026,
    state drift (arXiv 2608.19652); Dabas et al. 2026, "MemDrift"
    (arXiv 2605.24941).
  - Hu, Long and Wang 2026 (arXiv 2604.27003): abstract procedural
    memories transfer better than detailed trajectories.
  - Advani 2026, false success in LLM agents (arXiv 2606.09863).
