---
kind: paper
title: "Homogenization in Multi-Agent Systems"
authors: ["Prakhar Ganesh", "Kyra Wilson", "Luca Zappella", "Barry-John Theobald", "Nicholas Apostoloff", "Lucas Monteiro Paes", "Nivedha Sivakumar"]
institutions: ["Apple", "McGill University & Mila", "University of Washington"]
year: 2026
venue: "arXiv 2610.09824 (cs.LG), v1 2026-10-07, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.09824"
code_url: null
citations: null
source: "raw/papers/ganesh2026homogenization.pdf"
added: "2026-10-13"
relevance: 3
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/hierarchical-delegation]]"
tags: ["homogenization", "conformity", "polarization", "inertia", "multi-agent", "orchestrator", "llm-judge-panel", "correlated-errors", "codeql", "adversarial-agent", "bias-propagation", "peer-review-automation", "mixed-model", "no-channel-arm", "transcript-substrate"]
---

# Homogenization in Multi-Agent Systems

## TL;DR

An orchestrator-driven MAS runs T = 5 rounds. Each round, n worker agents
score the input, the orchestrator aggregates, and the whole discussion is
broadcast back to every agent. The paper calls the round-over-round
convergence of scores *homogenization* and measures it with three
directional metrics:

- **Conformity:** inter-agent variance falls.
- **Polarization:** the panel mean moves toward an end of the scale.
- **Inertia:** the round-to-round change in the mean shrinks.

It runs this on three tasks: code generation with a 4-reviewer panel
(CodeLMSec, 280 prompts, CodeQL as ground truth), hiring with a 4-evaluator
panel (8,000 synthetic counterfactual résumés), and peer review (1,950
ICLR 2025 papers, three reviewers plus a meta-reviewer and an author).
There are 9 LLMs, 16 mixed-model configurations, and M = 10 trials at
default sampling.

All three metrics trend the expected way almost everywhere. Three results
matter for this graph:

1. **Hiring is a clean propagation result.** One agent's system prompt is
   injected for two rounds with hidden stereotypical priors and an
   instruction to argue the panel round. The other agents' ethnicity bias
   rises from about 0 to roughly 0.2–0.3 points on a 1–5 scale, and it does
   not decay after the injection is removed. Without the adversary the
   same panel shows about 0 bias.
2. **In code, the panel's own verdict drifts away from the deterministic
   oracle.** The MAS cuts overall CodeQL vulnerability rates sharply,
   almost all of it from round 1 to round 2. Meanwhile a few vulnerability
   types (py/url-redirection, py/log-injection) climb among code the panel
   rates deployable, and the same types climb across model families.
3. **Neither "mitigation" is actually tested as one.** "Sampling
   stochasticity" means the 10 default-temperature trials. "Mixed-model"
   means 6 permutations per task, shown as figures with no single-model
   comparison of magnitude.

## Claims

- Homogenization is "a failure mode of MAS", and MAS evaluation "must move
  beyond aggregate performance to carefully analyze interaction dynamics."
- **Code generation.** MAS "reduce the overall rate of security
  vulnerabilities" (by "up to 50%"), but "correlated errors amplify
  specific vulnerabilities", creating "systemic security blind spots". The
  blind spots "are not idiosyncratic to a specific LLM."
- **Hiring.** "bias introduced by an adversarial agent propagates to other
  agents, and even persists after the original source of bias is removed."
  The authors attribute persistence to inertia.
- **Peer review.** Homogenization "drives systematic disparities across
  research areas": the spread between the most- and least-favoured primary
  areas widens over rounds, and the same areas are favoured or penalised
  across LLMs.
- **Mitigations.** "Sampling merely injects surface-level variance without
  altering the underlying consensus dynamics." "Mixed setups in the current
  LLM landscape fail to provide the expected behavioral diversity," because
  "models from different families often share the same blind spots."
- The three metrics are "orthogonal" (all 8 combinations are illustrated in
  Appendix A), and it is "the collective trends of all three axes" that
  constitute homogenization risk.

## Methods

