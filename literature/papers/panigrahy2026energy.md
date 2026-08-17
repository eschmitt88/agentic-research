---
kind: paper
title: "Energy per Successful Goal: Goal-Level Energy Accounting for Agentic AI Systems"
authors: ["Deepak Panigrahy", "Aakash Tyagi"]
institutions: ["Independent Researcher", "Texas A&M University"]
year: 2026
venue: "arXiv 2605.22883v1, cs.AI (preprint, 2026-05-20; ACM article template with placeholder DOI, 34 pages)"
peer_reviewed: false
url: https://arxiv.org/abs/2605.22883
code_url:
citations:
source: "raw/papers/panigrahy2026energy.pdf"
added: "2026-08-17"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/spend-forecast-calibration]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/pass-at-k]]"
  - "[[concepts/async-worker-pool]]"
  - "[[concepts/scripted-tool-pipelines]]"
tags: [budget, energy, cost-accounting, measurement, orchestration-overhead, retries, reproducibility, metrology, green-ai]
---

# Energy per Successful Goal

## TL;DR

Argues energy-per-inference is the **wrong unit** for agentic systems,
because inference count is an implementation artifact rather than a task
property: a system that retries four times before succeeding reports the
same energy-per-inference as one that succeeds immediately while consuming
5× the energy. Proposes **EpG** (total workflow energy across *all*
attempts, including failures and retries, divided by successfully completed
goals) and **OOI** (Orchestration Overhead Index — the same measurement
against a matched linear baseline on identical tasks and success criteria),
implemented in A-LEMS with hardware RAPL profiling. Headline: agentic
workflows consume **4.33× the energy per successful goal** of equivalent
linear baselines (888.1 J vs 205.3 J) across 827 agentic goals, and the
overhead comes from orchestration structure, not from more inference
compute.

## Claims

- **The unit problem.** Energy-per-inference counts implementation steps,
  not goal completions. "A planner issuing four LLM calls and two retries
  is not 'six inferences worth of energy': it is one goal-level
  computation whose energy is distributed across planning, execution,
  waiting, and recovery phases." No published agentic benchmark, to the
  authors' knowledge, normalizes energy by successfully completed goals.
- **Failed attempts are invisible by construction, and they are not
  cheap.** In a measured GSM8K trace, the failed attempt drew 2,256.1 J
  against 1,358.4 J for the successful one — so inference-level accounting
  missed **62.4%** of the true cost of that goal. Failures often cost
  *more* than successes because the model exhausts more computation before
  producing invalid output.
- **EpG.** Aggregate the energy of every attempt associated with a goal;
  divide by goals the evaluation function accepted. Goal granularity is
  fixed by the benchmark specification (one GSM8K row = one goal), never
  by the system's internal decomposition — so the metric cannot be gamed
  by re-splitting work.
- **OOI isolates orchestration from substrate.** A ratio of agentic to
  matched-linear energy under identical task definitions and success
  criteria, so it measures the cost of *orchestrating* rather than of
  inferring. Explicitly modeled on PUE: operationally defined, empirically
  measurable, and knowingly gameable, but useful for that reason.
- **Three further measurement failures the paper names and fixes.** The
  *boundary problem*: tools that allocate energy as TDP × wall-time
  include post-task framework teardown, a fixed cost shared by both
  workflow types — and because linear workflows finish faster, that fixed
  cost is a larger fraction of their denominator, biasing OOI toward 1.0×
  regardless of true overhead. The *attribution problem*: raw package
  energy conflates idle draw, concurrent processes, and workload-induced
  consumption, so without baseline subtraction and CPU-fraction isolation
  "the measurement reflects the machine, not the task." The
  *reproducibility problem*: without binding measurements to hardware
  identity, governor policy, and runtime configuration, the same workload
  on the same machine yields different numbers across runs.
- **Orchestration structure, not inference compute, drives the overhead.**
  The 4.33× is attributed to retries, intermediate planning, and recovery
  behavior rather than to more or larger model calls.
- **The metric is directional, not a fixed penalty — and that is the
  validation.** For tool-augmented tasks where agentic dispatch replaces
  costly token generation, **OOI inverts below 1.0×**: agentic execution
  is strictly cheaper than linear. A metric that always reported overhead
  would be suspect; this one responds to the structure it claims to
  measure.
- **Phase-level measurement reveals what duration-based allocation
  hides.** During the LLM-API wait phase in remote-inference runs, task
  power drops to 1.0 W against 0.2 W during active planning — a reduction
  invisible to any method that allocates energy by elapsed time, which
  therefore "overstate[s] inference cost while understating orchestration
  and coordination overhead."

## Methods

