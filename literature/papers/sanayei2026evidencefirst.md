---
kind: paper
title: "Evidence-First Agentic Verification"
authors: ["Reza Sanayei", "Razvan-Gabriel Dumitru", "Sunjie Hou", "MohammadHossein Rezaei", "Haris Riaz", "Robert Vacareanu", "Minglai Yang", "Ying Liu", "Mihai Surdeanu", "Daniel Yue Zhang"]
institutions: ["Scale AI", "University of Arizona"]
year: 2026
venue: "arXiv 2610.17724 (cs.AI), v1 2026-10-13, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.17724"
code_url: "https://github.com/rsanayei/sauron"
citations: null
source: "raw/papers/sanayei2026evidencefirst.pdf"
added: "2026-10-20"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/shared-substrate-contagion]]"
tags: ["agentic-verifier", "rubric-grading", "llm-judge", "evidence-atoms", "receipts", "state-vs-trace", "self-report", "fault-injection", "clean-controls", "false-alarm-rate", "mechanism-ablation", "bounded-queries", "criterion-interpretation", "reward-hacking", "trail", "gandalf", "sauron"]
---

# Evidence-First Agentic Verification

## TL;DR

Sauron is an agentic verifier for rubric-graded agent runs. A cheap planner
(GPT-5.6 Luna) groups rubric criteria by shared evidence and writes an
advisory gist. One investigator per cluster then reads the final state and
the execution trace through bounded, read-only queries. It proposes
**evidence atoms** (claim, evidence type, source locator). A separate
verifier re-fetches the cited source and checks each atom, mechanically for
exact values and with an isolated "faithfulness" model call otherwise. A
finalizer accepts a MET/UNMET verdict only if it cites verified atoms. Each
verdict ships with a receipt. Undecided criteria default to UNMET with an
evidence-missing marker.

Three evaluations, all graded on fixed trajectories:

1. **PA100** (100 internal personal-assistant episodes, 2,586
   human-labelled criteria). Sauron has the best F1 on all eight backbones.
   With GPT-5.6 Sol it scores 0.96 against Gandalf's 0.92, at $298 vs $388.
   The mean margin over Gandalf is +0.03 F1. On Claude Sonnet 5 it is +0.01.
2. **PAFault** (637 author-built fault/control pairs, 13 families × 49).
   Δfault = detection − false alarm = 0.95 (97% / 2%). The best baselines
   given the full trace reach 0.67 and 0.71.
3. **TRAIL** (148 real traces). Sauron leads Gandalf and TRAIL's judge on all
   three metrics, at about twice their cost on GPT-5.6 Sol.

The mechanism ablation says most of the gain comes from reading the trace
(trace-blind: −0.15 F1). The claim-verification layer adds 0.03 F1 and
accounts for about 40% of the cost.

## Claims

- Rubric judges "may look in the wrong places and miss relevant evidence,
  or reach a conclusion its observations do not establish." Access alone
  does not ensure a supported judgment.
- A judge that takes the agent's claims at face value can be exploited as
  an RL reward. Sauron targets this by grounding verdicts in claims checked
  against state and trace.
- Sauron "achieves the highest F1 of five verifiers on all eight backbones
  tested while costing less than Gandalf." Its lead "widens on process
  criteria."
- On PAFault it "detects 97% of injected failures while flagging only 2% of
  clean controls, outperforming baselines even when they are given the full
  trace."
- Bounded queries and clustering cut cost. Sauron reads fewer than half
  Gandalf's input tokens on PA100 and is 2.1–4.2× faster on a 10-episode
  slice.
- Limitations the authors state (App. A): one agent (Opus 4.8) in one
  domain; self-family preference untested; the faithfulness checker is
  still a model call; agent-authored text in state or trace can still
  mislead model-based checks; and the injected faults "do not establish
  robustness to policies optimized to exploit Sauron."

## Methods

- **Inputs.** User request, final response, final state and artifacts,
  execution trace, rubric. Each criterion is tagged *outcome* or *process*
  by a Gemini 3.7 Flash classifier (99% agreement with a manual review of
  200 criteria).
