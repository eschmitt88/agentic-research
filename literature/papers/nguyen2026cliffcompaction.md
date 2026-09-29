---
kind: paper
title: "CliffCompaction: Cost-Efficient Compaction for Long-Horizon Coding Agents"
authors: ["Trang Nguyen", "Eulrang Cho", "Bingqing Chen", "Tim Dettmers"]
institutions: ["Carnegie Mellon University", "Bosch Center for AI"]
year: 2026
venue: "arXiv (cs.AI; cs.LG, cs.SE)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.26779"
code_url: "https://github.com/nguyenvuthientrang/cliffcompaction"
citations: null
source: "raw/papers/nguyen2026cliffcompaction.pdf"
added: "2026-09-29"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/lossless-context-offload]]"
  - "[[concepts/pass-at-k]]"
  - "[[concepts/budget-as-ceiling]]"
tags: ["compaction", "context-management", "kv-cache", "prompt-caching", "cost", "truncation-vs-summarization", "coding-agents", "terminal-bench", "swe-bench-verified", "kernelbench", "test-time-scaling", "trajectory-selection", "claude-code"]
---

# CliffCompaction: Cost-Efficient Compaction for Long-Horizon Coding Agents

## TL;DR

A rule-based, LLM-free compaction for coding agents. The context grows
append-only, so the prefix cache stays valid, until a token threshold B.
Then the live session is reduced to verbatim fragments: tool results over
500 chars are dropped, tool calls cut to a ~150-char signature, thoughts
truncated to 300 chars, and system prompt, task and the last K turns kept.
The **previous compacted block is thrown away, not nested**. The result is
a "cliff" in context size and cost. The measured wins are real but mostly
about **cost**, from cache reads. On quality it roughly ties the
alternatives: on 1M-token KernelBench runs it scores 3.58× against 3.47×
for Claude-Code-style summarization, and summarization is slightly
cheaper. The headline test-time-scaling result, "matches Opus 4.7", depends
on a learned trajectory selector (SGV) whose own cross-validated number is
5 points lower.

## Claims

- **Precision over recall.** Summaries have high *compaction recall* (they
  carry information across many compactions) but low *precision* (distortion
  or omission). CliffCompaction deliberately keeps only the most recent
  window, verbatim: "it throws away all direct information after two
  compactions." Table 1 lists full-history recall as the one property it
  gives up.
- **Never compact a compaction.** C_t = CliffCompaction(L_t); C_{t−1} "is
  discarded entirely rather than nested into C_t". That rules out
  summary-of-summary drift by construction. The older information that
  survives does so only as *residual propagation*: the agent's behaviour in
  L_t was conditioned on C_{t−1}.
- **Cache reads are the cost.** Every context edit invalidates the KV
  cache, and uncached input is priced about 5–6× cached. So a policy should
  leave the prefix untouched between rare, large compaction events rather
  than edit it continually (microcompaction) or every step (sliding window).
- **Signatures are recovery handles.** Tool calls keep name, path and key
  arguments, so "any removed context remains recoverable" by re-issuing the
  call. Written file content is dropped because the file persists in the
  workspace.
- **Scaffold-agnostic.** It ships as an API proxy that works with closed
  harnesses (Claude Code, Codex).

## Methods

- **Algorithm 1.** On a `ContextWindowExceeded`, keep messages[0]
  (system) and messages[1] (task). Run CliffCompaction over the remaining
  turns, skipping prior compaction blocks. Keep the 2K most recent messages
  verbatim. Thresholds B ∈ {45K, 32K, 16K, 8K} for SWE/Terminal-Bench, and
  128K/200K for KernelBench.
- **Scaffolds and models.** mini-swe-agent and OpenHands on SWE-bench
  Verified; Terminus-2 on Terminal-Bench 2.0; real Claude Code, via the
  proxy, on Terminal-Bench 2.1 with GLM 5.3 Flash. OpenHands on KernelBench
  L3 (50 problems) with Kimi K2.7/K2.6 (CUDA) and GPT-5-mini (Triton).
  Models are all open-weight or small: Kimi K2.5–K2.7 and GLM 4.7 Flash–5.3.
  **No Claude model is ever run under CliffCompaction.**
- **Baselines (§6).** Sliding window; LLM summarization using Claude
  Code's summarization prompt; and "Summarization + Microcompaction", a
  *reimplementation* of Claude Code's two-tier scheme that adapts
  Anthropic's `clear_tool_uses`.
