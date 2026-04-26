---
description: >
  Reads the approved PRD from /forge-product:prd and generates detailed feature
  description files — one per feature — with user stories, acceptance criteria,
  edge cases, and vertical slices (walking skeleton first). Inherits sources +
  user_brief from prd.md frontmatter; accepts optional $ARGUMENTS to target
  specific features or apply per-feature steering. Safe to re-run after PRD
  edits to regenerate individual or all feature files.
disable-model-invocation: true
---

# backlog

## Usage

```
/forge-product:backlog                                       # generate all features in the PRD
/forge-product:backlog "regenerate 003-notifications"        # target specific features
/forge-product:backlog "for 002-auth, add magic link as an acceptance criterion"
/forge-product:backlog "skip Post-MVP features for now"
```

No `@`-paths needed — `sources.idea_validation` and `user_brief` are read from
`docs/product/prd.md` frontmatter. Pass freeform `$ARGUMENTS` for feature
targeting or per-feature steering.

## Role

Translate the approved PRD from `/forge-product:prd` into detailed feature
description files. One file per feature, each independently useful to a
developer or to downstream skills (`/forge-design:ux`, `/forge:architect`,
`speckit.specify`).

When `$ARGUMENTS` names specific features, generate or regenerate only those;
otherwise generate all features in the PRD.

## Inputs

- `docs/product/prd.md` — **required**. Read its YAML frontmatter to inherit
  `sources.idea_validation` and `user_brief`. Read its body for the feature
  list with RICE scores and release versions, personas, success metrics,
  open questions, and monetization model.
- `$ARGUMENTS` — optional freeform feature-targeting + per-feature directives (see below).

**Required-input gate:** if `docs/product/prd.md` does not exist or has no
frontmatter → STOP and tell the user:
> "No upstream PRD with frontmatter found. Run `/forge-product:prd` first
> (with `@idea-validation` path or a freeform brief)."

## Outputs

- `docs/backlog/NNN-feature-name/description.md` — one per feature, populated
  from the template at `${CLAUDE_SKILL_DIR}/templates/feature-description.md`.
  **Each file opens with YAML frontmatter** that copies `sources` +
  `user_brief` from prd.md and adds the PRD path itself:

```yaml
---
sources:
  idea_validation: <copied from prd.md>
  prd: /abs/path/to/docs/product/prd.md   # added by this skill
user_brief: |
  <copied from prd.md, plus any freeform $ARGUMENTS appended>
feature:
  number: 001                              # zero-padded 3-digit
  name: user-authentication                # kebab-case
  release: v1.0.0                          # from PRD's RICE-scored table
---
```

Folder naming: `NNN-kebab-case-name` (e.g. `001-user-authentication`).
Numbers must match the numbering in the PRD feature table. Do not renumber
existing folders — preserve any folders that already exist and are not
being regenerated.

This skill writes ONLY into `docs/backlog/`. It does **not** modify the PRD
or the upstream idea-validation document.

---

## Operator Profile (fixed — do not modify per run)

**Who:** Solo developer + AI agents, Ukrainian PE at MVP stage.
**Capacity:** 20–30 hours/week, ~$200/month infra budget at launch.
**MVP target:** Live product accepting real payments, first paying users
within weeks to a few months.
**Payment layer:** Merchant-of-record only (Paddle, LemonSqueezy, or equivalent).

---

## Handling $ARGUMENTS

`$ARGUMENTS` may contain:

- **Feature targeting** — "regenerate 003-notifications" or "only generate v1.0.0 features"
- **Feature-level constraints** — "for 002-auth, add magic link as an acceptance criterion"
- **Scope notes** — "skip Post-MVP features for now"
- **Any other steering** — apply as product intent

### Parsing rules

1. Read `$ARGUMENTS` in full before doing anything else.
2. If feature names or numbers are specified, generate only those. Otherwise
   generate all features listed in the PRD.
3. Apply any per-feature constraints to the relevant feature files.
4. Conflicts with the inherited `user_brief` are resolved in favor of the
   new `$ARGUMENTS` and noted in the affected feature file under a
   **User Input Notes** subsection.
5. Never silently ignore a directive.
6. The freeform `$ARGUMENTS` is **appended** to the inherited `user_brief`
   when written to each feature file's frontmatter.

If `$ARGUMENTS` is not provided, generate all features listed in the PRD.

---

## Process

### 1. Read upstream

1. Read `docs/product/prd.md`. Parse YAML frontmatter:
   - `sources.idea_validation`, `user_brief`
2. Read prd.md body. Extract:
   - Full feature list with RICE scores and release version (v1.0.0 / v1.1.0 / v1.2.0)
   - User personas (for writing user stories in the right voice)
   - Success metrics (to ground acceptance criteria)
   - Risky assumptions and open questions (to carry into feature files)
   - Monetization model (for acceptance criteria on payment-related features)

### 2. Determine scope

If `$ARGUMENTS` specifies features: generate only those.
Otherwise: generate all features in the PRD (across all release versions).

Check `docs/backlog/` for existing folders. If a folder already exists and is
NOT being targeted for regeneration, leave it untouched.

### 3. Write feature description files

For each targeted feature, create
`docs/backlog/NNN-feature-name/description.md` using the template at
`${CLAUDE_SKILL_DIR}/templates/feature-description.md`. **Prepend the YAML
frontmatter schema declared in `## Outputs`** (copy `sources` and `user_brief`
from prd.md; add `sources.prd` pointing at prd.md itself; append any
`$ARGUMENTS` text to `user_brief`; populate `feature.{number,name,release}`
from the PRD).

