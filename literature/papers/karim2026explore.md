---
kind: paper
title: "Explore or Confirm? LLM Agents Misallocate Measurements Under Noise"
authors: ["Abdul Karim", "Matt Keon"]
institutions: ["Xstendyl"]
year: 2026
venue: "arXiv 2610.25111 (cs.AI), v1 2026-10-18, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.25111"
code_url: "https://github.com/Xstendyl/wellocate"
citations: null
source: "raw/papers/karim2026explore.pdf"
added: "2026-10-27"
relevance: 3
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/pass-at-k]]"
tags: ["wellocate", "explore-vs-confirm", "re-testing", "replication", "measurement-noise", "noise-band", "best-arm-identification", "exploration-loss", "selection-loss", "winners-curse", "candidate-supply", "menu", "ledger", "prompt-advice-null", "minimal-agent", "reasoning-off", "nk-landscape", "gb1", "phoq", "protein-design", "scrambled-benchmark", "pre-registered", "failed-replication", "rl-environment", "ppo", "opus-4.7", "gemini-3.1-pro", "gpt-6.1-sol"]
---

# Explore or Confirm? LLM Agents Misallocate Measurements Under Noise

## TL;DR

Wellocate is a simulated assay campaign. A candidate is a 4-letter string
over 20 letters (160,000 candidates), each with a hidden true value. The
agent has 15 rounds × 4 slots = 60 noisy measurements (σ ≈ 7.3% of value,
sized from GB1 read counts). Only the top 7% of candidates are orderable,
and about 15% of orders fail at random. At the end it nominates one
candidate, which is scored on its true value. Three landscapes: a generated
NK grid (K = 1) and real GB1 and PhoQ data with letters scrambled. Five
models run as **minimal agents**: one call per round with the full history,
no tools, temperature 0. Opus 4.7, Gemini 2.5 Flash and DeepSeek V3.1 ran
with reasoning **off**; GPT-6.1 Sol and Gemini 3.1 Pro at their lowest
setting.

Three results matter for this graph:

1. **Misallocation runs both ways, and both lose on the grid.** Opus 4.7,
   Flash and DeepSeek spend 75–94% of slots re-testing. The scripted band
   is 24–38%. Gemini 3.1 Pro spends 14%, and its pick had a single reading
   in 78% of grid campaigns. Every model's grid pick (7.62–7.76) is below
   the guessing rule's 7.87. The shortfall splits cleanly. Over-confirmers
   lose to *exploration* (best measured 0.94–1.04 below the landscape best,
   against 0.67 for the rule). Pro loses to *selection* (0.38, against
   0.23).
2. **Over-confirmation is a candidate-supply failure, not a noise
   response.** It persists at zero noise: 51.8–97.5% re-testing after the
   first repeat across 15 cells. Advice does not move it. A ledger of
   per-candidate mean/SE and a landscape note shift it by ≤5 points.
   Explaining noise and √n *raised* Opus's re-testing (75.4% → 85.8%) and
   cut the distinct candidates it measured (11.3 → 7.0). A supplied list
   of 10 untried neighbours ranked by a fitted model (the "Menu") cut it by
   18–31 points.
3. **Where re-testing pays is set by the leader gap in noise-SD units.**
   The ten best grid candidates lie within 0.32 noise SD of the best; on GB1
   it is 2.95 and on PhoQ 6.80. So at real assay noise the best scripted
   rule re-tests about 1% of slots on GB1 and 0% on PhoQ. Re-testing starts
   to pay there only at about 8× and 4× the assay noise.

The digest's "confirmation has to run throughout the campaign" is the
authors' *suggestion* from one post-hoc replay. No arm runs throughout-
campaign confirmation for an agent.

## Claims

- As reasoning gets cheaper, the physical experiment is the bottleneck, so
  an agent's value depends on how it spends a fixed, noisy experiment
  budget. How LLM agents split that budget between new candidates and
  repeats "has not been measured against known true values."
- Five models misallocate in both directions (three over-confirm, one
  under-confirms, Sol sits just above the band). On the grid every model's
  final pick is worse than the guessing rule's.
- Each failure has its own cause. Over-confirmers "run short of new
  candidates worth trying"; the Menu cuts their re-testing and, on GB1,
  lifts two models' picks by more than three units. The under-confirmer
  picks on single readings. A final-round re-test of its top candidates
  recovers about a quarter of its selection loss, "which suggests
  confirmation has to run through the whole campaign."
