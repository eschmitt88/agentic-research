---
kind: paper
title: "Invariant-Aware Harness Enforcement to Protect LLM Agents from Silent API Failures"
authors: ["Sajib Acharjee Dip", "Brian Ouyang", "Lala Motlhabi"]
institutions: ["Virginia Tech", "Genentech"]
year: 2026
venue: "arXiv 2611.20595 (cs.CL), v1 2026-11-16, preprint (9-page main text + 16-page appendix)"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.20595"
code_url: "https://github.com/Sajib-006/silentguard"    # Apache-2.0 per the paper; dataset at huggingface.co/datasets/Sajib-006/AgentOrchBench; neither opened at ingest
citations: null
source: "raw/papers/dip2026invariantaware.pdf"
added: "2026-11-24"
relevance: 4
credibility: 4
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/refusal-cost-symmetry]]"
  - "[[concepts/evidence-gated-completion]]"
tags: ["silent-api-failure", "silent-tool-failure", "harness-enforcement", "warn-vs-withhold", "advise-vs-block", "metamorphic-check", "count-identity", "spec-check", "parameter-allowlist", "value-swap", "learned-contracts", "outcome-monitors", "abstention", "permission-to-abstain", "auto-repair", "selective-prediction", "risk-coverage", "noticing-without-acting", "prediction-log", "hash-frozen-code", "agentorchbench", "toolmaze", "real-public-apis", "arxiv-api"]
---

# Invariant-Aware Harness Enforcement to Protect LLM Agents from Silent API Failures

## TL;DR

A *silent API failure* is an HTTP 200 with a well-formed, self-consistent
body that does not answer the request (a misnamed filter ignored, a page
silently capped). No check on the response alone can see it. SilentGuard
puts the decision in the harness. After each filtered count it sends extra
queries to the same API and tests an exact invariant (count with the
condition + count with its negation = count without it, at every
position). A result that fails twice is **withheld**, and the harness then
accepts only counts that passed the check. Failed *learned* checks (mined
contracts, which can false-alarm) only warn.

Five results matter for this graph:

1. **The same check, acted on five ways (7 open models, % wrong under
   faults; range by model in brackets).** No guard 74.0 (32–87). Warning
   70.1 (19–87). Warning + recovery tip 37.2. Warning + a prompt line
   permitting "cannot be determined" **18.2 (0–59)**. Enforce (only checked
   counts accepted, failed value left visible) 8.8 (0.5–24.8). Withhold
   **1.7 (0.0–3.7)**. Permission without evidence (caution prompt) does
   nothing: 73.3.
2. **Evidence and enforcement are each necessary (a 2×2 on one judge).**
   Judge without the check's queries: 71.7 advising, 70.8 blocking. With
   them: 71.2 advising, **6.1 blocking**. A response-only detector has
   nothing to see: Outcome Monitors' contracts fired on 3.5% of faulted
   runs, and with blocking they still left 70.9% wrong.
3. **Agents notice and answer anyway.** The judge with evidence flags 82.4%
   of faulted runs (13.9% of clean runs). The agent still reports the
   flagged count in **77.4%** of flagged runs. After a warning, stated
   confidence falls from 1.00 to 0.43, and 40% of the wrong answers mention
   the warning.
4. **It works by abstaining, and what it buys depends on the fault being
   visible to the check.** Correct answers under faults fall from 16.4% to
   2.4%; 95.8% of faulted runs end "cannot be determined". Clean accuracy is
   unchanged (84.9 vs 84.9). On six fault types written after the freeze,
   run through the harness with no model, wrong answers fall only from
   61.1% to 24.5%. Every remaining wrong answer is a fault the check
   cannot see. An auto-repair variant recovers more true counts but
   released **49 wrong answers marked as verified** on those unseen types.
5. **On real APIs the cheap code check does all the work.** On 9 public
   research APIs with one filter name deliberately misdocumented, wrong
   answers fall from 89.8% to 21.3% and correct ones rise from 3.5% to
   69.5%. The **spec check alone** (reject any parameter outside the API's
   published list) gives 21.0% wrong. The invariant check adds +0.3 points
   [−2.5, +3.2]. Here the gain is recovery, not abstention: the agent reads
   the allowed list and rewrites the query.

## Claims

- Silent failures are common in real services. 10 of 15 public research
  APIs answer a plausibly misnamed filter with HTTP 200 and the unfiltered
  count. Counting all three probed fault kinds (misnamed filter, typo in a
  filter value, page over limit), 11 of 15 show at least one.
