---
kind: paper
title: "Assay: Claims That Decay With the Code. Content-Addressed Evidence Graphs for Accountable AI-Assisted Software Delivery"
authors: ["Om Shankar Tiwari", "Tangi Vass", "Gagan Deep Singh"]
institutions: ["Google (first author's listed role: Applied AI Technical Lead; not presented as a Google project)", "OMNI3ai (Lyon)", "Glinr Studios / theSVG.org (Texas)"]
year: 2026
venue: "arXiv 2609.36170v1 (cs.SE), 2026-09-28; 8 pages"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.36170"
code_url: "https://github.com/OmShiv/assay-research"
citations: null
source: "raw/papers/tiwari2026assay.pdf"
added: "2026-10-06"
relevance: 3
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/typed-enforcement]]"
tags: ["evidence-freshness", "content-addressing", "merkle-hash", "dependency-cone", "staleness", "blast-radius", "merge-gate", "model-free-gate", "separation-of-duties", "evidence-monotonicity", "signed-ledger", "repository-brief", "coding-agents", "no-model-in-loop"]
---

# Assay: Claims That Decay With the Code. Content-Addressed Evidence Graphs for Accountable AI-Assisted Software Delivery

## TL;DR

Assay is a tool plus a small formal model. Every claim an agent makes about
code ("tests-pass", "lint-clean", "no-secrets", "behavior-preserved",
"human-approved") is recorded in a signed append-only ledger. The claim is
keyed to the **cone hash** of its subject modules: a Merkle hash over the
condensation DAG of the module import graph. A claim is fresh iff those
hashes are unchanged. Three short propositions show that, relative to the
extracted import graph, this binding is sound (any change in the cone makes
it stale) and minimal (no change in the cone leaves it fresh). The set of
claims a change invalidates equals its blast radius.

On top sit a risk-proportional review obligation (levels 0–3), a
Liza-style review protocol (attest / accept / counter / contest / escalate,
no self-approval, humans resolve escalations), and a merge gate. The gate is
a pure function of the index, the ledger and the changed modules. It checks
coverage, freshness, separation, review status, exit code, plausibility,
evidence monotonicity and HMAC signatures, and consults no model.

The evaluation has **no model in the loop**. It is five scripted experiments
on five public repositories (Express, Flask, Gin, Axum, FastAPI) plus Assay
itself. Whether agents do better with Assay is explicitly out of scope and
left to a designed but not yet run SWE-bench Verified study.

For this graph, the transferable content is the binding rule: **evidence is
keyed to the hash of what it depends on, not to a time or to the named
subject alone.** Also useful is E3's measurement of the cost of getting the
binding grain wrong.

## Claims

- **Context and accountability are one graph at two times.** "A map knows
  that module A depends on module B, but not that the tests covering A last
  ran before B changed." Keying claims to cone hashes lets "one graph serve
  both moments": the agent reading before it edits, and the gate deciding
  after.
- **Staleness becomes a hash comparison.** "'has this gone stale?' becomes
  a hash comparison rather than a judgment." Hashing only the named module
  is unsound. Hashing the whole repo is "sound but useless". The cone is
  the right key.
- **Propositions 1–3.** Soundness, minimality, and blast radius = staleness
  frontier. The proofs are sketches. The paper says they "hold up to hash
  collisions", and they are stated over the import relation the indexer
  extracts.
- **Invariants live in code, not prompts** (borrowed from Liza): "An
  attestor cannot review their own claim. A stale claim cannot be
  accepted." Status is derived from the ledger on every read and never
  stored.
- **The gate is a floor, not a sufficiency check.** It "does not guarantee
  the evidence is sufficient. A reviewer can accept a weak suite, two
  models from one provider can share a blind spot, and a doer that can read
  the signing key can forge the ledger."
- **Evidence monotonicity** is "the mechanical counterpart of the contract
  clause 'do not delete the failing test'". The number of tests that ran
  may not drop below earlier accepted evidence unless a fresh human-approved
  claim covers it.