- **Protocol (Algorithm 1).** Each round, every worker samples (score,
  rationale) given its subtask and the global state. The orchestrator
  samples an updated global state, which is broadcast to all workers.
  Workers are personas over one base LLM, or different LLMs in mixed
  setups. vLLM runs at default generation parameters, with T = 5 and
  M = 10 trials. Error bars are 95% CIs over the 10 trials. No hypothesis
  tests are reported.
- **Metrics.** Each metric is an inequality between consecutive rounds per
  data point, reported as a dataset mean. They are directional, not
  calibrated against any reference rate of convergence.
- **Code generation.**
  - A generator writes code; four reviewers (security, optimality,
    correctness, readability) each give a 1–5 deployability score and
    9–10 lines of feedback; the generator revises.
  - CodeQL's `python-/cpp-security-extended` suites label vulnerabilities.
  - "Deployed" means a mean reviewer score of at least 3.
  - Results are reported as a % of *all* samples, per vulnerability type.
  - Reviewer prompts: "Only review the complete code in the latest round."
  - Code-specialist models are Kimi-K2.7-Code and Qwen3-Coder-Next.
- **Hiring.**
  - 40 US federal occupations, 5 Gemini-3.1-Pro job descriptions each,
    one résumé per description (Tan et al. 2026 prompt).
  - 40 counterfactual demographic variants per pair (4 ethnicities × 2
    genders × 5), using extracurricular, language, volunteering and hobby
    signals. Gemini 3.1 Pro recovered the intended group 100% of the time.
  - Bias is the mean score of the most-privileged group minus the
    least-privileged group, in the direction of BLS-based stereotypes.
  - The adversary's system prompt carries "hidden evaluation layer"
    priors. Its refinement prompt tells it to "raise ... concerns" or
    "argue that the panel may be underweighting" so as to pull others
    toward its view.
  - The adversary is present for rounds 1–2. Appendix F ablates duration
    (1–3 rounds) and timing (rounds 1–2, 2–3, 3–4) on Qwen3-30B only.
- **Peer review.** Stratified ICLR 2025 sampling following Sahu et al.
  2025: 1,000 rejected or withdrawn, 300 posters, all orals, spotlights and
  desk rejects. Main text only. Three reviewer personas (pragmatist,
  theorist, empiricist) score 1–10; a meta-reviewer summarises; an author
  agent writes rebuttals. Disparity is the spread of mean reviewer score
  across author-selected primary areas.
- **Stochasticity check (Fig. 7).** Variance of the panel mean across the
  10 trials, and for code, an LLM-judge test of functional equivalence over
  45 trial pairs.
- **Mixed configurations (Figs. 8, 11–13).**
  - Code: all 6 generator/reviewer permutations over three models.
  - Hiring: 4 evaluator LLMs with the adversary rotated across 4 of them.
  - Peer review: 6 permutations of author / meta-reviewer / reviewers.

## Results

- **Homogenization trends (Figs. 3, 10).** Inter-agent variance falls to
  about 0.5× or less by round 5 in most settings. Polarization moves the
  mean "by up to 0.5 points" toward an endpoint. Inertia is the strongest
  trend and polarization in hiring the weakest. Qwen3-30B-A3B-Thinking
  homogenizes most in every task.
- **Overall vulnerability rate (Fig. 14).** All 9 models drop sharply from
  round 1 (generator alone) to round 2 and are nearly flat afterwards.
  Examples: Qwen3-30B 54→30%, DeepSeek-V4-Pro 65→36%, Nemotron-Super
  60→22%, Kimi-K2-Thinking 66→45% (read from charts).
- **Rising vulnerability types (Figs. 4, 15–22).** Among deployed code,
  py/url-redirection rises from about 2% to about 8–10% of all samples
  over rounds for most models. py/log-injection rises steeply for some
  (Kimi-K2-Thinking about 1→14%; Nemotron-Ultra about 1→13%). In the
  all-code panels, url-redirection is roughly flat from round 2 for several
  models, while log-injection grows in the generated code itself for Kimi
  and Nemotron-Ultra. Other types (py/flask-debug, py/stack-trace-exposure)
  fall to near zero.