- **Cost accounting.** Main-text costs come from an idealized
  *perfect-caching* model: any shrink in prompt length counts as a full
  cache miss. Table 8 gives provider-metered real costs alongside.
- **Test-time scaling (§4).** k rollouts are scored by *Soft Group
  Verification* (SGV): a LightGBM classifier over 30 trajectory and solution
  features, including within-group line and symbol overlap. For the
  main-table numbers it is trained on the *other* model's rollouts over the
  same benchmark tasks.
- The number of seeds is not stated; results carry ± bands.

## Results

- **Terminal-Bench 2.0, Terminus-2 (Table 2).** Kimi K2.6: full context
  scores 59.16±3.41 at $0.40. CliffCompaction scores 61.42 at both 32K ($0.24)
  and 16K ($0.19). Native summarization at 16K scores 55.45 ($0.26). At 8K,
  Cliff drops to 50.00±2.40 and summarization to 42.97 at $0.60. GLM 5.1
  goes from 49.83 at full context to 53.20 (32K) and 54.33 (16K). These are
  the paper's "≈50% cheaper at held or better performance". The Kimi gain is
  +2.26 pp with overlapping bands.
- **Real Claude Code, Terminal-Bench 2.1, GLM 5.3 Flash.** Cliff at ~45K
  scores 76.69±1.41 at $0.16. Claude Code auto-compaction at 45K scores
  70.97 at $0.14, and at 200K 73.03 at $0.21. So against the native
  compactor at a matched budget Cliff is **more accurate but ~14% more
  expensive**, not cheaper.
- **SWE-bench Verified (Table 3).** Compaction is roughly flat to slightly
  negative. Kimi K2.6 scores 73.87 → 73.27 (32K) → 71.87 (16K) → 67.60 (8K).
  Real-cost savings on mini-swe-agent with Kimi K2.6 are 0% at 32K, −8% at
  16K, and **+1% at 8K** (Table 8). OpenHands saves 28–35% at 32K.
- **Cost decomposition (Table 7, TB2.0, GLM 5.1, 16K).** Cache reads are
  78% of the uncompacted bill. Cliff cuts cache-read cost 80% ($0.427 →
  $0.087). Re-prefill and extra-turn overhead add $0.06 against $0.34 saved.
  Total cost falls 51%. Tool results plus tool calls are 84% of context
  tokens. Provider cache-hit rate falls as B tightens: 79% → 47% for GLM 5.1
  on Terminal-Bench. Savings are therefore **non-monotonic** in B: for Kimi
  K2.6 on TB they are 53% at 16K and 37% at 8K.
- **Head-to-head (Table 6).** On SWE-bench Verified at 16K, all four methods
  are within 2.6 points. Sliding window scores highest on both models and is
  the only one that costs more than full context (+9%/+7%). On KernelBench L3
  (128K, 400 steps): **Cliff 3.58× at $8.32**, summarization 3.47× at $8.10,
  summarization + microcompaction 3.33× at $12.84, sliding window 2.86× at
  $21.52.
- **KernelBench continual learning (Table 5).** Without compaction, the
  256K window runs out at a median of 99 steps, and 98% of L40S runs end
  early, which caps speedup at 1.30×. Cliff at 128K reaches 2.23× at 200
  steps and 3.58× at 400. The same holds with Kimi K2.6 on RTX Pro 6000:
  1.51× → 1.87× → 2.48×. Against published kernel agents (reported numbers,
  not rerun), GPT-5-mini reaches 2.09× vs AdaExplore's 1.78× at 200 steps.
- **Precision proxy (Fig. 6, SWE-bench Verified, Kimi K2.7, 16K).** Relative
  to full context, re-reads rise +5.04 per instance under Cliff and +2.35
  under summarization. Summarization is the only strategy that also opens
  *fewer* new files (−0.68). The authors read this as a paraphrase that
  "appears to satisfy the agent and discourages it from returning to the
  ground-truth source."
- **Test-time scaling (Table 4).** Kimi K2.6 + Cliff 16K + SGV at k=3
  scores 69.7 practical (74.2 oracle) at $58.01. The same without Cliff at
  256K scores 64.0 at $91.65. Opus 4.7 is 69.4, taken from its system card
  with no scaffold listed. GPT 5.3 Codex scores 64.7 on Terminus-2. On
  SWE-bench Verified (Table 13) compaction makes scaling *cheaper but
  worse*: the best configuration is uncompacted Kimi K2.6 + SGV k=5 at 79.4,
  against 77.0 for the best compacted one.
