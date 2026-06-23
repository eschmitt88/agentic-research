---
kind: paper
title: "Less Context, Better Agents: Efficient Context Engineering for Long-Horizon Tool-Using LLM Agents"
authors: ["Abhilasha Lodha", "Mahsa Pahlavikhah Varnosfaderani", "Abir Chakraborty", "Abhinav Mithal"]
institutions: ["Microsoft"]
year: 2026
venue: arXiv
peer_reviewed: false
url: https://arxiv.org/abs/2606.10209
code_url: null
citations: null
source: "raw/papers/lodha2026less.pdf"
added: "2026-06-23"
relevance: 3
credibility: 2
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/context-eviction-policy]]"
tags: [context-engineering, context-pruning, summarization, tool-use, long-horizon, token-efficiency, mcp, enterprise]
---

# Less Context, Better Agents: Efficient Context Engineering for Long-Horizon Tool-Using LLM Agents

## TL;DR

On a 50-task enterprise expense-itemization benchmark (GPT-5 driving
Microsoft Dynamics 365 F&O via MCP), the paper compares four
context-management policies and shows that a deliberately simple one —
**prune the conversation to the last 5 tool call/response pairs and
re-insert a single automated summary of the evicted pairs** — reaches
**91.6% complete task completion** while cutting tokens by **62.7%** and
wall-clock by **60.2%** versus retaining full history. The headline
finding is counterintuitive: pruning *raises* accuracy (71.0% → 79.0%)
as well as cutting cost, because stale tool responses describe superseded
form state and actively mislead the agent.

## Claims

- In tool-heavy ERP workflows, verbose MCP tool responses (500–3,000
  tokens each, 15–30 tool calls per task) cause context overflow and
  prohibitive inference cost; full-history retention is impractical at
  scale (the full-context baseline burned 1.48M tokens and 14.56h on 50
  tasks).
- **Old context is actively detrimental, not merely redundant.** Early
  tool responses describe form states that later get superseded;
  retaining them causes stale-state errors (wrong field assignments,
  navigation errors). Pruning to recent interactions removes this noise.
- A **semantic-level** policy operating on whole tool call/response pairs
  (the agentic unit) beats token-level prompt compression (LLMLingua,
  Selective Context), which can corrupt structured form state an ERP
  agent must read verbatim.
- Recency pruning alone improves both accuracy and efficiency
  (C2→C3: +8 pp, −64% tokens); a single-pass summary of evicted pairs
  adds task-level situational awareness back (C3→C4: +12.6 pp at <4%
  token overhead) by suppressing the premature-termination failures that
  pure pruning introduces.
- The result is framed as **strong evidence for one class of enterprise
  tool-use workflow, not a proof of universal generalization** — an
  unusually explicit scope-limiting claim.

## Methods

- **Task:** hotel-expense itemization in D365 F&O — decompose a receipt
  total into 4–22 line items with correct subcategories and amounts such
  that the unallocated remainder reaches **exactly $0.00**. Zero-residual
  is a hard, business-defined success criterion (any nonzero residual =
  failure), and completion is verified by an independent read-back of
  saved D365 form state, not the agent's self-report.
- **Four configurations** on GPT-5, with the *user model held constant
  across C2–C4* to isolate the effect of context policy:
  - **C1** — no user model (a motivating ablation; agent stalls on
    unanswered clarifying questions in a non-interactive harness).
  - **C2** — full conversation history (the context-engineering baseline).
  - **C3** — context pruned to the last N=5 tool call/response pairs, no
    summary.
  - **C4** — last N=5 pruning **plus** an automated summary over the W=3
    most-recently-evicted interactions, re-inserted as one assistant
    message (costs exactly one extra LLM call per eviction event).
