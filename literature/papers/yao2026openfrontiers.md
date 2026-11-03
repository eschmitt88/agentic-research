---
kind: paper
title: "OpenFrontiers: A Collection of Open Research Problems and a Benchmark for Scientific Rediscovery"
authors: ["Jiarui Yao", "Ruida Wang", "Tong Zhang"]
institutions: ["University of Illinois Urbana-Champaign"]
year: 2026
venue: "arXiv 2610.37739 (cs.AI), v1 2026-10-27, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.37739"
code_url: "https://huggingface.co/collections/FlippyDora/openfrontiers"    # dataset + benchmark collection named in the abstract; the grading harness (evalkit-0.3) is not separately linked
citations: null
source: "raw/papers/yao2026openfrontiers.pdf"
added: "2026-11-03"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/information-firewall]]"
  - "[[concepts/web-grounded-literature]]"
  - "[[concepts/programmable-evaluator-oracle]]"
tags: ["rediscovery-benchmark", "open-problems", "lean4", "formal-grading", "two-checker", "no-llm-judge", "closed-book", "frozen-retrieval", "live-web", "retrieval-leakage", "solution-retrieval", "url-fetch-audit", "lineage-exclusion", "training-cutoff", "time-anchored-holdout", "known-good-preflight", "known-bad-preflight", "continuation-protocol", "early-stopping", "claude-code", "codex-cli", "sandbox", "mathematics"]
---

# OpenFrontiers: A Collection of Open Research Problems and a Benchmark for Scientific Rediscovery

## TL;DR

The authors built a corpus of 12,151 open-problem records from 62
sources, and a 200-item Lean 4 benchmark cut from it. Rediscovery-110
holds questions that were open into 2026 and now have a located
solution. Open-90 holds problems that are still open. The evaluated
subset, HQ-50, has 40 solved and 10 open items. Seven agents ran each
item once per condition: closed-book (A), frozen retrieval with a
pre-2026-01-06 corpus (B), and live web (C). Each ran under two
protocols, "native" (the session ends when the agent stops) and
"continuation" (the session is resumed until the 3-hour budget is
spent). A submission counts only if two independent Lean checkers
accept it with the reference answer. No model acts as judge.

Three results matter for this graph:

1. **Closed-book rediscovery is rare.** Agents solve 0–3 of 40 natively
   and 0–7 under continuation. Across all 14 A and B runs, 44 of the 53
   solves are on three short, human-solved, importance-3 items.
2. **Frozen retrieval does not help.** B beats A in two runs and trails
   it in others. Literature from before the solution adds nothing
   measurable.
3. **Live web measures retrieval, not rediscovery.** C solves 1–27
   items. 167 of the 181 C solves came from attempts that fetched a
   located solution source. The paper establishes this by URL matching:
   every fetched URL is normalized and compared with the item's
   solution sources. It does not inspect traces for copying.

The benchmark also excludes, per model, items whose solution came from
that model's own vendor ("lineage"). For 55 of the 110 solved items, the
solution materials credit an AI system with the key mathematics. So a
post-cutoff date alone does not make an item clean.

## Claims

- A research problem has no reference answer. So a formerly open problem
  can be graded only against a solution that appeared after the model was
  frozen, and the located solution must stay evaluator-only.
- A post-cutoff date is "necessary but not sufficient". A model whose
  lineage produced the solution "has in effect seen it". The protocol
  therefore excludes such items per model.
- Closed-book is the main condition. Live web is "a leakage reference
  rather than a rediscovery measure". The C–A gap "estimates how
  retrievable the answers are".
- Agents give up early. Under continuation they solve more, so harness
  and prompting have to induce persistence.
- Status labels in the corpus are the source's claim as of a date, not a
  local check. Uncertain items are labelled `unassessed`, not `open`.

## Methods

- **Corpus.** 12,151 records from 62 contributing sources (71
  registered), snapshot 2026-10-10. 93.4% are mathematics. One schema
  holds a verbatim statement, field and MSC 2020 codes, status-as-of date,
  provenance, licence and any Lean 4 attachment (1,946 records). The Lean
  attachments are display-only and not compiled at export.
- **Benchmark construction.** Solved candidates come from source-list
  status changes, 2026 arXiv preprints and public AI-solution releases.
  Nine sweeps gave 291 solved leads; a second reviewer then tried to
  overturn each one. Admission needs a public question before the
  solution, the same quantifiers and scope, no earlier solution, and no
  restatement or withdrawn paper. Evidence tiers: A (85, exact
  certificate recomputed or full Lean proof compiled locally), B (21,
  corroborated, not re-verified), C (4, pending). All located solution
  dates are after 2026-01-05.
