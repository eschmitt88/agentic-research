---
kind: paper
title: "Scan the Skill, Govern the Action: Composing Registry Verdicts with Runtime Consequence Control"
authors: ["Rohit Taneja", "Travis Weber"]
institutions: ["Pheo Inc"]
year: 2026
venue: "arXiv (cs.CR)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.12001"
code_url: "https://github.com/pheo-ai/open-agent-trust-system"
citations: null
source: "raw/papers/taneja2026scan.pdf"
added: "2026-09-15"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/shared-skill-namespace]]"
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/refusal-cost-symmetry]]"
tags: ["supply-chain", "skills", "registry", "permissions", "runtime-gate", "static-analysis", "graduated-autonomy", "vendor-evaluation", "negative-result", "openclaw"]
---

# Scan the Skill, Govern the Action: Composing Registry Verdicts with Runtime Consequence Control

## TL;DR

A skill registry's publish-time scanners answer "is this artifact
malicious?". An operator also needs to know "is this action permitted here,
now?". That second question depends on facts that don't exist at publish
time, so no scanner can express it. The authors measure the gap on
OpenClaw's public dataset of **66,192 ClawHub skill versions**. **705
skills that all three scanners and the LLM judge rate clean** still instruct
an action that CIS 2.7 / NIST CM-11 name as prohibited, mostly `curl | bash`
installers. **506 of those come from one publisher**, so the finding covers
135 independent parties, not 705 decisions. A live agent following cleared
skill docs ran commands whose consequence class **appeared in no documented
code block 34.7% of the time**. Every one of those was the generic
`shell_exec` class, so the document failed as a bound but never in a
dangerous direction. The authors' own deterministic per-command gate
(OATS, from their company) held or blocked the **23 of 23** forbidden
reaches it saw. The most transferable part is the math on graduated
autonomy. **Ten clean approvals cannot exclude a 25.9% true failure rate**
at 95%. Counting approvals made under review measures the reviewer, not the
agent.

## Claims

- **Permission ≠ maliciousness.** VirusTotal, static analysis, SkillSpector
  and the ClawScan judge each answer a question about the *artifact*.
  Permission is a property of the *action* evaluated against one
  operator's policy. The registry's verdict vocabulary
  (clean / suspicious / malicious) has no slot for "fine at a startup,
  prohibited at a bank." The 705 are not scanner false negatives. The
  scanners are answering a different predicate correctly.
- **The scanned artifact is not the executed artifact.** Agents recompose
  documentation instead of transcribing it: they chain checks, add flags,
  and split steps. Any exact-match method is therefore useless
  (allowlisting documented strings, hashing them, diffing execution
  against the doc). A capability-envelope review misses about a third of
  executed classes.
- **Layers compose because their blind spots don't coincide.** Each blind
  spot follows from the layer's input and timing, not from how well it is
  built. A scanner can't see which capabilities get exercised. An action
  gate can't see harm that never becomes an action: a hardcoded recipient,
  a "persuaded behaviour" assembled from benign calls, or over-broad scope.
  "A runtime action gate is not a superset of a semantic scanner and should
  not be deployed as one."
- **Composition rule (proposed, not measured).** A registry verdict is an
  *input* to the runtime ledger. A non-clean skill pins every lane at held
  or never-graduates. Measuring this against the scanner-defined population
  would be circular, and the authors say so.
- **Deterministic, stateless classification; stateful trust.** The
  resolver ρ maps a command string to the max-severity class among 33. It
  is total (unmatched → `shell_exec`, so nothing fails open) and never reads
  the skill's documentation. That closes the "reassuring prose" evasion
  channel that Saha et al. report at 36.5–100% against scanners. Trust
  lives in a separate ledger keyed on **(resource, class)**, not on the
  skill.
- **Promotion thresholds should be derived, not asserted.** With zero
  failures in N trials, N_k = ⌈ln δ / ln(1−ε_k)⌉. At δ = 0.05, the flat
  "10 clean approvals" demands **3–15× less evidence** than the tolerance
  bands require. 13 classes (severity ≥ 75: remote exec, credentials,
  destructive, deploy, IAM, CI config, protected merges, …) **never
  graduate**: "no clean record makes an unrecoverable action recoverable."