- **Investigation.** The tools are trace search/get/window/tool-calls/
  ordering/span-tree, state summary/get/search/diff, artifact
  list/read/search/table/formula, plus `run_python` in a scratch copy of
  the workspace. Responses are capped at 24,000 characters. Session budgets
  scale with cluster size n: turns min(round(min(20 + 7.5n, 75)), 60);
  tokens 250k + 150k·n. Clusters hold at most 12 criteria.
- **Prompt rules (Fig. 7).** "State is the source of truth. Trace is a
  source of leads and process evidence. Never award a criterion solely
  because the agent claimed it was done." "Actively check trace-vs-state
  consistency." There is no abstention. The prompt also carries
  domain-specific guidance: scan every spreadsheet sheet for error cells,
  and detect PDF layout collisions with pdfplumber word boxes.
- **Verification.** Twelve evidence types (Table 8), e.g.
  `state_value_match`, `temporal_ordering`, `trace_state_discrepancy`,
  `missing_required_action`. Mechanical checks cover tool presence/absence,
  ordering, state equality or substring, and artifact existence. Everything
  else goes to a faithfulness checker that sees only the claim and the
  cited source (truncated to 12,000 characters). Rejected atoms return to
  the investigator with a reason. In clusters of 8 or more, a set-coverage
  call checks "all/every/none" criteria.
- **Development split.** Prompts, evidence types, budgets, clustering and
  the decision policy were fixed on a separate 50-episode development set
  before any evaluation.
- **PA100.** A stratified sample of 100 from a 700-episode internal corpus.
  Median episode: 53 trace events and 20 tool calls. About 26 criteria per
  episode, ~30% UNMET. Two annotators graded independently, with 95%
  pre-adjudication agreement and a senior adjudicator. Scoring is F1 and
  MCC with UNMET as the positive class, pooled over criteria. Abstentions
  count as errors.
- **PAFault.** Episodes are disjoint from PA100 and the dev set. Each
  family edits the trace, response, state or criterion (Table 11). Twelve
  of the 13 families map to Raj et al.'s failure taxonomy. Three simulate
  reward hacking: eval awareness (an inserted statement that work was
  shortened because the run looked graded), spec gaming (the producing
  action removed while state is kept), and a false compliance claim.
  Δfault is an unweighted mean over families. Always-MET or always-UNMET
  scores 0 on 12 families.
- **Baselines.** Gandalf (state and workspace tools, no trace by default),
  Agent-as-a-Judge gray_box, Archipelago (snapshot diff), and Autorubric
  (response only). Each runs its released defaults "without any additional
  domain-specific guidance". On PAFault, Gandalf_trace and
  Autorubric_trace receive the full trace as text. Autorubric_trace still
  has no state access.
- **TRAIL.** Each of the 21 error categories becomes a criterion, "the
  trace contains no ⟨category⟩ error". Span locations come from verified
  atoms. The authors wrote a location normalization and applied it to all
  three verifiers.
- **Ablations.** PAAbl has 860 decisions: 506 criteria from 20 PA100
  episodes plus 354 PAFault variants. GPT-5.6 Sol only.

## Results

- **PA100, GPT-5.6 Sol (Table 1).**

  | Verifier | F1 | MCC | Cost | Process F1 | Outcome F1 |
  |---|---|---|---|---|---|
  | Sauron | 0.96 | 0.94 | $297.69 | 0.97 | 0.96 |
  | Gandalf | 0.92 | 0.89 | $388.25 | 0.90 | 0.93 |
  | Archipelago | 0.92 | 0.89 | $2,058 | 0.88 | 0.93 |
  | AAAJ | 0.52 | 0.25 | $234.78 | 0.82 | 0.46 |
  | Autorubric | 0.65 | 0.48 | $76.09 | 0.77 | 0.61 |