- **Lean statements.** Each item is one theorem `main` pinned to Lean
  4.33.1, Mathlib `0df444a` and Formal Conjectures `89294ea`. Value
  questions take the form `answer(sorry) ↔ P`, so the agent must supply
  the answer and the proof. Two AI reviewers per item: one checks
  fidelity to the source, one tries to prove or refute the statement
  cheaply through formalization loopholes. Up to two repair rounds.
  Final verdicts: 139 both pass, 38 with one minor issue, 23 with two
  minor issues, none must-fix.
- **Leakage controls in the statements.** Solved files carry no answers,
  upstream declaration names or live status links. Wikipedia links point
  to a pre-solution revision. A second pass removed semantic hints from
  47 files (declarative titles became questions, background sentences
  that pointed to the route were deleted). Items are renamed P001–P200 in
  a salted order.
- **Grader checks before use.**
  - Known-good: the reference statement with the answer filled in
    compiles for all 110 solved items. For 50 solved items, `main` was
    proved in this environment from public Lean proofs or recomputed
    certificates.
  - Known-bad: automated probes run `main` with the correct and a wrong
    answer. Automation (`decide`, `simp`, `omega`, `aesop`) proves or
    refutes none of the 200.
  - Grader self-test: accept and reject cases, including a compile-time
    file write that passes keyword bans.
  - The comparator's 16 official regression attacks pass. The authors
    ported the comparator to Lean 4.33 and fixed a gap: without counting
    structures referenced by projections, "its own projection attack
    would pass".
- **Grading.** Static rules allow only Mathlib imports. They reject
  `sorry`, extra axioms, `native_decide`, metaprogramming and syntax
  extensions. Checker 1 compiles the reference statement and the
  submission in separate networkless jails, replays the submission in
  the kernel, compares elaborated definitions and the statement of
  `main`, and allows only `propext`, `Classical.choice` and
  `Quot.sound`. Checker 2 (leanprover/comparator) exports both projects
  with `lean4export` and replays the whole export, without loading files
  the submission may have written. Solved = both accept with the
  reference answer. A checker disagreement counts as unsolved. A proof of
  the opposite answer, or an accepted open item, freezes the item for
  statement review. Results are counts with denominators, never accuracy,
  because a yes/no guess scores 50%.
- **Agent-facing pre-check.** An MCP tool, `check_submission`, applies
  the static rules, compares definitions and the statement, compiles, and
  lists axioms. It does not consult the reference answer.
- **Sandbox.** Fresh root filesystem per attempt, unprivileged user,
  private namespaces, seccomp, cgroups (5 CPU, 24 GB, 4,096 processes,
  10 GB disk). The agent CLI's only network exit is a model-API gateway.
  The gateway pins the model and strips server-side web tools in A and B.
  Answers, the evaluator-only package and the grading workspace are not
  visible. Status-revealing source lines are removed from the problem
  file.
- **Access conditions.** The conditions differ only in retrieval.
  - A: no retrieval.
  - B: search and fetch over arXiv, Wikipedia, OpenAlex and MathOverflow,
    limited to content public before 2026-01-06. arXiv returns the latest
    pre-cutoff version. Wikipedia and Stack Exchange return the last
    pre-cutoff revision. OpenAlex shows only journal or proceedings
    records, because merged preprint records can carry post-cutoff
    abstracts. Search indexes today's text, so every hit is re-checked
    against the displayed historical text.
  - C: live web through a proxy that refuses private addresses. Every
    fetched URL is logged, normalized (arXiv ID, DOI, repository, problem
    page) and compared with the item's located solution sources.
- **Models and harness.** Claude Code runs Claude Sonnet 5.5, Kimi-K3,
  GLM-5.3, DeepSeek-V4.1-Flash and Qwen3.8-27B (non-Anthropic models
  through a bridge). Codex runs GPT-6.1-sol and GPT-6-luna. Each attempt
  gets 3 h wall-clock. One attempt per item, condition and protocol
  (pass@1): 7 models × 2 protocols × 3 conditions × 50 items = 2,100
  attempts. No seeds or repeats.
- **Continuation protocol.** A session that ends with at least a minute
  left is resumed (`claude --resume`, `codex exec resume`). The resume
  message states the remaining time, the submission status and concrete
  tool work to try. An attempt ends as stalled after 20 consecutive
  rounds without tool use (or 30 of 40).
- **Clean subset.** Per model, remove items whose key mathematics came
  from the model's lineage and items with solution evidence on or before
  the model's cutoff. At a 2026-06-30 cutoff that leaves 15 items for
  the OpenAI models, 30 for Claude Sonnet 5.5, and 32 for unrelated
  models. The open-weight models' cutoffs are their release dates. Only
  5 items are clean for DeepSeek-V4.1-Flash.

## Results