#### User stories

Write from the persona's perspective. Use the format:
> As a [persona], I want to [action] so that [outcome].

Derive stories from:
- The feature's purpose in the PRD
- The persona's goals and frustrations
- The monetization model (include payment-related stories where relevant)

Aim for 2–4 stories per feature. More than 4 usually means the feature should be split.

#### Acceptance criteria

Write specific, testable criteria that a developer can implement against and
a QA engineer can verify. Avoid vague criteria like "works correctly" or "is
fast".

Good: `[ ] User receives an email within 60 seconds of triggering password reset`
Bad:  `[ ] Password reset works`

#### Edge cases

Derive from:
- The assumption audit / open questions in the PRD
- Failure modes specific to this feature (network errors, empty states, permission issues)
- Interactions with other features

#### Vertical slices

Decompose each feature into **vertical slices** — thin, end-to-end cuts
through all layers (DB → API → UI where needed) where each slice is
independently shippable and testable. Never decompose as layers
(DB schema → API → UI), where nothing runs until all layers are done.

Slice 1 is always the **walking skeleton**: the thinnest possible end-to-end
implementation that proves the feature works. Subsequent slices grow it
toward full capability.

**Connect slices to the release version.** Each feature belongs to a release
version (v1.0.0 / v1.1.0 / v1.2.0 from the PRD). The slices inside a feature
follow the same progression: slice 1 is the walking skeleton that ships in
the feature's assigned release; later slices ship in subsequent iterations.
Label each slice with its target release version.

Rules:
1. **Slice ≠ layer.** A slice crosses all layers (DB → API → UI if needed).
2. **Each slice is independently testable.** After merging a slice, an API
   test or end-to-end test can verify it works.
3. **Each slice delivers real value.** A user or caller can accomplish
   something meaningful with just this slice — even if limited.
4. **Order by value, not tech.** Start with the slice that proves the
   riskiest assumption or unlocks the most value.
5. **Aim for 2–5 slices** per feature. More than 5 usually means the feature
   should be split into two features.
6. **Walking skeleton ships first.** Slice 1 must always ship in the
   feature's assigned release version.

Then run the Self-Review Checklist + Iteration Loop below.

---

## Self-Review Checklist

Before declaring done, verify each item. Mark ✅/❌ inline.

**Structural:**
- [ ] Every targeted feature has a folder at `docs/backlog/NNN-feature-name/`
- [ ] Every feature folder has a `description.md` with valid frontmatter
- [ ] `frontmatter.sources` copied from prd.md, plus `sources.prd` added
- [ ] `frontmatter.user_brief` includes inherited brief + appended $ARGUMENTS
- [ ] `frontmatter.feature.{number,name,release}` populated from the PRD
- [ ] Folder numbering matches the PRD feature table — no renumbering
- [ ] Existing untargeted folders are untouched
- [ ] Output structure matches the template
- [ ] PRD and idea-validation documents are NOT modified

**Product/PM best practices:**
- [ ] Each feature file has 2–4 user stories in persona voice
- [ ] Acceptance criteria are specific and testable (no vague criteria)
- [ ] Edge cases derived from PRD assumption audit + feature failure modes
- [ ] Every feature has ≥ 2 vertical slices; slice 1 is a walking skeleton
- [ ] No slice is a pure layer (DB only, API only, UI only) — each crosses layers
- [ ] Each slice labeled with its target release version
- [ ] Walking skeleton's release version matches the feature's release version

**$ARGUMENTS handling:**
- [ ] Every directive applied OR noted under User Input Notes in the affected feature file(s)
- [ ] No directive silently ignored
- [ ] Targeted-feature scope respected — untargeted folders not touched

---

## Iteration Loop

```
attempts = 0
loop:
  run Self-Review Checklist
  if all items pass:
    finalize files and emit success block (see Chain)
    exit
  if any items fail and attempts < 2:
    attempts += 1
    fix the flagged items only
    continue
  else:  # attempts == 2 and items still failing
    APPEND a "## ⚠ Outstanding Issues" section to EACH feature file with
      unresolved items, listing each item with a one-line reason
    PRINT to console (verbatim):
      ⚠ /forge-product:backlog completed with N unresolved issues across M features — see docs/backlog/
    exit
```

---

## Quality Bar (pre-condition gate)

Do not begin the iteration loop until all of these pass.

- [ ] prd.md exists; frontmatter parsed
- [ ] PRD body read; feature list, personas, open questions, monetization extracted
- [ ] `$ARGUMENTS` parsed; scope determined (targeted or all)
- [ ] Existing feature folders identified — only targeted folders will be written
- [ ] Template read and being populated for each targeted feature

---

## Chain

After the iteration loop exits successfully, tell the user:
> "Feature descriptions written to `docs/backlog/`. Each feature folder
> includes a `description.md` with frontmatter cascaded from the PRD.
>
> Review them — edit acceptance criteria, adjust vertical slices, or re-run
> with specific feature names / numbers to regenerate individual files.
>
> When ready, run `/forge-design:ux` to define user flows and the screen
> inventory (it accepts `@docs/product/prd.md` and `@docs/backlog/` paths,
> or runs against any other project layout you point at)."

If the iteration loop exited with outstanding issues, additionally print:
> "⚠ N issues remain unresolved across M feature files — see
> `## ⚠ Outstanding Issues` sections in `docs/backlog/`. Resolve before
> proceeding, or accept and continue."
