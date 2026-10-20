---
kind: paper
title: "Search Discipline for Long-Horizon Research Agents"
authors: ["Adithya Srinivasan", "Devesh Paragiri"]
institutions: ["North Carolina State University", "University of Maryland"]
year: 2026
venue: "arXiv 2606.11522 (cs.AI), v1 2026-06-09, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2606.11522"
code_url: null
citations: null
source: "raw/papers/srinivasan2026search.pdf"
added: "2026-10-20"
relevance: 3
credibility: 2
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/shared-substrate-contagion]]"
tags: ["aggregate-inversion", "hidden-stratification", "disaggregated-evaluation", "candidate-selection", "external-monitor", "llm-judge", "premature-stopping", "reopen", "stall", "noise", "prompt-vs-controller", "same-model-judge", "case-study", "scientific-modeling", "ecosystem-demography", "hermes-agent", "gpt-5.5"]
---

# Search Discipline for Long-Horizon Research Agents

## TL;DR

A short (7-page) position-plus-case paper. It names **aggregate-verifier
inversion**: when a candidate's validity lives across regions or slices but
the verifier reports one pooled score, the top scorer can be the candidate
that breaks a part of the space that matters. The authors propose a
"candidate-effect audit" (helped, harmed and unchanged parts, a legal
separator, five candidate roles) and enforce it from an **external monitor
loop** that owns the agent's terminal, judges at decision boundaries with a
separate LLM call, and can demote a candidate or reopen a run the agent has
stopped.

The evidence is one cleanly certified run plus qualitative observations
from an unstated number of other runs, on one task: improving the fire
module of the Ecosystem Demography (ED) land model. The agent is Hermes
(Nous Research) on GPT-5.5 at high effort; the judge is the same model. In
the headline run, two candidates differ by 0.0007 on the global score
(0.6774 vs 0.6767), but the higher one drops boreal-region fidelity by
0.086 and the lower by 0.004. The loop demoted the higher one. Two further
logged instances show the loop pushing past an early stop. No counts, no
arms compared on outcome, no intervals, no code.

## Claims

- **Inversion.** "When scientific validity lives in that disaggregated
  structure, the aggregate can rank the wrong candidate first." It is
  likely in autoresearch because (1) the space is heterogeneous (hidden
  stratification, citing Oakden-Rayner et al. 2019) and (2) near a strong
  baseline, candidate-to-candidate aggregate differences fall within
  evaluation noise.
- **The decision must be external.** "A prompt is consumed at the start of
  the run; it has no way to wake when an evaluation finishes and no
  remaining turn once the agent has stopped." The producing agent's
  self-assessment "shares the bias that created the problem". These are
  argued, not tested.
- **Two moves only an external loop can make:** override an accept the
  agent made, and reopen a run it stopped.
- **Prompt reframing did not move the frontier.** One added paragraph
  asking the agent to borrow mechanisms from other domains "moved the agent
  earlier toward composing several mechanisms", but it "still stopped after
  one shallow pass and the final frontier was unchanged".
- **The loop forces search without manufacturing results.** In one run the
  forced extra pass still certified the baseline.
- **Scope, stated by the authors:** "The clearest evidence is a single
  certified run … We present the inversion as a demonstrated phenomenon …
  and leave to future work how often the loop changes an outcome across
  many runs."

## Methods

- **Task.** Improve the ED fire component, which predicts burned area,
  under a fixed input contract: model-derived inputs only (dryness,
  rainfall, temperature, vegetation productivity). No region-named rules,
  no latitude/longitude, no outside data, no per-location lookup tables, no
  fitting the target directly. Serious candidates are scored globally and
  per region. The region groupings are analysis-only; the agent never sees
  them.
- **The built-in trap.** The baseline over-predicts burning in several
  low-fire regions, so a suppressor raises the global score while it can
  damage regions that really burn (boreal forest).
- **Three conditions.** (1) A strong base prompt requiring baseline
  reproduction, global and regional checks, one legal formula, evaluation
  and logs. (2) Base plus one structural-reframing paragraph. (3) Base under
  the control loop. Runs per condition: not reported.
