---
description: >
  Entry point of the forge-product plugin. Translates a validated idea into a
  structured PRD with a RICE-prioritized feature summary table. Accepts an
  @-path to an idea-validation document and/or a freeform brief; persists those
  references in prd.md frontmatter so /forge-product:backlog inherits them
  without re-passing arguments. Does not modify the upstream idea-validation
  document.
disable-model-invocation: true
---

# prd

## Usage

```
/forge-product:prd @path/to/idea-validation.{md|pdf} "freeform brief"
/forge-product:prd "a Notion-like CRM for indie consultants — focus on freelancers"
/forge-product:prd                                       # uses docs/idea-validator/idea-validation.md if present
```

- `@`-prefixed tokens are file paths (absolute or relative; resolved to absolute).
- Supported formats: `.md`, `.pdf`. Convert `.docx`/`.xlsx` to PDF first.
- Non-`@` text is the freeform brief.

## Role

Translate a validated idea into a structured PRD with a RICE-prioritized
feature summary table. This skill is the **entry point of forge-product** —
it writes the frontmatter (`sources`, `user_brief`) that `/forge-product:backlog`
reads to discover the PRD path and the freeform brief without re-passing them.

Detailed feature description files (with user stories, acceptance criteria,
and vertical slices) are produced by the separate `/forge-product:backlog`
skill, which reads the approved PRD.

## Inputs

Accepted via `$ARGUMENTS`:

- `@<path-to-idea-validation>` — `.md` or `.pdf`. Optional.
- Freeform brief — non-`@` text. Optional.

### Resolution order for `sources.idea_validation`

1. **Explicit `@`-path** in `$ARGUMENTS`.
2. **Convention default** — `docs/idea-validator/idea-validation.md` (if exists).
3. **null** — leave unset.

### Required-input gate

If `sources.idea_validation` AND `user_brief` are both null, STOP and ask:
> "I need an initial instruction about the product. Provide one of:
> - An idea-validation document: `/forge-product:prd @path/to/idea-validation.{md|pdf}`
> - A freeform brief: `/forge-product:prd \"a Notion-like CRM for indie consultants\"`
>
> (I checked `docs/idea-validator/idea-validation.md` — not found.)"

Do not proceed with placeholder content.

## Outputs

- `docs/product/prd.md` — populated from the template at
  `${CLAUDE_SKILL_DIR}/templates/prd.md`. **Opens with YAML frontmatter** (see
  schema below). The PRD includes a RICE-scored feature summary table; full
  feature descriptions are written by `/forge-product:backlog`.

This skill writes ONE artifact. It does **not** modify the upstream
idea-validation document.

### Frontmatter schema (written by this skill)

```yaml
---
sources:
  idea_validation: /abs/path/to/idea-validation.md  # absolute; null if absent
user_brief: |
  <verbatim concatenated freeform text from $ARGUMENTS, or null if none>
---
```

This frontmatter is the contract `/forge-product:backlog` reads. Resolve
all paths to absolute (relative to project cwd at invocation).

---

## Operator Profile (fixed — do not modify per run)

**Who:** Solo developer + AI agents, Ukrainian PE at MVP stage.
**Capacity:** 20–30 hours/week, ~$200/month infra budget at launch.
**MVP target:** Live product accepting real payments, first paying users
within weeks to a few months.
**Payment layer:** Merchant-of-record only (Paddle, LemonSqueezy, or equivalent).
**Buyer profile:** B2C individuals/prosumers or B2B small businesses, indie
makers, freelancers, solo professionals. Card signup in under 10 minutes.

---

## Handling $ARGUMENTS

`$ARGUMENTS` may contain any mix of `@`-paths and freeform directives.
Freeform examples:

- **Feature requests** — "add mobile notifications with option to configure time"
- **Feature deprioritisation** — "deprioritize social sharing"
- **Feature exclusions** — "no onboarding wizard"
- **Tech or product constraints** — "use custom scoring formula"
- **Persona or scope notes** — "focus on freelancers, not agencies"
- **Any other steering input** — treat unrecognised directives as product intent

### Parsing rules

1. Tokenize `$ARGUMENTS`. Tokens starting with `@` are paths; everything else
   joins into the freeform brief.
2. Resolve `@`-paths to absolute (relative to project cwd).
3. Apply the convention default for `sources.idea_validation` (per `## Inputs`)
   if no `@`-path provided.
