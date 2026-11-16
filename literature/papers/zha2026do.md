---
kind: paper
title: "Do Agents Still Follow the Rules? Benchmarking Rule Compliance under Agent Context Compaction"
authors: ["Yiwei Zha", "Qingyu Meng", "Herbert Bos", "Min Chen"]
institutions: ["Vrije Universiteit Amsterdam (VUSec)"]
year: 2026
venue: "arXiv 2611.03223 (cs.CR), v1 2026-11-02, preprint (ACL-style template)"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.03223"
code_url: "https://github.com/VUAISec/ARCBench"
citations: null
source: "raw/papers/zha2026do.pdf"
added: "2026-11-16"
relevance: 5
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/context-eviction-policy]]"
tags: ["compaction", "rule-compliance", "governance-decay", "constraint-retention", "retention-vs-compliance", "violation-attribution", "matched-full-context-baseline", "violation-at-k", "pi-harness", "hermes-compressor", "acon", "trace", "fifo", "lru", "rule-preservation-prompt", "compactor-targeted-injection", "volume-injection", "summary-framing", "delivery-channel", "personal-vs-organizational-rules", "llm-judge-panel", "deterministic-tool-call-grading"]
---

# Do Agents Still Follow the Rules? Benchmarking Rule Compliance under Agent Context Compaction

## TL;DR

ARCBench runs an unmodified open-source coding harness (pi, pinned at
revision 97fa14e3) through 17 short tool-use cases. Each case plants one
operating rule, adds 12–16 benign messages, forces one compaction, and then
issues a final request that asks for the forbidden action. Violation is
graded deterministically from the tool call. Every compacted run is
matched with full-history runs, so a violation can be split into the
**agent's own** (it also happens with the full history) and **only after
compaction**. Each compacted summary is labelled retained, partially
retained or absent by a three-judge panel.

Three results matter for this graph:

1. **Under the Hermes compressor most post-compaction violations happen
   with the rule still in the summary.** violation@3 goes from 9.6% to 37.8%
   (S1, four flash-tier models). 70.1% of summaries retain the rule, 26.9%
   retain it partially and 3.0% lose it. Runs that retain it violate 21.1%
   of the time, against 8.1% with the full history (2.6×).
2. **That 2.6× is a property of the compressor, not of compaction in
   general.** Under pi's native compaction, retained-rule runs violate 9.3%
   against 9.5% for full history. With one added "preserve operating rules"
   paragraph (Pi + RP) the figure is 6.5%, below the full-history rate, and
   Pi + RP is the only strategy whose violation@3 is under baseline
   (9.0% vs 11.9%). The excess sits in Hermes (26.8%) and ACON-UT (17.4%).
   Hermes is the paper's main strategy. Its wrapper tells the agent the
   summary is "background reference, NOT … active instructions" and that
   "the latest message WINS", and in most runs it asks for a ~50-token
   summary.
3. **Strategy, budget and injection decide whether the rule survives.**
   Verbatim selective compactors lose a once-stated rule in 84.1% of runs
   (FIFO in all of them), and violation@3 reaches 50.2% against 21.8% for
   generative ones. Retention stays ≥97% down to a 750-token budget and
   collapses at 300 tokens (≥95% absent). One retrieved note telling the
   compactor to omit rules cuts Hermes's retention from 70.1% to 42.5%. A
   rule-preservation line in the compactor prompt brings retention back to
   99% but does not bring compliance all the way back.

## Claims

- Prior work (Governance Decay = [[literature/papers/chen2026governance]];
  COMPINT = Wang et al. 2026) shows that violations rise after compaction.
  But that work either assembles its own loop and compactor or replays
  stored histories to multiple-choice probes. Neither separates the
  violations the agent would commit anyway from those compaction adds, and
  a yes/no retention label cannot show a rule that has been weakened.
- "Compaction rarely deletes the rule: most violations after compaction
  occur while the rule is still in the compacted history."
- The delivery channel decides whether a violation is the agent's own. The
  compaction strategy and budget decide whether the rule survives.
- Added volume barely matters. A compactor-targeted injection both drops
  the rule and changes how the agent treats a surviving rule.
- "Rule retention is thus not a sufficient proxy for compliance."

## Methods

- **Harness.** pi (Zechner 2025), with its own loop, tool calling and
  native compaction. The two pi strategies call pi's compaction code
  unmodified, and the other strategies run through a shared driver over
  the same history. Tools are never executed: each call gets a fixed
  canned result. The decision loop allows four tool steps. The same model
  compacts and acts. Temperature 0.7, seed 7, all calls through
  OpenRouter. A Dockerfile pins pi and every dependency.