- More thinking and prompt wording do not fix it. Gemini 3.1 Pro reasoned
  for over 16,000 tokens on 35 calls and still nominated single-read
  candidates. The one wording change that passed the registered test did not
  replicate. A one-sentence wet-lab role lowered re-testing by at most ten
  points.
- The best fixed rule depends on landscape and noise, so the right split
  must be read from the situation. That makes Wellocate worth learning in.
  A small PPO policy trained only on generated grids beats all five models
  on their own grid campaigns.
- Advice for labs: measure the split (re-testing share against the best
  scripted share, plus the exploration/selection split); give an
  over-confirmer a cheap candidate generator; make an under-confirmer
  re-test a leader every round and the top few at the end.

## Methods

- **Environment.** NK landscape with N = 4 and K = 1. Each position's
  contribution depends on itself and the next letter, with uniform(0,1)
  tables. All landscapes are rescaled so the best scores 8.762. Viable =
  top 7% (11,199 on the grid, close to GB1's 7.2% with ≥10% of wild-type
  fitness). Viable candidates cluster: 25.8% of a viable candidate's 76
  one-letter neighbours are viable on the grid, 45.9% on GB1, 37.6% on
  PhoQ. A non-viable order is rejected before it uses a slot, with up to
  three replacements, and then the slot is lost. Viable orders fail 15% of
  the time with no reading. Noise is σ(f) = √(0.0035² + (0.073f)²),
  Gaussian, i.i.d. Every campaign starts from the median viable candidate,
  whose value is given. The agent sees the full order/reading record every
  round. The GB1/PhoQ names use a random per-position letter map, so
  single-mutation adjacency is preserved.
- **Scores.** Re-testing share = fraction of used slots spent on a
  candidate already ordered (it includes re-orders of the start and
  re-orders after a failed order; both alternative counts are reported).
  Shortfall from the global best is split additively into exploration loss
  (global best − best measured) and selection loss (best measured − pick),
  both in true values. No LLM judge anywhere; the environment scores.
- **Yardsticks.** The *guessing rule* re-tests its current best k times per
  round and fills the other slots with untried one-letter neighbours. The
  *informed rule* does the same but is told viability, so it is never
  rejected; this is an advantage the agents do not have. For both, k was
  set **after the runs** to the best mean pick per landscape, from 1,500
  seeds per setting (k* = 1 on the grid, 0 on the proteins). The
  *model-guided rule*, added later under a registered change, fits additive
  letter scores, re-tests one leader per round and the top four over the
  last three rounds. Three lab protocols are also simulated (triplicate,
  duplicate, screen-then-confirm).
- **Model runs.** Five-model study on the grid: 60 campaigns (Opus, Sol,
  Pro) or 120 (Flash, DeepSeek). On scrambled GB1: 60 / 60 / 60 / 30 / 30.
  On PhoQ: 30 each, paired by seed, with a named test registered before the
  first call. OpenRouter, temperature 0. DeepSeek's calls went to five or
  six hosts, and its results average over them. All five used the "Blind"
  prompt, which says nothing about noise or repeats. Sol + Pro runs cost
  $20.39 and $71.55, PhoQ $20.89.
- **Pre-registered prompt study** (App. H). 3 models × 5 prompt conditions
  (Blind → Quantitative σ(f) → Mechanism √n → Aggregated history →
  Decision-rule) gives 15 cells, 5,075 campaigns and 162,986 calls (695 /
  139 / 181 campaigns per cell for Opus / Flash / DeepSeek). There are six
  confirmatory contrasts, Holm-corrected and paired by seed. No condition
  instructs re-testing, "because that would measure instruction-following."
  Side runs: zero noise (40 seeds), two prompt-flaw checks (60 seeds), a
  free-rejection check, temperature 1.0, reasoning at medium effort (35–40
  seeds per cell) and a wet-lab role (40 per cell). An independent one-round
  check (270 calls) shares no code with Wellocate.
- **Aids** (grid, over-confirmers only, same seeds as control): ledger,
  landscape note, Menu. The Menu = 10 never-ordered candidates, each one
  letter from one of the agent's top three, ranked by a simple model fitted
  to the agent's own readings. Then the Menu on GB1 for all five models.
