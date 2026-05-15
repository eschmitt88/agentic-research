---
kind: concept
name: "scripted-tool-pipelines"
status: seedling
added: "2026-05-15"
source_papers: []
sources:
  - "[[literature/repos/nousresearch-hermes-agent]]"
used_by: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/budget-as-ceiling]]"
related_experiments: []
tags: [agent-architecture, tool-use, context-budget, efficiency, code-generation]
---

# scripted-tool-pipelines

## Definition

When an agent needs to chain several tool calls, write the chain as
a short script that calls the tools through a programmatic interface
(RPC, function calls, a thin SDK) rather than emitting each tool call
as a separate LLM turn. The LLM pays context cost only on the script
itself and the final return value — every intermediate tool result
that would have re-entered the conversation as an assistant/tool
message round-trip stays inside the script's local scope.

## Why it matters here

Frontier agents spend most of their context budget on round-tripping
intermediate tool results. A 10-step pipeline that grep-greps, filters,
parses JSON, then writes a summary file emits 10 tool-call messages
and 10 tool-result messages — easily tens of thousands of tokens for
work whose final output is a single line. Hermes's README states the
mitigation in one sentence:

> "Write Python scripts that call tools via RPC, collapsing multi-step
> pipelines into zero-context-cost turns."

The pattern is widely useful but rarely named. Claude Code's leaked
harness includes the same mechanism (the `Bash` tool plus the ability
to compose shell pipelines is the un-scripted analogue); OpenHarness
exposes tools via RPC for the same reason. Naming the pattern makes
it explicit so downstream projects choose it deliberately instead of
defaulting to per-step tool turns.

For research agents specifically, scripted pipelines are how a
long-running curation skill (e.g. `/digest`, `/lint`) avoids
exhausting its context on intermediate scans — a script can shell out,
filter, and aggregate before any of that touches the LLM's view of
the conversation.

## Implementation guidance

1. **Expose tools as a callable surface, not only as message-level
   tool-use.** The agent should be able to invoke its own tools from
   inside generated code. Hermes does this via Python + RPC;
   Claude Code does it via `Bash` + the shell + binary tools;
   OpenHarness exposes tools through both interfaces.

2. **The script is the agent's "compiled" plan.** When the model
   knows in advance what the chain looks like, emitting a script is
   strictly cheaper than emitting each step. When the chain branches
   on intermediate results in ways the model can't predict, fall back
   to step-by-step tool-use. Treat the choice as a runtime decision,
   not an architectural commitment.

3. **Sandbox the script's execution environment.** Scripted pipelines
   are a richer attack surface than message-level tool calls (one
   script can call many tools in one go without permission gates per
   call). Wrap script execution in the same permission/sandboxing
   layer used for free shell access — see Claude Code's permission-
   gate pattern, where Bash is one tool among many but the most
   permission-gated of them all.

4. **Return structured results, not transcripts.** The script's
   return value should be a small structured object the LLM can
   reason over, not a dump of intermediate output. Otherwise the
   "zero-context-cost" property is lost in the return path.

5. **Compose with subagent spawning when the chain is long.** A
   script that calls a subagent (which itself runs many tool calls
   in its own isolated context) is the limit case: the parent
   agent sees one input, one summary output, regardless of how
   much work the subagent did. This is where scripted-tool-pipelines
   composes with [[concepts/hierarchical-delegation]].

## Connections

- **[[concepts/agent-native-memory]]** — the same write-into-files
  philosophy: minimize the conversation-as-state-buffer footprint.
  Memory writes go to disk, tool chains run in scripts; what
  re-enters context is the small thing that needs reasoning.
- **[[concepts/hierarchical-delegation]]** — scripted pipelines and
  delegated subagents are two points on the same axis (cost-isolate
  the bulk work from the parent's context). Scripts are cheaper but
  less flexible; subagents are heavier but can adapt mid-chain.
- **[[concepts/budget-as-ceiling]]** — if a project tracks
  `max_tokens` as a hard ceiling, the choice between scripted and
  message-level tool chains can dominate whether a long curation
  loop fits under the ceiling. The pattern is not just a clarity
  win — it's a budget mechanism.

## Open questions

- **When does scripting hurt?** Branching pipelines that depend on
  intermediate LLM judgment lose value when scripted (the script
  has to call the LLM mid-execution, which negates the
  "zero context cost" claim). The break-even branching depth is
  unmeasured.
- **Tool-result schema discipline.** For a script to call N tools
  and return one summary, the tools need to expose their results in
  a structured way the script can reduce. Tools that only emit
  human-readable text force the script to do parsing it isn't well-
  suited for. How much friction this creates in practice depends
  on the tool ecosystem.
- **Debugging surface.** A script that runs 10 tool calls and
  returns one number is hard to debug post-hoc — the LLM never saw
  the intermediates. Hermes and OpenHarness both log RPC calls; the
  log becomes the audit trail. Whether this is sufficient or whether
  scripted pipelines need a richer trace format is open.
- **Status is `seedling`** because the pattern is named here from a
  single source (Hermes README) plus passing mentions elsewhere. A
  downstream project that actually wraps `/lint` or `/digest` as a
  scripted pipeline and measures the token saving would move it to
  `growing`.