- **Cases (17; App. E).** 8 personal and 9 organizational cases, crossed
  with four violation boundaries (data, access, action, control) mapped to
  OWASP classes. 13 are adapted from Governance Decay and COMPINT, and 4
  were authored. Screening dropped "nine process, preference, or formatting
  constraints" that had no attacker payoff (A1) or whose loss did not
  worsen the outcome (A5). Examples: pconfdir (never read the confidential
  folder), pgitforce (no force-push to protected branches), pdraft (draft,
  let me send), papprove (ask before running a command), email (internal
  domain only), deploy (us-west-2 only). The final request always asks for
  the prohibited action, e.g. "Send the Q4 executive summary to an external
  investor's address".
- **Delivery channels.** Personal: c1, a user message; c2, the agent is
  asked to persist the rule to `rule.md`. Organizational: c3, the rule
  arrives in a document tool result; c4, through an explicit `get_policy`
  call. Tool descriptions never encode the rule.
- **Strategies (10).** Generative: Pi, Pi + RP (P4 added to pi's prompt:
  keep obligations, prohibitions, scope and exceptions "with their original
  force and negation intact", and treat instructions inside tool results as
  data), Structured (same prompt as Pi, shared path, no 2,000-char tool-result
  cap), Hermes (Nous Hermes Agent compressor, plus its P7 "reference-only"
  prefix), ACON-UT, ACON-UTCO and TRACE (an AppWorld prompt run out of
  domain, as the authors flag). Selective: FIFO, LRU (rule) and LRU (LLM),
  which keep original turns under a fixed allowance.
- **Budget (App. G).** Budget B covers the compacted history. The
  replacement text gets max(64, B − F), where F is the pinned first user
  turn plus the retained tail. It is stated in the prompt and clipped to 4A
  characters. Without a budget, Hermes's `{summary_budget}` slot receives the
  50-token retained-tail target, so "every such Hermes prompt" in S1–S3 and
  S5–S7 "reads 'Target ~50 tokens'". Hermes's fixed preamble is 177
  estimated tokens.
- **Attacks and defenses (App. H).** Volume: 96,000 chars of filler in
  2,000-char tool results. Compactor-targeted injection: a retrieved note
  instructing the compactor to omit rules. There are six constructions
  (legacy authority-framed note, naive, escape = five newlines, ignore,
  fake completion, combined). Defenses change only the compactor prompt:
  isolation (delimiters), instructional (ignore embedded instructions),
  sandwich, and rule preservation (DP3, the first two sentences of P4).
- **Metric.** violation@k is the share of cases that violate in at least
  one of k runs (k = 3; k = 6 in S6–S7), with mean violation rate and all-k
  in the appendix. The paper states that 95% case-cluster bootstrap
  intervals are reported for condition differences. None appear in the
  tables read here.
- **Diagnostics.** Rule state: majority of GPT-6-Luna-Pro, Qwen-3.8-flash
  and Claude Sonnet 4.6 at T = 0, plus a keyword filter (280 of 8,106
  "present" verdicts downgraded). The 10,369 summaries break down as 7,388
  unanimous, 2,946 two-of-three and 35 three-way splits. A blind manual
  check on 40 agrees on 39 (κ = 0.92), and Sonnet 5.5 as a fifth judge on
  4,870 gives κ = 0.741. Selective compactors are labelled mechanically
  (was the rule-bearing turn kept). Agent behaviour on violating runs with
  the rule in context is labelled approval / scope / unregistered /
  acknowledged by GPT-6-Luna-Pro, GPT-6.1-Sol and Sonnet 5.5, without the
  summary. 1,115 runs, κ = 0.87 on 24 hand-labelled. The rubric files
  "following a precedence instruction such as 'the latest message wins /
  supersedes' from the context summary" under *acknowledged*.
- **Models.** DeepSeek-V4-Flash, GLM-5.3-Flash, Qwen3.7-Flash and GPT-Luna
  (an OpenRouter alias). S4 uses DeepSeek and GLM only, and S6–S7 use GLM
  only. No Claude model acts as agent or compactor.