- **Algorithm 1 (CONSTRUCTCONTEXT):** counts tool messages, evicts the
  oldest max(0, #tool − N) of them together with their preceding
  assistant tool-call message, summarizes the W most-recent evicted
  messages, and re-inserts the summary at the earliest evicted position.
  N=5 was chosen because one itemization line costs 2–3 tool calls, so 5
  pairs ≈ two complete working cycles.
- **Evaluation:** 5 independent runs; primary metric = Completely
  Itemized (%). Reports 95% CIs (Student-t over 5 runs + pooled Wilson
  over 250 task-runs), effect sizes, a six-mode failure taxonomy, an N/W
  sensitivity sweep, generalization to two more expense categories
  (Travel, Meals & Gifts), and a cross-model replication on Claude Sonnet
  4.5.

## Results

- **Headline (Hotel, n=50, 5 runs):** C1 8.0% → C2 71.0% → C3 79.0% → C4
  **91.6%** complete itemization. Tokens: C2 1,481K → C3 535K → C4 553K.
  Time: C2 14.56h → C3 5.39h → C4 5.79h. C4 vs C2: **−62.7% tokens,
  −60.2% time, +20.6 pp accuracy.**
- Pooled Wilson 95% CIs: C4 [87.5, 94.4] cleanly separated from C3
  [73.5, 83.6] — strong evidence summarization beats pruning alone.
  C4's run-to-run dispersion (±1.7) is far tighter than C3's (±8.2):
  summarization absorbs the variance aggressive pruning exposes.
- Input tokens are 99.75–99.87% of total usage; output tokens are flat
  (2,486–2,567) across C2–C4, confirming context management targets the
  dominant cost.
- **Generalization:** the C1→C2→C3→C4 accuracy ordering holds
  monotonically across all three structurally distinct expense categories
  (+19 to +20.6 pp from C2→C4).
- **Cross-model:** Claude Sonnet 4.5 does *not* stall without the user
  model (88.0% C1 baseline vs GPT-5's 8.0%), so the huge GPT-5 C1→C2 gap
  is model-specific stalling, not the value of context engineering. The
  context-engineering ordering itself still holds (Sonnet 92.0% → 94.5%
  with summarization).
- **Failure taxonomy confirms the mechanism:** stale-state references
  drop 34→6→4 across C2/C3/C4; premature terminations 9→18→3 (pruning
  trades stale-state for premature-termination, summarization fixes it).
  Residual modes (wrong subcategory mapping, duplicate/skipped items) are
  policy-invariant and bound the headroom.

## Critique / open questions

- **Single narrow domain.** One ERP product (D365 F&O), one task family
  (expense itemization), one MCP integration. The authors are explicit
  this is "one class of enterprise workflow," but that means the
  architectural transfer to *research* agents is by analogy, not
  attested.
- **No code or artifacts released** — the benchmark, harness, MCP proxy,
  and result files are all internal/synthetic, so the headline numbers
  are not independently reproducible.
- **The strongest single number (C1→C2, +63 pp) is shown to be a
  model-specific harness artifact**, not the value of context
  engineering — a refreshing admission, but it means the genuinely
  load-bearing comparison is the more modest C2→C4 (+20.6 pp), which
  rests entirely on this one domain.
- **Importance-pruning (retain the N most-recently-*referenced* pairs)
  is left as future work** — the policy is purely recency-based, which is
  exactly the heuristic that breaks when an old-but-still-relevant tool
  result gets evicted. The paper sidesteps this because expense state is
  cleanly superseding; research workflows may not be.
- N/W sensitivity plateaus past N=5, W>3 — convenient, but tuned on this
  single task structure (2–3 tool calls/line).

## Trust signals

- **Credibility:** 2 — an arXiv preprint (not peer-reviewed) from a
  Microsoft team with **no released code, benchmark, or result files**;
  the evaluation is on an internal synthetic D365 benchmark that cannot
  be independently checked. Methodology is unusually careful for the
  genre (5 runs, Wilson CIs, effect sizes, failure taxonomy, cross-model
  replication, explicit scope limits), which lifts it above a bare
  preprint — but the no-artifact, single-internal-domain combination
  caps trust at 2.

## Follow-up

- **Relevance:** 3 — a concrete, well-measured attestation of
  [[concepts/context-eviction-policy]]: "evict all but the last N=5 tool
  call/response pairs and re-insert one summary of the evicted window" is
  a fully specified eviction heuristic with a real token/quality tradeoff
  curve (Algorithm 1, the C2→C3→C4 ladder, the N/W sweep). The finding
  that *old context is actively harmful, not just redundant* — and that a
  summary of evicted state is the cheap fix — is directly importable to
  any long-horizon research-agent loop that accumulates verbose
  tool/search results. Held at 3 (not higher) because the domain is
  enterprise expense itemization, not research-agent architecture, and
  because the policy is purely recency-based (no importance/relevance
  gating), which is the harder case for research workflows where an old
  result can still matter. Provides a strong empirical data point for the
  concept rather than seeding a new one.
