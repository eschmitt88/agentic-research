---
kind: paper
title: "String: An Agentic OS Where Every App Is a Markdown File"
authors: ["Jookyung Song", "Nojun Kwak", "Simyung Chang"]
institutions: ["Seoul National University", "H1R.AI"]
year: 2026
venue: "arXiv (cs.AI)"
peer_reviewed: false
url: "https://arxiv.org/abs/2608.28027"
code_url: "https://github.com/ (open-source runtime; see paper)"
citations: null
source: "raw/papers/song2026string.pdf"
added: "2026-09-01"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/file-as-bus]]"
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/lossless-context-offload]]"
  - "[[concepts/scripted-tool-pipelines]]"
  - "[[concepts/information-firewall]]"
  - "[[concepts/enforcement-boundary-placement]]"
tags: ["agent-architecture", "coordination", "context-window", "safety", "governance"]
---

# String: An Agentic OS Where Every App Is a Markdown File

## TL;DR

Every surface an agent works through was designed for someone else — pages
for human eyes that can skim, tool schemas for programs that pay nothing to
carry unused definitions. An agent re-reads and re-pays for everything on
every turn. String moves tool knowledge **out of the context window** into a
runtime that renders it back **one view at a time as Markdown**, behind two
verbs: `/open` to see and `/act` to do.

## Claims

- The interface, not the model, is where agent token cost is set. A
  **53-token resident stub** suffices at any catalog size because the stub
  names a grammar rather than teaching a schema.
- **Partial views are correct by design, and staging is causal.** Showing
  everything available is actively harmful, not merely expensive.
- **Web and app are two renderings of one architecture**: an SFMD site
  serves styled HTML to browsers and the raw document to agents, so one
  grammar reaches apps, files, shells, and the web with no per-site
  integration.
- **Privilege follows provenance.** Trust is a property of where a document
  came from, enforced in code rather than by convention.

## Methods

- **SFMD** (String-Flavored Markdown) is a strict superset of CommonMark
  adding six constructs, each mapping onto an existing Markdown production,
  so any SFMD file still renders in an ordinary viewer: YAML frontmatter
  (identity, entry action, credentials), addressable blocks (invisible HTML
  comments), directives (menus, includes, requirements), shortcuts, and
  fenced actions.
- Four design principles, of which **P2 (uniform surface / location
  transparency)** and **P4 (recursive rendering — action output is
  re-parsed and re-rendered as SFMD, so results carry the same affordances)**
  do most of the work.
- **Topics** scope both state and privilege. Every command runs against a
  named topic, and privilege follows topic *type in the code, not by
  convention*: a remote SFMD page may invoke HTTP actions but never the
  shell; only a local document or installed app gets filesystem access;
  caller-supplied text never expands a stored secret.
- 87-task benchmark pairing each task with curated skills, six models from
  frontier to small. Plus three months of reported production use.

## Results

- **Staging is causal, and the effect is large.** Disclosing one tier of
  detail a single turn too early costs **up to 23 accuracy points**.
  Holding available information fixed, showing every action at once pushed
  wrong-action selection from **2% to 28%**; staged disclosure recovered it.
- **+1.3pp** aggregate success across six models while using **33.5% fewer
  tokens** among completed episodes.
- The resident interface stays a **constant 53 tokens at any catalog size**.

## Critique / open questions

- **+1.3pp is within noise**; the honest headline is "comparable accuracy at
  one third fewer tokens", which is what the abstract in fact says. The
  paper is a cost result, not a capability result.
- The 23-point staging effect is the most interesting number and the least
  characterized — "up to" across an unspecified set of conditions.
- Rewriting the world as SFMD is a large adoption cost, and the "no
  per-site integration" claim covers *reading* the web, not making
  arbitrary services act.
- Three months of production use is cited but the deployment is not
  characterized, so it functions as an assertion rather than evidence.
- The provenance-based privilege model is the strongest contribution and is
  presented almost in passing, without an adversarial evaluation.

## Trust signals

- **Credibility:** 3 — SNU (Nojun Kwak) with a startup; open-source
  runtime; a real 87-task benchmark across six models; controlled staging
  ablations that hold available information fixed, which is the right
  design for that claim. Held down by no peer review, a headline accuracy
  effect inside noise, and an unevaluated security model.

## Follow-up

- **Relevance:** 4 — pushes [[concepts/file-as-bus]] to its limit case. That
  concept argues plain files are a *sufficient coordination substrate*;
  here the file **is** the application, and the load-bearing addition is
  that this makes privilege *derivable from location* rather than
  configured separately.
- **"Privilege follows provenance" is the transferable rule** and it is
  what [[concepts/permission-gate-as-architecture]] has been reaching for.
  This project already has the substrate: `raw/` is immutable by rule,
  `concepts/` is agent-written, `~/.claude/skills/` is executable — three
  trust domains distinguished only by convention and prose today.
  String enforces the equivalent distinction in code. This is a sharper
  version of the same boundary [[literature/papers/rahman2026framing]] and
  [[literature/papers/leong2026recognition]] argue for from the attack
  side, and it lands on the *placement* axis the 08-31 `/promote-moc` entry
  flagged as uncovered by both [[concepts/typed-enforcement]] and
  [[concepts/permission-gate-as-architecture]].
- The staging result is a direct argument for how skills should be written:
  [[concepts/scripted-tool-pipelines]] and the project's own skill files
  currently front-load their full instruction text. A 2%→28% wrong-action
  penalty for showing all options at once is evidence that progressive
  disclosure is a correctness mechanism, not just a token economy — which
  bears on the `skill-authoring-concise-goal-level` preference already
  recorded for this project.
- Concrete support for [[concepts/lossless-context-offload]]: a constant
  53-token resident footprint at any catalog size is the strongest number
  yet for moving capability description out of the window entirely.