- **Agent and judge.** Hermes driven by GPT-5.5, reasoning effort high. The
  monitor's judge "uses the same model". The judge is described as
  "advisory": it returns a structured decision (contract satisfied?, which
  candidate role, intervene?), and the monitor renders and delivers any
  message.
- **Loop mechanics (§3.4).** The monitor launches the agent and owns its
  terminal. It judges only at completed evidence (process exit plus result
  files present and stable). Long instructions go via a file. A run is
  finished only when the loop records a terminal status, not when the agent
  writes its report. Runs where a reopen collides with agent shutdown are
  kept as "complete yet uncertified", separate from certified runs.
- **Candidate roles.** Score winner, defended candidate, tradeoff,
  informative failure, rejected shortcut.
- **Metric.** The "global score" is not defined beyond "a mean pooled over
  every region". The per-region "fidelity" measure is not defined either.
  The "independent public comparison" cross-check cites ref. [12], which is
  the second author's own blog.

## Results

- **Table 1, the one certified inversion.**

  | Candidate | Global score | Boreal change | Loop decision |
  |---|---|---|---|
  | Aggressive | 0.6774 | −0.086 | demote |
  | Gentler | 0.6767 (−0.0007) | −0.004 | accept |

  Both sit within 0.006 of baseline. The text gives the aggressive
  candidate's two boreal drops as "about 0.10 and 0.07", and the gentler
  one's as "about 0.006 and 0.003". The paper calls the 0.0007 global gap
  "within noise" and the 0.08 regional drop "well above the region's
  noise", but reports no seed variance or interval for either.
- **Premature stopping (§4.3), two logged instances.** In one run the loop
  rejected a finished-looking state, forced another pass through untried
  combinations, then certified a conservative outcome that kept the
  baseline. In another it blocked a stop at a candidate whose score sat
  within noise of baseline and required more work before closing. In both,
  the push came after the agent had returned to idle.
- **Across runs (§4.4).** "For almost every serious candidate" the global
  score stays within noise of baseline, so the audit, not the score,
  separates outcomes. The base prompt without the loop "also found a real
  mechanism on its own, while still damaging the boreal regions". One of the
  strongest candidates came from a run with no loop (§5).
- **Boundary (§4.5).** No clean candidate exists under the contract.
  Omitted causes (ignition, land use, lightning, wind) may explain why. The
  audit records which separations failed.

## Critique / open questions

- **Against the digest framing:** AT. The digest entry ("an external loop
  kept agents searching past self-chosen stops … blocked a stop at a
  candidate within noise of baseline. Case observations, not an ablation")
  matches §4.3 and already carries the right hedge. "Kept agents
  searching" is two logged instances, and "within noise" is never
  quantified, but the entry claims nothing the paper lacks. If anything it
  under-sells the paper's main point, the aggregate inversion, which is the
  part with a number attached. This is one of the few digest entries that
  does not read high.
- **n = 1, by the authors' own account.** One certified run, and an
  unreported number of other runs read through "candidate-effect traces".
  No condition-level outcome comparison exists. The loop's effect on what
  gets shipped cannot be estimated from this paper.
- **"Within noise" is asserted, not measured.** No seeds, no bootstrap over
  regions, no rerun. The claim that a 0.0007 global gap is noise while a
  0.08 regional drop is real is plausible from the scales, but per-region
  scores over few grid cells can be noisy too. Compare
  [[literature/papers/liu2026what]]'s rule: report the rerun flip rate next
  to any gap.
- **The judge is the same model as the agent.** GPT-5.5 judges GPT-5.5's
  candidates. The paper's case for externality is *position* (separate
  process, no stake, a turn after the stop). It never separates position
  from substrate. A same-model judge in a fresh context may share the
  agent's priors about what a good fire formula looks like. This is the
  [[concepts/shared-substrate-contagion]] question, and here it is not even
  raised. Judge accuracy is not checked against a human or a held-out
  verdict.
- **The judge saw the disaggregated scores; the base prompt also asked for
  them.** The base prompt already required "global and regional checks".
  The no-loop agent was apparently able to see regional damage and shipped
  anyway. That supports the "self-judgment is weak" reading (as in
  [[literature/papers/sun2026scipivot]] and
  [[literature/papers/woo2026youra]]), but the paper does not show the
  logs, so it is unclear whether the agent saw and ignored the boreal drop
  or never computed it.
