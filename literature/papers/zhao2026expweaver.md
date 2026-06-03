---
kind: paper
title: "Rethinking Experience Utilization in Self-Evolving Language Model Agents"
authors: ["Weixiang Zhao", "Yingshuo Wang", "Yichen Zhang", "Yanyan Zhao", "Yu Zhang", "Yang Wu", "Dandan Tu", "Bing Qin", "Ting Liu"]
institutions: ["Harbin Institute of Technology", "Huawei"]
year: 2026
venue: "arXiv 2605.07164"
peer_reviewed: false
url: "https://arxiv.org/abs/2605.07164"
code_url:
citations:
source: "raw/papers/zhao2026expweaver.pdf"
added: "2026-05-11"
relevance: 5
credibility: 4
status: read
related_experiments: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/evolutionary-expansion]]"
tags: [memory, experience-utilization, self-evolving-agents, retrieval-policy, uncertainty-gating, react, grpo, knowledge-organization]
---

# Rethinking Experience Utilization in Self-Evolving Language Model Agents

## TL;DR

Existing self-evolving agent work focuses almost entirely on the
*write-side* of memory — how experience is constructed, represented,
and updated — and applies experience at runtime through rigid fixed
schedules (inject once at init, or inject at every step). ExpWeaver
shows that the *read-side policy* — when the agent decides to invoke
memory during reasoning — is itself a critical design axis: a
minimal prompting intervention that exposes experience as an
optional resource triggered by a `[Retrieve]` token consistently
beats both fixed-schedule baselines across 4 self-evolving
frameworks, 7 LLM backbones, and 3 environments.

## Claims

- **Experience utilization is a first-class design dimension,
  orthogonal to experience construction.** Holding construction
  fixed and varying only when retrieval fires changes outcomes
  substantially; rigid initialization-only and always-on schedules
  both leave performance on the table.
- **Decision-time gating beats fixed schedules across frameworks,
  models, and tasks.** ExpWeaver wins on ReasoningBank
  (distilled insights), AWM (workflows), SkillRL (skills), and
  G-Memory (hybrid raw + insights) — i.e., the gain is independent
  of how experience is represented.
- **The gating behavior is learnable, not just a prompting effect.**
  GRPO training on Qwen3-4B/14B amplifies the ExpWeaver advantage
  (e.g., ALFWorld: Init-only 70.6% → ExpWeaver 75.9%).
- **The gate fires meaningfully, not mechanically.** Three
  diagnostics converge: usage is *task-aware* (ALFWorld induces far
  more retrievals than WebShop or QA), *model-aware* (stronger
  backbones retrieve less), and concentrated at decision points
  where the agent has high token-level entropy (= uncertainty).
- **The gating is causally beneficial.** Ablating either
  (a) experience at ExpWeaver-chosen positions or (b) the position
  selection (random-position retrieval) underperforms ExpWeaver,
  confirming that *where* retrieval fires — not just *how often* —
  drives the gain.

## Methods

- **Setup.** Augment the ReAct system prompt with an instruction
  that the agent may emit `[Retrieve]` during reasoning if extra
  guidance is needed. When the token appears, the current context
  is used as the retrieval query; otherwise the agent proceeds
  without memory consultation. Loop becomes:
  `reasoning → (optional experience utilization) → action`.
- **Frameworks tested** (memory representation in parens):
  ReasoningBank (distilled insights), AWM (executable workflows),
  SkillRL (reusable skills, RL-curated), G-Memory (raw trajectories
  + distilled insights, multi-agent).
- **Backbones:** GPT-5.2, DeepSeek-V4-Pro, Kimi-K2.5,
  Qwen3.5-397B-A17B, plus Qwen3 32B/14B/4B.
- **Environments:** ALFWorld (embodied), WebShop (web nav),
  6 QA datasets (HotpotQA, NQ, TriviaQA, 2Wiki, MuSiQue, Bamboogle).
- **RL extension:** Same trigger-token mechanism, trained end-to-end
  with GRPO (binary task-success reward, KL anchor to ref policy).
- **Cognitive-neuroscience framing:** prefrontal-cortex gating of
  memory retrieval during goal-directed behavior as the inspiration
  for decision-time uncertainty-conditioned retrieval.

## Results

- **Consistent gains across all 4 frameworks × 7 backbones.**
  E.g., ReasoningBank ALFWorld with Qwen3.5-397B-A17B: Init-only
  ≈ 84% → ExpWeaver ≈ 91%. SkillRL ALFWorld with GPT-5.2: similar
  pattern. Always-on never beats Init-only.