- **In errors (Table 12, our arithmetic).** Sauron vs Gandalf, false awards
  + missed awards out of 2,586: Sol 65 vs 121, Fable 5.1 78 vs 122, Opus 5
  99 vs 146, Terra 96 vs 149, Kimi K3 97 vs 170, Gemini 3.7 Flash 89 vs
  130, Luna 131 vs 182, **Sonnet 5 144 vs 148**. Sauron's errors lean to
  missed awards (Sol: 21 false awards, 44 missed).
- **Process vs outcome (Table 13, means over 8 backbones).** Process F1 is
  0.94 vs 0.87 for Gandalf; outcome F1 is 0.93 vs 0.92. Process criteria are
  411 of 2,586 (16%), so the "widening lead" sits on the smaller subset.
- **PAFault (Table 2, Fig. 11).**

  | Verifier | Δfault | Detect | False alarm | $/variant |
  |---|---|---|---|---|
  | Sauron | +0.95 | 0.97 | 0.02 | 0.21 |
  | Autorubric_trace | +0.71 | 0.84 | 0.13 | 0.26 |
  | Gandalf_trace | +0.67 | 0.74 | 0.07 | 0.99 |
  | AAAJ | +0.36 | 0.78 | 0.42 | 0.07 |
  | Gandalf | +0.09 | 0.52 | 0.43 | 0.67 |
  | Archipelago | +0.08 | 0.58 | 0.50 | 0.24 |
  | Autorubric | −0.17 | 0.51 | 0.68 | 0.02 |

  Sauron is 0.90–1.00 on 12 families and 0.69 on reorder. The gaps to
  trace-enabled baselines concentrate in two state-visible families. On
  state mismatch: Sauron +0.90, Gandalf_trace −0.27, Autorubric_trace
  −0.94. On silent write failure: +1.00, −0.10, −0.20.
  **Our arithmetic from Fig. 11: on the other 11 families,
  Autorubric_trace averages 0.95, the same as Sauron (0.95).**
  Gandalf_trace averages 0.83 there.
- **TRAIL (Table 3).** GAIA, GPT-5.6 Sol: wF1 / location / joint 0.53 / 0.73
  / 0.44 vs Gandalf 0.46 / 0.53 / 0.25, at $841 vs $441. On SWE-bench Lite
  (31 traces) joint accuracy is 0.10 vs 0.06. Sauron on GPT-5.6 Terra
  ($202) beats both Sol-backed baselines.
- **Ablations (Table 4, ΔF1 vs 0.96).**

  | Variant | ΔF1 | Rel. cost |
  |---|---|---|
  | State only (no trace tools) | −0.15 | 1.00× |
  | Trace in context, no trace tools | +0.01 | 3.03× |
  | No gist | 0.00 | 1.15× |
  | No faithfulness checker | −0.03 | 0.66× |
  | No mechanical checks | 0.00 | 1.01× |
  | No claim verification | −0.03 | 0.60× |
  | Unclustered | −0.01 | 1.63× |

- **Error analysis (App. G, all 65 PA100 disagreements on Sol).** 48 are
  stricter readings than the annotators': 34 positive criteria marked
  UNMET on literal wording (e.g. "next billing May 22" without the year)
  and 14 negative criteria read broadly. The rest: 6 lenient readings, 5
  misinterpretations or conditional criteria, 3 polarity conventions, 2
  budget exhaustions, 1 missed violation. "In all but three cases, the
  disagreement concerned how to read the criterion, not what the evidence
  showed."

## Critique / open questions

