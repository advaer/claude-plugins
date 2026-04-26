---
description: >
  Draws lo-fi layout shells in Figma — navigation placement, header, user menu,
  content zones — for desktop and mobile. Produces labeled spatial frames showing
  where everything goes, plus a layout-spec.md documenting decisions and mapping
  layouts to screens. Inherits PRD/features/user_brief from ux-spec.md
  frontmatter; accepts optional layout directives via $ARGUMENTS. Runs second
  in the forge-design pipeline (ux → layout → wireframe → concept → ds → ui).
disable-model-invocation: true
---

# layout

## Usage

```
/forge-design:layout                                      # inherits everything from ux-spec
/forge-design:layout "left sidebar nav, no top header"
/forge-design:layout "mobile: bottom tab bar with 4 items, settings uses dashboard layout"
```

No `@`-paths needed — `sources.prd` and `sources.features` are read from
`docs/ux-designer/ux-spec.md` frontmatter. Pass freeform `$ARGUMENTS` for
layout-specific directives.

## Role
Define and draw the spatial architecture of the product. Decide where navigation
lives, whether there's a header, where the user menu sits, how content areas are
structured — then draw these decisions as labeled lo-fi frames in Figma.

Output is structural, not visual: gray rectangles with text labels showing zone
names and dimensions. No color, no icons, no styling. The only decisions made
here are **where things go** on the page.

Most products need 2–4 layout variants:
- **Authenticated default** — the shell most screens use (dashboard, lists, detail views)
- **Auth / guest** — login, signup, password reset (typically no sidebar, centered content)
- **Minimal / focused** — onboarding wizards, checkout, single-task flows
- **Settings / admin** — if structurally different from the default

Not every product needs all of these. Derive the set from the UX spec's screen
inventory.

---

## Inputs

- `docs/ux-designer/ux-spec.md` — **required**. Read its YAML frontmatter to
  inherit `sources.prd`, `sources.features`, and `user_brief`. Read its body
  for the screen inventory and navigation structure.
- The PRD file at `frontmatter.sources.prd` (md/pdf) — read for personas,
  product type, platform scope.
- The features at `frontmatter.sources.features` (folder OR file) — read
  every feature file. Feature files are NOT enriched with UX notes by `ux`;
  derive feature-level UX context from user stories + acceptance criteria
  in each `description.md`, plus `docs/ux-designer/ux-spec.md` for
  cross-feature flows and screen inventory.
- `$ARGUMENTS` — optional freeform layout directives (see below).

**Required-input gate:** if `docs/ux-designer/ux-spec.md` does not exist or
has no frontmatter → STOP and tell the user:
> "No upstream UX spec with frontmatter found. Run `/forge-design:ux` first
> (with `@prd` and `@features` paths or a freeform brief)."

## Outputs

- Figma file: **[Product Name] — Wireframes** (layout shell frames on a reference page).
- `docs/design/layout-spec.md` — populated from
  `${CLAUDE_SKILL_DIR}/templates/layout-spec.md`. **Opens with YAML
  frontmatter** that copies inherited `sources` + `user_brief` from ux-spec
  and adds the figma block:

```yaml
---
sources:
  prd: <copied from ux-spec.md>
  features: <copied from ux-spec.md>
user_brief: |
  <copied from ux-spec.md, plus any freeform $ARGUMENTS appended>
figma:
  wireframes_file_key: <created in this run>
---
```

---

## Operator Profile (fixed)

Solo developer. No dedicated designer. Layout decisions must be explicit and
reviewable before any screen design begins. Speed and clarity over polish.

---

## Handling $ARGUMENTS

`$ARGUMENTS` may contain layout directives. Examples:

- "left sidebar navigation, no top header"
- "user menu bottom-left of sidebar"
- "top header with centered logo, no sidebar"
- "mobile: bottom tab bar with 4 items"
- "settings uses same layout as dashboard"
- "auth pages: centered card, no nav"

### Parsing rules

