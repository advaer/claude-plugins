---
description: >
  Entry point of the forge-design pipeline. Translates a PRD, feature
  descriptions, and/or a freeform brief into a self-contained UX spec
  (navigation, journeys, flows, screen inventory). Accepts file references
  via @-prefixed args and freeform text; persists those references in
  ux-spec.md frontmatter so downstream skills inherit them automatically.
  Does not modify the PRD or feature files — they remain the property of
  whatever upstream process produced them.
disable-model-invocation: true
---

# ux

## Usage

```
/forge-design:ux @path/to/prd.{md|pdf} @path/to/features "freeform brief"
/forge-design:ux @path/to/prd.pdf "mobile-first, no settings page"
/forge-design:ux "a Notion-like CRM for indie consultants"
```

- `@`-prefixed tokens are file/folder paths (absolute or relative; resolved to absolute).
- `features` may be a folder, a single file, or multiple `@`-args.
- Supported formats: `.md`, `.pdf`. Convert `.docx`/`.xlsx` to PDF first.
- Non-`@` text is the freeform brief.

## Role

Produce a complete UX specification — navigation structure, user journeys,
cross-feature flows, global interaction patterns, and a screen inventory —
from a PRD, feature descriptions, and an optional freeform brief.

This skill is the **entry point of the forge-design pipeline**. It writes
the frontmatter (`sources`, `user_brief`) that every downstream skill in
the chain reads to discover its upstream context. Get the frontmatter
right and `/forge-design:layout`, `:wireframe`, `:concept`, `:ds`, and
`:ui` can all run without re-passing arguments.

## Inputs

Accepted via `$ARGUMENTS`:

- `@<path-to-prd>` — `.md` or `.pdf`. Optional but strongly recommended.
- `@<path-to-features>` — folder OR single file OR multiple files. Each `.md` or `.pdf`. Optional.
- Freeform brief — non-`@` text. Optional.

### Resolution order for sources

1. **Explicit `@`-path** in `$ARGUMENTS` (highest priority).
2. **PRD convention default** — if no `@`-PRD given, check
   `docs/product-manager/prd.md`. If present, use it and record the path
   in `sources.prd` for future reference. If absent, leave `sources.prd`
   null.
3. **Features convention default** — if no `@`-features given AND
   `sources.features` ends up null, leave it null. (Features must be
   passed explicitly via `@` if you want them. No auto-discovery — that
   way the new flow stays decoupled from the `forge` pipeline's
   `docs/features/` layout.)
4. **Freeform brief** — non-`@` text in `$ARGUMENTS` becomes `user_brief`.

### Required-input gate

After resolution, if `sources.prd`, `sources.features`, AND `user_brief`
are all null, STOP and ask:
> "I need an initial instruction about the product we're going to build
> UX for. Provide one of:
> - A PRD path: `/forge-design:ux @path/to/prd.{md|pdf}`
> - A features path: `/forge-design:ux @path/to/features`
> - A freeform brief: `/forge-design:ux \"a Notion-like CRM for indie consultants\"`
>
> (I checked `docs/product-manager/prd.md` — not found.)"

Do not proceed with placeholder content.

## Outputs

- `docs/ux-designer/ux-spec.md` — global UX spec, populated from the
  template at `${CLAUDE_SKILL_DIR}/templates/ux-spec.md`. **Opens with
  YAML frontmatter** (see schema below).

This skill writes ONE artifact. It does **not** modify the PRD or any
feature files — those stay untouched. Each downstream `/forge-design:*`
skill writes its own dedicated artifact (`layout-spec.md`,
`wireframe-inventory.md`, etc.); design context lives in those artifacts,
not in PRD/features.

### Frontmatter schema (written by this skill)

```yaml
---
sources:
  prd: /abs/path/to/prd.pdf            # absolute; null if no PRD provided
  features: /abs/path/to/features/     # absolute; null if no features provided
user_brief: |
  <verbatim concatenated freeform text from $ARGUMENTS, or null if none>
---
```

This frontmatter is the contract every downstream `/forge-design:*` skill
reads. Resolve all paths to absolute (relative to project cwd at invocation).

---

## Operator Profile (fixed — do not modify per run)

**Who:** Solo developer + AI agents, Ukrainian PE at MVP stage.
**Capacity:** 20–30 hours/week, ~$200/month infra budget at launch.
**MVP target:** Live product accepting real payments within weeks to a few months.
**Payment layer:** Merchant-of-record only (Paddle, LemonSqueezy, or equivalent).
**Buyer profile:** B2C individuals/prosumers or B2B small businesses. Card signup
in under 10 minutes.

UX decisions must reflect this profile: prefer simple linear flows over complex
multi-step wizards; minimise cognitive load; avoid features that require
onboarding investment before value is demonstrated.

---

## Handling $ARGUMENTS

### Parsing rules

1. Tokenize `$ARGUMENTS`. Tokens starting with `@` are paths; everything else
   joins into the freeform brief.
2. Resolve each `@`-path to an absolute path (relative to project cwd).
3. Classify `@`-paths by what they point to:
   - First `.md`/`.pdf` file whose basename suggests a PRD (e.g. contains
     "prd", "product", "spec") → `sources.prd`.
   - Otherwise, first `.md`/`.pdf` file → `sources.prd` (best guess).
   - Folder, or remaining file paths → `sources.features`.
   - If ambiguous, ask the user once which is which.
4. **Apply convention defaults** (per `## Inputs` resolution order): for any
   slot still unset after Step 3, check the project default and use it if
   it exists.