- Fixes that leave the decision to the model (checklist, reviewer,
  Outcome Monitors, LLM judge) leave wrong answers near the unguarded rate.
  The reason is not that the fault goes unseen: models see it and answer
  anyway.
- Three ingredients are separable. **Evidence** (a test whose result shows
  the response cannot be right) is necessary. **Permission** to abstain
  lets some models act on it. Only **enforcement** in the harness is
  reliable across models.
- The harness's action should depend on how certain the failed check is:
  withhold on an exact check (estimated false-alarm rate 0), warn on a
  learned one (rule of succession, α̂ = 1/(n+2), cutoff τ = 1%).
- Harness builders should give the harness the final say over results that
  fail an exact check, and repair only fault types the check can confirm.
  API builders should reject unknown parameters and say when a page was
  cut. Benchmarks should include faults that repeat.

## Methods

- **AgentOrchBench (authors').** The agent works from a shell and issues
  one `curl` per turn against eight local REST services, reset before each
  task. 36 count tasks ask how many of 400 synthetic catalog records match
  two or three conditions. The catalog returns only `total_count`, so the
  agent cannot verify a filter by reading records. Faults are injected in
  the server, return HTTP 200, and **repeat on every call**. Six fault
  types are run (drop last filter, cap total, AND→OR, ignore field, ignore
  NOT, off by one). Two more (prefix match, consistent lie) keep the
  identity true by design. 2 seeds per cell. A second schema (retail
  orders, 30 tasks, 4 fault types) tests transfer. 30 list tasks test
  record-ID answers.
- **Checks (App. B).**
  - *Count identity:* 2k extra queries per filter call (at most 6 here).
  - *List checks:* every record meets the filter, no repeats, size equals
    the reported total.
  - *Spec check:* a request with a parameter outside the API's published
    list is stopped before it runs, and the agent gets an error naming the
    allowed parameters.
  - *Value-swap:* re-issue with the filter's value changed; an unchanged
    count means the filter was ignored.
  - *Learned checks:* Outcome Monitors' contract miner (presence, type,
    echoes, ranges, linear relations, date order), 5-fold by task.
- **SilentGuard's pipeline.** Pass → release. Fail → re-send the same call
  once. Fail again → withhold if the check is exact, warn otherwise. On
  counts this becomes an answer rule: only a count from a query that passed
  the check *and has at least as many conditions as any other query in the
  run* is accepted. Any other count is rejected twice, then the harness
  answers "cannot be determined". A result checklist is also added to the
  system prompt. The harness never repairs a value by itself.
- **Arms.** No guard, caution prompt, self-verification (the model is told
  to run the same every-position check itself), re-query, blind retry, LLM
  judge (gpt-oss-120b, sees the check's queries), checklist (after Sun et
  al. 2024), reviewer (after Ta et al. 2026, up to 5 loops), Outcome
  Monitors (Panthi & Abdelfattah 2026), the judge and OM with blocking, and
  harness-action ablations (warning, warning + tip, answer gate, Enforce,
  auto-repair). Prompts are quoted in App. C.
- **Other benchmarks.**
  - *Real public APIs:* 9 APIs (NCBI, UniProt, Europe PMC, GBIF, Zenodo,
    ChEMBL, arXiv, DataCite, iNaturalist) × 5 count tasks, with correct or
    one-filter-wrong documentation, 45 runs per model and arm. Four APIs
    were chosen before the method was frozen, five after.
  - *ToolMaze* (Zhu et al. 2026, external): 180 workflows never used in
    design, scored by its own judge. Only learned checks exist there, so
    SilentGuard only warns.
  - *τ-bench retail* with two silent read faults (App. J).
- **Models.** 7 open-weight models (Phi-4, gpt-oss-120b, Gemma-3-27B,
  Llama-3.3-70B, Llama-4-Scout, Llama-4-Maverick, Nemotron-3-Ultra) at
  temperature 0, max 2,000 output tokens per turn and 16 turns. Three
  frontier models (GPT-5.6 Sol, Terra, Luna) at provider default ran six
  AgentOrchBench arms and two real-API arms. They ran on Genentech's
  internal platform, were added after the planned experiments, had no
  predictions, and have no released logs or hash records.
- **Protocol.** Every planned experiment was written down with predictions
  before it ran. The code was frozen by file hash and runs refuse to start
  if it changed. **26 of 39 predictions held**, and the 13 failures are
  listed (Table 31). The method went through three versions, each tested
  on unseen ToolMaze workflows. τ was chosen after version 2, but any
  τ < 1.85% gives the same decisions.
