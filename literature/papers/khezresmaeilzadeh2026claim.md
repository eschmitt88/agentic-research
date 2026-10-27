---
kind: paper
title: "Claim Laundering: When Agent Decision Models Trust Records Over Reality"
authors: ["Tina Khezresmaeilzadeh", "Seyedarmin Azizi", "Erfan Baghaei Potraghloo", "Mehdi Kamal", "Massoud Pedram"]
institutions: ["University of Southern California"]
year: 2026
venue: "arXiv 2610.25947 (cs.AI), v1 2026-10-19, preprint (19 pages)"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.25947"
code_url: null     # no repository named in the PDF
citations: null
source: "raw/papers/khezresmaeilzadeh2026claim.pdf"
added: "2026-10-27"
relevance: 3
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/hce-evaluation]]"
tags: ["claim-laundering", "record-state-mismatch", "silent-failure", "false-success", "decision-model", "jev", "decider", "kev", "system-one", "counterfactual-pairs", "contrast-sets", "linear-probe", "activation-patching", "decodability-gap", "veto-gate", "evigate", "random-veto-control", "offline-online-transfer", "calibration-domain", "intent-written-records", "androidcontrol", "alfworld", "webshop", "t3a", "react"]
---

# Claim Laundering: When Agent Decision Models Trust Records Over Reality

## TL;DR

Agents increasingly hand small, repeated decisions (stop, proceed, buy) to
cheap one-pass "System-One" decision models such as hosted Jev or the open
decider and Kev models. Each decision reads a state built from the goal, the
agent's own record of what it did, and the current observation. The authors
build counterfactual pairs that change one input at a time. There are 6,175
states from 1,235 decision points across Android screens (AndroidControl),
ALFWorld and WebShop. They then ask which input moves the terminal decision.

Three results matter for this graph:

1. **The record moves the decision more than the observation does, in all 12
   model × domain cells.** After a failed step whose record claims success,
   the four main models still commit 84–96% (phones), 80–97% (ALFWorld) and
   53–100% (WebShop) of the time. That includes ALFWorld, where the
   observation literally reads "Nothing happens." With the false claim
   present, switching the observation from failed to successful moves log p
   by at most +0.24. The bias is already in the Qwen3.5-4B base model (claim
   +0.51 vs observation +0.02), so it predates decision tuning.
2. **The information is there and the output ignores it.** On phones, linear
   probes on intermediate hidden states separate worked from failed steps at
   AUROC 0.74–0.79 on claimed states, where the model's own output is
   0.51–0.61. Activation patching puts the observation at the decision
   position by mid-depth and the record's influence in the last layers. In
   ALFWorld and WebShop the probes hit 1.00, but so does a text-only
   classifier, so there the failure is plain in the input.
3. **A middle-layer veto helps live agents only when fit on the deployed
   agent's own states.** EviGate (a dot product at layer 20, 0.04–2.0%
   latency) raised live success by +5.1 (phone simulator), +18.7 (ALFWorld)
   and +5.0 (WebShop) points over no gate. Against random vetoes at the same
   rate, the gains are +4.7, +3.0 [−1.5, 7.5] and +9.0. The same gate fit on
   offline contrast sets vetoed 95% of WebShop purchases.

For this box the veto is not portable, since it needs weights. The portable
results are that an agent's own record outweighs the world for a decision
component, and that records written from intent are where the falsehood
enters.

## Claims

- Records are "written from the action's *intent*, often before its effect
  can be checked." A truthful log of an *attempted* action is enough to
  mislead: the model reads the attempt as evidence of success. "One that
  follows the record instead turns the agent's claim into the decision; we
  call this claim laundering."
- The failure is one of *evidence use*, not missing information. Relevant
  evidence is linearly decodable in middle layers ("decodability gap") but
  record-dependent influence dominates the final layers.
- Neither scale (0.8B–12B, plus hosted Jev) nor decision tuning
  consistently removes the bias. Kev-9B is a partial exception.
- EviGate needs no extra forward pass, improves task success in all three
  live domains, and beats a matched-rate random veto on phones and the web.
- Fine-tuning fixes the output in-domain but does not transfer across
  domains. Detection and fine-tuning both need labelled states from the
  target domain.
- Practical advice: "keep unverified records out of the decision state where
  possible, and otherwise check terminal decisions against what the model
  already encodes." The authors also ask hosted-model providers to expose
  intermediate states.

## Methods