5. The freeform brief becomes `user_brief` verbatim (preserve casing/wording).
6. Apply the brief to UX decisions. Classify each directive into:
   `flow-change`, `screen-add`, `screen-remove`, `nav-constraint`,
   `interaction-preference`, `platform-scope`, `persona-note`, `other`.
   Never silently ignore a directive.

### Required-input gate

If after Steps 3–4, all three of (`sources.prd`, `sources.features`,
`user_brief`) are null/empty → STOP per the message in `## Inputs`. Do not
invent context.

---

## Process

### 1. Read the PRD and features

If `sources.prd` is set: read the file (md or pdf — Read tool handles both).
Extract personas, MVP feature list with RICE/tier, monetization, out-of-scope
decisions, open questions.

If `sources.features` is set:
- If it's a folder: read every `.md` or `.pdf` inside (recursive).
- If it's a file: read it.
- If multiple paths: read each.

For each feature: extract overview, user stories, acceptance criteria, edge
cases, existing UX notes (do not overwrite).

If only the freeform brief is provided: treat it as the entire product brief.
Generate personas and feature list inline; note in the UX spec under
**User Input Notes** that this run had no PRD/features and listed assumptions
made.

### 2. Apply the freeform brief

Apply every parsed directive. Conflicts with the PRD or operator profile go
under **User Input Notes** with explanation.

### 3. Determine platform scope

Default: web (responsive, desktop-first). Mobile: include if PRD/brief says so.
State the decision explicitly in the spec.

### 4. Define navigation structure

List primary destinations, navigation pattern per platform, feature-to-level
mapping, context-sensitive nav. Every destination must be justified by an
MVP Core feature.

### 5. Map user journeys

One journey map per persona: entry point, activation path, recurring loop,
upgrade trigger. Numbered steps; what the user sees and does.

### 6. Define cross-feature flows

Required: onboarding, auth, payment/upgrade, error recovery, account/settings.
Plus any additional flows implied by the feature list.

### 7. Define global UX patterns

Empty states, loading states, error handling, success feedback, forms,
responsive behaviour.

### 8. Build the screen inventory

Every MVP screen: name (`[Feature] — [View]`), feature, platform, purpose,
entry points, key actions. Group by feature; global screens in their own group.
Post-MVP screens noted but not elaborated.

### 9. Write ux-spec.md with frontmatter

Read `${CLAUDE_SKILL_DIR}/templates/ux-spec.md`. Populate it. Prepend the
YAML frontmatter from the schema in `## Outputs`. Write to
`docs/ux-designer/ux-spec.md`.

Screen names in the spec are canonical — downstream design artifacts
(layout-spec.md, wireframe-inventory.md, etc.) reference them.

---

## Self-Review Checklist

Before declaring done, verify each item. Mark ✅/❌ inline.

**Structural:**
- [ ] `docs/ux-designer/ux-spec.md` exists with valid frontmatter
- [ ] `frontmatter.sources.prd` is absolute (or null with explanation in spec body)
- [ ] `frontmatter.sources.features` is absolute (or null with explanation)
- [ ] `frontmatter.user_brief` matches what the user typed (verbatim)
- [ ] Output structure matches the template
- [ ] PRD and feature files were NOT modified (this skill only writes ux-spec.md)

**UX best practices:**
- [ ] Navigation covers all MVP Core features and nothing more
- [ ] Journey map exists for every persona; each has activation, loop, conversion trigger
- [ ] Required cross-feature flows present (auth, onboarding, payment, error)
- [ ] Empty / loading / error states defined globally (no per-screen reinvention)
- [ ] Forms have validation timing (on blur vs submit) and required-field convention
- [ ] Responsive behaviour stated (breakpoint strategy, mobile collapse rules)
- [ ] Accessibility intent declared (target tier, focus order, hit-target convention)
- [ ] Every MVP Core user story has at least one screen
- [ ] No screen designed for an out-of-scope feature
- [ ] Realistic copy in journey/flow examples — no lorem ipsum

**$ARGUMENTS handling:**
- [ ] Every directive in the freeform brief was applied OR noted under User Input Notes
- [ ] No directive silently ignored
- [ ] Conflicts with PRD/operator profile resolved with explanation

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
    APPEND a "## ⚠ Outstanding Issues" section to docs/ux-designer/ux-spec.md
      listing each unresolved checklist item with a one-line reason
    PRINT to console (verbatim):
      ⚠ /forge-design:ux completed with N unresolved issues — see docs/ux-designer/ux-spec.md
    exit
```

---

## Quality Bar (pre-condition gate)

Do not begin the iteration loop until all of these pass — they are
pre-conditions, not part of the self-review.

- [ ] `$ARGUMENTS` parsed and `sources` + `user_brief` resolved
- [ ] Required-input gate passed (at least one of prd/features/brief is set)
- [ ] PRD read (or absent and noted)
- [ ] Feature files read (or absent and noted)
- [ ] Platform scope decided
- [ ] Navigation, journeys, flows, patterns, inventory drafted in working memory
- [ ] Template read and being populated

---

## Chain

After the iteration loop exits successfully, tell the user:
> "UX spec written to `docs/ux-designer/ux-spec.md` with frontmatter
> capturing PRD/features paths and the freeform brief. (PRD and feature
> files were not modified.)
>
> Next: review the screen inventory and flows, then run
> `/forge-design:layout` (no arguments needed — it inherits sources from
> ux-spec.md frontmatter)."

If the iteration loop exited with outstanding issues, additionally print:
> "⚠ N issues remain unresolved — see `## ⚠ Outstanding Issues` in
> `docs/ux-designer/ux-spec.md`. Resolve before proceeding to the next
> skill, or accept and continue."
