---
kind: paper
title: "Auto-Policy, not Auto-Skill: Compiled Agent Skills for the Physical World"
authors: ["Zhonghao Zhan", "Krinos Li", "Yefan Zhang", "Hamed Haddadi"]
institutions: ["Imperial College London", "Independent Researcher"]
year: 2026
venue: "arXiv (cs.AI)"
peer_reviewed: false
url: "https://arxiv.org/abs/2608.25091"
code_url: null
citations: null
source: "raw/papers/zhan2026auto.pdf"
added: "2026-08-31"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/shared-skill-namespace]]"
  - "[[concepts/permission-gate-as-architecture]]"
tags: ["agent-security", "skills", "safety", "agent-architecture", "governance"]
---

# Auto-Policy, not Auto-Skill: Compiled Agent Skills for the Physical World

## TL;DR

A Skill describes *how* an agent should behave; a Policy decides *which*
behavior is allowed to become an action. Today's Skill format covers the
first with markdown and scripts and leaves the second to the model — so
auto-generating more Skills scales the gap rather than the safety. The paper
names **Borrowed Authority** (the format gives a receiving agent no typed way
to reject an inter-agent permission claim) and proposes co-packaging a typed
invocation policy inside the Skill artifact itself.

## Claims

- Self-evolving skill harnesses (AutoSkills, Hermes Agent) report gains that
  are **efficiency, not safety**; generating more skills widens the
  skill/policy gap.
- **Borrowed Authority** is a named attack class at the intersection of two
  already-documented ones — malicious skills compromising cloud software, and
  jailbroken LLM-controlled robots causing physical harm. The intersection
  (malicious agent skills causing *physical* harm) follows directly but had
  not been reported.
- The right place for the guard is **inside the Skill artifact**, not between
  tools as workflow engines do — so that a high-risk skill ships its typed
  invocation policy alongside its procedural knowledge.
- Physical actions should depend on **machine-checkable evidence** (world
  state, sensor readings), not on peer-agent permission claims.

## Methods

- Edge Skillguard: a typed authority layer embedded in the Skill artifact,
  with guards expressed over world state and sensor evidence.
- Evaluated on a **live edge control-plane testbed**, five attack variants,
  plus a 5× scale-up and a cross-host run over a Tailscale mesh.

## Results

- Guards reject **60/60 borrowed-authority requests** across five attack
  variants, with no benign requests blocked.
- Result holds at 5× scale and across hosts over the mesh.

## Critique / open questions

- Six pages; the evaluation is small (60 requests, one testbed, five
  hand-built variants) and there is no adaptive adversary. This is a
  position paper with a demonstration, not a robustness study — read the
  60/60 as "the mechanism is well-formed", not "the mechanism is strong".
- No released code, so the typed-policy schema cannot be inspected or reused
  directly.
- The physical-world framing is the paper's novelty hook, but the argument
  (typed policy belongs in the artifact, not the orchestrator) is
  domain-independent and is the part that transfers here.

## Trust signals

- **Credibility:** 3 — Imperial College London (Haddadi is an established
  systems/privacy group), but an unreviewed 6-page preprint with no released
  code and a small single-testbed evaluation. Reputable group carries it to
  3; the thin evaluation keeps it from 4.

## Follow-up

- **Relevance:** 4 — the sharpest available statement of a distinction this
  project's own skill layer does not make. `~/.claude/skills/` is exactly the
  "markdown and scripts, policy left to the model" format the paper
  criticises, and [[concepts/shared-skill-namespace]] describes a namespace
  with no typed rejection mechanism.
- Joins [[concepts/typed-enforcement]] to
  [[concepts/skill-library-lifecycle]]: the lifecycle currently promotes
  skills on usefulness alone, with no compile-to-policy step.
- Read alongside [[literature/papers/wu2026evomal]] (same window) — that
  paper supplies the attack this one's defense is shaped against, on the same
  shared-library substrate.
- Contrast with [[literature/papers/guo2026when]], which places the same
  induction/authorization split *between* tools at runtime rather than inside
  the artifact. The two disagree on placement, not on the split.