- **Against the digest framing: BELOW (mild).** Every number in the digest
  entry matches the paper. The body narrows what those numbers mean in
  three ways the digest leaves out:
  1. The PA100 margin is small and has no interval: +0.03 F1 on average,
     +0.01 on Sonnet 5 (144 vs 148 errors), one run per cell, no seeds,
     no paired test, and 2,586 criteria clustered in 100 episodes from one
     agent. Some backbones show a real error-count gap (Sol 65 vs 121), but
     the paper does not test it.
  2. The 97% / 2% comes from a fault suite the authors built alongside the
     verifier. Its decisive families mirror Sauron's own evidence types
     (`trace_state_discrepancy`, `state_mutation`) and a prompt rule
     ("Actively check trace-vs-state consistency"). On the 11 families
     where a plain LLM judge can see the fault in the trace text,
     Autorubric_trace ties Sauron (0.95 vs 0.95, our arithmetic). The
     "even when given the full trace" gap is mostly state access plus that
     rule.
  3. The ablation does not credit "evidence-first validation" with much.
     Removing claim verification costs 0.03 F1, removing trace access 0.15.
     Putting the full trace in context instead of bounded queries gains
     0.01 at 3× cost. The digest's "relies on a claim only after
     independently validating it" is the design, but it is not what
     carries the result.

  The "several simulate reward hacking" wording is fair: the paper says
  these are scripted edits, "not exploits discovered through policy
  optimization."
- **Clean controls are reported, which is rare.** The 10-12 batch noted
  that false-flag rates on clean controls are usually missing. Here they
  are reported per family with matched controls, 49 per family, plus
  construction screens meant to keep the expected verdict unambiguous. The
  catch: these controls are clean by construction for one targeted
  criterion. They are not real clean runs graded on a full rubric. The
  PA100 false-award count (21 of ~1,810 MET criteria on Sol) is the better
  real-world false-accept figure, and it comes from a different
  population.
- **Uneven prompt guidance.** Sauron's investigator prompt includes
  spreadsheet and PDF-layout instructions that look tuned to this
  corpus's deliverables. The dev set was drawn from the same corpus.
  Baselines run with no domain guidance. The dev/test split is clean
  (prompts frozen first), but the comparison is not like for like. No arm
  gives a baseline the same guidance or removes it from Sauron.
- **Error direction.** Sauron's residual errors are mostly missed awards
  from literal readings. That is the safe direction for a completion gate
  and the costly one for an RL reward: a policy trained on it would learn
  to restate years and exact wording. The authors do not test Sauron as a
  reward.
- **Unresolved collapses to UNMET.** Budget exhaustion and absent evidence
  both score UNMET, with a marker. This is conservative, but it merges
  "evidence against" with "could not check", which guo2026epoch's
  three-valued gate keeps apart. Only 2 of 65 disagreements came from
  exhaustion here. On TRAIL, where whole traces must be scanned, bounded
  queries save less, and that share could grow.
- **Same backbone verifies itself.** The faithfulness checker and the
  investigator use the same model, so independence comes from context
  isolation only. This is the pattern xu2026dont and zhang2026who describe:
  independence from what the reviewer is shown, not from a different
  model. The graded agent is Opus 4.8. Claude-backed Sauron runs (Opus 5,
  Sonnet 5, Fable 5.1) do not stand out, but self-family preference is
  untested (App. A).
- **Data not released.** PA100, PAFault and PAAbl come from an internal
  Scale corpus. The code is released (MIT), so the verifier can be rerun on
  other data, but the headline numbers cannot be reproduced.
- **Live fronts.**
  - *Redirect-before-halt proposal:* no bearing.
  - *No-improvement noise band:* the ablation deltas (0.00–0.03 on 860
    items, one run) are the kind of differences a noise band would treat
    as no change. Read the ablation as ranking trace access far above
    everything else, not as resolving the smaller mechanisms.
  - *HCE:* the 50-episode dev set, frozen before test, is good practice.
    The fault suite is home-field: built by the same authors around the
    same evidence types (see [[concepts/hce-evaluation]]).
  - *Completion gates on this box:* the cheap transferable part is the
    atom schema (claim, type, locator), mechanical check first, model check
    only on the cited span, and state over narration. For `/ingest`-type
    skills, "the note exists and its frontmatter parses" is a
    `state_value_match`, not a sentence in the final message.

## Trust signals

