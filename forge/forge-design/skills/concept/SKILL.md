---
description: >
  Produces 3 visually distinct concept moodframes in Figma — each representing a
  different design philosophy — so the human can choose a direction before the full
  design system is built. Renders against the actual wireframe of the primary
  dashboard for realistic decision-making. Inherits inputs from
  wireframe-inventory.md frontmatter; accepts optional steering via $ARGUMENTS.
  Runs fourth in the forge-design pipeline (ux → layout → wireframe → concept
  → ds → ui).
disable-model-invocation: true
---

# concept

## Usage

```
/forge-design:concept                                     # inherits everything from wireframe-inventory
/forge-design:concept "calm authority, no playful, lean against typical purple AI clichés"
/forge-design:concept "Concept A must lean into raw-power archetype; Concept C should be expressive-brand"
```

No `@`-paths needed — `sources` and the wireframes Figma file key are read
from `docs/design/wireframe-inventory.md` frontmatter. Pass freeform
`$ARGUMENTS` for personality / archetype steering.

## Role
Translate product personality into 3 meaningfully different visual directions,
each expressed as a realistic moodframe of the product's primary dashboard screen.
The moodframe is the decision artifact. Everything else produced by this skill
is technical scaffolding to make that moodframe possible.

Concepts must differ in design *philosophy* — not just hue. Each represents a
genuine strategic choice about how the product presents itself to the world.

---

## Inputs

- `docs/design/wireframe-inventory.md` — **required**. Read its YAML
  frontmatter to inherit `sources.prd`, `sources.features`, `user_brief`,
  and `figma.wireframes_file_key`. Read its body to identify which screen
  is the primary dashboard and to look up its wireframe in the Figma file.
- The PRD at `frontmatter.sources.prd` (md/pdf) — brand tone, product type, personas, goals.
- The features at `frontmatter.sources.features` — for realistic moodframe content.
- `docs/design/ux-spec.md` body — navigation structure, primary dashboard definition.
- Figma wireframes file at `frontmatter.figma.wireframes_file_key` — open it
  to see the actual primary-dashboard wireframe; concepts render against
  this wireframe so token choices show how they hold up on real product structure.
- `$ARGUMENTS` — optional freeform personality / archetype directives (see below).

**Required-input gate:** if `docs/design/wireframe-inventory.md` does not
exist or its frontmatter lacks `figma.wireframes_file_key` → STOP and tell
the user:
> "No upstream wireframe-inventory with frontmatter found. Run
> `/forge-design:wireframe` first."

## Outputs

- Figma file: **[Product Name] — Concept Directions** — 3 moodframe artboards
  (1440×900 desktop, light theme), one page per concept.
- `docs/design/concept-brief.md` — populated from
  `${CLAUDE_SKILL_DIR}/templates/concept-brief.md`. **Opens with YAML
  frontmatter** that copies `sources` + `user_brief` from
  wireframe-inventory.md and adds the new figma file key:

```yaml
---
sources:
  prd: <copied>
  features: <copied>
user_brief: |
  <copied + appended $ARGUMENTS>
figma:
  wireframes_file_key: <copied from wireframe-inventory>
  concept_file_key: <created in this run>
---
```

---

## Operator Profile (fixed)

Solo developer. No dedicated designer. The moodframe must look like it was made
by a product designer. The human will choose one direction; that choice drives
everything in `/forge-design:ds`. Token efficiency matters — this skill exists
to make the expensive decision cheaply.

---

## Handling $ARGUMENTS

`$ARGUMENTS` may contain personality/archetype steering. Examples:

- "calm authority, professional confidence — no playful"
- "lean against typical AI-product purple; consider warm earthy palette"
- "Concept A must be raw-power archetype; Concept C must be expressive-brand"
- "all concepts must support both light and dark themes from day one"
- "audience skews older / less tech-savvy — bias toward Clinical Trust archetype"

### Parsing rules

1. Read `$ARGUMENTS` in full before any concept work.
2. Classify each directive: `archetype-pin` (specific concept must use a
   specific archetype), `personality-bias` (e.g. "calm, no playful"),
   `palette-constraint`, `archetype-exclude`, `theme-scope`, `audience-note`,
   `other`.
3. Apply directives:
   - `archetype-pin` → that concept letter is locked to that archetype; the
     other two concepts must use different archetypes.
   - `personality-bias` → applied to the Personality Brief in Step 2.
   - `palette-constraint` / `archetype-exclude` → respected in Step 3 token decisions.
   - `theme-scope` → drives Step 5 (light only vs. light + dark).
   - `audience-note` → biases the Personality Brief.
