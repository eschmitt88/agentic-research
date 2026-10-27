---
kind: concept
name: "shared-skill-namespace"
status: seedling
added: "2026-05-15"
source_papers: []
sources:
  - "[[literature/papers/banu2026harness]]"
  - "[[literature/papers/yang2026skillopt]]"
  - "[[literature/papers/zhou2026comprehensive]]"
  - "[[literature/papers/belikova2026managing]]"
  - "[[literature/papers/zhao2026generative]]"
  - "[[literature/papers/ge2026governance]]"
  - "[[literature/papers/huang2026skillwiki]]"
  - "[[literature/repos/nousresearch-hermes-agent]]"
  - "[[literature/repos/hkuds-openharness]]"
  - "[[literature/papers/kim2026why]]"
  - "[[literature/papers/zhan2026auto]]"
  - "[[literature/papers/wu2026evomal]]"
  - "[[literature/papers/tang2026wikiskill]]"
  - "[[literature/papers/paglieri2026case]]"
  - "[[literature/papers/kassis2026scientific]]"
  - "[[literature/papers/chen2026repo]]"
  - "[[literature/papers/kapner2026scanning]]"
  - "[[literature/papers/piriyakulkij2026subagents]]"
  - "[[literature/papers/taneja2026scan]]"
  - "[[literature/papers/liu2026lost]]"
  - "[[literature/papers/razmara2026one]]"
used_by: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/file-as-bus]]"
  - "[[concepts/shared-substrate-contagion]]"
related_experiments: []
tags: [skills, interop, format-portability, harness-ecosystem, standardization]
---

# shared-skill-namespace

## Definition

A convention by which skill files (markdown documents that the agent
loads on demand, with `name:` / `description:` frontmatter and a
structured body) are *portable across harnesses* — the same file works
whether loaded by Claude Code, OpenHarness, Hermes Agent, or any
other harness that adopts the convention. The portability is achieved
at three layers:

1. **A shared on-disk layout**: standard paths under the user's home
   (e.g. `~/.claude/skills/<name>/SKILL.md`, `~/.agents/skills/...`,
   `~/.openharness/skills/...`) that multiple harnesses read from
   without coordination.
2. **A shared file format**: the `SKILL.md` schema —
   `name` / `description` frontmatter plus a markdown body with `##
   When to use` / `## Workflow` sections (Anthropic's published
   convention).
3. **A community registry**: the
   [agentskills.io](https://agentskills.io) standard, which both
   Hermes and OpenHarness explicitly reference as the interop point.

## Why it matters here

Agent skills are *procedural memory* — knowledge of how to do
something, written down so the agent can re-execute it without
re-deriving it. If procedural memory is locked to one harness, the
agent's experience becomes non-portable: switching harnesses (or
running multiple harnesses against the same workspace) means
re-writing or re-discovering skills.

OpenHarness ([[literature/repos/hkuds-openharness]]) reads skills
from three user-level roots (`~/.openharness/skills/`,
`~/.claude/skills/`, `~/.agents/skills/`) plus the project-level
equivalents — explicitly so that skills authored for Claude Code work
unmodified. Hermes ([[literature/repos/nousresearch-hermes-agent]])
declares format-compatibility with the agentskills.io standard. The
emerging picture is a de-facto namespace where a skill is identified
by its on-disk path and its `name:` field, not by which harness
loaded it.

The namespace is now also a *research substrate*: Generative Skill
Composition ([[literature/papers/zhao2026generative]]) adopts the
agentskills.io skill definition (metadata / applicability condition /
procedural policy / termination / resources, with progressive
disclosure) as the formal object its composer predicts over — academic
work building directly on the shared standard rather than inventing a
bespoke skill format. A shared schema is what makes a trained composer
transferable across harnesses at all.

For this project specifically, this is operationally relevant:
the `/digest`, `/ingest`, `/lint`, `/fetch-paper`, `/discover` skills
under `~/.claude/skills/` are *already in the shared namespace*. A
downstream project using OpenHarness instead of Claude Code can
invoke them without porting. This is a feature of the project's
architecture that we did not explicitly choose but did inherit by
following the SKILL.md convention.

