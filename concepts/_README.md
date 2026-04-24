# concepts/ — the importable payload

Downstream projects `@import` files from this directory to inherit
architectural patterns by reference rather than by copy. When a
concept evolves here, downstream projects see the update at the next
session start. This README is what a downstream agent reads first
when it follows an `@import` link into this directory.

## Frontmatter contract

Every concept note in this directory carries this frontmatter:

```yaml
---
kind: concept
name: "<slug>"
status: active | experimental | retired
added: "YYYY-MM-DD"
source_papers:
  - <citekey>              # e.g. hambardzumyan2026aira
  - <citekey>
sources:
  - "literature/papers/<citekey>"       # actual entries wrap the path in double-bracket wikilink syntax; shown unwrapped here so this schema example does not resolve
used_by: []                # populated by /ingest on downstream side
related_concepts: []
related_experiments: []
tags: []
---
```

Fields specific to this project's import role:

- **`source_papers:`** — citekeys (plain strings), not wikilinks. This
  field is stable across whether the downstream project has the same
  literature notes. Downstream agents treat it as the canonical
  attribution.
- **`used_by:`** — a list of objects, each `{project_slug, imported_on}`.
  Maintained by `/ingest` on the downstream side when a project's
  `CLAUDE.md` contains `@import ~/projects/research/agentic-research/concepts/<name>.md`.
  Append is idempotent: re-ingesting does not duplicate entries for
  the same `project_slug`. See the root `CLAUDE.md` Import contract
  section.
- **`status:`** — differs from the template's `seedling | growing | mature`
  lifecycle (which is about concept *maturation* within a single
  project). Here, `status:` captures whether the pattern is
  *load-bearing for downstream projects*:

| status | meaning | downstream behavior |
|---|---|---|
| `active` | Stable, recommended. Pattern is realized in current skills or proves out across multiple projects. | Safe to import; follow guidance. |
| `experimental` | Plausible, under evaluation. No or limited track record. | Safe to import but expect churn; flag in project NOTES.md. |
| `retired` | Deprecated. Superseded, wrong in hindsight, or never earned `used_by:`. Body explains what replaces it. | `/ingest` emits a warning on import but allows it. |

## Body format

- **Definition** — one or two sentences. Precise enough to disagree with.
- **Why it matters** — the evidence from `source_papers:` that motivates
  adopting this pattern. Cite specific findings, not vibes.
- **Implementation guidance** — concrete steps for a downstream project
  to realize the pattern. File paths, frontmatter fields, enforcement
  mechanisms where they exist.
- **Open questions** — what the pattern doesn't settle; where it
  might be wrong; what evidence would move it to `retired`.

Aim for 200-500 words of body. Longer than that and you're restating
the paper rather than synthesizing.

## The import back-reference

Concepts earn their keep through `used_by:`. Empty `used_by:` on a
concept older than two weeks is a `/lint` review candidate — either
the concept isn't articulated clearly enough to adopt, or it isn't
load-bearing. Either state is fine; long-term empty `used_by:` means
`retire` or rewrite.

## Relationship to `_meta/templates/concept.md`

The project template's `concept.md` uses `status: seedling | growing | mature`
for internal maturation. This directory overrides the vocabulary to
describe load-bearing status for external importers. Both vocabularies
live side by side in the codebase: the template is for projects that
don't play the meta-project role; this README is for the one that does.
