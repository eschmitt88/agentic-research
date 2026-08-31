---
kind: paper
title: "EvoMal: Self-Poisoning in Self-Evolving Coding Agents"
authors: ["Xiaodong Wu", "Yu Shi", "Qi Li", "Zhimin Zhao", "Xiangman Li", "Bram Adams", "Ahmed E. Hassan", "Jianbing Ni"]
institutions: ["Queen's University"]
year: 2026
venue: "arXiv (cs.CR)"
peer_reviewed: false
url: "https://arxiv.org/abs/2608.25776"
code_url: null   # withheld; attack paper
citations: null
source: "raw/papers/wu2026evomal.pdf"
added: "2026-08-31"
relevance: 5
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/shared-skill-namespace]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/verified-memory-writes]]"
tags: ["agent-security", "skills", "memory", "safety", "agent-architecture"]
---

# EvoMal: Self-Poisoning in Self-Evolving Coding Agents

## TL;DR

Self-evolving coding agents write new tools by *imitating* retrieved skills.
A planted malicious skill can therefore become the **template** for a new
skill that preserves the payload — the agent authors, stores, and runs its own
malicious code. Each authored copy re-enters the library and can be imitated
again, forming a self-propagating worm that **persists after the planted
skills are removed**.

## Claims

- Self-poisoning is a distinct vulnerability from skill poisoning: the harmful
  artifact is *agent-authored*, not attacker-submitted, so defenses keyed on
  attacker-submitted names, code, and signatures do not see it.
- The attacker never has to invoke anything — planting is sufficient.
- The library is a **self-propagating** medium, not just a contaminated store.
  Removing the planted skills does not terminate the infection.
- The imitation trigger is structural: a **banner** of benign-looking
  structural elements wrapped around an interchangeable payload induces the
  authoring agent to reproduce the enclosed code.
- The threat is not hypothetical at production scale — the MCP Registry
  (backed by Anthropic, GitHub, Microsoft) opened in Sept 2025, skill
  marketplaces hold tens of thousands of entries, an audit of 98,380
  marketplace skills found 157 deliberately malicious, and CVE-2025-6514 was a
  CVSS-9.6 command injection in this surface.

## Methods

- **EvoMal**: wraps an interchangeable payload in a banner to amplify
  imitation. Payload and banner are separable, so the contribution of each is
  measurable.
- Metric: **agent self-poisoning rate (ASPR)** — the fraction of tasks that
  add a *newly authored* malicious skill to the library.
- Six models, **153 tool-relevant SWE-bench Verified tasks**; multi-round runs
  to observe propagation and post-removal persistence.
- Defense: **counter-prompt**, which discourages banner-style copying.

## Results

- ASPR ranges **20.3%–41.8%** across six models; poisoned libraries end up
  holding **4.9–9.0×** as many malicious skills as were planted.
- The vulnerability appears **without** a banner too: DeepSeek-V4-Pro reaches
  11.1% ASPR on the payload alone.
- Tailoring planted skill *descriptions* to one task family raises ASPR to
  **86.7%** — retrieval targeting is the strongest amplifier.
- **Persistence**: after planted skills are removed, Qwen3 still shows a
  round-5 ASPR of **68%**, carried entirely by agent-authored copies.
- Counter-prompt reduces EvoMal's ASPR to **≤6.7%** with no significant
  task-completion loss.

## Critique / open questions

- Counter-prompt is a *prompt-layer* defense, and
  [[literature/papers/rahman2026framing]] (same window) reports that
  prompt-layer defenses are brittle to reframing. ≤6.7% against the studied
  banner is not evidence against an adaptive banner. The authors do not claim
  otherwise, but the pairing is worth holding in mind.
- No released code — appropriate for an attack paper, but it means the ASPR
  numbers are not independently checkable.
- ASPR measures *addition of a malicious skill to the library*, not successful
  exploitation. The worm framing is well supported; the end-to-end harm claim
  leans on the referenced real-world CVE rather than on demonstrated impact.

## Trust signals

- **Credibility:** 4 — Queen's University, with Bram Adams and Ahmed E. Hassan
  (a major, long-established software-engineering research group). Real
  benchmark (SWE-bench Verified), six models, a defined metric, multi-round
  propagation evidence, and a proposed defense. Not peer reviewed and no code
  released, which is why it is not 5; code withholding is defensible for an
  attack paper.

## Follow-up

- **Relevance:** 5 — [[concepts/shared-skill-namespace]] has been argued in
  this project purely as a *benefit* (reuse across projects). This is the
  first source naming its failure mode, and the failure mode is specific to
  the design: a shared library plus imitation-based authoring is a
  self-propagating medium. This project runs a shared skill namespace under
  `~/.claude/skills/`.
- Extends [[concepts/verified-memory-writes]] from facts to *skills* — an
  executable memory write is strictly more dangerous than a factual one, and
  the write here is authored by the agent, which is exactly the case
  write-time attestation cannot distinguish.
- Forces a question on [[concepts/skill-library-lifecycle]]: the lifecycle
  currently has a promotion path and no quarantine or provenance-of-authorship
  step. "Which skills were agent-authored by imitation, and from what?" is not
  currently recoverable here.
- Direct counterpart to [[literature/papers/zhan2026auto]], which proposes
  compiling typed policy into the skill artifact — a structural defense on the
  same substrate this attack exploits, and a better bet than counter-prompt.
