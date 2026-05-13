---
kind: paper
title: "SkillOS: Learning Skill Curation for Self-Evolving Agents"
authors: ["Siru Ouyang", "Jun Yan", "Yanfei Chen", "Rujun Han", "Zifeng Wang", "Bhavana Dalvi Mishra", "Rui Meng", "Chun-Liang Li", "Yizhu Jiao", "Kaiwen Zha", "Maohao Shen", "Vishy Tirumalashetty", "George Lee", "Jiawei Han", "Tomas Pfister", "Chen-Yu Lee"]
year: 2026
venue: "arXiv 2605.06614"
url: "https://arxiv.org/abs/2605.06614"
source: "raw/papers/ouyang2026skillos.pdf"
added: "2026-05-11"
relevance: 5
status: read
related_experiments: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/selective-memory-retrieval]]"
tags: [skill-library, curation, self-evolving-agents, rl, grpo, procedural-memory, markdown-skills, curator-executor-split]
---

# SkillOS: Learning Skill Curation for Self-Evolving Agents

## TL;DR

SkillOS pairs a *frozen agent executor* (that retrieves and applies
skills) with a *trainable skill curator* (that updates an external
SkillRepo via `insert/update/delete` operations on markdown skill
files). The curator is trained with GRPO under a composite reward on
**grouped task streams** — earlier-task trajectories curate skills
that are evaluated by their utility on later related tasks in the
same group. This turns the central problem of skill curation from a
heuristic-driven process into a learnable long-horizon policy, and
yields the counterintuitive result that an 8B trained curator
outperforms Gemini-2.5-Pro used directly as the curator.

## Claims

- **Skill curation, not skill representation, is the bottleneck.**
  Holding the experience-construction pipeline of an existing
  framework (ReasoningBank, MemP) fixed and only changing what
  curates the skills produces consistent gains; conversely, swapping
  skill representations (insights vs workflows vs skills vs hybrid)
  under fixed heuristic curation produces only modest gains.
- **Long-horizon curation policies are learnable.** Grouped task
  streams + executor-grounded reward converts the indirect/delayed
  feedback problem (curation decisions only show value on *later*
  related tasks) into a usable GRPO signal.
- **Trained 8B curator > frontier curator on the same executor.**
  SkillOS-gemini (Gemini-2.5-Pro as curator) is consistently beaten
  by the trained 8B Qwen3 curator. The "curator-executor mismatch"
  finding: stronger curator reasoning produces skills miscalibrated
  to the executor's actual capacity and usage patterns.
- **The trained curator transfers across executors and tasks.** The
  same 8B curator trained on ALFWorld lifts WebShop and reasoning
  performance; cross-task transfer from reasoning to agentic tasks
  is particularly strong (more abstract strategies generalize).
- **Curator behavior evolves over training in a recognizable pattern.**
  Early: insert dominates (blindly add new skills). Mid: update grows
  (refine existing). Late: small but rising delete (prune). This is
  the natural progression of a knowledge-organization policy that
  learns when to stop expanding and start consolidating.

## Methods

- **Skill format.** Each skill is a single Markdown file. YAML
  frontmatter has two mandatory keys (`name`, `description`); the
  body has suggested sections (`# Workflow`, `# When NOT to Use`,
  `# Prerequisite Constraints`) plus emergent sections the curator
  invents over training.
- **Two-agent modular design.**
  - **Agent Executor** (`π_L`, frozen): retrieves top-k skills via
    BM25 against the current task, runs ReAct-style trajectory.
  - **Skill Curator** (`π_S`, trained): after each task, observes
    `(trajectory, self-judged correctness, retrieved skills)` and
    emits structured function calls
    `{insert_skill, update_skill, delete_skill}` that mutate the
    SkillRepo.