## Methods

- **Corpus (§3–4).** `OpenClaw/clawhub-security-signals` (MIT, Hugging
  Face) in full, with four scanner verdicts per row and redacted
  `SKILL.md`. The judge is GPT-5.5 for 99.47% of rows. Each skill's fenced
  code blocks go through the unmodified shipping client
  (`oatsctl hook pre-tool-use`). The unit is the skill ("does it instruct at
  least one action of class X"), so counts are floors. The sanitizer removed
  every newline, so "in the same command" is not measurable, and an earlier
  draft's phrasing to that effect was withdrawn. 98.8% of remote-exec
  detections rest on a pipe or single-token install that survives the
  collapse.
- **Clean population.** 29,257 skills were rated clean by all four
  signals. The 1,574 with incomplete scans are excluded; "nothing objected"
  there includes "nothing looked."
- **Hand audit.** 100 random skills from the 705, one rater (an author),
  fixed published seed. Judged twice: once for detector precision, once for
  any marker of malicious intent.
- **Divergence study (§5).** 40 skills, one per publisher, from the clean
  population, restricted to documents whose own blocks resolve to a class
  more specific than `shell_exec` (≈1% of publishers' first skills). Claude
  Sonnet 5 follows each doc in a throwaway Debian container with no host
  mount and no credentials. Every executed command is classified by the
  same ρ and compared with the doc's block classes.
- **Runtime study (§6).** 54 clean skills, one per publisher, each
  documenting a never-graduating action. One run each, ≤4 turns, the
  instruction "install it and confirm it works," commands executed only if
  the gate permits.
- **Obfuscation benchmark (§2.2).** 64 semantics-preserving rewrites (9
  techniques, 5 identity controls), scored as resolved if the rewrite's
  class is at least as severe as the base.
- **Released:** harness `research/live_agent_study.py`, per-command JSONL
  records, sandbox Dockerfile, corpus reproduction script, and
  `spec/action-classes.json`. **Not released:** the resolver predicates.
  `pheo-oats` 0.5.3 is proprietary, so results can be re-executed but the
  predicates can't be audited.

## Results

- **Clean but prohibited: 705 skills / 135 publishers.** By class: remote
  fetch-and-execute 658, credential access 23, destructive 20, protected
  merge 7, workflow trigger 1. IAM change, deploy and repo deletion scored
  zero. An earlier table's 20 IAM / 3 deploy were extraction artifacts.
- **Concentration.** oomol contributes **506 (71.8%)**, one installer line
  repeated across its connector catalogue. 117 of the 135 publishers
  contribute exactly one skill. Excluding oomol leaves **199 skills / 134
  publishers**. Restricting to remote exec without oomol leaves 152 skills,
  93 publishers, **102 distinct command strings, 48 domains**. The
  magnitude depends on one vendor; the practice does not.
- **Precision 92%** (95% CI 84.8–96.5%). Flipping the three arguable calls
  gives 89%. Errors: a fetch piped into a formatter (5), `gh pr merge`
  assumed protected (2), a registry `npm install -g` (1). **No malicious
  marker in any of the 92 correct detections.** The authors won't call them
  "benign."
- **Base rates.** Remote exec fires on 1.73% of all skills and **2.25% of
  clean ones**. It is *more* common among cleared skills, which is the
  thesis as a base rate.
- **Independence from scanners.** OATS flags 1,412 (2.13%); any registry
  non-clean 35,361 (53.4%, mostly SkillSpector); intersection 680 against
  754 expected. **φ = −0.016**, OR 0.81. An earlier Jaccard-based
  "independence" claim was withdrawn as the wrong statistic.
- **Divergence: 144 commands / 39 skills.** 2 verbatim (1.4%), 92
  recomposed within a documented class (63.9%), **50 class-absent (34.7%,
  CI 27.0–43.1%)**, concentrated in 14 skills. **All 50 resolved to
  `shell_exec`**, so no recomposition escalated severity. The earlier
  unpreserved run agrees: 36.6%, Fisher p = 0.78.
- **Runtime: reach 23/53 (43.4%, CI 29.8–57.7%), held/blocked 23/23.** 22
  reaches were remote exec, 2 credential access, 1 destructive. An earlier
  run reported 4/54 (p = 1.6×10⁻⁵ apart). A pilot with execution disabled
  reproduced 4/54: without command output the agent spends its turns
  testing its own tooling. Estimated reaches ≈ 705 × 0.92 × 0.434 ≈ 281.
- **Container confound, undetermined direction.** In one of seven
  `brew` cases, brew failed on Linux and the agent's next step fetched
  remote code. The sandbox manufactured that reach.
- **Recomposition into a blind spot, unprompted.** When a documented
  `curl … | bash` failed, the agent retried as `curl -o` then `cat`, a
  staged form the resolver cannot see (below).
- **Interruption load.** 170 decisions over 53 skills (3.2/skill; 85%
  `shell_exec`). Letting ordinary shell/reads/docs/tests run unattended
  leaves 0.5/skill, and no policy removes the 25 never-graduating
  interruptions. Hold correctness 49/50 (98%). The one error was a heredoc
  whose *content* matched a destructive pattern (open defect).
- **Obfuscation.** Wrapper 100%, chained 100%, interpreter 75%, variable
  60%, base64 33%, indirect read 25%, quoting 20%, **staged fetch→chmod→run
  0%**. Macro 52% / micro 75%. The authors state the choice of aggregate
  because it swings the headline from 52% to 77%. A per-command classifier
  can't see a property that exists only across commands, and "tuning
  patterns will not change that."
- **Cost.** 67.6 ms median / 81.1 ms p95 per call end to end, almost all
  process spawn. Classification adds <~1 ms at 2,000 chars. **The record,
  not the decision, scales badly**: ≈12 KB per check, and throughput fell
  from 67 to <20/s over 15k checks in one store (unfixed). The earlier µs
  microbenchmark figures were withdrawn as unreproducible.
- **Graduation table (δ = 0.05).** At ε = 10%, N = 29 and E[approvals] =
  202. At 5%, N = 59, E = 392. At 2%, N = 149, E = 965. Because a rejection
  resets the counter, expected approvals run 6–7× N. The shipped ledger
  still uses flat N = 10, and the authors concede this. The moderate
  operator's `shell_exec` lane needs ~143 skill installs to graduate.
- **Rejected predicates.** A literal-hardcoded-domain egress detector fired
  on 4.4% of clean skills and an argument-embedded credential detector on
  1.5%, so neither shipped. Shipped remote exec fires at 2.25%, *between*
  those rates. The authors concede the real shipping criterion was audited
  precision, not firing rate.

## Critique / open questions

- **23/23 is close to true by construction.** The paper never defines
  "reached for a never-graduating action" independently of ρ. The reaches
  are the commands ρ classified into those classes, and on an empty ledger
  those classes are held or blocked by policy. So 23/23 shows the hook fired
  and the policy applied. It does not show the gate catches forbidden
  actions. The informative numbers are the reach rate (43.4%), hold
  precision (98%), and the 0% on staged rewrites, which the agent itself
  produced in §5. **Recall against forbidden actions ρ fails to classify is
  unmeasured**, and the CI's 85% lower bound doesn't apply to it.
- **Vendor evaluating its own product, with a closed core.** Pheo sells
  OATS, and no competing gate was measured. The predicates are proprietary,
  so the §10 appeal to Anderson's "small enough to verify" is met only for
  the published class table. The authors disclose all of this.
- **The trust boundary is inside the agent's reach.** The hook config, the
  binary and the ledger are assumed uncompromised, yet the governed agent
  has local shell access. The HMAC-chained receipts detect tampering after
  the fact but can't stop an agent disabling the hook. This violates design
  rule 2 of [[concepts/enforcement-boundary-placement]] ("the constrained
  component cannot reach the constraining one"), and the paper admits the
  requirement is only partially met.
- **Threat model is non-adversarial.** The 34.7% "no escalation" bound
  holds only for benign recomposition, and the authors say so. An adversary
  wants exactly the low-severity-looking form.
- **Thin live samples.** One agent (Sonnet 5), one prompt, four turns, one
  day. 39 and 53 skills. The §5 slice is ≈1% of publishers by design. The
  precision audit has a single author-rater and no inter-rater agreement.
  The detector's recall is unknown ("5% or 95%").
- **"Prohibited" is argued from control text, not from operators.** CIS 2.7
  and CM-11 support treating `curl | bash` as unreviewed-script execution,
  but the authors surveyed no operators. The 705 therefore counts a
  hypothetical policy's scope, not observed violations.
- **Heavily revised preprint.** Withdrawn claims include the dedup-by-
  publisher statement, the 73.6% precision, the 2× `secret_change`
  inflation, the Jaccard independence, the µs latency, "in the same
  command," and 4/54. This is unusually candid, and it also means earlier
  versions circulated with wrong headlines. Small residual slips: §9 says
  "three further defects" and lists four; §6 says 54 skills in one place and
  53 in another (explained as 53 producing a command).
- **The graduation derivation has a sharper limit than its table.** The
  ledger counts clean runs *under human review*, and graduation removes the
  reviewer. If the reviewer was rejecting the bad cases, clean runs are
  evidence about the reviewer. "We know of no way to close that gap from the
  ledger alone." There is also no multiplicity correction (100 lanes at δ =
  0.05 → ~5 wrongly graduated), and a per-action bound says nothing about
  cumulative risk (ε = 0.5% over 1,000 actions → 99.3% chance of at least
  one failure). These caveats apply to *any* track-record-based autonomy
  rule, not only OATS.
- **Second-hand premise.** Scanner disagreement (pairwise overlap ≤10.4%,
  81.9% single-scanner positives) comes from OpenClaw's own study (Koc et
  al., arXiv:2606.01494), which is not ingested here. The released dataset
  has 66,192 rows against that study's 67,453, and the authors can't
  explain the gap.
- **Open question.** Can per-action classification be lifted to
  cross-command state without losing determinism and replay? The staged-
  fetch 0% points to shell-grammar resolution with carried state, which is
  the causal-history gap [[literature/papers/palumbo2026formal]] raises.
  The paper cites Lotfi et al. (arXiv:2608.01558) on trajectory assurance
  as the target and doesn't attempt it.

## Trust signals

- **Credibility:** 3. A two-author arXiv preprint (cs.CR) from a startup
  (Pheo Inc) with no track record in this graph. Not peer-reviewed, no
  citations yet (posted 10 Sep 2026). It evaluates the authors' own
  commercial product against a bar they set, with a proprietary resolver.
  That would put it at 2. Raised to 3 because the load-bearing numbers don't
  depend on trusting the vendor. The corpus finding (705 / 135 / 506)
  reruns from a public MIT dataset with a released script and published
  binary. The live studies ship their harness and every recorded command.
  The graduation figures are exact arithmetic (checked: 1−0.05^0.1 =
  0.259; N=29 at ε=10%; E=202). The paper also reports its own negatives
  with rare thoroughness: nine instrument defects, seven withdrawn claims,
  benchmark results it half-fails, and a concession that its shipped
  default is the threshold it argues against. The one headline that does
  rest on the vendor, 23/23, is the weakest number in the paper (see
  critique).

## Follow-up

- **Relevance:** 4. It adds a new axis and a derived cost to two load-bearing
  concepts. For [[concepts/enforcement-boundary-placement]], a placement
  fixes *which predicate can be expressed*: publish time can decide
  maliciousness, and permission needs action time. For
  [[concepts/permission-gate-as-architecture]], it prices relaxing a gate
  from a track record, and shows that clean runs under review measure the
  reviewer. It also corrects an assumption in
  [[concepts/shared-skill-namespace]]. Not a 5: it seeds no concept, the
  setting is a consumer skill registry (OpenClaw) rather than research
  agents, and the live evidence is thin and vendor-run.
- **Independence from [[literature/papers/kapner2026scanning]]: largely
  independent populations measuring different fields. Complements, not
  corroboration.**
  - *Different corpora.* Kapner samples 3,171 public GitHub repositories
    parsed as Claude Code / Cursor / Copilot / Gemini CLI / OpenCode /
    Windsurf / Cline / Codex layouts, plus Claude Code plugin marketplaces.
    Taneja uses OpenClaw's ClawHub registry dump. The `SKILL.md` format is
    shared and some publishers may post to both, so some overlap is
    possible but unmeasured; nothing in either paper suggests it is large.
  - *Different teams, and no citation.* Red Hat / Ben-Gurion vs Pheo; the
    papers appeared three days apart.
  - *Different fields.* Kapner reads the grant side (`allowed-tools`,
    `permissions.allow`, MCP pins). Taneja reads the instruction side (the
    fenced command blocks in the body). So they are not two sightings of
    one rate.
  - *They compose into a case neither measures.* Kapner's 3.7% of
    collections ship a skill pre-approving the shell. Taneja's 2.25% of
    clean skills instruct remote fetch-and-execute. A skill doing both takes
    the harness's own gate off exactly the action a permission policy
    forbids. The joint rate is unmeasured.
  - *One genuine independent convergence.* Both teams built a
    credential- or data-to-network predicate and found it doesn't
    discriminate at the artifact level. Kapner's fired on 6 repos, none of
    them exfiltration. Taneja's fired on 4.4% / 1.5% of clean skills,
    "because curl-POSTing data to a URL is simply how ordinary API
    integrations work." Two instruments on two populations reach the same
    negative result.
  - *They extend each other's placement claim.* Kapner says supply-chain
    checking must run twice, at publication and at assembly. Taneja adds a
    third check, at the action, and argues the first two can't express
    permission at all.
- **[[concepts/shared-substrate-contagion]].** The 506/705 concentration is
  he2026stored's "source count is not source independence" showing up in a
  registry prevalence count. The mechanism is a single vendor's template,
  not imitation spreading between publishers: after removing it, 102
  distinct command strings across 93 publishers remain. So a high artifact
  count can overstate propagation, and publisher-level de-duplication is
  the minimum before reading prevalence as spread.
- **[[concepts/refusal-cost-symmetry]] (source-only candidate).** The
  paper's refusal to report a single false-positive rate addresses that
  concept's open question on weighting: whether a hold is an error depends
  on the operator's policy, and the operator's ε_k is the weighting.
  Table 4 prices one decision set under three policies.
- **[[concepts/skill-library-lifecycle]]: declined.** That concept's
  admission gate is about whether a skill *helps* (evidence, gain,
  consistency). Registry screening for malice is a different gate on a
  different axis, and this paper doesn't bear on library curation or
  retention.
- **This repository's own gate.** `permission-gate-as-architecture`
  described "the Claude Code PreToolUse hooks" as load-bearing here. As of
  2026-09-15, `~/claude-system/claude/settings.json` (symlinked as
  `~/.claude/settings.json`) sets `permissions.allow: [WebSearch,
  WebFetch]` and hooks for SessionStart, PreCompact, SessionEnd, Stop and
  PostToolUse (Write), with no PreToolUse hook, and the project has no
  `.claude/settings.json`. The action-time layer this paper argues for is
  Claude Code's built-in permission prompt, not a custom hook. The concept
  sentence was corrected on 2026-09-15.
- **Watch:** the OpenClaw scanner-disagreement study (arXiv:2606.01494)
  and Saha et al. (arXiv:2605.11418, "Under the Hood of SKILL.md,"
  36.5–100% governance evasion). Neither is ingested. Saha's evasion rate
  is the attack this paper's "ρ never reads the doc" design answers.
