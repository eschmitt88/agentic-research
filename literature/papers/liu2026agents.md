---
kind: paper
title: "Agents Cannot Hand Over What They Have Not Yet Understood"
authors: ["Yibo Liu"]
institutions: ["Beijing University of Posts and Telecommunications"]
year: 2026
venue: "arXiv 2611.18642 (cs.AI), v1; the PDF's arXiv stamp reads '14 Oct 2026', preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.18642"
code_url: null
citations: null
source: "raw/papers/liu2026agents.pdf"
added: "2026-11-23"
relevance: 4
credibility: 2
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/lossless-context-offload]]"
  - "[[concepts/compression-as-generalization-test]]"
  - "[[concepts/hierarchical-delegation]]"
tags: ["handoff", "compaction", "transmission-chain", "iterated-learning", "summary-of-summary", "repeated-compression", "record-relay", "compaction-relay", "handoff-loss-decomposition", "hidden-rule-worlds", "induced-knowledge", "fact-level-grading", "note-length-sweep", "writer-instruction-null", "frontier-of-ability", "single-author", "no-code"]
---

# Agents Cannot Hand Over What They Have Not Yet Understood

## TL;DR

56 small text worlds with hidden, checkable rules. An agent that is told
nothing fails within one life, and an agent given the designer's ≤300-char
statement of the rules almost always succeeds. The paper compares one agent
that plays eight lives with its whole history (**Memory**) against a chain
of eight agents that each play one life and inherit a 300-char note written
by the previous one (**Relay**). Two more chains split the gap: **Compaction
relay** (same 300 chars, but the writer may read the raw record of every
earlier life) and **Record relay** (no note; each fresh agent gets the raw
records). Life g of sequence k is the same instance in all four, so the
conditions differ in one respect each.

Three results matter for this graph:

1. **Every model loses at a handoff.** Handoff loss M − R is 0.26–0.67
   marks (of 5), 19–48% of what Memory gains over Solo, all intervals above
   zero. The eighth Relay life scores where Memory stood after 3.9–4.1
   lives for four models and 2.5–3.0 for the two weakest.
