---
kind: moc
name: "evaluation-integrity"
status: active
added: "2026-06-23"
concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/pass-at-k]]"
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/compression-as-generalization-test]]"
  - "[[concepts/information-firewall]]"
tags: [moc, evaluation, evaluation-integrity, overfitting, grounding, architecture]
---

# Evaluation Integrity for Autonomous Research Agents

How do you keep the evaluation signal of an autonomous research agent
*honest* — so a loop that runs for hundreds of cycles measures real
capability rather than learning to game its own measuring stick? This is
the failure mode that distinguishes a long-horizon agent from a
short-horizon one: over enough iterations, any signal the search loop can
read, it will eventually optimize *against* rather than *toward*. The seven
concepts here are the structural defenses, each guarding a different point
where the signal can rot. Where the sibling MoC
[[mocs/knowledge-organization-for-research-agents]] is about the knowledge
the agent works *with*, this one is about whether the numbers and claims
that come *out* can be trusted to mean what they say.

Two of these concepts (`citation-anchoring`, `typed-claim-partition`) also
appear in the knowledge-organization MoC — they are the seam where "how
claims are organized" meets "how claims are scored," and they earn a place
in both views. AIRA_2 ([[literature/papers/hambardzumyan2026aira]]) names
validation-selection overfitting as one of three dominant bottlenecks in
research agents; every concept below is a response to some facet of it.

## Keeping the search loop from gaming itself

The first defense is structural: the loop must not be *able* to read the
signal that would let it cheat.

- [[concepts/hce-evaluation]] — Hidden Consistent Evaluation. A hard split
  between a validation signal the loop reads every cycle (`metrics.json`)
  and a held-out test split revealed only by a final-scoring pass
  (`final_metrics.json`). AIRA_2's ablations show that without it, longer
  runs trade real capability for validation gaming and the val/test gap
  diverges. This is the load-bearing rule (`~/.claude/rules/evaluation.md`)
  that the other five operate under.
- [[concepts/pass-at-k]] — a reported result is a *distribution over seeds*,
  not a single number. MLE-bench ([[literature/papers/chan2024mle]]) and
  AIRA_2 both find headline performance is seed-sensitive at a magnitude
  that rivals architectural change: a 2-point gap can vanish at k=5, and a
  real improvement can be invisible at k=1. Honest selection requires k≥3
  and a stated distribution, so the loop can't promote noise.
- [[concepts/information-firewall]] — the same boundary-drawing applied to
  the *task construction* rather than the split: withhold the source
  method (identity, code, outputs) so the agent must discover a solution
  rather than reproduce the published one. NatureBench
  ([[literature/papers/wang2026naturebench]]) operationalizes it with
  file-level keep/exclude rules plus eval-time web-search disable. Where
  HCE hides the *answers* from the loop, the firewall hides the *method*
  — one protects the estimate, the other the construct being measured.
  No longer single-source: search-time contamination
  ([[literature/papers/wang2026search]]) attests the same boundary from
  the measurement side and shows the two firewalls share **one bypass** —
  a search-enabled agent routes around the method firewall and the answer
  holdout with the same tool call, recovering benchmark artifacts and gold
  labels from question banks and forums. Two lessons transfer to every
  concept in this MoC: audit for *answer-level* leakage rather than corpus
  overlap (metadata retrieval carries hazard ratios below 1 for correct
  prediction, so URL-matching audits over-report), and treat a restricted
  retrieval corpus as relocating the boundary rather than drawing it
  (0% leakage on one benchmark, 78% on another built from the same
  sources).
- [[concepts/compression-as-generalization-test]] — the audit that says
  whether a validation-selected gain was *real*. Split hygiene prevents the
  loop from reading the truth but can't certify what it selected;
  [[literature/papers/bertran2026fits]] shows honest strategies compress
  into ~32-token prompts a fresh reproducer (training data only) can
  replay, while validation-exploiting gains fail to reproduce (100%
  sensitivity / 91% specificity) — and that a one-bit "improved?" feedback
  channel matches scalar feedback while yielding confidence intervals that
  certify progress. HCE's certify-after-search pass, made operational per
  checkpoint.

## Grounding the claims the agent emits

Even with an honest scalar, the *prose* an agent writes drifts. These two
keep every claim traceable — and turn that provenance into a control signal.

- [[concepts/citation-anchoring]] — every concrete claim traces to a code
  reference (`metrics.json:<field>`, `train.py:42-58`) or a `literature/`
  wikilink; unanchored assertions are defects, not stylistic slips. Kosmos
  ([[literature/papers/mitchener2025kosmos]]) measured ~20% unsupported
  statements over 12-hour runs — the dominant defect type — and the fix was
  structural, not better prompting.
- [[concepts/typed-claim-partition]] — the operationalization that turns
  anchoring from a flag into a control signal. GSAR
  ([[literature/papers/kamelhar2026gsar]]) partitions claims four ways
  (grounded / ungrounded / contradicted / complementary) with evidence-type
  weights, then drives a *tiered recovery* (proceed / cheap-regenerate /
  replan). Crucially it makes silently-dropped contradictions detectable as
  score inflation — closing a hole a single scalar leaves open.

## The oracle that defines what's discoverable

The deepest layer: in search-driven systems the evaluator is not a
component of the agent — it is the *environment* the agent optimizes
against, and it implicitly bounds what can be found at all.

- [[concepts/programmable-evaluator-oracle]] — a user-supplied function that
  verifies, runs, and scores every candidate and serves as the loop's only
  ground truth. FunSearch ([[literature/papers/romeraparedes2024funsearch]])
  uses it to neutralize hallucination (invalid programs never enter the
  population); AlphaEvolve ([[literature/papers/novikov2025alphaevolve]])
  makes it the *unit of domain transfer* (same core, swap the evaluator). The
  consequence: agent competence collapses into evaluator quality, and a
  subtly-buggy oracle steers the whole search wrong without tripping its own
  success signal. [[concepts/hce-evaluation]] is the named structural defense
  — the evaluator may be gamed; the final scorer must not be.

## Open thread

The seven form a containment hierarchy: the information firewall keeps the
task from leaking its own solution, HCE keeps the loop from reading the
truth, pass@k keeps it from promoting noise, compression-testing certifies
what survived selection, anchoring + typed-partition
keep the prose honest, and the programmable oracle is the one thing the loop
*is* allowed to optimize — which is exactly why it must be kept off the
held-out truth. The working hypothesis is that these compose multiplicatively,
not additively: an agent with a pristine HCE split but an un-anchored report
still ships ~20% fabrication, and one with perfect grounding but a gameable
oracle optimizes a corrupt target with full provenance. The most directly
testable interaction for this project is the citation-anchoring × pass-at-k
pairing — whether requiring anchored, distribution-reported claims actually
closes Kosmos's accuracy gap, or whether the gap is in the underlying
reasoning and anchoring only makes the defect grepable (the ablation flagged
in `citation-anchoring`'s open questions).