- **Statistics.** The model is the unit. Mean ± SE over 7 models, t
  intervals, an exact sign-flip test over models (floor p = 0.016), and a
  2,000-resample task-cluster bootstrap for key differences.

## Results

- **Real-API probes (Tables 8, 11, 12).**
  - Misnamed filter → HTTP 200 with the unfiltered count on 10 of 15 APIs.
    Overcount by API: ChEMBL 691×, UniProtKB 83×, **arXiv 67×**, NCBI 24×,
    Europe PMC 19×, Zenodo 4×, GitHub 3×, DataCite 3×, GBIF 3×,
    iNaturalist 2×. ClinicalTrials.gov, OpenAlex, Crossref, openFDA and
    Semantic Scholar reject the unknown parameter.
  - Value-swap caught 26 of 28 ignored names.
  - Page over limit: 6 of 14 testable APIs return fewer records with no
    error. A typo in a valid filter's value is silently ignored by three
    APIs (Europe PMC, GitHub search, iNaturalist). The checks flagged 15 of
    15 such probes with 0 of 15 false alarms on the correct spelling.
  - Of 84 documented operations in 13 APIs, 61 allow an exact check and all
    84 allow the spec check. Search/list operations: 43 of 43. Get-by-ID: 5
    of 16. Writes: 0 of 5.
- **AgentOrchBench, all arms (Table 17, 7 open models, % of faulted runs
  wrong / correct / unknown; clean % correct).**

  | Arm | Wrong | Correct | Unknown | Clean |
  |---|---|---|---|---|
  | No guard | 74.0 | 16.4 | 9.6 | 84.9 |
  | Caution prompt | 73.3 | 16.4 | 10.3 | 86.1 |
  | Checklist / Reviewer / OM | 72.3 / 73.8 / 72.2 | ~16 | ~11 | 84.1 / 80.4 / 86.1 |
  | Blind retry / Re-query | 74.4 / 74.9 | ~17 | ~8 | 88.3 / 84.7 |
  | Self-verification | 56.2 | 14.2 | 29.6 | 76.6 |
  | LLM judge (with evidence, advising) | 71.2 | 17.4 | 11.4 | 86.5 |
  | Judge, no evidence + blocking | 70.8 | 16.7 | 12.5 | 83.7 |
  | OM + blocking | 70.9 | 16.6 | 12.5 | 86.5 |
  | Warning | 70.1 | 14.1 | 15.9 | 86.3 |
  | Warning + tip | 37.2 | 9.8 | 52.9 | 82.5 |
  | Warning + may abstain | 18.2 | 5.0 | 76.8 | 83.5 |
  | Enforce | 8.8 | 3.0 | 88.2 | 86.3 |
  | Judge + blocking | 6.1 | 3.5 | 90.4 | 81.7 |
  | Auto-repair | 3.0 | 57.3 | 39.7 | 83.3 |
  | **SilentGuard** | **1.7** | 2.4 | 95.8 | 84.9 |

  - SilentGuard has fewer wrong answers than no guard, each published
    method and blind retry on 7 of 7 models (p = 0.016). Bootstrap
    differences vs no guard −72.3 [−74.0, −70.5]; vs Enforce −7.1 [−8.5,
    −5.6].
  - Enforce vs withhold (App. E): of Enforce's 266 wrong answers, **130**
    come from agents re-querying until some other (looser) query passed the
    check, against 13 of 52 for SilentGuard, which removes the value.
- **Frontier models (Table 5, % wrong; clean in brackets).**

  | Model | None | Self-verify | Warn | Warn + abstain | Enforce | SilentGuard |
  |---|---|---|---|---|---|---|
  | GPT-5.6 Sol | 89.4 (98.6) | 2.5 (29.2) | 11.3 (77.8) | 2.3 (86.1) | 31.5 (100) | 0.0 (100) |
  | GPT-5.6 Terra | 62.3 (93.1) | 3.9 (15.3) | 53.9 (68.1) | 35.9 (76.4) | 34.0 (90.3) | 0.2 (98.6) |
  | GPT-5.6 Luna | 59.0 (84.7) | 6.9 (16.7) | 63.2 (69.4) | 45.1 (73.6) | 36.1 (87.5) | 1.4 (95.8) |

  - Self-verification "works" by refusing: clean accuracy falls to 20.4%
    (mean).
  - Enforce does worse on frontier models than on open ones (33.9% vs
    8.8%). With no logs, the authors cannot say why.
  - Real APIs, frontier: 57.8% → 8.9% wrong.