- **Five-variant contrast sets** per decision point, goal/question/context
  fixed: DONE (logged, observation shows the effect), DONE+CLAIM, HONEST (not
  attempted, not logged, preceding state), LOGGED (logged, no effect),
  LOGGED+CLAIM (logged, no effect, false success claim). Effects are
  differences in log p of the commitment option:
  Δobs = L(DONE) − L(LOGGED), Δlog = L(LOGGED) − L(HONEST),
  Δclaim = L(LOGGED+CLAIM) − L(LOGGED). CIs are paired bootstraps over
  decision points or groups (1,000–2,000 resamples).
- **Domains (Table 1).** Phones: 3,200 states from AndroidControl (245 apps)
  in T3A's request format. The failed variant pairs the action record with
  the unchanged earlier screen; it is constructed, not an observed failed
  Android execution. Claims are one of four fixed sentences. ALFWorld: 1,475
  states from the hand-coded expert on 134 `valid_unseen` tasks (117
  solved), with "Nothing happens." as the failure and a ReAct-style "think:
  Now I have done …" as the claim. WebShop: 1,500 states, where failed
  variants keep the logged clicks but show the options unselected.
- **Models.** decider-4b, Kev-4B, decider-12b (Gemma-4) and hosted Jev
  (`jev-1.13-20260917`) in all three domains. On phones also decider-2b,
  Kev-0.8B, Kev-9B, Laya (ModernBERT encoder), and Qwen3.5-4B base/chat.
- **LLM-written records.** Qwen3.5-9B writes the final step summary with
  T3A's own prompt for 200 phone failure episodes. Whether a summary
  "claims success" is judged by the same model with a yes/no question and
  was not checked by humans.
- **Probes.** ℓ2 logistic regression on the answer-position hidden state
  every second layer, GroupKFold by app/task/goal, layer chosen within
  training folds. Baselines: the model's own score and a TF-IDF text-only
  classifier.
- **Patching.** Replace the decision-position state at one layer with the
  paired state's and measure how much of the log p difference is restored.
  decider-4b (about 100 pairs per domain, |Δlog p| > 0.2) and decider-12b
  (48–94 pairs).
- **EviGate.** s = σ(wᵀh_ℓ + b) at layer 20; if the proposed option is
  terminal and s < τ, take the best non-terminal option instead. τ maximizes
  balanced accuracy on held-out calibration states.
- **Live agents.** All use decider-4b as the decision model.
  - Phone *text simulator* (no KVM for AndroidWorld): 24 tasks × 30 seeds =
    720 trials per condition. The decisive tap fails 40% (first try) / 20%
    (retries), and the record generator says "goal – done" after 70% of
    failures. The gate is fit on 384 template states.
  - ALFWorld: Qwen3.5-9B writes thoughts; 134 unseen tasks, one run each.
    Gate fit on 148 states from 100 train games, τ = 0.158.
  - WebShop: Qwen3.5-4B writes thoughts; 200 goals, one run each. Gate fit
    on 480 states from 300 other goals, τ = 0.229.
  - Silent failures are *injected* (30% of actions or option clicks).
  - Arms: no gate, second yes/no question, random veto at the gate's
    measured rate, EviGate fit offline, EviGate fit on target-domain states.
- **Fine-tuning.** Rank-16 LoRA on decider-4b (0.8% of parameters), one
  epoch, seed 0. Commit BCE plus a KL term keeping non-commit preferences.
  Three training mixes; general accuracy checked on HWU64 and DBpedia.
- **Compute.** 2× RTX A6000; about 20 GPU-h offline, 60 GPU-h live, 5 GPU-h
  fine-tuning.

## Results

- **Which input moves the decision (Table 2, LOGGED+CLAIM "Wrong").**

  | Domain | Wrong (4 models) | Obs. effect | Largest record effect |
  |---|---|---|---|
  | Phones | 84–96% | +.05 to +.61 | +.32 to +2.31 |
  | ALFWorld | 80–97% | +.28 to +1.19 | +.32 to +4.41 |
  | WebShop | 53–100% | −.09 to +.20 | +.31 to +3.71 |

  Eleven of the 12 cells are ≥80% wrong; the 53% is Kev-4B on WebShop. In every
  cell the larger record effect exceeds the observation effect with a paired
  95% CI above zero. The tightest is Kev-4B ALFWorld, claim − obs +0.04
  [0.01, 0.07], where the logged-action effect is in fact *below* the
  observation (−0.03). With the claim present, obs | claim is +0.00 to +0.24
  (Table 7).