- **Training instance = group of related tasks.** Tasks are tagged
  by Gemini-2.5-Pro with attributes (topic, skills, concepts,
  heuristics, pitfalls); grouping uses a soft-Jaccard phrase
  similarity with a dependency gate (shared foundation, shared
  reasoning, not-near-duplicate, progression, forward-curriculum).
  Within a group, earlier-task curations are graded by later-task
  success.
- **Composite reward**: `r = r_task + λ_f·r_fc + λ_u·r_cnt +
  λ_c·r_comp`.
  - `r_task` — average success over remaining tasks in the group
    after the curation.
  - `r_fc` — fraction of curator-emitted function calls that are
    valid and execute.
  - `r_cnt` — Qwen3-32B LLM-as-judge score on abstraction,
    reusability, actionability, faithfulness of curated content.
  - `r_comp` — compression vs. raw trajectory length, discourages
    verbatim trajectory copying.
- **GRPO optimization** (no KL term, to encourage exploration);
  advantage = within-rollout normalized reward.

## Results

- **ALFWorld average SR** (Qwen3-8B executor): No-memory 47.9 →
  ReasoningBank 55.7 → SkillOS-base 53.1 → SkillOS 61.2
  (+13.3 absolute over no-memory; +5.5 over best baseline).
- **Same with Gemini-2.5-Pro executor**: No-memory 66.4 →
  ReasoningBank 71.4 → SkillOS-gemini (frontier curator) 79.3 →
  SkillOS (trained 8B curator) **80.2**. Trained 8B beats Gemini
  curator.
- **Steps efficiency**: SkillOS reduces ALFWorld interaction steps
  by 2.2–3.1 per episode vs. no-memory; consistently fewer than all
  memory baselines.
- **Reasoning** (AIME24+AIME25+GPQA avg, Qwen3-8B executor):
  no-memory 69.6 → SkillOS 73.8.
- **Generalization** (Fig. 3): off-diagonal entries (train on task A,
  test on task B) almost all positive vs no-memory baseline.
  Reasoning-trained curator transfers best to agentic tasks.
- **Behavior dynamics** (Fig. 4): operations rebalance from
  insert-dominant (~75%) at training step 1 to update-dominant
  (~50%+) by step 50; delete grows from ~5% to ~15%.
- **Content dynamics** (Fig. 5): early skills contain "advanced
  guidance / tips / recommendations" sections (verbose, weak
  operational value); later skills contain "retry logic /
  alternative paths / special considerations" (failure-handling,
  conditional branches). Repository shifts from task-specific
  skills to meta-strategy skills (verification, fallback, systematic
  search) representing ~29% of late-training skills.
- **Skill usage** (Fig. 6): SkillOS invokes skills on 100% of
  evaluations (vs SkillOS-base 87.9%), uses larger fraction of
  repo (61.2% vs 53.6%), but fewer skills per example (1.95 vs
  2.24). More precise selection.
- **Ablations** (Table 3): removing grouped task streams hurts most
  (61.2 → 57.3). Content-quality reward worth +2.6; compression
  reward worth +1.2.

## Critique / open questions

- **Read-side policy is the obvious next axis, and SkillOS doesn't
  attack it.** Retrieval is fixed-schedule BM25 top-k at every step
  — exactly the *always-on* pattern that
  [[literature/papers/zhao2026expweaver]] (ingested same batch)
  shows underperforms decision-time-gated retrieval. The two papers
  are concurrent (May 7 and May 8 2026) and don't reference each
  other; combining curator-learning (write-side) with
  uncertainty-gated retrieval (read-side) is the obvious next
  experiment.
