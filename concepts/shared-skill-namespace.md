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
  - "[[literature/repos/nousresearch-hermes-agent]]"
  - "[[literature/repos/hkuds-openharness]]"
used_by: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/file-as-bus]]"
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

## Open questions

- **How much actually transfers?** The shared layout and shared
  format are necessary but not sufficient — a skill that calls the
  `WebSearch` tool only works in harnesses that expose that tool.
  The empirical portability rate (fraction of skills that "just
  work" across harnesses) is not documented in either source.
- **Versioning.** The SKILL.md schema may evolve (Anthropic has
  already shipped a multi-file variant — `SKILL.md` plus auxiliary
  files in the same directory). How harnesses negotiate version
  differences is unspecified.
- **agentskills.io as a registry.** Hermes and OpenHarness both
  point at it; whether it becomes a genuine PyPI-style registry
  (with discovery, install, versions) or stays a curated link list
  is open. The trajectory matters for whether downstream projects
  should publish skills there.
- **Conflict resolution.** If `~/.claude/skills/wrap/SKILL.md` and
  `~/.openharness/skills/wrap/SKILL.md` both exist with different
  bodies, which wins? OpenHarness's docs don't make the precedence
  rule explicit. A future shared-namespace spec needs one.
- **Status is `seedling`** because the concept is named here from
  two sources that *implement* it but no source that *specifies*
  it. The agentskills.io spec, if read closely, is likely the
  canonical reference and would move this to `growing`.