- **Runtime.** OpenHands wall-clock falls 23% (Kimi) and 34% (GLM) at 32K.
  Self-hosted vLLM with GLM 4.7 Flash speeds up 1.60× / 2.10× / 2.62× at
  32K / 16K / 8K.

## Critique / open questions

- **The drift argument is asserted, not measured.** No experiment isolates
  summary-of-summary degradation. On the one benchmark long enough to
  compound it (KernelBench, 1M+ tokens), plain summarization reaches 3.47×
  against 3.58×, single runs, no error bars, at 2.6% lower cost. The
  re-read proxy (Fig. 6) is suggestive. It measures behaviour, not fidelity.
- **The KernelBench headline is mostly liveness plus budget.** 1.30× →
  2.23× compares a run that died at median step 99 with one that ran to 200.
  2.23× → 3.58× doubles the step budget. That is
  [[literature/papers/fan2026empirical]]'s "most of what a policy buys is not
  dying", in a new regime. Policy vs policy at equal steps is the Table 6
  spread (2.86–3.58×). The claim of beating specialized agents uses numbers
  copied from those papers, with a different model and hardware for
  CUDA-Agent (Seed 1.6 RL, H20). Dettmers is also a co-author of AdaExplore.
- **The Opus-4.7 match leans on the selector, and on tasks it has seen.**
  Table 4's SGV is trained on GLM 5.1 rollouts of the *same 89 tasks*.
  Table 17's "data efficiency" check trains on a fraction and still
  evaluates on all 89. Table 16 reports 5-fold *instance-level* CV for the
  same Kimi K2.6 k=3 setting: cross-model **64.4 at 16K and 60.7 at 256K**,
  against 69.7 and 64.0 in Table 4. The paper never reconciles the two. At
  64.4, the result sits below GPT 5.3 Codex (64.7), not level with Opus 4.7.
  The comparator is cross-harness in any case.
- **"Up to 50%" is Terminal-Bench, perfect-cache.** The savings are
  scaffold-, model- and threshold-dependent. They are near zero on
  mini-swe-agent with Kimi and negative against Claude Code's own compactor
  at matched budget. Tighter is not always cheaper, because cache-hit rate
  collapses.
- **Lossless only for environment-backed content.** Truncated thoughts (300
  chars) have no recovery path. The signatures that serve as handles are
  themselves discarded two windows later. Recovery works because the
  *workspace* is the store, which is fan2026empirical's re-derivability
  scope condition.
- **Scope.** Coding and terminal tasks with open-weight models only. The
  limitations section says the benefit is "meaningful only for
  medium-to-long-horizon tasks" and that no comparison was made against
  trained or external-memory context managers.
- **Relation to [[literature/papers/semenov2026beyond]].** Cited as
  "structure-based … cache-inefficient", but not run as a baseline.

## Trust signals

- **Credibility:** 3. A reputable academic group: CMU (Tim Dettmers) with
  Bosch Center for AI. Code is public (the GitHub repo resolves). Four
  scaffolds, about ten models, ± bands, and a real-cost table alongside the
  idealized one. Held at 3, not 4: it is a days-old, unreviewed preprint; the
  seed count is unstated; the KernelBench tables are single runs; the
  specialized-agent and Opus 4.7 comparisons use reported numbers across
  harnesses; and the SGV headline disagrees with the paper's own
  cross-validated table. Trust the cost decomposition and the Table 6
  head-to-head more than the SOTA and "matches Opus" framing.

## Follow-up

- **Relevance:** 4. It is the first head-to-head in the graph of truncate
  vs summarize vs microcompact vs sliding window, priced in billed dollars
  with the cache term decomposed, and one baseline reimplements Claude
  Code's own scheme. It sharpens [[concepts/context-eviction-policy]]
  guidance #3 ("compact, don't truncate") and gives measured support to
  guidance #8 and mason2026missing's "batch structural mutations". It is not
  a 5 because the quality delta over summarization is within noise where it
  matters, and no Claude model was tested.
- **Digest correction (item 17).** "Each pass works on original content"
  understates the design: everything before the previous compaction is
  *discarded*, not recompacted. "2.23× → 3.58×" is a doubled step budget
  against a baseline that dies at step 99, not a gain from compaction
  quality. "Held or better Terminal-Bench performance" holds at 32K and 16K
  but not at 8K (50.00 vs 59.16).
- **For this box.** The proxy is directly usable with Claude Code, but the
  only Claude Code result shows +5.7 pp at +14% cost against native
  auto-compaction, with GLM 5.3 Flash, not a Claude model. Worth a trial on
  a long `/curate` run before adopting.