## Implementation guidance

1. **Author skills in the published format.** Use Anthropic's
   `SKILL.md` schema — flat YAML frontmatter with `name`, `description`,
   and a markdown body. Avoid harness-specific extensions to the
   frontmatter unless they degrade gracefully (i.e., an unknown
   harness reads the file and ignores fields it does not understand).

2. **Put skills at canonical paths.** User-global skills go in
   `~/.claude/skills/` (the original Anthropic location); project-
   global skills go in `<project>/.claude/skills/`. Other harnesses
   read these locations; harness-specific overrides go in harness-
   specific subdirectories (e.g. `~/.openharness/skills/`).

3. **Treat `name:` as the public identifier.** When one skill links
   to another (e.g. `/wrap` chains into `/digest`), reference by
   name, not by absolute path. This keeps the link stable across
   harnesses that resolve `name` differently.

4. **Avoid harness-specific tool calls inside skill bodies.** If a
   skill says `use the Bash tool` and one harness names it `Bash`
   while another names it `Shell`, the skill breaks. Either use the
   highest-common-denominator name (most harnesses accept Anthropic's
   tool names), or reference tools by capability ("run a shell
   command") and let the host harness resolve.

5. **Test against multiple harnesses if portability matters.**
   OpenHarness ships a "Real Skills + Plugins" E2E suite that runs
   skills authored for Claude Code under OpenHarness — the test
   suite *is* the portability contract. A project that genuinely
   needs cross-harness skills should adopt the same discipline.

## Connections

- **[[concepts/skill-library-lifecycle]]** is the *intra-harness*
  story: how skills are created, refined, and pruned over time.
  Shared-skill-namespace is the *cross-harness* dimension: how the
  same lifecycle artifacts move across executors. Both concepts are
  needed; neither subsumes the other.
- **[[concepts/agent-native-memory]]** — shared-skill-namespace is
  the inter-harness specialization of "memory lives in files the
  agent owns." When the files are at canonical paths and follow a
  shared schema, the substrate becomes portable not just inside one
  agent over time, but across the agent ecosystem.
- **[[concepts/file-as-bus]]** — analogous pattern at a different
  scope. File-as-bus is multiple agents within one session sharing
  state via files; shared-skill-namespace is multiple harnesses
  (across time / installations) sharing skill artifacts via files.
  The unifying observation: the filesystem with a shared schema is a
  more robust interop substrate than IPC, RPC, or in-process state.

## A shared global root is a growing list, and skill loading drops steeply between 10 and 100 entries (2026-10-20)

Portability has a cost the definition above does not name. Every
harness that reads a shared root (`~/.claude/skills/`, `~/.agents/skills/`)
lists every skill in it, for every task. [[literature/papers/liu2026lost]]
measures what a growing list does to a progressive-disclosure agent. With
golden skills always installed, loaded-skill recall falls as random
distractors are added. Aggregate (3 open-weight models, 82 SkillsBench
tasks) is 73.6 → 68.2 → 53.3 → 27.4% at 0 / 10 / 100 / 1,000. Claude Opus
4.6 on 37 tasks is 81.3 → 77.7 → 43.3 → 30.9%. The steep part is between
10 and 100 entries. Masking the distractors' descriptions restores most
of the loss, so the cost is in what the entries say, not how long the list
is.

For this namespace that means portability and scoping pull in opposite
directions. A skill in the global root is portable to every harness and
also a distractor in every project that does not need it. This session's
listing has about 40 entries (13 user skills plus built-in and plugin
skills), inside the range where the curve steepens. Guidance that follows:

- Keep the global root small. Put specialised skill groups in
  project-level `.claude/skills/`. The namespace already supports this
  layer; it is the scope key.
- Write descriptions that state when *not* to use the skill. Description
  text is the interference channel, so vague, broad descriptions cost
  every other skill.
- Judge a skill by whether it fires, not by task success. Opus 4.6 kept
  Pass@3 at 64.9 → 62.2 while its loading collapsed. Outcome alone would
  read "no harm."

Caveats:
- The harness is a custom terminal agent, not Claude Code. Distractors
  are random market skills, not a user's own related skills.
- 37-task, implicit-only validation for the proprietary models; no CIs.
- Our ~40 are not distractors in the paper's sense. Many are relevant to
  some task here. The probe is skill-fire telemetry, not this paper's
  numbers.

## Two same-job skills in one context: one takes over, and the worked example decides which (2026-10-27)

The open question below on conflict resolution assumes the harness must
pick one file. [[literature/papers/razmara2026one]] measures what happens
when it does not pick, and both same-job skills are in context.
SkillClash has 16 jobs (commit message, SQL query, Dockerfile, …), each
with two author-built SKILL.md files. Each file has six code-checked rules
and one worked example. The readers are seven models from Qwen3-4B to
Qwen3-235B and GPT-4.1, greedy, with 8 requests per job and CIs over the 16
jobs. Both files are force-loaded into the system prompt with an
instruction to follow every rule.

- **A rival costs a third of the rules, wholesale.** A skill alone is
  followed 93–99%. Next to an unrelated skill nothing changes. Next to a
  same-job skill it falls to 61–62% on every reader, and in 45–78% of
  outputs one skill's non-conflicting exclusive rules are all kept and the
  other's all dropped.
- **The worked example is the channel.** Removing examples cuts takeover to
  7–41%. A skill that alone keeps its example wins 67–93% of takeovers.
  Ending both files with one example that follows both cuts takeover to
  1–15%.
- **Load order picks the winner, inconsistently.** The first-listed skill
  wins 74–77% of takeovers on Llama-3.1-8B and GPT-4.1 but 3–24% on
  Phi-4-mini and Qwen2.5-7B. So a precedence-by-path rule in a namespace
  spec would not predict behaviour across models.

For the namespace this means a shared root has a second cost beyond
liu2026lost's loading interference. Two portable skills for the same job,
from different authors, do not compose by default even when every rule
could hold. Implementation guidance that follows:
- Do not install a second skill for a job a house skill already covers.
  Merge the two instead. A file merged once by the strongest available
  model (here Claude Opus 5.5) reached 96–99% of achievable rules on all
  seven readers.
- End the merged file with one correct example that follows both sources.
  Do not carry over the sources' examples. Adding those back restored
  takeover to 32–55%.

Caveats:
- No frontier reader. Claude Opus 5.5 was only the merger.
- Selection is removed by force-loading, unlike Claude Code's on-demand
  `Skill` loading.
- The skills are synthetic and six rules long.
- The only real-skill evidence is observational. A reanalysis of Yan et
  al.'s Claude Code runs gives about −8 points.

This box has no same-job pairs today: user-level and project skill lists
are the same set. Our inference: the exposure arrives with plugin or
vendor skills that overlap a house skill.

## Open questions

- **How much actually transfers?** The shared layout and shared
  format are necessary but not sufficient — a skill that calls the
  `WebSearch` tool only works in harnesses that expose that tool.
  The empirical portability rate (fraction of skills that "just
  work" across harnesses) is not documented in either source.
- **Execution semantics are not part of the format.** The same
  `SKILL.md` can be run inline (body loaded into the main context) or as
  a fresh-context subagent (body seeds a new context and only the final
  message returns). [[literature/papers/piriyakulkij2026subagents]] shows
  the choice changes results: on SkillsBench the ranking of the two modes
  flips depending on whether the skill's description states an expected
  input and output, with gaps up to ~0.28 accuracy for smaller models
  (read from bar charts). Nothing in the name/description schema tells a
  harness which mode a skill was written for, so two harnesses can load
  the identical file and behave differently — a portability gap below the
  layout and format layers this concept names. A contract field (input /
  output) in the description is the cheapest candidate for closing it.
- **Versioning.** The SKILL.md schema may evolve (Anthropic has
  already shipped a multi-file variant — `SKILL.md` plus auxiliary
  files in the same directory). How harnesses negotiate version
  differences is unspecified.
- **agentskills.io as a registry.** Hermes and OpenHarness both
  point at it; whether it becomes a genuine PyPI-style registry
  (with discovery, install, versions) or stays a curated link list
  is open. The trajectory matters for whether downstream projects
  should publish skills there. SkillWiki
  ([[literature/papers/huang2026skillwiki]]) is the first concrete
  proposal for what that registry would need — lifecycle states,
  version lineage, provenance links to evidence, health monitoring,
  and governed (reviewed) contribution — though it demonstrates
  feasibility, not adoption.
- **Trust and permission verification.** The namespace now has a named
  threat model: ge2026governance formalizes the *malicious skill plugin*
  class — a skill whose runtime behavior exceeds its self-declared
  permissions while still performing its declared function — and finds
  intent-alignment judges detect it at only 75–94% (vs ~92–100% for
  prompt injection) because the covert side-channel rides a legitimate
  operation. Its OpenClaw audit names the exact gap a shared registry
  inherits: permission declarations are self-reported, with no
  independent runtime verification (G2). A skill that is *portable
  across harnesses* is also an attack that is portable across harnesses;
  a future namespace spec needs declared-permission enforcement at the
  protocol level, not just judge-based screening. **Now measured:**
  [[literature/papers/kapner2026scanning]] puts numbers on the gap.
  Across 511 published skill collections, **3.7%** ship a skill whose
  `allowed-tools` pre-approves an unrestricted shell (121/121 pairs
  confirmed at pinned commits). That is the one field through which a
  portable skill carries *execution authority* into every repository
  that installs it. The paper's fix belongs at the namespace level:
  require a scoped form in the spec, or display the field at install
  time.
  [[literature/papers/taneja2026scan]] bounds what any namespace-level fix
  can do, on a largely independent population (OpenClaw's 66,192 ClawHub
  skills, not kapner's GitHub harnesses), reading the skill *body* rather
  than its grants. A skill's documentation doesn't bound what an agent runs
  from it: 34.7% of executed commands had a consequence class in no
  documented block. All were generic shell in this non-adversarial sample.
  So "declared-permission enforcement" checks a proxy. Whether an action is
  *permitted* is also a property of the operator, not the artifact. 705
  skills that every scanner and the registry's judge cleared still instruct
  actions CIS/NIST controls prohibit, mostly `curl | bash` installers. A
  namespace spec can carry declarations and verdicts as inputs, but the
  permission decision has to be made operator-side, at the action. The
  stronger reading of "enforcement at the protocol level" above doesn't
  hold.
- **The shared format is not enforced by its reference client.** The
  Agent Skills spec requires `name` and `description`. Claude Code makes
  both optional: it defaults the name to the directory and improvises the
  description from the body's first paragraph. kapner2026scanning finds
  **3.5% of collections** ship a skill the spec's own validator rejects
  and Claude Code loads without comment. Layer 2 of the definition ("a
  shared file format") therefore holds only as loosely as the most
  permissive loader, and a skill that works in Claude Code is not
  evidence it conforms. This strengthens implementation guidance 1:
  declare `description:` explicitly, because a missing one is not an
  error — it is silently replaced by whatever the body opens with.
- **Conflict resolution.** If `~/.claude/skills/wrap/SKILL.md` and
  `~/.openharness/skills/wrap/SKILL.md` both exist with different
  bodies, which wins? OpenHarness's docs don't make the precedence
  rule explicit. A future shared-namespace spec needs one. But
  divergence is usually deliberate: 17.5% of setups in
  kapner2026scanning configure more than one assistant, and where two
  assistants' context files or MCP declarations differ, an adjudicator
  judged the difference a real defect in only 29/71 and 8/22 pairs; the
  rest were intentional per-assistant variants. A spec therefore cannot
  flag divergence as an error. It first needs a way to *declare* a
  variant (the paper recommends one source of truth plus imports).
- **Status is `seedling`** because the concept is named here from
  two sources that *implement* it but no source that *specifies*
  it. The agentskills.io spec, if read closely, is likely the
  canonical reference and would move this to `growing`.