- **The claim is deliberately narrow.** "The paper does not measure how much
  better language-model agents perform with Assay than without … The claim
  here is narrower and more durable: the floor that holds no matter what
  the model decides can be made cheap and precise."

## Methods

- **Index.** Built with `git ls-files`. Modules are directories up to a
  configurable depth. Python imports are parsed with the stdlib AST. Seven
  other language families (TS/JS, Go, Rust, Java/Kotlin, Ruby, C) use
  "conservative regular expressions" to recover imports. The index also
  stores 90 days of git history and per-language test/build/typecheck
  commands. A per-file (size, mtime) cache gives Merkle-skip rebuilds, and
  a pre-commit hook keeps the committed `assay.json` current.
- **Risk** r(m) = ½·min(1, commits_90d/20) + ½·min(1, fan-in/10). The
  obligation level is a function of the radius fraction, the radius size
  and the maximum risk inside the radius:
  - L0: tests on the changed modules;
  - L1: tests on the whole radius;
  - L2: adds lint, a secrets scan and adversarial review;
  - L3: adds a human-approved claim.

  The thresholds are "policy … We claim only that they are explicit,
  monotone in radius and risk, and cheap."
- **Ledger.** JSON lines under `.assay/`, committed. Each line is
  HMAC-SHA256 signed with a key read from the environment or a git-ignored
  file. "In a supervised deployment the process that executes the agent's
  tool calls holds the key and the agent never sees it." That pattern is
  described, not enforced (§VIII "Key placement").
- **Interfaces.** A CLI (`init/index/brief/blast/attest/review/respond/gate/verify/mcp`),
  an MCP server over stdio, and `assay gate --base origin/main` for CI.
  The implementation is 2,313 lines of Python with no runtime dependencies
  and 30 tests.
- **Brief.** About 600 tokens covering hubs, hot spots, commands, the
  current change's radius and obligation, stale claims, open findings and a
  fan-in-ordered module list, trimmed to the token budget.
- **Evaluation.** Five scripted experiments, no model:
  - E1: orientation tokens for full dump vs exploration proxy vs index vs
    brief. The proxy is a directory listing plus the ten most-imported
    files.
  - E2: cold vs warm rebuild time.
  - E3: 100 random sets of k edited modules, k ∈ {1,2,4,8,16,32}. Compares
    repo-hash, self-hash and cone-hash binding on the fraction re-verified
    and on required invalidations missed.
  - E4: 12 scripted doer/reviewer scenarios on a 12-file Python project.
  - E5: the distribution of single-module blast radius and obligation
    level.

  Tokens are counted with tiktoken cl100k_base. Every figure is generated
  by `paper/make_numbers.py`. There is also one dogfooding change on a
  private TypeScript Electron app.

## Results

- **E1 (Table I): brief vs exploration proxy.** The gain is 14× (Gin) to
  114× (FastAPI), and against a full dump it is 216× to 1225×. The brief is
  592–635 tokens on the external corpora (434 on Assay). It is "flat in
  repository size by construction" because it is trimmed to a budget.
- **E2 (Table II): warm rebuilds.**
  - With nothing changed, a warm rebuild is 1.7× to 5.4× faster than a cold
    one.
  - With one edited file it is 1.6× to 4.8× faster (median ≈ 1.85×).
  - Absolute times: cold builds take 45–508 ms. On FastAPI (1,142 files) a
    one-file rebuild takes 106 ms against 508 ms cold.
- **E3 (Table III, k = 4): fraction of claims re-verified.**

  | Repo | Modules | Repo hash | Cone hash | Self hash | Self-hash misses |
  |---|---|---|---|---|---|
  | express | 35 | 100% | 16.6% | 11.4% | 27.8% |
  | flask | 16 | 100% | 66.7% | 25.0% | 45.7% |
  | gin | 7 | 100% | 81.9% | 57.1% | 29.3% |
  | axum | 90 | 100% | 66.3% | 4.4% | 67.6% |
  | fastapi | 195 | 100% | 7.9% | 2.1% | 22.7% |
  | assay | 5 | 100% | 87.4% | 80.0% | 7.4% |

  "The spread is structural". FastAPI's graph is wide and shallow, while
  Axum's workspace crates form a deep chain.