- **Table 3, solved / 40 (clean / eligible):**

  | Model | A nat | B nat | C nat | A cont | B cont | C cont |
  |---|---|---|---|---|---|---|
  | Claude Sonnet 5.5 | 0 (0/30) | 1 (1/30) | 6 (3/30) | 4 (3/30) | 3 (3/30) | 24 (18/30) |
  | GPT-6.1-sol | 2 (2/15) | 2 (2/15) | 15 (6/15) | 7 (4/15) | 5 (3/15) | 27 (12/15) |
  | GPT-6-luna | 0 (0/15) | 0 (0/15) | 4 (3/15) | 2 (2/15) | 0 (0/15) | 16 (6/15) |
  | Kimi-K3 | 2 (2/33) | 2 (2/33) | 15 (12/33) | 3 (3/33) | 2 (2/33) | 15 (13/33) |
  | GLM-5.3 | 2 (2/16) | 2 (2/16) | 15 (8/16) | 0 (0/16) | 2 (2/16) | 13 (5/16) |
  | DeepSeek-V4.1-Flash | 3 (1/5) | 3 (1/5) | 13 (0/5) | 3 (1/5) | 3 (1/5) | 15 (2/5) |
  | Qwen3.8-27B | 0 (0/19) | 0 (0/19) | 1 (0/19) | 0 (0/19) | 0 (0/19) | 2 (0/19) |

- **Closed-book.** The 53 A and B solves across 14 runs fall on eight
  items. Three importance-3 items, all solved by humans through short
  constructions, account for 44 of them. Of the 21 importance-5 items,
  one was solved, once.
- **Frozen retrieval.** The paper says B beats A in two runs (by one and
  two items) and trails it in three. By my count from Table 3 it trails
  in four (Claude cont, GPT-6.1-sol cont, GPT-6-luna cont, Kimi-K3 cont).
  Either way there is no systematic gain.
- **Live web.** C totals 181 solves (69 native, 112 continuation). 167
  of them came from attempts that fetched a located solution source
  (preprint, repository or problem page), often one with a public Lean
  proof to adapt. 34 of the 40 solved items have public Lean proofs. In
  one example item with a public prover-agent proof, A and B solved it
  in 0 of 28 attempts and C in 11 of 14, each after fetching the problem
  page or its forum thread, which links to that proof. No model solved
  all 40 even with live web.
- **Early stopping.** Natively, Claude Sonnet 5.5 and the GPT models end
  most attempts within minutes (median 0.3–4.1 of 180 min). Claude
  Sonnet 5.5 native A left a formal submission in 3 of 50 attempts.
  Under continuation most agents use nearly the full budget, but most
  Claude Sonnet 5.5 attempts stall (41 of 50 in A). Pooled over models,
  continuation raises A from 9 to 19 solves, B from 10 to 15 and C from
  69 to 112.
- **Open items.** Table 7 lists three open-item acceptances "by both
  checkers, before review", all under continuation with live web
  (GPT-6.1-sol, GLM-5.3, DeepSeek-V4.1-Flash). The review outcome is not
  reported. The intro says no open problem was solved without live web.
  The conclusion says models "can solve some open problems".

## Critique / open questions

- **Against the digest framing: at framing.** "≤7/40 closed-book" is
  right: 7 is the best continuation run, and the native best is 3. The
  quote is the abstract's. "Mostly reflects retrieval" rests on 167/181,
  which is in the body. Two things the digest left out:
  1. **The retrieval attribution is URL co-occurrence, not trace-proven
     use.** An attempt counts if it fetched a URL matching a located
     solution source, including a problem page that links to the proof.
     That is wang2026search's exposure tier, which that paper found
     over-reports. The paper does not report how many C attempts fetched
     a source and still failed. It also does not check whether the 14
     non-fetch C solves differ from A. The A-vs-C gap on the same items
     is the stronger evidence, and it is large (pooled 9 → 69 native,
     19 → 112 continuation).
  2. **The lineage exclusion is the most transferable idea**, and the
     digest does not mention it. Half the solved items have an AI-credited
     solution, and clean counts move a lot. GPT-6.1-sol's continuation C
     score drops from 27/40 to 12/15. DeepSeek's native C drops from 13 to
     0/5.
- **"Clean datum" holds, with small n.** Same items, same sandbox, three
  retrieval arms, formal two-checker grading. But it is one attempt per
  item, no repeats, and 40 items. Clean denominators differ by model
  (5 to 33), so clean counts are not on a common item set. Model ranks on
  clean A or B rest on differences of one to three items.
- **B ≈ A supports HCE §1's technique exemption, weakly.** Pre-solution
  literature gave no lift here. It also did no harm. That fits §1's line
  that technique-level retrieval is fine. But these items need a new idea,
  and the solving rate is near the floor, so a null here says little about
  tasks where technique lookup matters (e.g. ML engineering).