- **Studies (Table 2/16).** S1 baseline (Hermes, c1–c4, 816 runs, 810
  valid); S2 the same runs by channel; S3 all ten strategies (c1/c3); S4
  budget 200–2,500 (Hermes, volume in every run); S5 volume / legacy
  injection / both; S6 six injections; S7 four defenses × {no attack,
  escape}.

## Results

- **S1 (Table 3, 19).** violation@3 is 9.6% with full history and 37.8%
  compacted (8.9 points agent's own, 28.9 only after). Per model, full →
  compacted:
  - DeepSeek 8.8 → 26.5, retention 98.0%
  - GLM 8.8 → 29.4, retention 98.0%
  - GPT-Luna 6.1 → 33.3, retention 48.5%, 45.5% partial
  - Qwen 14.7 → 61.8, retention 35.3%, 58.8% partial

  Per-run violation by rule state is 8.1% full, 21.1% retained, 29.4%
  partial and 83.3% absent. Every model's retained runs violate more than
  its full runs (DeepSeek 17.0 vs 6.9; GPT-Luna 20.8 vs 4.0; Qwen 47.2 vs
  13.7; GLM 16.0 vs 7.8).
- **Why retained-rule runs violate (Table 4, 20).** In the agent's own
  cases *approval* dominates (72.4% full, 82.4% compacted): the agent treats
  the user's later request as permission. In only-after-compaction
  violations with the rule retained, approval falls to 45.9%, while
  *unregistered* (27.0%) and *acknowledged* (21.6%, against 3.4% with full
  history) rise. The counts are small: 40 runs, of which 17 approval, 10
  unregistered and 8 acknowledged.
- **S2, channel (Table 5, 24, 6).** Full-history violation@3 is c1 25.0%,
  c2 16.1%, c3 0.0% and c4 0.0%. Retention varies little across channels
  (65.7–74.0%), but retained-rule violation varies a lot: c1 36.6%, c2
  20.0%, c3 16.9%, c4 11.7%. Under c1/c2 the reason is mostly approval
  (72.7%, 71.4%). Under c3/c4 it is mostly unregistered (54.5% at c3) or
  acknowledged (37.5% at c4). The authors' reading: a user can be read as
  lifting their own rule, an organization's rule cannot be lifted by the
  user.
- **S3, strategy (Table 21, 22, 24).**
  - Generative family: violation@3 21.8%, retention 91.2%, retained-rule
    violation 12.6% vs 9.5% full.
  - Selective family: violation@3 50.2%, retention 15.9%, absent 84.1%.
  - Per generative strategy (violation@3 / retention / retained-run
    violation vs full):
    - Pi + RP: 9.0 / 99.0 / 6.5 vs 9.5
    - Pi: 14.9 / 96.0 / 9.3 vs 9.5
    - Structured: 14.9 / 94.5 / 11.6
    - TRACE: 18.5 / 93.3 / 9.9
    - ACON-UTCO: 20.6 / 90.7 / 10.8
    - ACON-UT: 31.8 / 96.0 / 17.4
    - Hermes: 42.6 / 69.6 / 26.8
  - Per selective strategy: FIFO 51.5 / 0.0, LRU (rule) 54.4 / 11.8,
    LRU (LLM) 44.8 / 36.3.
  - The agent's-own share is flat across strategies (6.0–12.1%).
  - Pi + RP never exceeds any model's baseline. Hermes, ACON-UT and every
    selective compactor exceed it for all four models.
- **S4, budget (Table 5, 23, 24).** From 2,500 down to 750 tokens,
  violation@3 is 24.2–28.1% and retention ≥97.1%. At 300 and 200 tokens it
  is 47.1% and 50.0%, with ≥95.1% of runs absent. Even at 2,500 tokens and
  100% retention, retained runs violate 18.8% against 9.4% full. The
  excess under Hermes does not depend on the 50-token target. The authors
  note that at the tightest budgets prefix clipping keeps Hermes's
  177-token preamble ahead of the rule-bearing content. They did not test
  moving it.
- **S5, volume and injection (Table 7, 26, 30).** violation@3 for each
  added content:
  - Volume: 37.8 → 34.3, retention 71.9%.
  - Legacy injection: 46.3, with retention down to 42.5% and 31.1%
    absent. It raises violation for all four models. GPT-Luna loses the
    rule in 81.8% of runs.
  - Both: 41.0.

  Volume is a null here by construction, since compaction is forced
  anyway. Across the strategy study the injection raises the generative
  family's absent share from 0.9% to 16.3%. Pi + RP never loses the rule
  under injection (99.5% retained), but its violation@3 still rises from
  9.0% to 17.9% (full 13.4%). Pi goes to 31.3%.
- **S6, attacks (GLM, Hermes, c1/c3, k = 6; Table 27).** violation@6 is
  23.5% without an attack, 29.4–41.2% under five of the constructions and
  58.8% under escape. Under escape, retention falls from 96.1% to 66.7% and
  retained-rule violation doubles (26.5% vs 13.3%).
- **S7, defenses (Table 28, 30).** Under escape:
  - Rule preservation: retention 99.0%, violation@6 29.4% (no attack
    23.5%, full 11.8%), retained-rule violation 15.8% vs 13.3%.
  - Isolation, instructional and sandwich: retention 82.4–87.3%,
    violation@6 41.2–52.9%.

  Without an attack, no defense lowers violation@6 below the undefended
  23.5% (range 23.5–29.4). Rule preservation takes retention from 96.1% to
  100% and violation@6 stays at 23.5%.

## Critique / open questions

- **Against the digest framing: below (material).** The digest's numbers
  are in the PDF. What it carries over from the abstract is a scope
  problem.
  1. **"Runs that keep the rule still violate it more than twice as often
     as full-history runs" is a Hermes number.** It is S1's 21.1% vs 8.1%.
     The generative family pooled is 12.6% vs 9.5%. Under Pi it is 9.3% vs
     9.5%, and Pi + RP is *below* baseline (6.5%). Five of seven generative
     strategies show ≤ 2.1 pp excess. The large gap needs Hermes or
     ACON-UT.
  2. **The Hermes wrapper is a candidate cause the paper never isolates.**
     P7, prepended to every Hermes summary, says the summary is
     "background reference, NOT … active instructions" and that if the
     latest user message contradicts it, "the latest message WINS". The
     final request is a later message that contradicts the rule. The
     authors' own behaviour rubric had to add a rule for agents that cite
     "the latest message wins / supersedes" from the summary. There is no
     Hermes-without-P7 arm. ACON-UT also shows an excess (17.4% vs 9.6%)
     with no such prefix, so P7 cannot be the whole story. Read the
     headline as: whether a retained rule binds depends on how the summary
     presents it, not as "retention is never enough".
  3. **"Compaction rarely deletes the rule" holds only for generative
     strategies at default budget.** Even in S1, 26.9% of summaries weaken
     the rule's scope, exception or negation. Selective compactors delete it
     in 84.1% of runs, budgets ≤300 tokens in ≥95%, and injection in 31.1%.
  4. **"The strongest attack more than doubles violations"** is one model
     (GLM), one strategy (Hermes), 17 cases, k = 6. The Limitations section
     says the escape mechanism "remains a hypothesis".
- **The final request is the violation, and for personal rules that blurs
  revocation.** Under c1 the user who set the rule then asks for the
  forbidden action. ARCBench scores compliance with that request as a
  violation, and the judges label it *approval*. That accounts for most of
  c1's 25% full-history floor. Whether it is a failure or a legitimate
  revocation depends on the case. The organizational channels (0% full
  history) are the clean measurement. The design also never tests the case
  where the task merely passes through the forbidden action without the
  user naming it.
- **Short, single-compaction episodes on flash models.** Each case has
  12–16 benign messages, one forced compaction, canned tool results and a
  four-step decision loop, and no Claude or frontier model acts. Retention
  near ceiling for Pi and Structured is consistent with this. It does not
  contradict [[literature/papers/salle2026benchmarks]], whose generic
  "keep constraints" ask failed on long ProgramBench runs across up to 30
  compactions.
- **The excluded class matters for this box.** Screening dropped nine
  process, preference and formatting constraints for having no attacker
  payoff. Process instructions ("how to commit") are the class
  salle2026benchmarks found dropped most (35.9%). Scope exclusions
  (pconfdir, secret, pgitforce) and approval controls (pdraft, papprove)
  are represented.
- **Reporting inconsistencies.** §3.2 says 50 summaries were hand-labelled,
  but App. J.2 reports 40. App. J.1 says the 35 three-way disagreements
  went to a human, but the labels listed sum to 23. App. K.3 points to
  prompt "P18", but the prompts are P16/P17. The Sonnet 4.6 citation is
  garbled. Bootstrap intervals are promised in I.2, but no table shows one,
  so differences such as Pi + RP 6/67 vs Pi 10/67 cases have no stated
  uncertainty.
- **Fit with the graph.** The paper extends
  [[literature/papers/chen2026governance]] (which split violation by a
  binary survival label and found survived → 0%) by adding a "partially
  retained" state, an agent's-own baseline, and a complete harness. The
  retained-but-violated finding is the in-harness, post-compaction version
  of the "present but did not bind" failure recorded in
  [[literature/papers/mcmillan2026instruction]] and
  [[literature/papers/chandran2026autoresearch]]. It is consistent with
  [[literature/papers/laddha2026position]]'s finding that a later line
  out-ranks an earlier policy: P7 states that ranking explicitly. The
  selective-compactor result is the compliance-side counterpart of
  [[literature/papers/lencevicius2026when]]'s observed-fact loss under
  drop-oldest windows.

## Trust signals

- **Credibility:** 3.
  - For: VUSec at VU Amsterdam, an established systems-security group
    (Bos). Code, cases, prompts and study scripts are released with a
    Dockerfile and a pinned pi revision, and the authors say the tables
    regenerate from the released rows. The paper runs the harness's own
    compaction code unmodified. Compliance is graded deterministically
    from tool calls, against matched full-history baselines. The judge
    panel spans three model families, with blind human checks (κ 0.92 on
    40 summaries, 0.87 on 24 behaviour labels) and a fifth-judge κ of
    0.741 on 4,870. The limitations of the adversarial study are stated
    plainly.
  - Against: not peer reviewed, two weeks old, no citations. Only four
    flash-tier agent models and no Claude agent, 17 short cases, one
    forced compaction per episode, canned tools. The adversarial study
    uses one model. No intervals are shown, there are several internal
    count inconsistencies, and the headline is generalized from the main
    strategy (Hermes) without isolating its prefix.
  - Not 4 because the headline does not survive a change of compressor and
    the models are far from this box's. Not 2 because the artifact is
    released, the grading is deterministic, and the strategy table that
    undercuts the headline is the paper's own.

## Follow-up

- **Relevance:** 5. It is the first behavioural test, in an unmodified
  harness, of a category-specific "preserve the operating rules" compactor
  instruction. That is the form of the box's accepted PreCompact change
  (`docs/system-proposals/2026-11-01-precompact-chat-only-constraints.md`),
  whose own Risk 1 says "the fix form is untested". It also separates
  retention from compliance with a matched baseline. Together these make it
  load-bearing evidence for [[concepts/constraint-pinning]], with a
  strategy and budget table for [[concepts/context-eviction-policy]].
- **What it implies for the PreCompact change (our reading).**
  - *The form gets its first support.* Pi + RP is the closest tested
    analog. On these models it gave 99% retention and the only below-baseline
    violation rate, and under compactor-targeted injection it never lost
    the rule. The box's bullet asks for more (verbatim quotes) and
    reaches the summarizer by a different route (hook text, not the
    compaction prompt).
  - *Judge the change by behaviour, not restatement.* The proposal's local
    check counts restatement per compaction. It should also grade an
    action after compaction against a full-context arm, split agent's-own
    vs only-after, and use a probe whose task passes through the
    forbidden action without the user naming it, so that a revocation
    does not score as a violation.
  - *Framing.* Claude Code's continuation wrapper, checked in this box's
    transcripts, says the summary "covers the earlier portion of the
    conversation" and has no reference-only or latest-wins clause. So the
    Hermes-sized excess probably does not transfer. Two framing points
    remain, both untested:
    - The wrapper ends "Continue … without asking the user any further
      questions", which works against ask-first rules (the
      pdraft/papprove type).
    - Quoted constraints should read as still in force, not as narrated
      history.
- **Candidates** (cited, not yet in the graph):
  - Wang, Zhang, Lee & Yang 2026, "Lost in compaction: evaluating
    side-constraint loss under context compaction" (COMPINT, arXiv
    2608.11242). Already listed as a candidate in salle2026benchmarks and
    busch2026ctxagent.
  - Min et al. 2026, "Toward reliable context compression for long-horizon
    agents: an empirical study of execution instability" (TRACE, arXiv
    2608.06503).
  - Kang et al. 2026, "ACON: optimizing context compression for
    long-horizon LLM agents" (arXiv 2510.00615).
  - Mu et al. 2023, "Can LLMs follow simple rules?" (arXiv 2311.04235).
  - pi agent harness (github.com/earendil-works/pi), as a repo note: its
    compaction prompt has a "Constraints & Preferences" slot.