- **RL gains compound.** Qwen3-4B on ALFWorld: w/o experience 61.7,
  Init-only 70.6, Always-on 67.9, ExpWeaver **75.9** — Always-on
  underperforming Init-only is itself a signal that excessive
  retrieval introduces interference.
- **Usage-pattern analysis (Table 1):** ALFWorld induces 2.17
  retrievals/sample (Qwen3-32B, ReasoningBank); WebShop 0.13;
  HotpotQA 0.42. Strong backbones (GPT-5.2, Qwen3.5-397B-A17B)
  retrieve near zero on WebShop/QA but rise on ALFWorld.
- **Temporal pattern (Fig 5):** retrieval is concentrated at step 1
  (high-level planning) and resurges sporadically later — typically
  during ALFWorld object-search failures.
- **Causal ablation (Table 2):** removing experience at
  ExpWeaver-selected positions drops Qwen3-32B G-Memory ALFWorld
  from 83.33 → 75.37; random-position retrieval performs even worse
  (71.64). The position selection itself carries causal weight.
- **Entropy analysis (Fig 6):** retrieval invocations co-occur with
  token-entropy peaks during reasoning — direct mechanistic evidence
  that the gate tracks decision uncertainty.

## Critique / open questions

- **The intervention is a single prompt-engineering tweak.** That's
  a strength for transferability but a weakness for stress-testing:
  the agent must reliably interpret an "emit `[Retrieve]` when
  unsure" instruction, and the limitations section concedes weaker
  small-scale models may not. The reported gains on Qwen3-4B come
  through RL — the prompt-only version may not transfer cleanly to
  weaker backbones without training.
- **No joint optimization of construction and utilization.** The
  paper explicitly studies utilization with construction held
  fixed — a clean isolation, but it leaves open whether the optimal
  experience *representation* depends on the read-side policy, or
  vice versa. The limitations gesture at fully-automated memory
  design (citing M-Star, agentic-context-engineering) as the
  natural next step.
- **Token-entropy as uncertainty proxy is a correlation, not a
  mechanism.** The agent decides to retrieve before computing the
  next token, and entropy is measured during reasoning — the causal
  story (high uncertainty → emit `[Retrieve]`) is consistent with
  the data but not directly tested by intervention.
- **Cognitive-neuroscience framing is decorative.** The PFC-gating
  citations motivate the design but don't constrain it. The
  evaluation succeeds or fails on the engineering claim alone.
- **Safety angle is acknowledged but not evaluated.** Selective
  retrieval amplifies whatever is in the repository; if memory
  contains biased or stale content, the gate is now also a
  vulnerability surface (own cited follow-up: zhao2026safety,
  shao2025misevolve).

## Trust signals

- **Credibility:** 4 — Harbin Institute of Technology (a strong NLP
  lab) with Huawei; arXiv preprint, not yet peer-reviewed; no code
  URL located. Unusually broad validation (4 frameworks × 7 backbones
  × 3 environments) plus causal/entropy ablations gives it strong
  empirical footing; held below 5 only by the absence of peer review
  and a located public artifact.

## Follow-up

**Relevance: 5** — directly seeds the new concept
[[concepts/selective-memory-retrieval]] (decision-time read-policy,
uncertainty-gated retrieval) and materially extends
[[concepts/agent-native-memory]] by exposing the read-side policy
axis that note's existing implementation guidance leaves implicit.

**Connections to our own implementation.** This project's `/digest`
and `/iterate` skills today follow an *initialization-only* read
pattern — relevant concepts and recent notes are loaded at the
start of an invocation and then static. ExpWeaver suggests this is
the wrong default. Two concrete porting ideas:

1. `/digest` could let the WebSearch sub-agent emit a "consult
   prior concepts" trigger mid-query, rather than dumping all of
   `concepts/` into its initial context.
2. `/iterate`'s implementer sub-agent could emit a "consult related
   literature" trigger when its current proposal stalls, instead of
   always loading the top-5 nearest lit notes upfront.

**Strong pairing with [[literature/papers/kamelhar2026gsar]]**
(ingested same batch): GSAR is the write-side typology (how to
*structure* what is grounded); ExpWeaver is the read-side policy
(when to *consult* what is stored). The two together cover the
full read/write split of agent memory architecture and should be
co-cited in the upcoming knowledge-organization-for-research-agents
MoC.

**Suggested next reads** from this batch:
- SkillOS (`ouyang2026skillos`, pending) — the curator/executor
  split is a *write-side* analogue to ExpWeaver's read-side gating.
- SkillRet (`cho2026skillret`, pending) — empirical anchor on
  what skill libraries look like at scale; informs whether the
  selective-retrieval pattern needs additional infrastructure.