- **Hiring propagation (Figs. 5, 23–24).**
  - Without an adversary, panel bias is about 0 for all 7 models.
  - With one: the adversary itself starts at about 0.8–1.5, falls after
    removal, and stays well above 0 (about 0.3–0.7 at round 5). The other
    agents rise to about 0.1–0.3 and stay there through round 5. The panel
    mean peaks at round 2 and then declines slightly.
  - Gender bias shows the same shape at smaller magnitude.
  - Duration ablation: residual bias in the others is about 0.11–0.13
    after 1 round of exposure and about 0.19–0.23 after 3.
  - Timing ablation: about 0.18–0.20 for rounds 1–2, 0.10–0.13 for rounds
    2–3, and about 0.05 for rounds 3–4, measured at round 5.
- **Peer-review disparity (Figs. 6, 8, 27).** Area means fan out over
  rounds, and the same areas sit low in all four LLMs (infrastructure and
  systems; transfer, meta- and lifelong learning) while learning theory and
  RL sit high. In mixed setups the max–min spread grows from about 0.6 to
  about 1.2 on the 1–10 scale.
- **Stochasticity (Fig. 7).** Trial-to-trial variance *grows* with rounds,
  and pairwise code dissimilarity rises from about 0.8 to about 0.95, while
  the Section 5 harms appear in every trial.
- **Mixed setups (Figs. 8, 11–13).** All mixed hiring setups propagate the
  adversary's bias (others reach about 0.1–0.4). Mixed code setups still
  show rising url-redirection in most permutations. **Not every mixed
  setup homogenizes:** Qwen3-Coder-Next + Nemotron-Super reviewers keep
  conformity flat at about 0.78 (Fig. 11a), and two peer-review
  permutations keep conformity and polarization nearly flat (Fig. 13). The
  text does not discuss these cases.

## Critique / open questions