- **Real APIs, open models (Tables 4, 9, 24).** No guard 89.8 wrong (92.1
  correct with the right docs). Blind retry 89.5. Spec check alone 21.0 /
  70.8 correct. Value-swap alone 17.8 / 25.7 (mostly unknown). Typed calls
  (strict parameter schema, as in function calling) 28.6. Typed calls +
  SilentGuard 28.3. SilentGuard 21.3 / 69.5.
  - The gain is uneven: 38.6% wrong on the four APIs chosen before the
    freeze, 7.4% on the five after. Most remaining errors are on APIs whose
    filters live inside a query string (NCBI, UniProt, Europe PMC,
    **arXiv**), which neither the schema nor value-swap sees.
  - Wrong value format for a valid parameter (iNaturalist dates): no guard
    97.1, spec check 100.0, SilentGuard 20.0 wrong / 2.9 correct / 77.1
    unknown. Four of its seven wrong answers passed value-swap because the
    live total drifted between request and check.
  - Replaying the final queries of wrong answers on the six query-string
    APIs: the count identity would have caught 8 of 60.
- **Unseen fault types, harness only (Table 6; 216 tasks).** Wrong /
  correct / unknown: no guard 132/84/0, **SilentGuard 53/41/122**,
  auto-repair 102/79/35. Where the check fired, SilentGuard never released
  a wrong count. 49 of auto-repair's repairs were wrong and marked verified.
- **Lists (Table 26).** Wrong lists 49.3 → 9.1 (seen faults), 37.8 → 11.5
  (unseen). Correct lists on unseen faults 56.7 → 7.1. Clean 80.5 → 84.8.
- **Retail.** 78.6 → 11.0 wrong (Enforce 12.3, auto-repair 15.0). Clean
  accuracy is low in every arm (36.2 vs 38.5).
- **ToolMaze (Tables 4, 21–23).** Silent-fault success: SilentGuard 36.0
  vs Checklist + OM 32.7 (+3.3 [+1.8, +4.9], 5 of 7 models, p = 0.125).
  Clean success 52.5 vs OM 57.1 (−4.6). **Blind retry beats it: 43.7**,
  because half of ToolMaze's silent faults hit only the first call (60.4 vs
  47.6 on one-time faults; 27.0 vs 24.4 on lasting ones). Blind retry
  repeats bookings, payments and messages 2.36 times per run (0.24 for
  SilentGuard). Withholding on learned checks gave 32.1 vs 33.5 for
  warning.
- **Risk vs coverage (Fig. 2).** At a 25% fault rate: SilentGuard answers
  67% of tasks at 3.3% risk; no guard 90% at 24.6%; LLM judge 91% at 24.0%;
  judge + blocking 66% coverage; auto-repair 80% at 3.9% (on its own fault
  types).
- **Break-even cost of a wrong answer (Table 16, score = correct − λ ×
  wrong).** SilentGuard beats no guard for λ > 0.2 (counts, seen faults),
  0.6 / 1.9 (lists, seen / unseen), 0.5 (unseen counts). It beats
  auto-repair only for λ > 43.6 on seen count faults, but for λ > 0.8 on
  unseen ones.
- **Check budget (Table 30, harness only).** Last condition only: 88.0%
  caught, 8.8% wrong released. All positions: 97.2%, 0.0%. All positions,
  stopping at the first failure: same detection with 2.3 extra queries on
  faulted results. No budget raised a false alarm on clean runs. The
  consistent lie (an API that keeps the identity true on the checker's
  queries) is caught at most **2.8%**.
- **Cost (Table 27).** Per faulted run: 18.9 s, 3.8 tool calls, 9,055
  tokens, no extra LLM call. No guard: 9.2 s, 3,183 tokens. LLM judge:
  47.2 s, 6,287 tokens.
- **Fine-tuning (Table 28).** SFT to refuse (Phi-4-mini, Mistral-7B,
  trained on drop-last with the check shown) transfers to unseen fault
  types (6.7%, 8.7% wrong). SFT to repair does not (45.3%, 42.0%). RL with
  DAPO is in between (28.7%). Every trained model still fails the
  consistent lie at 100%. Training makes the model follow the check; it
  does not replace it.