- **LLM-written summaries.** 78 of 200 (39%) summaries of no-effect steps
  claimed success, even though the summarizer saw the unchanged screen and
  was told to "be critical". Given such a summary, decider-4b / Kev-4B / Jev
  chose `complete` 77% / 94% / 80% of the time, against 26% / 48% / 27% with
  an honest one (+46–53 points). This measures summarizer behaviour on
  constructed failures, not deployment prevalence.
- **Scale and tuning (Tables 8, 11).** For models up to 4B the claim
  outweighs the screen. Kev-9B weights the screen most (+0.34 vs +0.27 /
  +0.24) yet still completes after more than 65% of claimed failures. Decision
  tuning *raised* decider's logged-action effect from +0.15 (base) to +1.12.
- **Probes vs output (Tables 3, 9).** Phones, claimed states: head
  0.51–0.61 vs probe 0.74–0.79 for the four main models (gap +0.14 to +0.15
  on all states, CIs above zero). Smaller models gain less (Kev-0.8B +0.04
  [−0.01, 0.09], not significant). Text-only classifier 0.59. ALFWorld and
  WebShop probes 1.00, text-only also 1.00. Ten labelled phone episodes
  already beat the output. A probe trained on synthetic template states
  reaches only 0.63 on real phone screens.
- **Patching (Fig. 2, Table 14).** decider-4b at layer 24 of 32: observation
  60% (phones) / 72% (ALFWorld) restored vs claim 25% / 21%, logged web
  click 12%. Nothing before layer 16 matters. decider-12b shows the same
  order. Steering the answer position raised accuracy on claimed-unfinished
  synthetic states only from 0.48 to 0.70; the middle-layer probe reached
  0.89.
- **Removing the record (Tables 12, 13).** Deleting record tokens from layer
  12 onward cut ALFWorld premature advancement 59% → 26% but barely moved
  WebShop (99.7% → 93%); from layer 20 it changed nothing in either. Deleting the record from the input is worse than the gate:
  - On phones it vetoes 93% of correct completions (the model loses track of
    progress).
  - In ALFWorld and WebShop it passes 9.5% / 6.0% of failed steps and vetoes
    13% / 7.5% of good ones, against 0.5% / 0% and 0% / 0.5% for EviGate.
- **Offline gating (Table 4).** Phones balanced accuracy EviGate 0.71–0.73
  vs own score 0.59–0.63 and second question 0.60–0.62. For decider-12b the
  wrong-stop rate stays 33% → 33%. ALFWorld 0.99–1.00, WebShop 1.00. A
  second question helps only in ALFWorld (0.75–0.99), where failure is an
  explicit phrase.
- **Live agents (Table 5; paired 95% CIs over tasks).**

  | Condition | Phone sim | ALFWorld | WebShop |
  |---|---|---|---|
  | No gate | 77.2 | 42.5 | 8.5 |
  | Second question | **83.2** | **61.2** | 7.0 |
  | EviGate, offline-fit | – | 58.2 | 2.5 |
  | EviGate, target-fit | 82.4 | **61.2** | **13.5** |
  | Random veto, matched rate | 77.6 | 58.2 | 4.5 |
  | EviGate − no gate | +5.1 [1.8, 8.9] | +18.7 [11.9, 26.1] | +5.0 [1.0, 9.5] |
  | EviGate − random veto | +4.7 [1.7, 8.2] | +3.0 [−1.5, 7.5] | +9.0 [4.5, 13.5] |

  In ALFWorld 78.6% of gated steps follow a real failure, so a random veto
  recovers most of the gain. After a vetoed purchase the WebShop agent buys
  less (46 vs 78 purchases). In the simulator a separate 9B LLM judge reached
  81.5% at 6.18 calls per success, against EviGate's 4.86. The second
  question costs 93–98% extra latency.
- **Best-of-16 (WebShop, 50 goals).** The probe picks a successful attempt
  40.0% of the time against 28.0% for the model's own judgement (+12.0
  [2.0, 22.0]); the oracle gets 44.0%.
- **Fine-tuning (Table 6).** Training on phones only cut phone wrong
  decisions 91% → 47% (AUROC 0.50 → 0.84), with ALFWorld and WebShop
  unchanged (91%, 100%). With 1,000 states per domain, ALFWorld and WebShop
  went to 0% but phones only to 89%. Adding all 2,755 phone states gave
  47% / 0.3% / 0% with general accuracy unchanged. Offline only.

## Critique / open questions