1. Read `$ARGUMENTS` in full before any design decisions.
2. Classify each directive: `nav-placement`, `header`, `user-menu`,
   `content-grid`, `mobile-layout`, `variant-scope`, `other`.
3. If a directive conflicts with the UX spec or the inherited `user_brief`,
   follow the directive (human intent overrides spec) and note the conflict
   in layout-spec.md under User Input Notes.
4. Never silently ignore a directive.
5. The freeform `$ARGUMENTS` is **appended** to the inherited `user_brief`
   when written to the layout-spec frontmatter — both cascade downstream.

---

## Process

---

### Step 1: Read all inputs

1. Read `docs/ux-designer/ux-spec.md`. Parse its YAML frontmatter:
   - `sources.prd` → path to PRD
   - `sources.features` → path to features folder/file
   - `user_brief` → cumulative freeform brief from upstream
2. Read the PRD at `sources.prd` (md or pdf — Read tool handles both).
3. Read every feature file under `sources.features` (folder = recursive).
4. Read the UX spec body for navigation structure and screen inventory.
5. Read `$ARGUMENTS` layout directives.

Extract:
- Navigation structure defined in the UX spec
- Full screen inventory
- Platform scope (web, mobile, both)
- Screen groupings that imply different layouts (auth screens vs. app screens vs. wizard flows)
- Any `$ARGUMENTS` layout directives

---

### Step 2: Determine layout variants needed

Scan the screen inventory. Group screens by structural similarity:
- Which screens share a sidebar + content layout?
- Which screens have no persistent navigation (auth, onboarding)?
- Which screens need a different content structure (settings, wizard)?

Define 2–4 layout variants. Each variant gets a name and a description of which
screens use it.

State the variants in the conversation:
```
Layout variants identified:
1. [Variant name] — [description] — used by: [screen list]
2. [Variant name] — [description] — used by: [screen list]
...
```

---

### Step 3: Design each layout variant

For each variant, make explicit decisions on:

**Desktop (1440×900):**

| Decision | Options |
|----------|---------|
| Navigation | Left sidebar (persistent/collapsible/toggle), top bar, combined, none |
| Header | Present/absent; contents (breadcrumbs, page title, actions, search) |
| User menu | Position (top-right, bottom-left sidebar, etc.); contents |
| Content area | Single column, two-column, adaptive; max-width; panel structure |

**Mobile (375×812):**

| Decision | Options |
|----------|---------|
| Navigation | Bottom tab bar (N items), hamburger drawer, combined, none |
| Header | Top bar contents (back arrow, title, action icons) |
| User menu | In drawer, tab bar, settings screen |
| Content collapse | How desktop multi-column maps to single column |

---

### Step 4: Draw layout frames in Figma

Create a new Figma file: **"[Product Name] — Wireframes"**

**Visual rules for layout frames:**
- Background: white (`#FFFFFF`)
- Zone rectangles: light gray fill (`#E5E7EB`) with `#9CA3AF` border, 1px
- Zone labels: black (`#111827`), Inter 14px, inside each zone
- Dimension annotations: `#6B7280`, Inter 12px (e.g. "240px", "64px", "flex")
- No content inside zones — just the zone label
- Auto-layout on all frames
- Named layers: `Shell/[VariantName]/[ZoneName]`

**Page: Layout Shells**

For each variant, create:
1. **Desktop frame** (1440×900) — zones labeled:
   - Navigation zone (with position, width)
   - Header zone (if present, with height)
   - User menu zone (with position)
   - Content zone (with width constraints)
   - Any persistent panels (filter sidebar, detail panel)

2. **Mobile frame** (375×812) — zones labeled:
   - Navigation zone (bottom bar, drawer trigger)
   - Header zone (top bar)
   - Content zone
   - Any overlay zones (drawer, bottom sheet)

Label each frame: `Layout: [Variant Name] — Desktop`, `Layout: [Variant Name] — Mobile`

---

### Step 5: Present for review