- **Skill1 (Shi et al., 2026, [arXiv
  2605.06130](https://arxiv.org/abs/2605.06130)) takes the inverse
  architectural bet** — same date (2026-05-07), same theme, but
  unifies skill selection / application / extraction under a single
  RL policy instead of curator/executor separation. SkillOS doesn't
  cite Skill1 (concurrent submission). The architectural contrast
  matters: SkillOS's separation is what enables the curator to
  transfer across executors (the headline generalization result);
  a unified policy in Skill1's setup can't separate those two
  concerns. Worth a direct head-to-head, which neither paper does.
- **Simplified skill format (single Markdown file)** drops
  Anthropic's full SKILL.md format (multi-file, scripts, hierarchical
  composition). The paper flags this as a limitation. For a research
  project, this is actually the right simplification — single-file
  is what works in a git-tracked, grep-able knowledge graph.
- **Curator-executor mismatch finding deserves more attention than
  it gets in the paper.** The result that Gemini-2.5-Pro curator
  *underperforms* an 8B trained curator on the same executor is
  buried in §4.3. The deeper claim — that *good knowledge curation
  is reader-specific*, not absolutely good or bad — is the more
  generalizable lesson and probably warrants a paper of its own.
- **Training reward is heavy on engineering choices** (`λ_f`,
  `λ_u`, `λ_c` hand-tuned; Qwen3-32B judge for content quality is
  itself a moving target). Reproducing the recipe cleanly outside
  Google Cloud AI Research will be non-trivial.

## Follow-up

**Relevance: 5** — directly seeds the new concept
[[concepts/skill-library-lifecycle]] (insert/update/delete as a
learned policy operating on a markdown-file skill repository) and
provides material new evidence for [[concepts/agent-native-memory]]
(literal markdown-file substrate, file-I/O operations) and
[[concepts/hybrid-model-backends]] (curator/executor split with
strong calibration evidence: stronger curator alone isn't better
— it must match the executor's needs).

**Connections to our own implementation.** This project IS already a
skill-library-shaped knowledge graph (`concepts/`, `literature/`,
`experiments/` are all markdown-file libraries curated through
human-driven `/ingest` and `/lint` operations). Three takeaways for
evolving it:

1. **The behavior-evolution pattern is normative for `/lint`.**
   SkillOS's empirical insert→update→delete progression
   (Fig. 4) is exactly the curation arc a healthy knowledge graph
   should follow: early stages dominated by new concept seeding,
   maturing into refinement and pruning. `/lint` today catches
   orphan concepts and dead wikilinks but doesn't yet surface
   "concepts that should be merged" or "concepts that should be
   retired" — that's the next axis.

2. **Curator-executor mismatch generalizes.** For a research-memory
   project, the "executor" is *future-Claude consulting the graph*.
   Notes optimized for human reading may be miscalibrated for
   future-Claude's retrieval patterns. The frontmatter
   `relevance:` and `related_concepts:` fields are the project's
   current calibration mechanism; SkillOS suggests those fields
   should be tuned by *whether downstream sessions actually use
   the note*, not by curator judgment alone.

3. **Grouped-task curation maps to ingest cycles.** SkillOS's
   key trick — grouping related tasks so curation has dense reward
   — corresponds in this project to ingesting *clusters* of related
   papers in one cycle (as this 2026-05-11 batch is doing).
   Concepts seeded by paper N can be validated by whether they
   absorb papers N+1, N+2 cleanly. The MoC pattern (≥5 concepts on
   a theme) is the project's version of this.

**Strong pairing with [[literature/papers/zhao2026expweaver]]** —
together they cover the full read/write axes of agent memory:

| Axis | Paper | Mechanism |
|---|---|---|
| Read-side: when to consult memory | ExpWeaver | `[Retrieve]` trigger on reasoning uncertainty |
| Write-side: how memory evolves | SkillOS | Trained curator with `{insert, update, delete}` over markdown files |

Neither paper does the other side. Combining them is the obvious
next system: a *trained read-side gate* on top of a *trained
write-side curator*. This is the strongest concrete experiment idea
emerging from this batch.

**Suggested next read** from this batch:
- SkillRet (`cho2026skillret`, pending) — 18K-skill empirical
  retrieval benchmark; tests whether the curator-executor mismatch
  finding holds at population scale.