- **Confirmation replay** (App. G). Each grid campaign of Pro and Sol is
  rebuilt exactly from its log and seed. Every campaign passed the
  reproduction check. The final round's four orders are replaced with
  re-tests of the top m by mean reading. The pick is made by the
  **registered selection rule**, a shrinkage-to-campaign-mean ranking, not
  by the agent.
- **Learnability.** A noise × K sweep with a two-parameter learner (App. K).
  A PPO policy trained on grid only (Run A, 3 + 5 seeds) or on grid + GB1
  (Run B). Registered success criteria, stress tests (batch effects, floor,
  failed wells, spikes), real-name proteins, and replay on each LLM's own
  campaigns.
- **Registration.** "The design, outcome, confirmatory contrasts and
  analysis were registered before it ran". Every later change was
  registered with its date before its data. The PDF does not name where the
  registry lives.

## Results

- **Main table (Table 1; share % / true-value pick).**

  | | grid | GB1 | PhoQ |
  |---|---|---|---|
  | Opus 4.7 | 74.9 / 7.62 | 63.3 / 4.78 | 62.8 / 1.89 |
  | Gemini 2.5 Flash | 94.0 / 7.70 | 91.9 / 1.12 | 63.7 / 2.13 |
  | DeepSeek V3.1 | 91.0 / 7.76 | 96.0 / 0.84 | 89.9 / 0.83 |
  | GPT-6.1 Sol | 44.1 / 7.67 | 19.4 / 6.36 | 23.9 / 2.37 |
  | Gemini 3.1 Pro | 14.0 / 7.69 | 11.1 / 4.90 | 14.7 / 1.72 |
  | guessing rule | 23.8 / 7.87 | 0.9 / 6.09 | 1.0 / 2.05 |
  | informed rule | 23.3 / 8.00 | 0.0 / 6.02 | 0.0 / 2.63 |
  | model-guided | 37.6 / 7.93 | 22.6 / 5.61 | 25.3 / 1.89 |

  Grid pick CIs (Table 6) all lie below 7.87. Gemini Pro's is
  [7.580, 7.802]. On GB1, Sol's lead over the guessing rule is +0.28
  [+0.06, +0.53] as registered. Counting its 12-of-30 repeated nominee once
  widens it to include zero. On PhoQ Sol nominated one candidate in 14
  campaigns, and its lead is likewise uncertain.
- **Loss split, grid (Table 6).** Exploration / selection: Opus 1.02 /
  0.12, Flash 1.04 / 0.02, DeepSeek 0.94 / 0.06, Sol 0.93 / 0.16, Pro
  0.70 / 0.38. Guessing rule 0.67 / 0.23. In the prompt study, exploration
  is 88.9–99.2% of every cell's shortfall. Distinct candidates per
  campaign run 2.57–11.34, against 28.94 (guessing) and 39.14 (informed).
  On GB1, Flash and DeepSeek measured only 3.9 and 2.4 distinct candidates
  in 60 slots.
- **Aids on the grid (Table 7, Δ re-testing points).**

  | | ledger | landscape note | Menu |
  |---|---|---|---|
  | Opus 4.7 | +4.4 | −4.7 | −18.2 [−20.8, −15.5] |
  | Flash | −0.9 | −0.5 | −26.2 [−27.9, −24.5] |
  | DeepSeek | +0.8 | +1.5 | −31.4 [−33.5, −29.4] |

  With the Menu, re-testing is still about 57–68% (our subtraction), well
  above the 24–38% band. On the grid the Menu raised best-measured (Flash
  +0.17, DeepSeek +0.08) and *worsened* selection loss for all three (+0.09
  to +0.13). With the agent's own nomination, net grid pick change is not
  reported. Our arithmetic from Table 9 gives roughly +0.04 (Flash) and
  −0.02 (DeepSeek). On GB1 the Menu lifted Flash and DeepSeek by more than
  three units, from 1.12 and 0.84 (exact values only in Fig. 5c), and moved
  Pro and Sol by ≤4 points with no detectable pick change.
- **Zero noise and the independent check.** At σ = 0, post-first-repeat
  re-testing has a median of 90.6% across the 15 cells. All 15 paired drops
  from noise are negative, 12 significant after Holm. The separate one-round
  test with exactness stated gives repeat shares of 24.2% (Opus), 45.0%
  (Flash) and 55.8% (DeepSeek). Adding noise raises them by +37.5 to +50.0
  pp. So the models do respond to noise, from a high floor. Stating
  exactness made no detectable difference.