- **Bundle, not a placement contrast.** The loop condition adds a second
  LLM call, a structured role rubric, extra turns after stop, and a
  terminal-status rule, all at once. There is no arm with the same rubric
  run by the agent itself at the same decision points. So "the decision
  must be external" is an argument, as in
  [[literature/papers/rodionov2026improving]], not a measured placement
  effect like [[literature/papers/zhang2026judged]]'s prompt-vs-mask arms.
- **What it does mean for the live fronts.**
  - *Redirect-before-halt proposal
    (docs/system-proposals/2026-10-18-iterate-redirect-before-halt.md).*
    Weak, consistent support only. The failure here is the opposite end of
    a chain: stopping after one shallow pass, not refining until the stall
    halt. The useful datum is that a forced extra pass once ended by
    certifying the baseline. Forced search did not invent a gain, which is
    the outcome the proposal's "judge the pivot against the chain's best"
    clause relies on. n = 1, and still no redirect-vs-halt arm. The
    reframing paragraph is one more prompt-form nudge that changed where the
    search started but not where it ended, after cai2026forkandflush's
    ignored "pick a DIFFERENT approach".
  - *Accepted no-improvement noise band.* The band is computed on a scalar
    metric. This paper's case shows two candidates the band would call tied
    while one breaks a slice. The band correctly refuses to reset the
    counter on such a "new best", but a tie inside the band is exactly where
    the champion choice should go to a per-slice check, not to the raw
    higher number. That is our inference; the paper has no band.
  - *HCE.* Inversion is a validation-split problem, not a test-leak problem.
    A hidden test split scored with the same pooled metric would hide it
    too.
- **Open.** Does a per-slice tiebreak inside the noise band change which
  candidate `/iterate` keeps on a real downstream task? Is the boreal effect
  stable over seeds? Would a different-family judge demote the same
  candidate?

## Trust signals

- **Credibility:** 2.
  - For: the setup is concrete and checkable in principle (a named public
    model, a stated input contract, a real scientific-modeling task where
    regional validity matters). The authors state the n = 1 scope plainly
    and do not claim an averaged effect. The loop-reliability details
    (judge on completed evidence, terminal status owned by the loop,
    certified vs uncertified runs kept apart) read like real engineering.
  - Against: two authors, apparently students, at NCSU and UMD department
    level, no lab or senior author. Not peer reviewed. No code, prompts,
    logs or run counts. Metrics are undefined. The "independent" cross-check
    is the second author's blog. The judge is the agent's own model, with no
    accuracy check. Every number in the paper comes from one run.
  - Not 3 because the only quantitative result is one run with no noise
    estimate and nothing is released. Not 1 because the authorship is
    traceable, the task is real, and the claims are scoped honestly.

## Follow-up

- **Relevance:** 3. It is on two active fronts at once: agents stopping
  early, with an external loop that reopens, and selection on a pooled
  metric. It gives [[concepts/hce-evaluation]] a clean, concrete instance of
  aggregate inversion inside a research-agent loop, which the graph lacked,
  and adds one more controller-not-prompt observation to
  [[concepts/budget-as-ceiling]]. It does not shift either concept: it has
  no arms, no counts and one run. Not 2 because the inversion case is
  directly about how `/iterate` picks a champion.
- **For `/iterate` (not proposed).** If the noise band ever gains a
  tiebreak, "inside the band, prefer the candidate that does no per-slice
  damage beyond the band" is the cheapest form. It needs only the per-slice
  metrics `/implement` would have to emit. Hold until a second, measured
  source exists.
- **Candidates** (cited, not yet in the graph):
  - Oakden-Rayner et al. 2019, "Hidden stratification causes clinically
    meaningful failures in machine learning for medical imaging" (arXiv
    1909.12475). The canonical source for the subgroup point.
  - Nguyen 2026, "Physics is all you need? A case study in
    physicist-supervised AI development of scientific software" (arXiv
    2605.30353).
  - Paragiri 2026, "Extending the autoresearch loop" (blog,
    paragiri.com/blog/2026/autoresearch-paradigm-fire/). Likely holds the
    run details the paper omits.