## Critique / open questions

- **Against the digest entry (item 20): at.** The numbers are in the PDF
  and quoted correctly, but two framings overreach and the entry misses
  the paper's most importable controls.
  - **"A warning from the same check leaves 70.1% wrong"** omits the
    middle of the ladder. Warning + a one-line permission to abstain gives
    18.2%. So a prompt closes 52 of the 68 points between warning and
    withholding. Enforcement closes the last 16.5 and compresses the model
    spread from 0–59 to 0–4. The contrast is a gradient, not warn vs
    enforce.
  - **The 1.7% and 0.5% hold largely by construction.** On the six fault
    types the exact check was designed to see, the check fires and the
    answer rule then admits only checked counts. A wrong answer can only
    come from a missed detection or an unchecked route. The informative
    numbers are the unseen-fault run (61.1 → 24.5, harness only), the
    correct-answer loss (16.4 → 2.4) and the clean twin (84.9 = 84.9). The
    abstract does report the first and says "abstaining rather than
    recovering", so the paper's own framing is fair. The digest headline
    is not.
  - **"11 of 15 ... inflated up to 691×".** 11 is across three fault
    kinds; for the misnamed filter alone it is 10 of 15. The overcount is
    the misnamed filter's selectivity (the unfiltered total over the
    filtered one), not a property of the API's failure. Both depend on the
    client sending a wrong name. In the agent study that wrong name comes
    from documentation the authors altered on purpose.
  - **"Only the response changed"** is nearly true for Warning vs
    SilentGuard: both use the full check. SilentGuard also adds the
    re-check (inert, since faults repeat), the result checklist (inert
    alone: 72.3%) and the answer rule. The single-step contrasts are the
    judge 2×2 and Enforce vs withhold (value visible vs removed).
  - **The *Why* line's `/digest` angle is real but narrow.** The arXiv API
    silently ignores a misnamed parameter (67×). Its filters live inside
    the `search_query` string, though, the class that neither the spec
    check nor value-swap can see, and where most of SilentGuard's
    real-API residual sits. `/digest`'s own guard (select by ID prefix and
    datestamp, not by the API's filter) is the record-membership check of
    Table 1, applied informally.
  - No check-at-ingest hint was given. The PDF is the described work
    (title, arXiv 2611.20595v1, cs.CL, 2026-11-16, 3 authors). No wrong
    number or ID found.
- **Exact checks have a narrow domain.** The count identity assumes
  two-valued filters and stable totals. Nulls need a three-way partition,
  which is not implemented. Live totals drift, and that let four iNaturalist
  wrong answers through. The ignore-NOT fault breaks the check's own
  queries and fires on runs whose count may be right. Get-by-ID lookups
  mostly allow only the spec check, writes none. Where only learned
  checks exist (ToolMaze), SilentGuard reduces to "re-check, then warn",
  and the gain is +3.3 points, below blind retry.