- **Prompt study.** Only one of the six confirmatory contrasts survived Holm
  (DeepSeek Decision-rule +0.0699, Holm p 0.0075). A pre-registered
  replication on 120 new seeds gave +0.008 [−0.036, +0.052]. The authors
  attribute the original to a low-scoring Mechanism sample. Opus was
  nominally worse under both Aggregated and Decision-rule (−0.033, −0.032;
  Holm p 0.087). Free rejections (check B) left every cell at ≥74.8%
  re-testing, so rejection risk does not explain the habit.
- **Reasoning on** (medium effort, Opus and Flash, two conditions).
  Re-testing fell 7.5–12.4 points and distinct candidates rose 3.7–7.1. Yet
  the pick *fell* by 0.099–0.263, all four cells significant after Holm,
  because the best measured got worse. 9 of 80 Flash campaigns returned no
  answer and were excluded; including them leaves the result unchanged.
- **Confirmation replay (Table 11).** Gemini Pro, last round, m = 3:
  +0.088 [+0.011, +0.171], about 23% of its 0.38 selection loss. With m = 4:
  +0.090 [−0.004, +0.190]. With the last two rounds (not planned in
  advance): +0.100 [+0.013, +0.191], about 27%. The registered rule on
  Pro's unchanged readings gives +0.004, so the gain is from the extra
  readings. Sol gains nothing (−0.041 [−0.100, +0.011]) because its top
  candidates already have repeats. The gain is small because Pro ends with
  dozens of single-read candidates and four slots can check only a few.
- **Noise sweep (Fig. 6, Table 3).** Grid peaks at k = 1 at assay noise.
  GB1 and PhoQ peak at k = 0, and every added re-test lowers the pick. On
  the grid, part of re-testing's value is avoiding rejections. Never
  re-testing loses 23 slots to rejections, against 12 at k = 1. With free
  rejections k = 1 still wins (7.95 vs 7.79). In the 18-setting
  learnability sweep k = 1 is best or tied in all 18, and 16 of 18 are
  within 2 SE of the runner-up.
- **PPO policy.** Run A (grid only) re-tests 9.3% on the grid and about 2%
  on the proteins, mostly retries of failed orders. Of its new candidates,
  61% are one letter from a top-three candidate. It beats random ordering
  by +0.40 [0.36, 0.44] on held-out grids and ties the guessing rule. Run B
  failed its registered transfer criterion and passed only after a post-hoc
  rerun with a 5× entropy bonus. On the LLMs' own grid campaigns Run A wins
  by +0.10 to +0.25 against every model. On scrambled GB1 it loses to Sol
  by −1.69. On real-name GB1 it loses to Opus by −2.84, where Opus scores
  7.59 against 4.78 scrambled.
- **Memorisation.** With real GB1 letters, Opus 4.7 named the best measured
  variant in round one in 11 of 20 campaigns; random picks would do so
  0.035% of the time. This justifies the scrambling.

## Critique / open questions

- **Against the digest framing: below framing (mild).** Every number the
  digest quotes is in the PDF. Two of its sentences claim more than the
  paper shows.
  1. **"A list of untried candidates fixes over-confirmation."** It reduces
     it. On the grid the three models still re-test about 57–68% of slots.
     Selection got worse, and the grid pick gain with the agent's own
     nomination is not shown. Only a fixed post-hoc picker improves Opus and
     Flash. The large pick gains are on GB1, for two models that had
     collapsed to 2–4 distinct candidates. The Menu is also not a neutral
     aid. It is the model-guided rule's proposal step (fitted letter scores,
     one letter from the top three) handed to the agent. So "supply
     candidates" partly means "outsource the search to a scripted proposer".
     The Menu was tested on the grid and on GB1 only, not PhoQ.
  2. **"Confirmation has to run throughout."** This is the authors'
     *suggestion* ("which suggests"), drawn from one replay of one model
     (Gemini 3.1 Pro) on one landscape (grid). The pick in that replay is
     made by a registered shrinkage rule, not the agent. No arm makes an
     agent, or a replay, confirm throughout, so the remaining 77% is not
     shown to be recoverable that way. The model-guided rule does confirm
     throughout, but it also proposes differently. Its grid pick (7.93) is
     not separable into confirm vs propose. On GB1, Pro's loss is almost
     all exploration (3.76 vs 0.10 selection), so under-confirmation is a
     one-model, one-landscape finding, as the limitations section says.