4. The freeform brief becomes `user_brief` verbatim.
5. Apply the brief to PRD decisions. Classify each directive into:
   `feature-add`, `feature-deprioritize`, `feature-exclude`, `constraint`,
   `persona-note`, `scope-note`, `other`.
6. Apply directives:
   - `feature-add` → include as a feature; derive user stories and
     acceptance criteria from the idea and the directive text
   - `feature-deprioritize` → include but score lower in RICE; note user
     input as the reason
   - `feature-exclude` → omit from feature list; note in Out-of-Scope
     section of PRD with reason "excluded by user input"
   - `constraint` → apply to relevant PRD section
   - `persona-note` → refine the persona section accordingly
   - `scope-note` → apply to problem statement or success metrics
   - `other` → apply best judgement; note where applied
7. Never silently ignore a directive. If a directive conflicts with the
   validated idea or the operator profile, note the conflict in the PRD
   under a **"User Input Notes"** subsection and explain how it was resolved.

### Required-input gate

If after parsing both `sources.idea_validation` and `user_brief` are
null, STOP per the message in `## Inputs`. Do not invent context.

---

## Process

### 1. Read upstream

If `sources.idea_validation` is set: read the file (md or pdf — Read tool
handles both). Extract:
- Validated idea: working title, one-liner, target user, core value, positioning
- Go / No-Go verdict and any conditions
- Assumption audit results (carry risky assumptions into PRD as open questions)
- Monetization recommendation
- Differentiation wedge

If the verdict is **No-Go ❌**, stop and inform the user. Do not proceed
unless the user explicitly overrides via `$ARGUMENTS` acknowledging the risk.

If only the freeform brief is provided (no idea-validation document): treat
the brief as the entire product definition. Generate personas, monetization,
and feature list inline; note in the PRD under **User Input Notes** that
this run had no idea-validation document and listed the assumptions made.

### 2. Apply the freeform brief

Apply every parsed directive. Conflicts go under **User Input Notes**
with explanation.

### 3. Define personas

Derive 1–2 user personas from the validated idea and any persona-note directives.

Each persona:
- Name and role (fictional but grounded in the target segment)
- Primary goal when using the product
- Key frustrations with existing solutions
- Usage context (when, where, how often)
- Willingness-to-pay signal (from landscape or assumed)

### 4. Define success metrics

Propose 3–5 measurable success metrics for the MVP launch phase.
Ground them in the monetization model and the operator's MVP target.

Examples:
- First paying user within N weeks of launch
- MRR target at 30/60/90 days
- Activation rate (users who complete core action within first session)
- Retention rate at 7 and 30 days
- Support ticket volume per active user (quality signal)

Avoid vanity metrics (raw signups, page views) unless tied to a conversion goal.

### 5. Define out-of-scope

List what is explicitly excluded from the MVP. Sources:
- Features that would push build complexity to High without clear validation value
- Any `feature-exclude` directives from `$ARGUMENTS`
- Enterprise features, admin dashboards, API access (unless core to the idea)
- Localisation, accessibility compliance beyond basics

### 6. Generate and RICE-score the feature list

Generate a feature list for the MVP. Sources:
- Core features implied by the validated idea
- Features added via `feature-add` directives
- Features adjusted via `feature-deprioritize` directives

For each feature apply RICE scoring calibrated to this operator:

**Reach** — What proportion of target users need this feature? (1–10)
**Impact** — How much does this feature drive activation, retention, or payment? (1–10)
**Confidence** — How certain are we this feature is needed, based on evidence? (1–10)
**Effort** — How hard to build solo using the preferred stack? (1 = easy, 10 = hard)

```
RICE Score = (Reach × Impact × Confidence) / Effort
```

Rank features by RICE score, then assign each a **release version** using
semantic versioning and the skateboard principle:

| Version | What it means |
|---------|---------------|
| **v1.0.0** | Core loop closed. User can sign up, do the primary job, and pay. The product ships. Rough edges acceptable. This is the MVP. |
| **v1.1.0** | Key friction removed. Core experience is smooth and recommendable. Retention-driving features added. |
| **v1.2.0** | Competitive feature set. Meaningful differentiation. Power-user and growth capabilities. |
| **v2.0.0** | Reserved for major platform evolution — pivot, redesign, or breaking change to the core product contract. Do not assign features here at PRD stage. |

**Semantic versioning conventions:**
- **MAJOR** (v2.0.0+): Platform pivot, complete redesign, or breaking change to the core product contract. Not used for normal feature planning.
- **MINOR** (v1.1.0, v1.2.0 …): Each minor version is a coherent batch of features that extends the product. Features within a batch should be cohesive and shippable together.
- **PATCH** (v1.0.1, v1.1.1 …): Bug fixes, copy changes, minor UX tweaks. Not planned at PRD stage — assigned during execution.