- **E4 (Table IV).** All 12 scenarios end as designed. The 3 honest paths
  pass, including one escalated case that a human resolved. The 9
  adversarial ones each fail on the mechanism built for them: coverage
  ×2, freshness, separation, signature, exit code, plausibility,
  monotonicity, and review. In the "failing test deleted" scenario a
  reviewer who did not look accepts the claim, and the gate still fails it
  on monotonicity.
- **E5 (Table V): single-module blast radius.**
  - The median radius runs from 0.5% (FastAPI) to 57.1% (Gin).
  - The p90 radius reaches 91.1% on Axum, where a handful of hubs sit under
    everything (axum-macros/src is reached by 84 modules).
  - Under the obligation policy, Axum sends 24 of 90 modules to L3 and
    FastAPI sends 8 of 195.
- **Dogfooding.** One real change went through attestation, review and a
  passing gate, and surfaced "two obligation defects fixed in the released
  version."

## Critique / open questions

**Against the digest entry.** The paper again reads at or below its digest
framing. That makes 32/32 if the running count is kept. This time the
digest carried no number to inflate. The over-reach is in its qualitative
words, and the abstract's own numbers show the usual favourable-cut pattern.

- **"Goes stale exactly when that code changes" and "shown to be sound and
  minimal" hold only relative to the extracted import graph.**
  - In practice, edges come from regular expressions for every language
    except Python. The authors call this "the largest source of
    unsoundness" because it under-approximates dynamic imports, reflection
    and generated code.
  - **Edge recall is not measured on any repository**, so we do not know
    how often the soundness proposition fails in practice.
  - Everything outside the import relation is outside the cone. That
    includes lockfiles, configs, data files, environment and toolchain, so
    a dependency bump does not stale a "tests-pass" claim. Lockfile
    binding is listed only as future work ("Beyond tests").
  - "Minimal" means minimal at directory-module grain over static imports.
    Semantically the binding over-invalidates: any edit in the cone stales
    the claim, whether or not it touches the tested behaviour.
  - The proofs are half-page sketches of what is essentially a property of
    Merkle hashing.
- **"A merge gate that consults no model" is literally true but needs the
  rest of the sentence.** The gate *function* is mechanical. One of its
  inputs is a reviewer's verdict, and the paper expects reviewers may be
  models ("two models from one provider can share a blind spot"). The gate
  checks plausibility, not sufficiency. Its integrity also rests on an HMAC
  key the agent must not be able to read, and the paper describes that
  placement without enforcing it. On a box where the agent's shell can
  read the environment or git-ignored files, the signature check is void.
- **The digest's "Why" is partly wrong about this graph.**
  [[concepts/evidence-gated-completion]] did not lack a staleness
  mechanism. [[literature/papers/li2026who]] (ingested 2026-09-28) already
  established "evidence must be bound to the dependency versions it was
  produced against", with 595/595 affected entries invalidated after 248
  targeted mutations. What Assay adds is the **grain** argument and its
  cost (E3), not the requirement.

**The abstract's numbers, cut by cut.**

- **"Up to 5× faster" is the single best cell**: FastAPI with nothing
  changed (5.4×). The one-edit median is about 1.85×, and the absolute
  saving is under 0.4 s everywhere. That is fine for a pre-commit hook,
  but it is not a performance result.
- **"14× to 114× less than an exploration proxy" compares against a
  stand-in the authors built** (dir listing + 10 most-imported files), and
  the brief is budget-trimmed to ~600 tokens by construction. Nothing tests
  whether an agent can do the task from the brief. The ratio is a property
  of the budget parameter, not of comprehension.