- **What the digest omitted.** The three over-confirmers ran with reasoning
  *off*, and Opus 4.7 is one of them. With reasoning on they explore more
  and pick worse. Over-confirmation persists at zero noise, so it is a habit,
  not a noise response. Explaining noise in the prompt made Opus
  over-confirm *more*. At real assay noise, re-testing never pays on either
  protein landscape, so "every model re-tests more than the best rule" on
  GB1/PhoQ is nearly automatic: that rule re-tests ≤1%.
- **The yardstick is mildly oracle-tuned.** k for the guessing and informed
  rules was chosen after the runs, per landscape, on the best mean true
  pick. The learnability sweep shows k = 1 is best or tied almost
  everywhere on grids, so the tuning advantage on the grid is small. On the
  proteins "k = 0" is a choice an agent could only make if it knew the
  leader gap. The informed rule also knows viability. The guessing rule at
  k = 0 (7.71) is beaten by 5 of the 15 prompt-study cells, so "every model
  below scripted" depends on comparing against the k = 1 setting.
- **The noise model is idealised.** It is i.i.d. Gaussian with no plate
  effects or drift. Rejections are free of wells. A new variant costs the
  same as a re-test. Appendix A says each of these biases toward *less*
  re-testing than a real lab would want. The authors say so themselves:
  "an agent that re-tests more than the scripted rules here would not
  necessarily re-test too much in a real lab." The stress tests in Table 21
  (batch effects, floors, failed wells) do not move k*. 2% spikes do. Batch
  size and budget effects on the agents were not tested.
- **K = 1 is an easy landscape.** The search is "simple" by design, so losses
  are about spend. That isolates allocation but says nothing about rugged
  search. Agents were run at K = 1 only.
- **Sample sizes are adequate for the headline and thin for the
  mechanisms.** 30–120 campaigns per model and landscape, temperature 0,
  one prompt (Blind) for the five-model study. The per-model claims (Sol
  "just above the band", Pro "under-confirms") rest on one run set each.
  Sol's apparent wins on the proteins rest partly on one repeated nominee.
- **Credit for discipline.** The paper reports its own positive
  confirmatory result failing replication. It checks rejection risk with a
  free-rejection arm and runs a code-independent reproduction. It reports
  both alternative re-testing counts, and reports the PPO Run B failure and
  the post-hoc fix as such. This is unusually careful for the genre.
- **Fit with the rest of the graph.**
  - The leader-gap result is the experimental-design form of the noise-band
    question. Whether a repeat is worth a slot depends on how far apart the
    contenders are in noise-SD units, not on the noise alone. That agrees
    with [[literature/papers/ning2026implementation]]'s allocation rule:
    replicates pay only when their variance term dominates their cost.
  - Pro's single-read winners are the winner's curse that
    [[literature/papers/chen2026auditing]] prices for selection over many
    noisy candidates. The paper's shrinkage selection rule is the standard
    partial fix.
  - That advisory aids fail and a supplied candidate list works matches
    [[literature/papers/yan2026traceml]] (a prompt self-check reduced
    switching) and [[literature/papers/vats2026good]] (advisory redirect
    text bought nothing). Changing what the agent can choose from moves
    behaviour; telling it more does not.
  - "The best fixed rule changes with the landscape" parallels
    [[literature/papers/zou2026fmlbench]]: which search topology wins
    depends on improvement-opportunity density.
- **Transfer to ML-research agents is indirect.** These are bare models with
  no tools, memory or plan. The ML analogue (re-run the incumbent vs try a
  new idea) has correlated noise (shared validation items, as in
  [[literature/papers/yao2026explicit]]) and very unequal costs. Both
  violate the environment's assumptions.

## Trust signals