- **The main benchmark is built for the check.** The catalog returns only
  a count and the faults repeat on every call. That makes retry and
  record-reading useless by design and makes abstention the only safe
  action. The paper says so ("Benchmarks should include faults that
  repeat") and supplies ToolMaze as the counterweight, where the
  ordering flips.
- **Real-API recovery comes from the spec check's error message.** The
  allowed-parameter list in the rejection is a discharge path. That is
  probably why the spec check beats strict typed calls (21.0 vs 28.6, a
  failed prediction, T1), but no arm isolates the message. This is our
  reading, not the paper's.
- **The frontier arms are the weakest evidence.** They are post hoc, with
  no predictions, no hashes and no logs. They use one vendor. Enforce's
  frontier result (33.9%) is unexplained. Self-verification's clean
  collapse (20.4%) suggests refusal, but it can't be audited.
- **Seven models put a floor of p = 0.016 on every sign-flip test**, so
  "7 of 7" is the strongest result the design can return. The authors
  read it as a count of agreeing models, which is right.
- **Retail clean accuracy is ~36% in every arm**, so wrong-answer cuts
  there are measured on tasks agents mostly fail anyway.
- **No Claude model is tested**, and the agent protocol (one `curl` per
  turn, no pipes, 1,500-character response view) is far narrower than a
  Claude Code session. Whether a strong Claude model acts on a warning the
  way Sol did (11.3%) or the way Terra and Luna did (54–63%) is open.
- **Consistent with neighbours in the graph.**
  - [[literature/papers/zhang2026are]]: agents state that the data cannot
    decide and still give the key (84%). An "insufficient data" option
    helped there, as permission to abstain does here, and covered absence
    rather than contradiction.
  - [[literature/papers/li2026who]]: qualify a validator before it may
    block. α̂ ≤ τ is the same rule with a counted false-alarm bound.
  - [[literature/papers/elkoussy2026agentltl]]: block-and-warn and a
    kill-switch hurt strong models where violations were locally
    recoverable. Here the fault is not recoverable by retry, and the
    harshest action wins.
  - [[literature/papers/liu2026silent]]: silent errors in an evaluation
    pipeline that each "produced a plausible number and raised no error".
    That is the same failure class one layer up.

## Trust signals

- **Credibility:** 4.
  - For:
    - The authors are at Virginia Tech and Genentech. Code (Apache-2.0)
      and the AgentOrchBench dataset (CC-BY-4.0) are released, though
      neither was opened at ingest.
    - The protocol is unusually disciplined. Predictions were written
      before each run and the code frozen by hash. 26 of 39 predictions
      held, and all 13 failures are reported and discussed, including
      ones against the method (blind retry, typed calls).
    - Controls separate evidence, permission and enforcement in matched
      cells. Six harness-action ablations share one detector.
    - There are four benchmarks, one of them external (ToolMaze, on
      workflows unused in design), plus real public APIs added after the
      freeze with the method unchanged.
    - The uncertainty is well reported: model-level SE, t intervals, a
      task-cluster bootstrap and exact sign-flip tests.
    - The limits section names the by-construction parts: consistent lie,
      prefix match, abstention instead of an answer, and blind retry
      winning on short faults.
  - Against:
    - Not peer reviewed, and a week old.
    - The main benchmark is the authors' own and favours the method by
      design (count-only responses, repeating faults).
    - The frontier and Nemotron control runs have no logs, hashes or
      predictions.
    - τ was chosen after seeing results, though it is shown to be
      insensitive.
    - Seven models limit the model-level tests.
  - Not 5: unreviewed, uncited, and the headline benchmark is
    author-built. Not 3: the code is released, the predictions are
    registered with failures reported, and the controls isolate each
    claim. That is well above most preprints in the graph.

## Follow-up

- **Relevance:** 4.
  - It is the cleanest matched evidence in the graph on *what the harness
    does* once a correct check fires. The detector is held fixed across
    warn, warn + tip, warn + permission, gate-the-answer, repair and
    withhold. A judge 2×2 separates evidence from enforcement.
  - Material new evidence for [[concepts/enforcement-boundary-placement]],
    [[concepts/typed-enforcement]] (the on-violation action axis) and
    [[concepts/refusal-cost-symmetry]] (clean twin, a two-sided break-even
    λ).
  - Not 5: tool-result handling on count and list tasks, not an
    ML-research agent loop. The headline holds only where an exact
    invariant exists.
- **Candidates** (cited, not yet in the graph):
  - Zhu et al. 2026, "When Tools Fail: Benchmarking Dynamic Replanning and
    Anomaly Recovery in LLM Agents" (arXiv 2606.05806). ToolMaze. Implicit
    failures hurt more than explicit ones.
  - Panthi & Abdelfattah 2026, "Outcome Monitors: Recovery Affordances for
    Silent Tool Failures" (arXiv 2608.19303). Learned contracts with a
    non-binding advisory; the baseline this paper beats.
  - Li et al. 2026, "SilentProbe: Measuring Silent Failure in Production
    APIs Used as Agent Tools" (arXiv 2609.00035).
  - Yang et al. 2026, "Agents' Overreliance on Unreliable Tools" (arXiv
    2609.05587). Agents adopt corrupted tool returns even when they
    question them.
  - Zhang et al. 2026b, "How Do Agent Harnesses Create Value? Planning
    Information and Release Control in Stateful LLM Agents" (arXiv
    2609.20474). An episode-level release gate.
  - Liu et al. 2026, "ToolGate: Contract-Grounded and Verified Tool
    Execution for LLMs" (arXiv 2601.04688). Commits a tool result only
    when a hand-written postcondition holds.
  - Sethi et al. 2026, "Fabrication After Tool Failure" (arXiv
    2609.14758). Agents invent values mainly when a tool failure is not
    signalled.
- **Open on this paper:** does a warning-only harness act like Sol (11.3%)
  or like Terra/Luna (54–63%) on a Claude model? And does the spec check's
  allowed-list message, on its own, carry the real-API recovery?
