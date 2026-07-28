---
kind: paper
title: "Autoformalization of Agent Instructions into Policy-as-Code"
authors: ["Adam Mondl", "Matthew Maisel", "John H. Brock"]
institutions: ["Sondera"]
year: 2026
venue: "arXiv preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2606.26649"
code_url: "https://github.com/sondera-ai/sondera-h"
citations: null
source: "raw/papers/mondl2026autoformalization.pdf"
added: "2026-07-28"
relevance: 4
credibility: 3
status: read
related_concepts:
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/scripted-tool-pipelines]]"
related_experiments: []
tags: [policy, autoformalization, cedar, policy-as-code, neurosymbolic, generator-critic]
---

# Autoformalization of Agent Instructions into Policy-as-Code

## TL;DR

The translation step the typed-enforcement thread was missing: an LLM
generator–critic pipeline that compiles agent prompts, MCP tool schemas,
and natural-language policy documents into **Cedar** authorization
policies, enforced at runtime by a deterministic external engine. The
argument is a scaling one — hand-coded symbolic enforcement is sound but
does not cover the breadth of a real policy specification, and
autoformalization does.

## Claims

- The field has two inadequate options: probabilistic guardrails
  (fine-tuned classifiers, prompt steering) with no formal guarantee, and
  hand-coded symbolic enforcement that does not scale to the breadth of
  real policy specifications. Autoformalization is proposed as the third.
- **Coverage, not soundness, is the binding constraint in practice.** Prior
  work hand-implemented 23 of an 88-rule policy; the other 65 rules were
  unenforced not because anyone disagreed with them but because writing
  guardrails by hand is expensive.
- Verification should be layered rather than monolithic — a
  deterministically checkable critic and a semantic critic catch disjoint
  classes of error.

## Methods

**The "Verification Sandwich"** — three layers:

1. **Grounding layer** (bottom) — extracts entities, identifies tool
   schemas (e.g. OpenAI/MCP JSON schemas), and defines the
   principal–resource–action ontology, producing the Cedar schema. This
   constrains candidate generation to real entities and valid identifiers.
2. **Model layer** (middle) — the policy generator (Gemini 3 Pro),
   prompted with the agent's system instruction, tool definitions, and the
   auto-generated Cedar schema.
3. **Safety layer** (top) — a **hard critic** (the Rust `cedar-policy` CLI:
   syntax, schema mismatch, plus static analysis flagging *vacuous
   policies* and *conflicting rules*) and a **soft critic** (two-stage
   Judge → Verifier, both Gemini 2.5 Flash at temp 0.3 / 0.1, checking
   semantic alignment against a rubric). Validation errors feed back to the
   generator, retrying up to three times.

Other details worth noting:

- **Cedar** (AWS's open-source authorization language) as the target, not a
  bespoke DSL — so the enforcement engine is off-the-shelf and
  deterministic.
- Rules carry **source annotations marking their lineage** (`policy.md` vs
  MCP server), letting the authors separate rules derivable from the policy
  text from FHIR constraints transcribed from the reference MCP server.
- Evaluation replays MedAgentBench trajectories through the Cedar engine
  via an ETL pipeline. The policy under test is Hong et al.'s synthetic
  88-rule natural-language policy (drafted by GPT-5.2, expanded via STPA
  hazard analysis).

## Results

On MedAgentBench (Cedar Block Rate, trajectories blocked / total):

| Dataset | Condition | Hong et al. Unsafe ↓ | Cedar Block Rate |
|---|---|---|---|
| Original | Raw | 39.0% | 30.3% (91/300) |
| Original | Baseline | 23.0% | 57.7% (173/300) |
| Original | Guardrail | 0.0% | 55.7% (167/300) |
| Adversarial | Raw | 78.0% | 72.0% (36/50) |
| Adversarial | Baseline | 62.0% | 82.0% (41/50) |
| Adversarial | Guardrail | 0.0% | 85.7% (42/49) |

The authors are explicit that these columns are **not comparable as
quality scores**: Hong et al.'s Unsafe rate counts violations of their 23
implemented rules (0% by construction in the Guardrail condition), while
the Cedar Block Rate counts violations of the broader autoformalized rule
set — i.e. the 55.7% is largely *the rest of the policy that was never
enforced before*. The claim is coverage, not superiority.

Also reported: trajectory-level rates understate the effect because many
MedAgentBench tasks are retrieval-only and never issue a write, and Cedar
can deny tool calls but not dialog-only behavior — such trajectories are
unblockable by construction.

## Critique / open questions

- **No false-positive measurement.** The paper states it evaluates
  "enforcement coverage rather than agent utility," which leaves the most
  important question open: of the 167/300 benign-split trajectories Cedar
  blocked, how many were genuine violations and how many were the policy
  over-firing? A pipeline that generates 88 rules' worth of enforcement is
  exactly the pipeline most likely to over-block, and this is the axis on
  which autoformalization would fail if it fails.
- The soft critic is an LLM-as-judge validating LLM-generated policy — the
  generator and its semantic verifier share a failure mode. The hard critic
  is the only genuinely independent check, and it can only catch syntactic,
  schema, vacuity, and conflict errors, not *wrong* policy.
- Short paper; single benchmark; one synthetic policy that was itself
  LLM-drafted. Whether the pipeline handles a real, human-authored,
  internally inconsistent policy document is untested — and that is the
  realistic case.
- The FHIR constraints "not derivable from the policy text alone" had to be
  transcribed separately by Claude Opus 4.7. That is an honest disclosure
  and also an admission of the method's boundary: some enforceable
  constraints live in the implementation, not the prose, and no amount of
  reading the policy recovers them.

## Trust signals

- **Credibility:** 3 — small startup team, no peer review, single-benchmark
  evaluation with no utility/false-positive measurement. Raised above 2 by
  a released open-source implementation
  (`sondera-ai/sondera-h`, with Python Cedar bindings), an off-the-shelf
  standard target language rather than a bespoke DSL, and unusually candid
  reporting of why its headline numbers are not head-to-head comparable.

## Follow-up

- **Relevance:** 4 — with [[literature/papers/palumbo2026formal]] this is
  the pair the 07-20 NOTES predicted would tip the **typed-enforcement
  thread** to concept ripeness. Palumbo supplies the enforcement semantics;
  this supplies the *compiler* from prose to policy. Neither alone answers
  how an organization's actual written rules become machine-checkable
  gates; together they do.
- The **hard/soft critic split** is the transferable idea and it
  generalizes past policy: pair a deterministic checker that can only find
  mechanical faults (syntax, schema, contradiction, vacuity) with a
  semantic judge, and route the deterministic errors back as retry
  feedback. Same structure as
  [[concepts/programmable-evaluator-oracle]]'s crisp-oracle-vs-judge
  tension, resolved by using both at different layers rather than choosing.
- **Rule lineage annotation** connects to [[concepts/citation-anchoring]]
  and [[concepts/typed-claim-partition]]: each generated rule records which
  source it came from, which is what makes the policy auditable and lets
  the authors separate policy-derived from implementation-derived rules.
  Any generated artifact in this graph should carry the same annotation.
- Closest thing to a directly applicable idea for this box: the static
  checks the hard critic runs — **vacuous policy** and **conflicting rule**
  detection. This repo's `CLAUDE.md` + `.claude/rules/` are exactly a
  natural-language policy set that has never been checked for either.
  `/lint` currently checks graph hygiene, not rule consistency.
  See the same note under [[literature/papers/palumbo2026formal]].