- **Credibility:** 3.
  - For: a registered design with dated amendments, and a failed
    replication of its own positive result reported in full. An
    environment-scored outcome with no LLM judge. A code-independent
    reproduction of the core behaviour. Exact log-replay with a 100%
    reproduction check. CIs on nearly every number and paired seeds
    throughout. Honest limits (Appendix A gives the direction of every
    simplification). Code, data, environment, campaign logs and analysis
    are released (github.com/Xstendyl/wellocate, named in the PDF; not
    opened at ingest).
  - Against: two authors at an unknown company (Xstendyl), not peer
    reviewed, 9 days old. The registry location is not given. AI coding
    assistants were used "for implementation, analysis and drafting". The
    yardstick k was tuned post hoc. The reasoning-off configuration is not
    how these models are normally deployed. The under-confirmation mechanism
    and the "throughout" claim rest on one model, one landscape and a
    post-hoc replay. Model names come via OpenRouter, and DeepSeek mixes
    hosts.
  - Not 4 because there is no peer review or known-lab signal, the
    registration cannot be checked from the PDF, and the most quotable
    mechanism claim is an inference. Not 2 because the design is
    pre-registered and over-checked, the artifacts are released, and the
    main allocation effect (75–94% vs a 24–38% band, persisting at σ = 0)
    is large and replicated across prompt conditions, seeds and an
    independent harness.

## Follow-up

- **Relevance:** 3. It is the first measurement in this graph of how LLM
  agents themselves split a fixed noisy budget between new candidates and
  repeats, scored against known truth. It gives a clean diagnostic, the
  exploration/selection loss split. It adds material to
  [[concepts/budget-as-ceiling]] (when re-measuring the incumbent pays),
  [[concepts/evolutionary-search-grain]] (breadth vs depth chosen by the
  agent, and candidate supply vs advice) and [[concepts/pass-at-k]] (the
  selection term, measured). Not 4 because the agents are minimal, three of
  them run with reasoning off, the domain is a simulated assay with i.i.d.
  noise, and the most transferable result (the leader-gap rule) is
  classical experimental design rather than new.
- **For the noise band (our inference, not proposed here).** The paper
  supports a *conditional* incumbent re-run in `/iterate`. When a candidate
  beats the champion by less than about one to two band widths, spend the
  next run re-measuring both, which is wang2026pear's RETAIN state. When
  the margin is several band widths, a re-run is GB1-like waste. A champion
  crowned on a single run (`band=none`) is Gemini Pro's failure mode. One
  end-of-chain re-check of the top few would recover only part of it if
  many single-run "bests" accumulated. This is an inference from a
  bandit-like simulator; it was not tested on ML loops.
- **For instruction files (our inference).** Explaining noise and √n made Opus
  4.7 (an earlier model of the family this box runs, here with reasoning
  off) over-confirm more. Any noise-band wording in an `/iterate` prompt should
  be checked for that side effect. A controller-computed band is safer than
  telling the agent about noise.
- **Missing arms worth asking for.** Throughout-campaign confirmation
  enforced for the agent (replay with one leader re-test per round). A
  Menu-free candidate generator that is not the yardstick's own proposer.
  The five-model study with reasoning on. Correlated noise or unequal
  re-test cost.
- **Candidates** (cited, not yet in the graph):
  - Krishnamurthy et al. 2024, "Can large language models explore
    in-context?" (NeurIPS). LLMs in bandits; a summarised history helped
    only GPT-4 with CoT. Here a ledger did not help.
  - Park et al. 2026, "Exploration and exploitation errors are measurable
    for language model agents" (arXiv 2604.13151). Programmatic
    environments, noise-free.
  - Gandhi et al. 2025, "BoxingGym" (arXiv 2501.01540). The nearest
    experimental-design benchmark, scored by EIG, with no repeat choice.
  - Wang & Xu 2026, "AllocBench" (arXiv 2607.23332). Online budget
    allocation, noise-free.
  - Huai et al. 2026, "AgentHPOBench" (arXiv 2607.29626). LLM agents as
    sequential HPO optimisers on 30 ML tasks. The closest ML-research
    analogue; already named in guo2026epoch, no note.
  - Kim & Romero 2026, "Benchmarking and behavioral characterization of LLM
    agents for protein design" (bioRxiv). Agents check candidates too
    shallowly and stop exploring early.
  - Gupta et al. 2025, "LLMs for Bayesian optimization in scientific
    domains: are we there yet?" (EMNLP Findings) and Wainrib et al. 2026
    (arXiv 2603.26177). Do LLM optimisers use the feedback at all?
  - Binois et al. 2019, "Replication or exploration?" (Technometrics) and
    Dai et al. 2023, batch BO for replicable experimental design (NeurIPS).
    The classical anchors for the leader-gap rule.