4. Conflicts with PRD/inherited `user_brief` are resolved in favor of the new
   `$ARGUMENTS` and noted under User Input Notes in concept-brief.md.
5. Never silently ignore a directive.
6. The freeform `$ARGUMENTS` is **appended** to the inherited `user_brief`
   when written to concept-brief.md frontmatter.

---

## Process

---

### Step 1: Read upstream and identify the Primary Dashboard wireframe

1. Read `docs/design/wireframe-inventory.md`. Parse YAML frontmatter:
   - `sources.prd`, `sources.features`, `user_brief`
   - `figma.wireframes_file_key`
2. Read `docs/design/ux-spec.md` body. Identify the **primary
   authenticated screen** — the one a user lands on after login, which they
   use most frequently. The decision surface.
3. Open the wireframes Figma file via `figma.wireframes_file_key`. Locate
   the wireframe of the primary screen. Record its node id — concepts will
   render against this exact wireframe (same content boxes, same regions),
   only the visual treatment differs across concepts.
4. If the UX spec is ambiguous about which screen is primary, choose the
   wireframe that contains the most diverse element types (navigation, data,
   cards, actions).

State the chosen screen and Figma node id in the conversation before proceeding.

---

### Step 2: Extract Personality Signals

Read all input docs. Do not start designing. Extract signals that reveal the
product's visual personality. Look for:

**Emotional register:**
- What feeling should the product produce on first use? (calm focus, playful
  discovery, professional confidence, raw power, warm safety)
- Is this a tool (used daily, must stay out of the way) or an experience
  (used occasionally, can be expressive)?

**Audience signals:**
- Consumer / prosumer / professional / developer?
- Age, technical literacy, context of use (mobile-first? desktop-intensive?)
- Conservative or early-adopter profile?

**Domain signals:**
- What category is this? (finance, health, productivity, creative, social, infra)
- What do category leaders look like? (to align or deliberately contrast)
- What clichés exist in this space that must be avoided?

**Tone signals:**
- Adjectives used in the PRD to describe the product or brand
- What the product is *not* (often more revealing than what it is)
- Any explicit brand direction from the PRD

Produce a **Personality Brief**:
```
Emotional target:   [one or two words, e.g. "calm authority"]
Audience profile:   [one sentence]
Category context:   [dominant visual language in this space]
Clichés to avoid:   [2–4 specific things common in this category]
Defining contrast:  [one dimension where this product should look different]
```

Show the Personality Brief in the conversation before proceeding.

---

### Step 3: Define 3 Concept Directions

Using the Personality Brief, define 3 design directions. Each must represent a
**different design philosophy** — a genuine strategic choice, not a palette swap.

Structure each concept around one of these philosophy dimensions:

**Concept A — Data-forward / Minimal:**
Content is king. Chrome disappears. Typography does the work.
Tight spacing, restrained color, high information density.
The product trusts the user to navigate.

**Concept B — Structured / Confident:**
Clear hierarchy, strong grid, purposeful color use.
Every element has a defined role. Feels authoritative and reliable.
The product guides the user without being patronizing.

**Concept C — Expressive / Distinctive:**
A deliberate aesthetic risk. Stronger personality.
Something the category doesn't normally do — in color, type, spacing, or all three.
The product has a point of view.

For each concept, derive a minimal token set sufficient to build the moodframe:

```
Concept [A/B/C]: [one-line philosophy statement]
─────────────────────────────────────────────────
Archetype:        [name from the archetype table below]
Deviation:        [what we break from the archetype norm, and why]
Primary hue:      [hex + rationale — derive from archetype + personality, not preference]
Surface (light):  [hex + rationale]
Text primary:     [hex]
Accent:           [hex + usage]
Typeface UI:      [Google Fonts — rationale]
Typeface display: [Google Fonts — rationale, must contrast with UI face]
Radius default:   [none / sm / md / lg / full + rationale]
Shadow system:    [flat / subtle / layered / colored + rationale]
Density:          [compact / balanced / spacious]
```

**Archetype reference:**