Rules:
- **v1.0.0** contains the fewest features that create a closed loop (sign up → do the thing → pay). Nothing else.
- A feature belongs in v1.0.0 only if removing it would prevent the product from shipping at all.
- **v1.1.0** makes the product worth recommending — the "scooter" iteration.
- **v1.2.0** expands reach or deepens engagement post-validation — the "bicycle" iteration.
- Additional minor versions (v1.3.0 …) may be added if the feature list warrants further batching.
- A `feature-deprioritize` directive applies a −2 to RICE score before version assignment.
- When in doubt, push to v1.1.0. v1.0.0 scope creep is the most common failure mode.

### 7. Write prd.md with frontmatter

Read `${CLAUDE_SKILL_DIR}/templates/prd.md`. Populate it. Prepend the YAML
frontmatter from the schema in `## Outputs`. Write to `docs/product/prd.md`.

The feature list in the PRD is a summary table — feature number, name,
one-liner, RICE dimensions, RICE score, and release version. Full feature
descriptions (user stories, acceptance criteria, vertical slices) are
produced by the separate `/forge-product:backlog` skill, which reads this
PRD once the user has reviewed and approved it.

Then run the Self-Review Checklist + Iteration Loop below.

---

## Self-Review Checklist

Before declaring done, verify each item. Mark ✅/❌ inline.

**Structural:**
- [ ] `docs/product/prd.md` exists with valid frontmatter
- [ ] `frontmatter.sources.idea_validation` is absolute (or null with explanation in PRD body)
- [ ] `frontmatter.user_brief` matches what the user typed (verbatim)
- [ ] Output structure matches the template
- [ ] The upstream idea-validation document was NOT modified

**Product/PM best practices:**
- [ ] Personas derived and grounded in target segment (1–2)
- [ ] Each persona has goal, frustrations, usage context, willingness-to-pay
- [ ] Success metrics are measurable and MVP-appropriate (3–5)
- [ ] No vanity metrics not tied to conversion
- [ ] Out-of-scope section present and reasoned
- [ ] Every feature has a RICE score computed correctly
- [ ] `feature-deprioritize` directives applied as −2 modifier
- [ ] Feature summary table includes release version (v1.0.0 / v1.1.0 / v1.2.0 …) per feature
- [ ] v1.0.0 contains only features required to close the core loop — no more
- [ ] Risky assumptions from the idea-validation are carried forward as open questions
- [ ] Monetization model is consistent with the operator's MoR-only constraint

**$ARGUMENTS handling:**
- [ ] Every directive in the freeform brief was applied OR noted under User Input Notes
- [ ] No directive silently ignored
- [ ] Conflicts with idea-validation or operator profile resolved with explanation

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
    APPEND a "## ⚠ Outstanding Issues" section to docs/product/prd.md
      listing each unresolved checklist item with a one-line reason
    PRINT to console (verbatim):
      ⚠ /forge-product:prd completed with N unresolved issues — see docs/product/prd.md
    exit
```

---

## Quality Bar (pre-condition gate)

Do not begin the iteration loop until all of these pass — they are
pre-conditions, not part of the self-review.

- [ ] `$ARGUMENTS` parsed and `sources.idea_validation` + `user_brief` resolved
- [ ] Required-input gate passed (at least one is set)
- [ ] Idea-validation read (or absent and noted)
- [ ] If verdict is No-Go, user override acknowledged via $ARGUMENTS
- [ ] Personas, success metrics, out-of-scope, and RICE-scored feature list drafted in working memory
- [ ] Template read and being populated

---

## Chain

After the iteration loop exits successfully, tell the user:
> "PRD written to `docs/product/prd.md` with frontmatter capturing the
> idea-validation path and the freeform brief.
>
> Review the PRD — adjust scope, personas, success metrics, or the feature
> list as needed. When you're happy with it, run `/forge-product:backlog`
> to generate detailed feature descriptions with user stories, acceptance
> criteria, and vertical slices (no arguments needed — it inherits sources
> from prd.md frontmatter)."

If the iteration loop exited with outstanding issues, additionally print:
> "⚠ N issues remain unresolved — see `## ⚠ Outstanding Issues` in
> `docs/product/prd.md`. Resolve before proceeding to the next skill, or
> accept and continue."
