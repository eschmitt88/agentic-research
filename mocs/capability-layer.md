---
kind: moc
name: "capability-layer"
status: active
added: "2026-07-18"
concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/shared-skill-namespace]]"
  - "[[concepts/scripted-tool-pipelines]]"
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/typed-enforcement]]"
tags: [moc, capability-layer, skills, tool-use, harness-ecosystem, governance]
---

# The Capability Layer: Skills, Tools, and the Harness

What can the agent actually *do*, and who decides? The sibling MoCs map
the knowledge the agent works with
([[mocs/knowledge-organization-for-research-agents]]), the machine that
runs it ([[mocs/agent-architecture]]), how it searches
([[mocs/autonomous-search-loop]]), and how it stays honest
([[mocs/evaluation-integrity]]). This one maps the agent's **action
surface**: procedural capability packaged as skills and tool access, plus
the harness machinery that curates it, ports it across executors,
executes it cheaply, and gates it safely. The six concepts belong
together because a capability is only real when all four clauses hold —
well-curated, loadable where the agent runs, affordable in context, and
safe to fire. The first three clauses take one concept each; the fourth
takes two, because gating splits into *where the check sits*
(`permission-gate-as-architecture`) and *what the policy is written in*
(`typed-enforcement`). The cluster is anchored
by the 2026 skill-ecosystem wave
([[literature/papers/ouyang2026skillos]],
[[literature/papers/zhang2026skillcomposer]],
[[literature/papers/shen2026dynamic]],
[[literature/papers/huang2026skillwiki]]) and the harness
reverse-engineering line ([[literature/repos/nousresearch-hermes-agent]],
[[literature/repos/hkuds-openharness]],
[[literature/papers/liu2026dive]]), which appear across nearly every
member's sources.

Two members are deliberately shared with [[mocs/agent-architecture]]
(`scripted-tool-pipelines`, `hybrid-model-backends`) — they are the seam
where "how the agent is built" meets "what the agent can do," and they
earn a place in both views.

## Authoring: the write-side lifecycle

A skill library that only grows becomes noise; one that is pruned by age
or frequency destroys value it never measured.

- [[concepts/skill-library-lifecycle]] — skills are curated through
  explicit, governed operations (insert / update / delete, extended by
  merge, split, compress, retire). SkillOS's central result: holding the
  pipeline fixed and changing *only the curation policy* yields the
  largest reported gains in the self-evolving-agents literature. SLIM
  ([[literature/papers/shen2026dynamic]]) adds the retirement warning —
  removal is not internalization, so retire on measured marginal
  contribution, never on age or LRU.

## Portability: the shared namespace

The lifecycle's artifacts outlive any single harness only if they are
portable.

- [[concepts/shared-skill-namespace]] — the `SKILL.md` schema, canonical
  on-disk paths, and the agentskills.io registry make the same skill file
  load under Claude Code, OpenHarness, or Hermes without porting.
  Academic work now builds directly on the shared standard
  ([[literature/papers/zhao2026generative]] predicts over agentskills.io
  objects), and [[literature/papers/huang2026skillwiki]] sketches what a
  governed registry needs. The dark side hands off to the governance
  section: a skill portable across harnesses is an attack portable across
  harnesses ([[literature/papers/ge2026governance]]'s malicious-skill-
  plugin class).

## Execution: dispatch and context economics

Owning a capability is not the same as affording to run it.

- [[concepts/scripted-tool-pipelines]] — chain multi-step tool use as a
  short script so intermediate results stay in local scope; only the
  script and its return value spend context. The difference between a
  curation loop that fits under `max_tokens` and one that doesn't.
- [[concepts/hybrid-model-backends]] — which model tier executes which
  role is a dispatch decision, and the intuitive default ("frontier model
  for the strategic side") is empirically wrong twice: SkillOS's trained
  8B curator beats a frontier model as curator, and SkillComposer's 4B
  composer lifts a 27B executor. The capability layer, not the model
  card, decides who runs what.

## Governance: the gate as architecture

The layer that decides whether a capability fires at all.

- [[concepts/permission-gate-as-architecture]] — the permission gate as a
  first-class regulator over the agent's policy: stateful across turns
  (FinHarness's risk cumulant, [[literature/papers/jia2026finharness]]),
  deterministic at its core
  ([[literature/papers/madatha2026deterministic]]), time-scoped so grants
  expire with their subgoal (PORTICO,
  [[literature/papers/santosgrueiro2026lingering]]), and never relying on
  an LLM judge alone — at 1% attack prevalence the best judge's precision
  collapses to 22.7% ([[literature/papers/ge2026governance]]). Scripted
  pipelines raise the stakes here: one script can fire many tools behind
  a single approval, so the gate and the script surface must be designed
  together.
- [[concepts/typed-enforcement]] — the gate's *policy*, factored out from
  the gate's placement. Constraints written as a machine-checkable
  artifact in a language with decidable static analyses, held outside the
  agent's reasoning: Datalog policies with a reference monitor and a
  correctness theorem ([[literature/papers/palumbo2026formal]]), affine
  budget types that make double-spending a compile error
  ([[literature/papers/khan2026token]]), a conservation law over delegated
  budgets ([[literature/papers/ye2026agent]]), and a compiler from prose
  policy documents into Cedar
  ([[literature/papers/mondl2026autoformalization]]). The pairing with the
  gate concept is the useful frame: one asks *where the check happens*, the
  other *what the check is written in and whether it can be checked before
  it runs*.

  Two results from this concept bear directly on the rest of the layer.
  First, fewer than 1% of 6,145 real agent-config files declare any
  permission boundary at all
  ([[literature/papers/madatha2026deterministic]]) — the governance section
  describes machinery almost nobody deploys. Second, every instance in the
  cluster keeps a **semantic escape hatch** (FORGE's probabilistic
  `llm_check`, the autoformalizer's LLM soft critic, the impossibility of
  true pre-flight token reservation), so "formally enforced" always means
  "modulo the hatch," and a design's quality is how much it pushes into the
  formal skeleton rather than the hatch.

## Open thread

The four sections are converging on a single artifact standard. SLIM
emits `SKILL.md` files from inside an RL loop; Generative Skill
Composition trains composers over agentskills.io objects; SkillWiki
proposes registry lifecycle states; ge2026governance names the registry's
threat model. Working hypothesis: the capability layer is standardizing
the way package ecosystems did (npm, PyPI), and will inherit their
failure modes — supply-chain trust, version conflict, abandoned
artifacts — with the permission gate playing the role package signing and
sandboxing play there. Notably, this project sits *inside* the namespace
it studies: the `/digest`, `/ingest`, `/lint` skills under
`~/.claude/skills/` are themselves artifacts of this layer, so lifecycle
findings (evidence-based retirement, curate-against-the-reader) are
directly testable on our own skill library.