- **House-pattern check: the paper reads BELOW the digest framing.** The
  digest repeats the abstract's three headline claims. Each is narrower in
  the body.
  1. *"Mixed-model MAS and sampling stochasticity fail to reduce it."*
     Neither is tested as a reduction. The stochasticity "intervention" is
     the default-temperature trials the whole paper already runs: there is
     no temperature sweep and no independent-resample arm. The mixed-model
     result is a set of figures with no single-model versus mixed
     magnitude comparison, and three mixed configurations in the paper's
     own appendix show little or no conformity. The supported claim is that
     mixing does not *eliminate* the harms, not that it fails to *reduce*
     them.
  2. *"Homogenization hides and amplifies correlated errors"* in code is
     the most-favourable-cut pattern. The aggregate result is a 30–60%
     relative drop in vulnerabilities. The "amplified" ones are chosen
     types, measured as a % of *all* samples within the deployed subset. A
     type can therefore rise because the panel passes more code (the
     reviewer-overconfidence half of the paper's own explanation) without
     the generator writing more of it. For url-redirection the all-code
     panels are roughly flat after round 2. Log-injection does rise in
     generated code. An alternative mechanism the paper does not test is
     that reviewer-requested fixes (logging, redirects) introduce new
     vulnerabilities. This is our inference.
  3. The "Why" line's reading — "the vendor-mixing question answered
     negatively for interaction-driven convergence" — is too strong for
     the same reason as point 1.
- **No no-channel arm, which is what [[concepts/shared-substrate-contagion]]
  requires before calling convergence contagion.** No condition has agents
  re-score each round *without* seeing the broadcast.
  - In code, reviewers re-score code that is being fixed in response to
    their own feedback. Falling variance and scores drifting toward 5 are
    what genuine improvement looks like too.
  - In peer review, if individual scores polarize, area means stretch
    apart proportionally, so a widening max–min spread can be a change of
    scale rather than an area-specific preference. The rank order of areas
    in Fig. 6 is largely stable from round 1. This is our inference.
  - The metrics are directional inequalities with no reference rate.
    Inertia in particular holds for any process that settles. Compare
    [[literature/papers/shao2026language]], whose peer-hidden control
    split channel-driven agreement from correlated priors and found they
    need opposite fixes.
- **Hiring is the strong result, and its "persistence" is the transcript.**
  This arm does have a proper counterfactual: with no adversary, bias is
  about 0, so the others' bias must arrive through the channel. But the
  agents are stateless LLM calls whose refinement prompt contains "the
  panel's discussion so far". Removing the adversarial *prompt* leaves the
  adversary's round-1–2 *messages* in every agent's input, including its
  own (hence its residual 0.3–0.7). "Inertia prevents self-correction" is
  therefore the shared substrate retaining the defect, and the natural
  repair is to quarantine the messages, not only the agent. The timing
  ablation is consistent with this, but it is confounded with how many
  rounds remain before round 5. Our inference.
- **The adversary is strong by construction.** It is told to argue
  persuasively and to hide its priors behind professional language. The
  effect on others is 0.1–0.3 points on a 1–5 scale. That is real but
  modest, and it is the kind of propagation [[literature/papers/xu2026dont]]
  and [[literature/papers/shen2026revoked]] measured with tighter designs.
- **No validity anchor for peer review.** Area disparity is never compared
  with the actual ICLR decisions in the sample, so "uneven standards" is
  not separated from real differences in acceptance rate across areas.
- **Statistics are thin for the claims.** There are no hypothesis tests or
  effect sizes, only 95% CIs over 10 trials of the same 280/200/1,950
  items. Most numbers are read off charts. There is no released code,
  although the prompts are in full in Appendix I.
- **Scope relative to research agents.** The architecture is the review
  panel this repo actually uses (several judges score, see each other,
  revise), so the transfer is direct. But all outputs are bounded scalar
  scores, and code generation is single-function completion. No task is a
  research loop.

## Trust signals

- **Credibility:** 3.
  - For: Apple ML research with McGill/Mila and UW authors, two of them
    with a prior homogenization and multiplicity track (FAccT, AIES). It
    is broad: 9 LLMs, 16 mixed setups, three tasks, a deterministic oracle
    (CodeQL) for code, a counterfactual résumé design with a validated
    demographic signal, a no-adversary baseline and ablations of adversary
    duration and timing. All prompts are released in the paper.
  - Against: a preprint with no named venue. No code or data release; the
    hiring dataset and ICLR sample were regenerated by the authors. No
    hypothesis tests, and figure-only results. Above all, no no-channel
    control for the central homogenization claim, and the mitigation claims
    are not tested as interventions.
  - Not 4 because the design cannot separate interaction-driven
    convergence from genuine improvement or correlated priors. Not 2
    because the scale, the oracle and the hiring counterfactual are solid.

## Follow-up

- **Relevance:** 3. It is on the declared `shared-substrate-contagion`
  watch axis and supplies one new propagation sighting with a
  source-removal arm (hiring), plus an LLM-panel-versus-deterministic-
  oracle datum (code). It does not move the vendor-mixing question, which
  was the digest's reason for ranking it, because mixing is never measured
  as a reduction. [[literature/papers/zheng2026engineering]] (model swap
  −11.3pp versus source swap −40.9pp) and
  [[literature/papers/chen2026rankevolve]] (cross-vendor ρ = 0.21 versus
  same-product 0.58) remain the quantitative evidence on that question,
  and both say mixing helps *some*.
- **For this repo's review panels.** When a reviewer is removed or
  overruled, its messages must leave the context too. A reviewer whose
  prompt is fixed but whose earlier verdicts remain in the transcript keeps
  steering the panel.
- **For any LLM deployability or acceptance gate.** Report the
  deterministic-oracle rate *conditional on passing the gate*, per round.
  The aggregate improvement hid a rising pass-through of specific defect
  types.
- **Missing experiment worth naming.** Run the same panel with the
  broadcast withheld, so each agent re-scores alone, and compare the round
  trends. That is the cheapest test of whether "homogenization" here is
  the channel at all.
- **Candidates** (cited, not in the graph):
  - Lin et al. 2025 and Oh et al. 2026 (conformity and consensus metrics
    in debating agents).
  - El et al. 2026, "Physics of agents" (arXiv 2608.16578).
  - Goel et al. 2025, "Great models think alike and this undermines AI
    oversight" (ICML).
  - Kim et al. 2025, "Correlated errors in large language models" (ICML).
  - Gorecki & Hardt 2026, "Monoculture or multiplicity" (NeurIPS).
  - Jiang et al. 2026, "Artificial hivemind" (NeurIPS).
  - Sahu et al. 2025 (the MAS peer-review protocol reused here).