- **Against the digest framing: below framing (mild).** Every number in the
  digest summary is in the PDF and stated correctly. The "Why" (the finding
  transfers, the veto needs weights) is right. The digest leaves out four
  things that change how far the numbers reach.
  1. **"Probes detect the failure (0.77–1.00)" is two different results.**
     The 1.00s in ALFWorld and WebShop are matched by a TF-IDF text-only
     classifier, so there the input states the failure plainly and the
     probe shows only that the model saw it. The real decodability-gap
     evidence is phones: about 0.77–0.80 against an output of 0.63–0.69 and
     text-only 0.59. The 0.77 floor also covers only the four main models.
     Kev-0.8B, decider-2b and Laya reach 0.62–0.65.
  2. **"The veto adds 5–19 pts" is against no gate.** Against a random veto
     at the matched rate the ALFWorld gain is +3.0 with an interval through
     zero. The headline +18.7 there is mostly the value of vetoing at all,
     because most gated steps really did fail. A second yes/no question tied
     or beat the gate in the simulator (83.2 vs 82.4) and in ALFWorld
     (61.2 vs 61.2). The gate is uniquely useful only in WebShop, from an
     8.5% base.
  3. **The live setting is narrow.** All three live agents use decider-4b as
     the decision model. Silent failures are injected at 20–40%. ALFWorld and
     WebShop are one run per task, and the phone arm is a text simulator with
     a stochastic record generator, not AndroidWorld.
  4. **The phone failures are constructed** by pairing a record with the
     earlier screen. The 39% false-summary rate was decided by the
     summarizer model itself, with no human check.