```
## ⏸ Layout Shells — Ready for Review

**Figma file:** [link]

**Variants:**
1. [Variant name] — [one-line summary] — [N] screens use this
2. [Variant name] — [one-line summary] — [N] screens use this
...

**Desktop decisions:**
- Navigation: [summary]
- Header: [summary]
- User menu: [summary]

**Mobile decisions:**
- Navigation: [summary]
- User menu: [summary]

Reply with:
- **approved** — I'll write the layout spec
- **changes: [list]** — I'll update and re-present
```

Do not write layout-spec.md until the human approves.

---

### Step 6: Write docs/design/layout-spec.md with frontmatter

After approval, populate the template at
`${CLAUDE_SKILL_DIR}/templates/layout-spec.md`. **Prepend the YAML frontmatter
schema declared in `## Outputs`** (copy `sources` and `user_brief` verbatim
from ux-spec.md frontmatter; append any `$ARGUMENTS` text to `user_brief`;
add `figma.wireframes_file_key`). Write to `docs/design/layout-spec.md`.

Then run the Self-Review Checklist + Iteration Loop below.

---

## Self-Review Checklist

Before declaring done, verify each item. Mark ✅/❌ inline.

**Structural:**
- [ ] `docs/design/layout-spec.md` exists with valid frontmatter
- [ ] `frontmatter.sources` copied verbatim from ux-spec.md frontmatter
- [ ] `frontmatter.user_brief` includes inherited brief + appended $ARGUMENTS
- [ ] `frontmatter.figma.wireframes_file_key` set
- [ ] Output structure matches the template

**UX best practices:**
- [ ] Every screen in the UX spec is assigned to a layout variant
- [ ] Layout variants are minimal — no variant exists without ≥1 screen using it
- [ ] Navigation pattern is consistent within a variant (no "sidebar on some, top nav on others" inside the same variant)
- [ ] Mobile reachability: primary actions reachable with one thumb on a 375-wide frame
- [ ] Onboarding/wizard layout has no persistent sidebar (focused-task convention)
- [ ] Auth/guest layout is centered card without app navigation

**UI best practices (lo-fi structural):**
- [ ] Zone labels are clear and unambiguous
- [ ] Dimension annotations present on navigation and header zones
- [ ] All frames use auto-layout
- [ ] All layers named (no "Frame 47" or "Rectangle 12")
- [ ] Spacing intent is consistent (4 or 8 px grid implied by dimensions)

**Visual screenshot review:**
- [ ] Render each layout variant frame via `mcp__plugin_figma_figma__get_screenshot`
- [ ] **Look at the rendered image.** Confirm: zones don't overlap, nothing
      bleeds off-canvas, labels are legible, dimensions look balanced
- [ ] Confirm desktop and mobile frames look like the same product family

**$ARGUMENTS handling:**
- [ ] Every directive applied OR noted under User Input Notes with a reason
- [ ] No directive silently ignored

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
    APPEND a "## ⚠ Outstanding Issues" section to docs/design/layout-spec.md
      listing each unresolved checklist item with a one-line reason
    PRINT to console (verbatim):
      ⚠ /forge-design:layout completed with N unresolved issues — see docs/design/layout-spec.md
    exit
```

---

## Quality Bar (pre-condition gate)

Do not begin the iteration loop until all of these pass.

- [ ] ux-spec.md exists and frontmatter parsed
- [ ] PRD and feature files at the inherited paths read
- [ ] `$ARGUMENTS` parsed
- [ ] Layout variants identified and presented for review
- [ ] Human approval received before writing layout-spec.md

---

## Chain

After the iteration loop exits successfully, tell the user:
> "Layout shells approved and spec written to `docs/design/layout-spec.md`.
> Frontmatter cascaded from ux-spec.
>
> Next: run `/forge-design:wireframe` to fill all screens with wireframe
> content using these layouts (no arguments needed — inputs are inherited)."

If the iteration loop exited with outstanding issues, additionally print:
> "⚠ N issues remain unresolved — see `## ⚠ Outstanding Issues` in
> `docs/design/layout-spec.md`. Resolve before proceeding, or accept and continue."