- **Frozen retrieval is harder to build than a date filter.** Indices
  serve today's text, and merged preprint records carry later abstracts.
  The authors re-check every hit against the historical revision and drop
  OpenAlex preprint records. A downstream "pin retrieval to a frozen
  corpus" has to do the same.
- **The open-item contradiction is unresolved.** Three kernel-accepted
  open items are listed with no review outcome, and the text says both
  "no open problem was solved without [live web]" and "models can solve
  some open problems". The paper's own prior is that a statement error is
  likelier than new mathematics. All three came with live web.
- **The grader preflight is the strongest part.** It has both directions:
  a known-good replay of 50 public proofs, plus known-bad probes, a
  self-test with a file-write attack, and 16 regression attacks. The
  regression set found a real gap in the stock comparator (projection
  attack), which was patched. A do-nothing submission alone would not
  have found it. This is a direct datum for the pending known-bad
  preflight proposal and for the reviewers' known-good addition.
- **The continuation protocol is not a redirect arm.** It resumes a
  session the agent ended, with a budget reminder. It does not pivot
  strategy on a stall, so it does not test redirect vs halt. It does show
  that an agent's own stop is a weak signal: closed-book solves double
  pooled (9 → 19) when the stop is overridden.
- **Domain.** Mathematics with formal proof, not ML research. The
  information-firewall note's "ML-research tasks untested" gap stays open.
  Curation used AI agents heavily (sweeps, reviews, statements, harness,
  scripts), as the paper discloses.
- Table 3's "†" on Qwen3.8-27B's native C clean count is not explained in
  the text.

## Trust signals

- **Credibility:** 3.
  - For: UIUC group with a senior ML theorist (Tong Zhang). Dataset and
    benchmark released on Hugging Face. Fully formal grading by two
    independent checkers with no LLM judge. Grader validated in both
    directions. Detailed sandbox, gateway and retrieval specification.
    Per-run end reasons and submission counts (Table 7). Counts are
    reported with denominators. Honest about limits (C as leakage
    reference, statement error as the likelier explanation for open
    acceptances).
  - Against: not peer reviewed, one week old, no citations. One attempt
    per cell, no seeds, 40 scored items. The retrieval attribution is
    URL-level. The open-item review outcome is missing, and the text
    contradicts itself on it. One small count mismatch (B trails A in
    "three" runs; Table 3 shows four). Harness code (evalkit) not linked.
  - Not 4 because of single-attempt cells and the unresolved open-item
    claims. Not 2 because the grading and access-condition design are
    unusually careful and the data are released.

## Follow-up

- **Relevance:** 4. It is the first same-item, same-harness measurement
  in this graph of closed-book vs frozen vs live retrieval on a
  rediscovery task. It adds a lineage-exclusion rule to the time
  boundary and a two-direction grader preflight with a found bug. It adds
  material evidence to:
  - [[concepts/hce-evaluation]]: §1's retrieval clause, measured as an
    arm gap with a URL-fetch audit; the grader preflight.
  - [[concepts/information-firewall]]: lineage as part of the time
    boundary; a dated retrieval corpus.
  - [[concepts/web-grounded-literature]]: what "pin retrieval to a frozen
    corpus" takes in practice.
  Not 5 because it is mathematics, one attempt per cell, and the
  retrieval attribution is exposure-level.
- **Bearing on the pending known-bad preflight proposal:** a second
  attestation that adversarial reject cases catch checker gaps that a
  floor probe would not. Here the stock checker's own attack suite had a
  hole after porting.
- **Cheap local audit for downstream HCE projects with retrieval:** the
  paper's C-condition audit is implementable as is. Normalize every
  fetched URL in the trajectory and match it against a list of the task's
  known solution sources (leaderboard, Kaggle discussion, published
  write-ups). Log hits to `log.md`, as §1 asks. Report the pass-rate gap
  between a retrieval-off and a retrieval-on arm, not just the hit rate
  (ludwig2026shortcutting's reading).
- **Related in the graph:** [[literature/papers/wang2026search]]
  (exposure vs answer tiers), [[literature/papers/wang2026evobrowsecomp]]
  (tool-free ablation as contamination audit),
  [[literature/papers/ludwig2026shortcutting]] (pass-rate gap as the
  firewall's value), [[literature/papers/chen2026when]] (known-answer
  pre-flight).
- **Candidates** (cited, not yet in the graph):
  - Liu et al. 2026, "Danus: Orchestrating mathematical reasoning agents
    with fact-graph memory" (arXiv 2607.06447).
  - Adamczewski 2026, "OEIS Open: How many conjectures can language models
    turn into theorems?" (arXiv 2608.11941).
  - Petrov et al. 2025, "Proof or bluff? Evaluating LLMs on 2025 USA Math
    Olympiad" (arXiv 2503.21934). Final-answer grading overstates.