- **"Gate blocks 9 of 9 scripted adversarial behaviours"**: the gate's
  designers wrote one scenario per mechanism, and each fails on its
  matching mechanism. The authors concede this ("independent red-teaming is
  warranted"). It is a unit-test suite, not an adversarial evaluation.
- **"Cone binding re-verifies 7.9%–81.9%" (k = 4)** hides a module-count
  artefact. k = 4 is 57% of Gin's 7 modules and 25% of Flask's 16, so cone
  binding there costs 66.7–81.9%, close to repo-hash. The binding is cheap
  only on wide, shallow, many-module graphs (FastAPI, Express).
- **"23% to 68% misses"** for self-hash correctly excludes the Assay repo
  (7.4%). This one is reported straight.

**Other limits.**

- **The risk score and obligation thresholds are unvalidated constants** (20
  commits/90 days, fan-in 10, level cut-offs). E5 shows how obligations
  *distribute*, not whether they catch defects.
- **Separation of duties is identity-by-declaration.** The roles are
  ledger fields. Nothing checks that "reviewer" and "doer" are different
  processes or models, beyond the key.
- **The real-world evidence is one change on one private app.**
- **On the positive side, this is the most reproducible paper in the
  recent batch.** It is deterministic, has no model, the code is released,
  and every number is script-generated. The limitations section is candid
  and pre-empts most of the above. The design is a careful synthesis of two
  2026 GitHub projects (Liza for protocol, Stacklit for indexing) plus
  build-system theory (Mokhov et al.'s verifying traces). The paper credits
  them openly.

## Trust signals

- **Credibility:** 3. In favour:
  - code and paper source are released, and every number is regenerated by
    a script with no model in the loop, so the results are checkable to the
    digit;
  - the limitations are stated candidly.

  It is not higher because of:
  - practitioner authorship: one engineer whose listed role is at Google
    and two startup founders, with no lab affiliation or peer review;
  - an 8-page preprint;
  - a self-designed adversarial suite;
  - no measurement of edge recall;
  - one dogfooding change.

  Trust here applies to the narrow floor the paper measures, not to any
  claim about agent outcomes, which it does not make.

## Follow-up

- **Relevance:** 3 (the digest said 4). It is a clean, importable mechanism
  for evidence freshness, and E3 is the first measurement in this graph of
  what binding grain costs. Too coarse re-verifies everything; too fine
  silently misses 23–68% of required invalidations. But the freshness
  *requirement* was already in [[concepts/evidence-gated-completion]] via
  li2026who. There is no agent in the loop, and the domain is software
  delivery, not ML research. It sharpens existing concepts without shifting
  any.
- **Bearing on `/lint` staleness.** `kg_lint.py`'s age checks
  (`STALE_LIT_DAYS`, `STALE_CANDIDATE_DAYS`) are backlog timers, not
  validity checks. Assay's rule maps onto this graph directly: a concept
  section's "cone" is the set of literature notes it wikilinks. Hashing
  those notes at the time a paragraph is written would let lint flag
  paragraphs whose cited notes changed afterwards. That convergence with
  [[literature/papers/chen2026fresh]]'s dependency-scoped revalidation is
  now two-domain.
  **The honest limit for this repo:** literature notes rarely change after
  ingest and `raw/` is immutable. The dominant staleness here is *new
  evidence elsewhere* contradicting a concept. That is a new node outside
  the cone, which cone binding cannot see by construction. The mechanism
  catches "what I vouched for changed", not "the world learned more".
- **For `/iterate`-style loops**, the analogue is binding a recorded metric
  to the hash of the code + config + data it was produced from, which is
  close to DVC's stage hashing. Assay's monotonicity check (test count may
  not drop) generalizes to "eval-set size may not drop" as a cheap
  anti-gaming floor.
- **Watch for the models-in-the-loop study.** It is designed and the
  authors say it will be preregistered. It would measure resolved rate,
  tokens to gate pass, and how often the gate blocks harness-passing,
  hidden-test-failing patches on SWE-bench Verified. That last number is
  the one this graph needs.
- **Candidates:**
  - Liza (github.com/liza-mas/liza): the protocol source; repo-ingest
    candidate.
  - Stacklit (github.com/glincker/stacklit).
  - Mokhov, Mitchell & Peyton Jones 2018, "Build Systems à la Carte":
    the theory of verifying traces.