- **A-LEMS**: a temporal boundary model (t₀/t₁/t₂ defining the attribution
  window of a workflow); a five-layer observation pipeline (L0–L4) mapping
  hardware RAPL signals to workflow-level energy via baseline subtraction
  and CPU-fraction attribution, with every quantity tagged MEASURED,
  CALCULATED, or INFERRED at each layer; and a **three-hash reproducibility
  protocol** binding each measurement to its exact context — H_hw
  (CPU model, microcode version, kernel release, available RAPL domains),
  H_env (Python version, OS, git commit hash and dirty flag, framework
  version, data schema), and H_run (governor policy, turbo state, plus the
  other two hashes and a baseline id).
- Five reasoning task families (factual QA, science QA, arithmetic,
  multi-step reasoning, logical reasoning) plus three tool-augmented
  families; 827 agentic goals; hardware-level RAPL energy profiling;
  controlled failure injection to exercise the retry path; local
  (llama_cpp) and remote-inference configurations.
- Five validation axes: measurement validity, reproducibility, boundary
  correctness, discriminative power, and orchestration dominance.

## Results

4.33× mean energy per successful goal for agentic vs matched linear
(888.1 J vs 205.3 J) over 827 goals; 62.4% of one traced goal's true cost
invisible to inference-level accounting; OOI < 1.0× on tool-augmented
tasks (the directional-correctness check); 1.0 W vs 0.2 W power split
between API-wait and active-planning phases.

## Critique / open questions

- **Energy, not dollars, and a local-inference bias.** RAPL measures
  host-package energy, so the *model's* inference energy is only captured
  in the local llama_cpp configuration; for remote inference the measured
  quantity is largely orchestration on the client. That is precisely what
  the paper wants to isolate, but it means "agentic workflows use 4.33×
  the energy" should not be read as a claim about total datacenter draw.
- The matched linear baseline is doing heavy lifting. What counts as "the
  same task solved linearly" is a construction choice, and OOI inherits
  whatever generosity it contains.
- 827 goals on small reasoning benchmarks (GSM8K-scale) is a narrow
  workload relative to the long-horizon agentic loops this project cares
  about, where a single goal can include a training run. The *unit*
  argument transfers cleanly; the 4.33× constant almost certainly does
  not.
- The retry amplification is partly **controlled failure injection**, not
  observed field failure rates, so the 62.4% figure demonstrates the
  metric rather than measuring the wild.
- The paper concedes OOI is gameable (as PUE is) and depends on reported
  measurement conditions. No public artifact URL is given despite the
  reproducibility protocol hashing a git commit — the protocol is designed
  for reproducibility that the release does not yet enable.

## Trust signals

- **Credibility:** 3 — careful metrology with hardware RAPL measurement,
  explicit baseline subtraction and provenance tagging, a genuine
  falsification check (the tool-task OOI inversion), and honest treatment
  of the boundary/attribution biases most energy papers ignore. Held at 3:
  preprint on an ACM template with a placeholder DOI, one independent
  researcher plus one university co-author, no located artifact release,
  narrow benchmark suite, and a headline amplification figure that leans
  on injected failures.

## Follow-up

- **Relevance:** 4 — second source for
  [[concepts/spend-forecast-calibration]] and the mechanism the
  besanson2026green energy thread was missing. Its contribution here is
  the **denominator**: cost normalized by *completed goals* rather than by
  calls, which is the only unit under which retries and dead ends appear
  at all. That is the same accounting move [[concepts/pass-at-k]] makes
  for scores, and the two together suggest a single rule — always
  normalize by outcomes, never by attempts.
- Direct consequence for `budget.yaml`: `max_tokens` counts attempts, so a
  chain that burns its ceiling on three failed cycles and one success
  reports the same consumption as one that succeeded four times. A
  per-successful-outcome counter would make
  `max_consecutive_no_improvement` and the token ceiling commensurable —
  worth considering the next time the ceilings are revisited.
- OOI is the metric that would answer whether this project's own
  subagent-heavy patterns pay for themselves. The tool-task inversion
  (OOI < 1.0× when dispatch replaces generation) is an argument *for*
  [[concepts/scripted-tool-pipelines]] measured in energy rather than
  correctness: handing arithmetic to code is cheaper as well as more
  reliable.
- The API-wait power finding (1.0 W vs 0.2 W) is a caution for
  [[concepts/async-worker-pool]] accounting: wall-clock-based cost
  attribution mis-prices a design whose whole point is overlapping waits.
- The metrology framing is the transferable part for
  [[concepts/hce-evaluation]]: a unit whose value depends on the system's
  internal decomposition can be gamed by re-decomposing, which is why goal
  granularity must come from the benchmark specification. Same argument
  shape as fixing the holdout outside the agent's reach.
