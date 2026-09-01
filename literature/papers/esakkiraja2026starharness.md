---
kind: paper
title: "StarHarness: Evolving Harnesses with Stratified Search for Enterprise Environments"
authors: ["Esakkivel Esakkiraja", "Denis Akhiyarov", "Vikas Yadav", "Sai Rajeswar", "Patrice Bechard", "Sridhar Nemala", "Sagar Davasam"]
institutions: ["ServiceNow", "Mila", "Université de Montréal"]
year: 2026
venue: "arXiv (cs.AI)"
peer_reviewed: false
url: "https://arxiv.org/abs/2608.24804"
code_url: "https://github.com/ServiceNow/StarHarness"
citations: null
source: "raw/papers/esakkiraja2026starharness.pdf"
added: "2026-09-01"
relevance: 5
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/enforcement-boundary-placement]]"
tags: ["agent-architecture", "evolutionary-search", "harness", "skills", "evaluation"]
---

# StarHarness: Evolving Harnesses with Stratified Search for Enterprise Environments

## TL;DR

Evolve the **harness** — prompts, tool interfaces, skills, MCP providers,
subagent structure, agent-loop config — while **model weights stay frozen**.
20–35 percentage points on three stateful enterprise benchmarks after only
**4–12 accepted changes**, and the evolved harness transfers frozen to other
models and model families without re-evolution.

## Claims

- Persistent **model–environment mismatch**, not model capability, is the
  binding constraint in tool-rich stateful tasks. The harness is where that
  mismatch lives and where it can be repaired.
- Harness evolution is **cheap in edits**: 4–12 accepted changes per
  environment produce the full gain. This is an outer loop with a very
  small accepted-change budget, not a long search.
- Gains are **generalization, not memorization** — they persist on tasks
  excluded from evolution and transfer across model families.
- What is learned is *reusable environment behavior*: interface repairs,
  environment conventions, operational knowledge that compresses search.

## Methods

- **Harness evolution as outer-loop optimization** of the executable
  scaffold around a fixed LLM. Editable surface: prompt and task framing,
  tool interfaces, skills, MCP-backed providers, subagent structure,
  agent-loop configuration.
- **Three-way task partition** — proposer-visible *search* tasks,
  proposer-hidden *selection* tasks, and reserved *held-out* evaluation
  tasks. This is the paper's methodological contribution: it separates
  search performance from generalization, which prior harness-evolution
  work conflates.
- **Stratified sampling**: the evolution pool is built by stratifying tasks
  by *baseline failure behavior*, keeping the pool compact.
- Each candidate is a **git diff** against the current frontier, scoped to
  the benchmark's editable directories. A **validator** checks scope,
  imports, and constraints before evaluation.
- **Explicitly forbidden**: ground-truth tables and hidden-state access —
  i.e. the search is barred from reaching the oracle.
- Benchmarks: ITBench SRE, EnterpriseOps-Gym ITSM, AutomationBench Finance.

## Results

- **+20–35 percentage points** full-benchmark over the default harness,
  after 4–12 accepted changes per environment.
- On held-out tasks excluded from evolution: **+13.8 / +22.3 / +17.6**
  points on ITBench, EnterpriseOps-Gym, and AutomationBench respectively —
  most of the gain survives the generalization test.
- **Frozen cross-model transfer**: the evolved harness improves *every*
  transferred model tested, including models from different families (GPT
  and Qwen). No re-evolution required.
- Trace analysis attributes gains to interface repairs, environment
  conventions, and operational knowledge that compresses search — with
  fewer false-positive diagnoses and shorter trajectories.

## Critique / open questions

- Enterprise ITSM/SRE/finance workflows are not research workflows. The
  transferable claim is about *harness evolution as a method*; the specific
  repairs are environment-bound by construction.
- 4–12 accepted changes is a striking number, but "accepted" hides the
  proposal count — the search cost to find those 4–12 is not the headline.
- A large fraction of the gain plausibly comes from repairing genuinely
  broken tool interfaces, which is a one-time debt payment rather than
  open-ended improvement. The paper's own attribution ("interface repairs")
  partly concedes this: the ceiling may be "as good as a well-built
  harness" rather than "better indefinitely".
- ServiceNow evaluating on ServiceNow-adjacent enterprise benchmarks is a
  home-field advantage worth noting, though ITBench is external.

## Trust signals

- **Credibility:** 4 — ServiceNow Research with Mila / Université de
  Montréal co-authors; **code released** at `ServiceNow/StarHarness`; three
  independent benchmarks; and an unusually disciplined evaluation protocol
  (proposer-visible / proposer-hidden / held-out) that the paper correctly
  identifies as absent from comparable work. Not peer reviewed.

## Follow-up

- **Relevance:** 5 — [[concepts/evolutionary-search-grain]] asks what unit
  evolution should operate on, and the ingested literature has answered at
  function-grain (FunSearch) and whole-file-grain (AlphaEvolve). **"The
  harness, weights frozen" is a third grain**, and it is the only one of
  the three this project can actually operate at: it has no training
  capability, but it does have a harness — skills, rules, hooks,
  `budget.yaml` — that is exactly the described editable surface.
- The **three-way partition is the directly importable discipline**.
  Proposer-visible search tasks / proposer-hidden selection tasks /
  held-out evaluation is a clean statement of what
  [[concepts/hce-evaluation]] demands, arrived at independently for a
  performance reason (measuring generalization) rather than an integrity
  reason. The validator's explicit prohibition on ground-truth-table and
  hidden-state access is the same firewall
  [[concepts/information-firewall]] describes, implemented as a scope check
  on a git diff.
- "4–12 accepted changes" is a useful prior for `/elevate`, which has
  produced zero proposals for several consecutive cycles against a stable
  system. This paper suggests the correct number of harness edits is small
  but non-zero, and that the value is concentrated in interface repairs —
  which argues for looking at where this project's skills *mis-fit their
  environment* rather than for new capability.
- Frozen cross-model transfer supports [[concepts/hybrid-model-backends]]:
  if a harness evolved on one model lifts every other model tested, the
  harness is a durable asset independent of the backend, and `budget.yaml`'s
  `models:` roles can change without invalidating it. Pairs with
  [[literature/papers/tang2026wikiskill]], which finds the same
  transferability for skills.