2. **On the worst worlds, most of the loss comes from writing each note
   from the previous note.** On each model's 10 highest-loss worlds, letting
   the 300-char writer read the full record recovers 1.82 of 2.59 marks for
   gpt-6.1-sol (70%) and 1.08 of 1.31 for gpt-6-astra (82%), but only 0.81
   of 2.03 for gpt-5.6-terra (40%). Swapping the agent costs nothing
   (astra's fresh agents do *better*). Over all 56 worlds, sol recovers
   0.39 of 0.64 (61%).
3. **The writer's view is the lever, not its instructions.** Asking the
   writer to revise rather than rewrite, keep a confirmed / ruled-out / open
   ledger, or mark what was verified and record unexplained observations
   does not help (sol, 10 worlds). A longer plain note does: at 1,500 chars,
   Relay reaches 3.83 against Compaction relay's 3.47 at 300.

The paper's account: a rule that takes several lives to see exists, for
most of a chain, only as scattered observations. The writer of a note sees
one life, does not know which observations matter, and writes conclusions.
What is written is mostly kept. What was never concluded is lost.

## Claims

- Handoff (sub-agent reports, takeovers), context compaction and
  cross-episode reflection all compress what an agent has learned into
  short text for a successor. Existing evaluations measure task completion
  or recall of facts the agent was *given*, and cannot tell "the writer
  never understood" from "the writer understood and did not say".
- The handoff loss decomposes exactly:
  M − R = (M − F) change of agent + (F − C) one compression +
  (C − R) repeated compression.
- "Changing the agent costs nothing"; compressing once costs 0.72–0.92
  marks; "compressing repeatedly costs the most".
- The loss sits at each model's frontier: absent in worlds learned within
  two lives and in worlds never learned, largest in between. A stronger
  model moves it to harder worlds rather than removing it.
- Learning from experience and handing it over are separable abilities;
  completion rates understate the loss.
- Recommendation (§6): "keep the raw record outside the working context and
  give it to whoever writes the next summary: this costs storage, not
  context. Where the record cannot be kept, a longer summary is the next
  best choice, and a more elaborate instruction to the writer is not a
  substitute for either."

## Methods

- **Worlds (§3, Table 2).** Six themes: grid, machine, craft, tool, care,
  trade (10/10/10/10/6/10 admitted). Each world is a deterministic program
  with 16 instances that share rules but differ in layout, starting values
  and solution path. Replaying one instance's solution on another reaches
  the goal ≤5% of the time, so only rules carry over. Each world has 3–5
  mechanisms, 13–60 moves per life (median 33), a 97–298-char opening
  screen, pseudo-word names (banned meaning-bearing words) so pretraining
  cannot help, 5–8 facts (323 in all) each with a multiple-choice question
  including "the note does not say", and a designer's note of 194–300
  chars (median 253).
- **Admission.** 60 drafted with a coding agent's help, 56 admitted.
  Automatic checks: the solver uses 55–70% of the moves; 2,000 random lives
  reach the goal ≤2 times and the fourth mark ≤1%; ≥10% end on a rule; no
  first move ends a life. Model check with deepseek-v4.1-flash: with the
  designer's note ≥8 of 10 lives reach the goal, without ≤2 of 10. On
  admitted worlds, 0.6% without and 95.8% with. Four care worlds were
  dropped. Some designer notes that stated every rule still failed until
  reworded.
- **Conditions (§4, Fig. 3).** Solo (nothing), Oracle (designer's note),
  Relay, Compaction relay, Record relay, Memory. The record h_g is screens
  and actions only, not reasoning. The note request is fixed: "Write a note
  of at most 300 characters for the next player, who will meet the same
  world in a different starting situation." Longer notes are cut.
- **Measures.** Score of a life = highest mark (0–5). Sequences averaged
  over lives 5–8. Unit of analysis is the world; 95% intervals by
  resampling worlds. Also loss share (M − R)/(M − S), an exchange rate (the
  Memory life whose score equals Relay's eighth), and the discovery gap
  O − M.
- **Note grading.** A fixed reader model sees only the note and answers
  each fact question: right / wrong / absent. Two readers agree on 91.1% of
  1,591 judgements (κ = 0.81). In a sample of disagreements the default
  reader was right 10 of 10. A second reader, given the true rules, labels
  each claim in a note as a rule, a false claim or a single-game record.
  Neither reader model is named.
- **Variants.** Relay with L = 150, 600, 1,500, 3,000. At fixed L: revise
  the inherited note (as in Tessler et al. 2021's human chains); mark each
  rule verified or not and record unexplained observations; keep a ledger
  of confirmed / ruled out / open.
- **Models and runs.** gpt-6-astra, gpt-6.1-sol, gpt-5.6-terra,
  gpt-5.6-luna, gpt-6-luna, deepseek-v4.1-flash ("two providers"),
  reasoning effort high, other settings default. Solo/Oracle: 5 lives per
  world. Relay/Memory: 3 sequences per world (5 Relay chains for deepseek,
  which covers only 31 worlds). Compaction relay: all 56 worlds for sol, 29
  for astra, 10 for terra. Record relay and note variants: each of three
  models' 10 highest-loss worlds. >25,000 lives. A request that keeps
  failing is recorded missing, not zero.
- **No appendix.** The PDF is 12 pages of text plus references. Prompts
  other than the note request, temperatures, seeds and per-world tables are
  not given. Code: "not publicly available".

## Results

- **Main conditions (Table 3; Solo / Relay / Memory / Oracle; loss M − R
  [95% CI]; share; exchange rate; goal % Relay / Memory):**
  - gpt-6-astra: 2.21 / 4.12 / 4.57 / 4.99; 0.44 [0.24, 0.66]; 19%; 4.0;
    65 / 82
  - gpt-6.1-sol: 1.96 / 3.77 / 4.41 / 4.96; 0.64 [0.31, 0.96]; 26%; 4.1;
    57 / 74
  - gpt-5.6-terra: 1.41 / 2.40 / 3.07 / 4.68; 0.67 [0.45, 0.89]; 40%; 3.9;
    17 / 27
  - gpt-5.6-luna: 1.30 / 1.94 / 2.52 / 4.61; 0.59 [0.35, 0.84]; 48%; 2.5;
    7 / 15
  - gpt-6-luna: 1.25 / 1.87 / 2.13 / 4.62; 0.26 [0.07, 0.46]; 30%; 3.9;
    4 / 10
  - deepseek-v4.1-flash (31 worlds): 1.12 / 1.79 / 2.22 / 4.89; 0.43
    [0.20, 0.65]; 39%; 3.0; 6 / 14

  S < R < M < O for every model. The discovery gap O − M is 0.42–0.55 for
  the two strongest and 1.61–2.67 for the rest. The Relay curve keeps
  rising to life 8, so a chain is a slower learner, not a stalled one. For
  the four weaker models the goal rate barely moves (4–17% vs 10–27%), so
  a completion-only benchmark would miss the loss.
- **Decomposition, 10 highest-loss worlds (Table 4; R / C / F / M;
  repeated C − R; once F − C; agent M − F):**
  - astra: 3.10 / 4.18 / 4.90 / 4.41; 1.08 [0.40, 1.84]; 0.72 [0.22,
    1.29]; −0.49 [−1.00, −0.01]
  - sol: 1.65 / 3.47 / 4.29 / 4.24; 1.82 [1.16, 2.42]; 0.83 [0.28, 1.57];
    −0.05 [−0.57, 0.53]
  - terra: 1.56 / 2.37 / 3.29 / 3.59; 0.81 [0.23, 1.38]; 0.92 [0.14,
    1.68]; 0.30 [−0.25, 0.84]

  Wider sets: sol on all 56 worlds R 3.77, C 4.16, M 4.41, so repeated
  compression is 0.39 [0.15, 0.64] of 0.64 and the rest 0.25 [0.02, 0.49];
  astra on 29 worlds 0.45 [0.14, 0.80] of 0.54. For terra, one compression
  costs as much as repeating it: "the weaker model is also the weaker
  summariser of a full record". For astra a fresh agent with the raw record
  beats the agent that lived it, with an interval that excludes zero.
- **Length vs request (§5.2, Fig. 5 right; sol unless stated).** Relay at
  150 / 300 / 600 / 1,500 / 3,000 chars: ≈1.8 (read off the plot) / 1.65 /
  ≈2.5 (plot) / 3.83 / 3.65. Compaction relay at 300 comes close to or
  beats Relay at 1,500: sol 3.47 vs 3.83, astra 4.18 vs 3.97, terra 2.37
  vs 2.31. Giving Compaction relay 1,500 chars "adds nothing" (3.47).
  Requests: revise 1.98; ledger 1.57 at 300 and 1.95 at 600 (plain 1.65,
  2.48); verify-and-record-observations at 1,500, 3.49 (plain 3.83). No
  intervals are given for these.
- **Where the loss falls (§5.3, Fig. 6a).** sol loses 2.23 marks in machine
  worlds and 0.96 in craft, astra 1.20 and 0.79; grid, care and trade lose
  ≤0.22 for both. The two models' world-level losses correlate (rank 0.60).
  By the life in which Memory first reaches the goal (sol): 13 worlds
  within two lives lose 0.03; 20 in lives 3–4 lose 0.62; 23 later or never
  lose 1.01 (astra 0.09 / 0.65 / 0.56). The four weaker models lose most in
  tool, care and trade (up to 1.22), uncorrelated with the strongest
  (−0.21 to 0.08).
- **What the notes contain (§5.4, Fig. 6b–c).**
  - Facts stated correctly in Relay notes rise from 30% after life 1 to
    50% by life 5 and plateau (sol; astra 24% → 50%). sol's late notes in
    machine worlds state 17%. The designer's note states all of them in the
    same space.
  - Per hop, a fact right in one note is right in the next 89% (sol) / 87%
    (astra) of the time. An absent fact appears 14% / 15%. A wrong fact is
    corrected 16% / 20% and repeated 60% / 49%, but wrong statements are
    rare (6% / 4% of facts in late notes).
  - With the record, late Compaction-relay notes state 47% / 54% of facts
    against 24% / 40% for Relay on the same worlds, and an absent fact is
    supplied twice as often (16% vs 8%; 17% vs 12%). Only 12 (sol) and 9
    (astra) worlds were graded ("so far").
  - On the ten high-loss worlds sol's late notes are 16% true rules, 18%
    false claims, 65% single-game records; on ten no-loss worlds 44 / 8 /
    48. astra 33 / 14 / 53 vs 44 / ≈3 / 53. Every kind of fact (goals,
    dangers, mechanisms, orderings, quantities) is stated rarely in machine
    worlds (4–27%) and more often elsewhere (37–69%).

## Critique / open questions

- **"Repeated compression" is not paraphrase drift.** C − R changes two
  things: the Relay writer re-compresses a compressed text, *and* it sees
  only one life of evidence. The Compaction-relay writer sees every life.
  The paper's own account and note data point at the second. Stated content
  survives a hop well (89% / 87%). What fails is that observations never
  accumulate into a rule. No arm gives a writer the previous note plus all
  records, or re-summarizes a summary with no new evidence, so drift in the
  strict sense is not isolated. The mechanism is closer to
  [[literature/papers/goyal2026does]]'s "lost at write time" and
  [[literature/papers/lencevicius2026when]]'s observed-vs-derived split than
  to the summary-of-summary drift that
  [[literature/papers/nguyen2026cliffcompaction]] argues from.
- **The headline decomposition is on worlds selected by the outcome.** The
  10 highest-loss worlds are picked on M − R, and Table 4 appears to reuse
  the same R and M runs. Regression to the mean then inflates R's
  shortfall and the C − R share. The wider sets partly answer this (sol
  61% on all 56 worlds, astra 83% on 29), and the CIs there exclude zero.
  Still, read 70–82% as an upper-range estimate.
- **Not uniform across models.** For terra, one compression (0.92) costs
  more than repeating it (0.81), and the record recovers 40%. "Changing the
  agent costs nothing" is really "costs nothing or helps": astra's fresh
  agents beat Memory by 0.49. Possibly the Memory agent is hurt by its own
  long history, which the paper does not discuss.
- **The budget regime is tight, and the paper's own sweep weakens the
  transfer to long summaries.** At 1,500 chars a plain Relay note matches
  or beats a 300-char note written from the record (3.83 vs 3.47 for sol).
  A Claude Code compaction summary is one to two orders of magnitude longer
  than 300 chars. How large C − R is at that length is untested here, and
  for sol the gap is already gone at 5× the budget (no intervals).
- **The writer's record fits in its window.** Eight lives of ≤60 short
  moves on ~190-char screens is small. At an overflow compaction the record
  by definition does not fit, so "write from the full record" is a
  hand-off or brief-writing prescription. At overflow it becomes "keep the
  record addressable and make the writer use it", which is a different
  intervention, and an offered address is rarely used in this graph
  ([[literature/papers/busch2026ctxagent]],
  [[literature/papers/sheshanarayana2026do]],
  [[literature/papers/fan2026empirical]]).
- **The knowledge is induced, not given.** What is lost is a rule no one
  has stated yet. The box's PreCompact concern is constraints the user
  *stated* in chat, which are conclusions from the start. On this paper's
  account those are the content that survives a hop (87–89%), though that
  per-hop rate would compound over many compactions. The paper does not
  test given constraints.
- **The writer-instruction null is thin.** One model, 10 worlds, no
  intervals, and the "verify and record observations" variant was only run
  at 1,500 chars. It agrees with
  [[literature/papers/chu2026when]], where compaction-prompt wording was not
  the lever either, and with salle2026benchmarks's failed generic "keep
  constraints" ask.
- **Reproducibility.** No code, no worlds, no prompts beyond one line, no
  appendix, unnamed reader model, notes only partly graded, no stated
  temperature or seed. The reference model used to admit worlds
  (deepseek-v4.1-flash) is also an evaluated model, so its Solo and Oracle
  scores are fixed near the admission thresholds by construction.
- **Date oddity.** The arXiv stamp on the PDF says "arXiv:2611.18642v1
  [cs.AI] 14 Oct 2026", matching the API's v1 date, but a 2611 ID implies a
  November submission. The newest references are October 2026 (arXiv
  2610.12114). The body gives no date. Unresolved.
- **Fit with the graph.** First measurement in the graph that separates
  compressing once from compressing repeatedly, which
  [[concepts/context-eviction-policy]] records as "argued, not measured".
  The same direction as [[literature/papers/xu2026what]] (a rewrite from
  full context beats the agent's own running reports at matched length)
  and [[literature/papers/zhang2026correct]] ("correct now, insufficient
  later"). [[literature/papers/zha2026do]] left the re-compaction regime
  untested; this covers re-compaction, but of induced facts, not rules.

## Trust signals

- **Credibility:** 2.
  - For: a clean matched design. Life g uses the same instance in all four
    sequence conditions, neighbouring conditions differ in one respect,
    and the decomposition is exact. Intervals come from resampling worlds,
    and scores are graded mechanically from the program. Note grading has a
    second reader (κ = 0.81) and none of the conclusions rest on grades
    alone. Limitations are stated plainly, including that size of loss is
    not claimed to transfer to real work.
  - Against: single author, no peer review, no code, worlds or prompts
    released, no appendix. Headline decomposition on outcome-selected
    worlds. Several key comparisons (variants, length sweep) have no
    intervals and cover one model. Note grading is incomplete. The arXiv
    date is inconsistent with the ID.
  - Not 3 because nothing can be rerun or checked. Not 1 because the design
    and reporting are careful and internally consistent.

## Follow-up

- **Relevance:** 4. First matched measurement of single vs repeated
  compression of what an agent has learned, which is material new evidence
  for [[concepts/context-eviction-policy]] (its drift argument was
  unmeasured) and for [[concepts/lossless-context-offload]] (a raw record
  handed to a fresh agent fully replaces having lived it). Not 5 because
  the knowledge is induced in toy worlds at a 300-char budget, and the
  paper's own 1,500-char result weakens transfer to long summaries.
- **What it implies for the box (our reading).**
  - *PreCompact bullet.* It does not test the bullet's target (stated
    chat-only constraints) and does not argue against its wording. Its
    data say written conclusions are carried; the unconcluded class is
    what is lost. It does argue against adding writer instructions
    ("record unexplained observations", ledgers) as a fix for that class.
  - *"Compact from the transcript, not the last summary"* is implementable
    where the record fits the writer: subagent briefs, session hand-offs,
    `NOTES.md` "Next" written from the journal and git log rather than from
    the previous `NOTES.md`. At overflow compaction it is not, by
    definition.
  - *Evaluation.* For any handoff or compaction change, run a no-handoff
    arm (Memory) and a raw-record arm, and score progress, not completion.
- **Candidates** (cited, not yet in the graph):
  - KC & Budathoki 2026, "Handoff debt: the rediscovery cost when coding
    agents take over interrupted tasks" (arXiv 2606.02875). The closest
    real-task handoff study.
  - Wang, Zhang, Lee & Yang 2026, "Lost in compaction" (COMPINT, arXiv
    2608.11242). Already a candidate in salle2026benchmarks and zha2026do.
  - Liu 2026, "What does context compression cost an agent? Interaction
    costs unrevealed by task-completion metrics" (arXiv 2608.16370).
  - Spencer 2026, "Ground truth first" (arXiv 2607.21962): memory-writer
    audit against a fixed script.
  - Kwon 2026, "Reclaim evaluation: a lossy memory is worse than an empty
    one" (arXiv 2606.25449).
  - Tessler et al. 2021, "Learning to solve complex tasks by growing
    knowledge culturally across generations" (arXiv 2107.13377): the human
    protocol this one copies.