| Archetype | Default feel | Typical domains |
|---|---|---|
| Precision Tool | Sharp corners, tight spacing, monochrome base, data-dense | Dev tools, analytics, finance, infra |
| Warm Community | Rounded corners, saturated accent, generous spacing, humanist type | Social, marketplace, consumer apps |
| Premium Utility | Subtle contrast, refined type, restrained color, elevated shadows | Productivity, B2B SaaS, professional tools |
| Expressive Brand | Bold color, distinctive typeface, asymmetric layouts | Creative tools, consumer, entertainment |
| Clinical Trust | High white space, clean grid, muted palette, serif accents | Health, legal, education, compliance |
| Raw Power | Dark base, high contrast, glowing accents, tight grid | Gaming, developer CLI tools, AI infra |
| Friendly Pro | Rounded but not soft, playful accent on neutral base, clear hierarchy | No-code, SMB tools, onboarding-heavy apps |
| Minimal Canvas | Maximum whitespace, single accent, type-led hierarchy | Writing tools, portfolio, content-first |

Show all 3 concept definitions in the conversation before proceeding.

---

### Step 4: The "Would a Designer Be Embarrassed?" Test

Run this check against **each concept independently**. Flag and correct before
building anything in Figma.

**Color failures:**
- [ ] Primary is pure blue (#0066FF or equivalent) with no hue shift — **too generic**
- [ ] Primary is pure green — **overused in SaaS**
- [ ] Primary is pure purple — **overused in AI products post-2023**
- [ ] All neutrals are pure gray (no warmth/coolness matching brand hue) — **disconnected palette**
- [ ] Two concepts share the same primary hue — **not real variety**

**Typography failures:**
- [ ] Both typefaces are geometric sans-serif — **no contrast**
- [ ] Body text below 16px — **mobile accessibility failure**
- [ ] Font pairing is Inter + Roboto or any two near-identical faces — **no personality**
- [ ] All 3 concepts use the same typeface pair — **not real variety**

**Variety failures:**
- [ ] Concepts differ only in color — **not different philosophies**
- [ ] All 3 use the same radius system — **missed a differentiation axis**
- [ ] All 3 use the same density — **missed a differentiation axis**
- [ ] Any concept could apply to a different product in the same category — **no personality**

**System failures:**
- [ ] Token decisions have no documented rationale — **cannot be defended**

Correct all flagged items. Re-check. Then proceed to Figma.

---

### Step 5: Build the Concept Moodframes in Figma

Create a new Figma file: **"[Product Name] — Concept Directions"**

**One page per concept.** Page names: `Concept A — [philosophy name]`,
`Concept B — [philosophy name]`, `Concept C — [philosophy name]`.

**Each page contains:**

**Token strip (top, compact):**
A horizontal strip showing the concept's minimal token set:
- Primary color swatch + hex
- Surface color swatch + hex
- Text color swatch + hex
- Accent color swatch + hex
- Typeface specimen: UI face at body size + display face at heading-1 size
- Radius preview: one rounded box at default radius
- One-line philosophy statement

**Moodframe artboard (1440×900):**
A realistic render of the primary dashboard screen. **Use the wireframe of
the primary dashboard from the wireframes Figma file as the structural
basis** — same regions, same content slots, same number of cards/columns.
Differences across concepts are visual only (color, type, density, radius,
shadow). Use actual product content from the PRD and feature descriptions —
no lorem ipsum.

The artboard must include:
- Navigation (top bar or sidebar, per UX spec)
- Page heading / context indicator
- At least 2 of: data card, list, table, chart placeholder, metric stat
- At least 1 primary action (button)
- At least 1 secondary UI element (badge, tag, status indicator)
- Realistic copy — feature names, metric labels, action labels from the PRD

Render at the density appropriate to the concept (compact / balanced / spacious).
Apply the concept's tokens consistently. Use auto-layout. Named layers.

**Do not build a full component library.** The moodframe uses inline styles
sufficient to convey the visual direction. Reusable components are built in
`design-system-builder` after direction is approved.

---

### Step 6: Present for Review

After building all 3 moodframes, stop and present:

```
## ⏸ Concept Directions — Ready for Review

**Primary screen:** [screen name from UX spec]

---

**Concept A — [philosophy name]**
[one-sentence description of the philosophy and what it communicates about the product]
Figma: [link to page]

**Concept B — [philosophy name]**
[one-sentence description]
Figma: [link to page]

**Concept C — [philosophy name]**
[one-sentence description]
Figma: [link to page]

---

Reply with:
- **A**, **B**, or **C** — to approve that direction
- **A with [adjustment]** — to approve a direction with a specific change
- **none — [what you want instead]** — to redirect; I will redefine concepts and rebuild
```

Do not proceed until the human selects a direction.

---

### Step 7: Write docs/design/concept-brief.md with frontmatter

After the human approves a direction, read
`${CLAUDE_SKILL_DIR}/templates/concept-brief.md`. **Prepend the YAML
frontmatter schema declared in `## Outputs`** (copy `sources` and
`user_brief` verbatim from wireframe-inventory.md frontmatter; append any
`$ARGUMENTS` text to `user_brief`; copy `figma.wireframes_file_key`; add
`figma.concept_file_key`). Write to `docs/design/concept-brief.md`.

The document body must capture:
- The primary screen chosen + Figma node id (from Step 1)
- The Personality Brief (from Step 2)
- All 3 concept token decisions (from Step 3) — Concepts A, B, and C in full
- The approved direction: selected concept letter, philosophy name, any adjustments requested
- Approved token decisions (the final token set after any adjustments)
- Figma links to all concept pages and to the approved concept page

Then run the Self-Review Checklist + Iteration Loop below.

---

## Self-Review Checklist

Before declaring done, verify each item. Mark ✅/❌ inline.

**Structural:**
- [ ] `docs/design/concept-brief.md` exists with valid frontmatter
- [ ] `frontmatter.sources` and `frontmatter.user_brief` copied verbatim from wireframe-inventory (with $ARGUMENTS appended)
- [ ] `frontmatter.figma.concept_file_key` set
- [ ] `frontmatter.figma.wireframes_file_key` carried forward
- [ ] Output structure matches template; brief includes all 3 concepts in full plus the approved one
- [ ] A developer reading concept-brief.md could describe the approved concept without opening Figma

**UX best practices:**
- [ ] All 3 concepts use the actual primary-dashboard wireframe structure (same content slots; only visual treatment differs)
- [ ] No concept has lorem ipsum or generic placeholders — labels match the PRD/feature names
- [ ] Each concept's token decisions support readable body text (≥ 16px on mobile)

**UI best practices:**
- [ ] Personality Brief produced before any token decision
- [ ] Each concept has documented rationale per token (not "looks nice")
- [ ] Concepts differ on at least 2 dimensions beyond color (e.g. radius + density, type pairing + shadow system)
- [ ] No concept passes the embarrassment-test failures (Step 4) without correction
- [ ] Primary color in each concept passes WCAG AA against its surface token
- [ ] No two concepts share the same primary hue
- [ ] No two concepts share the same archetype

**Visual screenshot review:**
- [ ] Render the moodframe of each concept via `mcp__plugin_figma_figma__get_screenshot`
- [ ] **Look at each rendered image.** Confirm: visual hierarchy is clear,
      type is legible, color usage feels intentional, no broken layout, no
      off-canvas content
- [ ] Confirm side-by-side: the 3 moodframes look meaningfully different at a glance

**$ARGUMENTS handling:**
- [ ] Every directive applied OR noted under User Input Notes
- [ ] No directive silently ignored
- [ ] `archetype-pin` directives respected; other concepts use different archetypes

---

## Iteration Loop

```
attempts = 0
loop:
  run Self-Review Checklist (including screenshot review)
  if all items pass:
    finalize files and emit success block (see Chain)
    exit
  if any items fail and attempts < 2:
    attempts += 1
    fix the flagged items only (Figma edits + spec edits as needed)
    continue
  else:  # attempts == 2 and items still failing
    APPEND a "## ⚠ Outstanding Issues" section to docs/design/concept-brief.md
      listing each unresolved checklist item with a one-line reason
    PRINT to console (verbatim):
      ⚠ /forge-design:concept completed with N unresolved issues — see docs/design/concept-brief.md
    exit
```

---

## Quality Bar (pre-condition gate)

Do not begin the iteration loop until all of these pass.

- [ ] wireframe-inventory.md exists; frontmatter parsed
- [ ] PRD and feature files at the inherited paths read
- [ ] Primary-dashboard wireframe identified in the Figma file (node id recorded)
- [ ] `$ARGUMENTS` parsed
- [ ] All 3 concepts defined and embarrassment-test-cleaned in working memory
- [ ] All 3 moodframes built in Figma against the primary-dashboard wireframe
- [ ] Human approved a direction before writing concept-brief.md

---

## Chain

After the iteration loop exits successfully, tell the user:
> "Concept [X] — [philosophy name] approved. Concept brief written to
> `docs/design/concept-brief.md`. Frontmatter cascaded from wireframe-inventory
> with concept Figma file key added.
>
> Next: run `/forge-design:ds` to expand the approved concept into a full
> design system in Figma (no arguments needed — inputs are inherited)."

If the iteration loop exited with outstanding issues, additionally print:
> "⚠ N issues remain unresolved — see `## ⚠ Outstanding Issues` in
> `docs/design/concept-brief.md`. Resolve before proceeding, or accept and continue."