- **The phone observation is weak evidence on its own.** A text-only
  classifier on the screen gets 0.59. A model that leans on the record when
  the screen barely separates success from failure is not obviously
  irrational there; real logs usually mean real success. The case is strong
  in ALFWorld and WebShop, where the failure is explicit ("Nothing
  happens.", options visibly unselected) and models still commit 80–100%.
  Read the paper's headline from those two domains, and its probe headline
  from phones.
- **The counterfactuals measure influence, not base rates.** The authors
  say so. Δlog and Δobs are taken from different baselines (HONEST vs DONE),
  so "record > observation" compares mean shifts, not a decomposition of one
  decision. The obs | claim column (≤ +0.24) is the cleaner statement: once
  the claim is there, the world barely matters.
- **"Removing the record" is the result most worth carrying over, and it
  cuts against a simple reading of the advice.** On phones the record is the
  only place progress lives, and deleting it vetoes 93% of correct
  completions. The paper's own advice ("keep unverified records out of the
  decision state where possible") therefore works only where the
  observation encodes progress. Our inference: what helps is separating the
  *attempt* from the *claimed effect* inside the record, not deleting the
  record. No arm tests that. The nearest is DONE vs LOGGED, which shows that
  even a truthful attempt entry moves the decision (Δlog +0.20 to +4.41 in
  all 12 cells).
- **Offline-to-live non-transfer is a second datum for the same pattern.**
  The gate fit on offline contrast sets vetoed 95% of live WebShop purchases
  (2.5% success) and was no better than random in ALFWorld. A synthetic-trained
  probe reached only 0.63 on real screens. That matches
  [[literature/papers/vats2026good]], where Jev's offline AUROC (0.78) did not
  predict online value. A detector's offline score is not evidence about the
  deployed loop.
- **Fit with the rest of the graph.**
  - The same failure in an LLM judge: [[literature/papers/xu2026dont]] showed
    that a supportive log decides a video judge's verdict even with the
    frames in view, and that the dangerous line is a claimed check.
    [[literature/papers/sun2026scipivot]] showed a frontier Claude model
    siding with a claimed check in STATE.md against the code.
    [[literature/papers/sanayei2026evidencefirst]]'s trace-only judge
    believed the account where it contradicted the state. This paper adds
    (a) the same pattern in sub-12B typed decision models and a hosted one,
    (b) a *truthful attempt entry* as a trigger, not only a false claim,
    and (c) layer-wise evidence that the observation is represented and then
    overridden. Neither xu2026dont nor sun2026scipivot had (c).
  - [[literature/papers/zhang2026are]] ("noticing without acting") found
    text-level evidence inside the trajectory being ignored by the answer.
    This is the representation-level version.
  - [[literature/papers/hu2026memory]] found stale stored facts overriding
    current evidence, worst in larger models. Here scale helps a little
    (Kev-9B) but does not fix it.
  - [[literature/papers/qin2026llm]] shows the record is agent-writable.
    This paper shows that even an honest record, written from intent, is
    enough.
- **Self-citation and throughput.** Azizi et al. 2026a/b (option-name and
  one-word-gate attacks on Jev-style models) and Fayyazi et al. 2026
  (hidden-state probes forecast unauthorized actions) are the same USC group
  (Pedram, Kamal) in the same two months. The probing method is shared with
  Fayyazi et al. That does not make the results wrong. It does mean the
  "decision models fail in several ways" literature is largely one lab.

## Trust signals

- **Credibility:** 3.
  - For:
    - The five-variant counterfactual design isolates each input.
      Group-held-out evaluation (GroupKFold by app/task/goal) and paired
      bootstrap CIs are reported for every effect, including the tight cells.
    - Seven open models from three families plus a hosted one; base vs
      tuned backbone comparison.
    - Matched-rate random-veto and second-question baselines in the live
      runs, with negative results reported: offline-fit gate failure, the
      ALFWorld random-veto tie, second-question wins.
    - A text-only baseline that honestly deflates the 1.00 probe results.
      Latency is measured on a real serving engine.
    - The limitations section names injection, the simulator and
      weight-access dependence.
  - Against:
    - Not peer reviewed, no code or data release named in the PDF,
      8 days old.
    - Live agents use one decision model and one run per task, with
      injected failures. The phone live arm is a simulator.
    - The phone failure states are constructed. The false-summary label is
      auto-judged by the summarizer model.
    - The surrounding Jev-failure literature is heavily the same group's.
  - Not 4 because there is no release, no peer review, and the live evidence
    rests on one 4B decision model under injected failures. Not 2 because
    the offline design is careful and well-controlled, the CIs are paired and
    honest, and the paper reports the controls that weaken its own method.

## Follow-up

- **Relevance:** 3. This box does not put a typed decision model in its loop,
  and the fix needs weights. The finding is portable, though, and it
  sharpens four concepts:
  - [[concepts/evidence-gated-completion]]: deleting the account is not
    free when the observation does not encode progress.
  - [[concepts/typed-claim-partition]]: an *attempted* entry and a
    *took-effect* entry are different types, and an untyped attempt is
    already read as success.
  - [[concepts/verified-memory-writes]]: records written from intent before
    the effect is observed are where the falsehood enters.
  - [[concepts/hybrid-model-backends]]: cheap decision models inherit the
    record bias from their base LLM.

  Not 4: no ML-research loop, no frontier model, and the mechanism story
  repeats xu2026dont and sun2026scipivot at a lower level rather than
  changing what this box should do.
- **For this box (our inference).** The analogue is a session that reads
  its own `NOTES.md` "Did" lines, `log.md` entries or a subagent's "done"
  report and decides to move on. Those are records written from intent.
  The cheap transferable rule is typing, not deletion: log an action as
  *attempted* until its effect has been re-observed (a passing check, a
  file that exists, a pushed commit hash). A terminal decision ("wrap",
  "mark experiment done") should read the effect, not the entry.
- **For the `hce-known-bad-preflight` proposal.** The LOGGED+CLAIM variant is
  a known-bad preflight for a *decision* component: a state whose record
  claims success and whose observation shows failure has a known right
  answer. This is the same logic as scoring a do-nothing submission against
  an evaluator. It adds no evidence to the proposal itself, which is about
  evaluators. It does show the "feed a known-bad case first" pattern from a
  fifth group, aimed at a decision component instead of a scorer.
- **Candidates** (cited, not yet in the graph):
  - Ling, Yu, Chen & Fang 2026, "When faster decisions do not make better
    agents: an empirical study of Jev in agent workflows" (arXiv
    2610.13092). Swapping an LLM action selector for Jev cut AndroidWorld
    success by 13–18 points, mostly from premature completion. This is the
    end-to-end cost the present paper explains.
  - Hu 2026, "Silence is endorsement: verification-status laundering in LLM
    agent pipelines" (arXiv 2609.20211). Monitors approve risky actions when
    an upstream summary drops an "unverified" marker. A typed-claim datum.
  - Advani 2026, "From confident closing to silent failure" (arXiv
    2606.09863). 45–76% of failures on τ²-bench and AppWorld are false
    successes. Already a candidate under park2026when and zhang2026how.
  - Fayyazi, Kamal & Pedram 2026, "The agent knew: internal states forecast
    unauthorized actions and stop them, and the lies that follow" (arXiv
    2610.17301).
  - Wang, Wang & Wu 2026, "The unreliable progress bar" (arXiv 2609.08589).
    Agents' self-reported progress during execution.
  - Mammen et al. 2026, "Do agents know when they succeed?" (arXiv
    2609.09448), and Ruan et al. 2026, "Doomed from the start" (arXiv
    2607.06503). Hidden-state success and failure probes.