- **Credibility:** 3.
  - For: Scale AI with a University of Arizona co-author (Surdeanu). Code
    released under MIT at github.com/rsanayei/sauron (repo verified
    2026-10-20, last pushed 2026-10-14, with src/, tests/, docs/ and an
    adapter example). Prompts are printed in full in the appendix.
    Development and evaluation are separated. Eight backbones, three
    benchmarks, including one external (TRAIL) with independent labels.
    Labels have 95% pre-adjudication agreement. There is a full
    disagreement audit, and the limitations section is candid.
  - Against: not peer reviewed. PA100 and PAFault are internal and not
    released. Single runs with no intervals anywhere. The fault suite and
    its expected verdicts are author-constructed, and the TRAIL adaptation
    and location normalization are the authors'. The verifier prompt has
    domain guidance that the baselines lack. Model names are
    provider-internal snapshots.
  - Not 4 because the headline comparisons have no uncertainty and the
    main datasets cannot be inspected. Not 2 because the code, the full
    prompts, the per-family tables and an external benchmark make the
    claims checkable in part.

## Follow-up

- **Relevance:** 4. It is the first verifier in this graph to report a
  mechanism ablation next to paired clean-control false alarms per fault
  family. That is material evidence for
  [[concepts/evidence-gated-completion]]:
  - Most of the gain comes from the verifier reading the state the claim
    is about, plus an explicit state-vs-claim cross-check. A plain judge
    given only the trace trusts claims that contradict state: Δ −0.94 on
    state mismatch, flagging controls more often than faults.
  - The claim-checking layer is a small increment, +0.03 F1 for ~40% of
    the cost.
  - Once evidence is verified, the residual error is how the criterion is
    read (62 of 65 disagreements). That puts the burden on requirement
    wording, as in [[literature/papers/waks2026what]] and
    [[literature/papers/kim2026into]].

  It qualifies [[literature/papers/zhang2026veriharness]]'s "access alone
  does nothing". Gandalf_trace, with state and trace, still fails state
  mismatch. Autorubric_trace, with the trace alone, matches Sauron on
  trace-visible faults. Receipts tie each verdict to cited, verified
  sources, which is [[concepts/citation-anchoring]] applied to verdicts.
  The typed atom schema relates to [[concepts/typed-claim-partition]]. Not
  5 because the domain is personal-assistant tasks rather than ML research,
  it grades fixed trajectories rather than gating a live loop (so the hold
  on a refusal-driven outcome delta is untouched), and the main numbers
  have no intervals.
- Related in the graph:
  - [[literature/papers/xu2026dont]]: least-privilege routing; Sauron's
    isolated faithfulness checker is the same move.
  - [[literature/papers/qin2026llm]]: the trace is agent-writable; Sauron
    treats state as truth and the trace as leads, and its injected faults
    do not include a tampered trace.
  - [[literature/papers/huang2026reward]]: the red-team hacks that evade
    review "tend to resemble ordinary work", and evading pairs rise from
    7/150 to 56/150 once reviewer feedback is in the loop. Sauron's
    simulated hacks are scripted edits with a fixed signature, a much
    easier target.
  - [[literature/papers/zhu2026claimreceipt]]: sufficiency and coverage
    receipts. Sauron's set-coverage check for "all/every" criteria is the
    coverage half.
  - [[literature/papers/ding2026autonomous]]: cited as [6], the
    verification-gap survey.
- **Candidates** (cited, not yet in the graph):
  - Dücker et al. 2026, Gandalf, "Verifying agents in rubric-graded
    environments" (openreview 5YK4zLxBfG).
  - Deshpande et al. 2025, TRAIL (arXiv 2505.08638).
  - Raj et al. 2026, "Model or harness? An interaction-centric taxonomy for
    localizing agent failures" (arXiv 2607.28802).
  - Mahmoud et al. 2026, "Reward hacking in rubric-based reinforcement
    learning" (arXiv 2605.12474).
  - Zhuge et al. 2025, Agent-as-a-Judge (ICML 2025).
  - Vidgen et al. 2026, APEX-Agents / Archipelago (arXiv 2601.14242).
