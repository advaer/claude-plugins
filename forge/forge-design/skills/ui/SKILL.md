---
description: >
  Designs all application screens in Figma by composing instances from the
  established Design System file. Produces pixel-perfect, developer-ready
  screens with named layers and linked variables. The DS file is the source
  of truth — components are instanced from it, never recreated. Inherits
  inputs from design-system.md frontmatter; accepts optional steering via
  $ARGUMENTS. Final skill in the forge-design pipeline (ux → layout →
  wireframe → concept → ds → ui).
disable-model-invocation: true
---

# ui

## Usage

```
/forge-design:ui                                          # inherits everything from design-system
/forge-design:ui "desktop only for now, skip mobile frames"
/forge-design:ui "use metric cards instead of a table on the overview; collapsible sidebar by default"
```

No `@`-paths needed — `sources` and the upstream Figma file keys are read
from `docs/design/design-system.md` frontmatter. Pass freeform `$ARGUMENTS`
for screen-level overrides or platform scope changes.

## Role
Design every screen in the application by **instancing components from the
Figma Design System file** and composing them into full layouts. The Design
System file has been human-reviewed and is the single source of truth for
all visual decisions — tokens, components, and variables.

This skill does not create new components. It consumes them. If a screen
requires a component that does not exist in the Design System file, flag it
and pause — do not improvise.

---

## Inputs

- `docs/design/design-system.md` — **required**. Read its YAML frontmatter
  to inherit `sources.prd`, `sources.features`, `user_brief`,
  `figma.wireframes_file_key`, `figma.concept_file_key`, `figma.ds_file_key`.
  Read its body for the component inventory and token reference.
- `docs/design/wireframe-inventory.md` — **required**. Read its body to get
  the screen-to-wireframe mapping (each screen has an existing wireframe to
  use as structural reference).
- `docs/ux-designer/ux-spec.md` body — screen inventory, navigation, flows, interaction patterns.
- The PRD at `frontmatter.sources.prd` (md/pdf) — product copy, personas, brand tone.
- The features at `frontmatter.sources.features` — each feature's
  description (user stories + acceptance criteria). Feature files are NOT
  enriched with UX notes by `ux`; derive screens, flows, and edge cases
  from `ux-spec.md` + each feature's description + the wireframe inventory.
- `docs/design/concept-brief.md` body — personality, archetype (for editorial moments).
- **Figma DS file** at `frontmatter.figma.ds_file_key` — read-only source
  of truth for components, tokens, variables. Published as a library.
- **Figma Screens file** — target file. If
  `frontmatter.figma.screens_file_key` already exists in a prior
  screen-inventory.md (re-run scenario), reuse it. On first run, this skill
  creates a new Figma file `[Product Name] — Screens` and records its key
  in screen-inventory.md frontmatter.
- `$ARGUMENTS` — optional freeform screen/layout directives (see below).

**Required-input gate:** if `docs/design/design-system.md` does not exist
or its frontmatter lacks `figma.ds_file_key` → STOP and tell the user:
> "No upstream design-system with frontmatter found. Run `/forge-design:ds` first."

If `docs/design/wireframe-inventory.md` is also missing → STOP with the
same kind of message pointing to `/forge-design:wireframe`.

---

## Outputs

- **Figma Screens file** — created on first run (or reused on re-runs).
  All screens composed from DS library component instances with linked
  variables and named layers. The DS file is read-only.
- `docs/design/screen-inventory.md` — populated from
  `${CLAUDE_SKILL_DIR}/templates/screen-inventory.md`. **Opens with YAML
  frontmatter** that copies inherited fields and adds the new screens key:

```yaml
---
sources:
  prd: <copied>
  features: <copied>
user_brief: |
  <copied + appended $ARGUMENTS>
figma:
  wireframes_file_key: <copied>
  concept_file_key: <copied>
  ds_file_key: <copied>
  screens_file_key: <created on first run; reused on re-run>
---
```

---

## Operator Profile (fixed)

Solo developer. No dedicated designer. Screens must look like they were
composed by a product designer, not assembled from generic component
library defaults. Every screen is a composition of the reviewed Design
System — visual consistency is non-negotiable.

---

## Handling $ARGUMENTS

`$ARGUMENTS` is free-form text. It may contain any mix of:

- **Screen overrides** — "make the dashboard single-column"
- **Content changes** — "use metric cards instead of a table on the overview"
- **Layout preferences** — "sidebar should be collapsible by default"
- **Screen additions** — "add an onboarding welcome screen"
- **Screen removals** — "skip the settings page, handle inline"
- **Platform focus** — "desktop only for now, skip mobile frames"
- **Any other steering input** — treat unrecognised directives as layout/design intent

### Parsing rules

1. Read `$ARGUMENTS` in full before doing anything else.
2. Classify each directive into one of: `screen-override`, `content-change`,
   `layout-preference`, `screen-add`, `screen-remove`, `platform-focus`, `other`.
3. Keep parsed directives in working memory.
4. Apply every directive to the relevant screen or layout decision.
5. Never silently ignore a directive. If a directive conflicts with the UX spec
   or the inherited `user_brief`, note the conflict under **User Input Notes**
   in `screen-inventory.md` and explain how it was resolved.
6. The freeform `$ARGUMENTS` is **appended** to the inherited `user_brief`
   when written to screen-inventory.md frontmatter.

If `$ARGUMENTS` is not provided, proceed using only the upstream documents
and inherited `user_brief`.

---

## Process

---

### Step 1: Resolve File Keys from Frontmatter

1. Read `docs/design/design-system.md`. Parse YAML frontmatter:
   - `sources.prd`, `sources.features`, `user_brief`
   - `figma.wireframes_file_key`, `figma.concept_file_key`, `figma.ds_file_key`
2. Read `docs/design/wireframe-inventory.md` body for the screen-to-wireframe mapping.
3. Check if `docs/design/screen-inventory.md` already exists with a populated
   `figma.screens_file_key` in its frontmatter:
   - **Re-run case**: reuse that Screens file.
   - **First run case**: the Screens file does not yet exist. Create a new
     Figma file via the Figma MCP named `[Product Name] — Screens`. Record
     the new file key as `figma.screens_file_key` for the output frontmatter.

Open both files via Figma MCP:
- **DS file** (`figma.ds_file_key`) — read-only source of truth.
- **Screens file** (`figma.screens_file_key`) — write target.

Confirm access:

```
Design System file loaded: [DS file key]   (read-only)
Screens file:              [Screens file key]   (write target — [created this run | reused])
Components in DS:          [count]
Variable collections:      [list]
DS library enabled in Screens file: [yes/no]
Proceeding with screen composition...
```

**If DS library is NOT enabled** in the Screens file, stop and tell the user:
> "The Screens file does not have the Design System library enabled.
> In the Screens file: Assets → Libraries → toggle the DS library on.
> Then re-run `/forge-design:ui`."

If `figma.ds_file_key` is missing from design-system.md frontmatter, stop and
tell the user to re-run `/forge-design:ds`.

---

### Step 2: Inventory Components Available

Using Figma MCP against the **DS file**, walk every page and collect, for each
`COMPONENT` / `COMPONENT_SET`:
- Name
- **Component key** (the `key` property, NOT the node `id`) — required for
  `importComponentByKeyAsync` / `importComponentSetByKeyAsync` when creating
  instances in the Screens file
- Variants / variant property definitions
- Page it lives on (for human reference in `screen-inventory.md`)

Also collect variable collections and their variables, capturing each
variable's **key** (for `importVariableByKeyAsync` when rebinding in the
Screens file).

Categorise the inventory:
- Navigation components (top nav variants, bottom tab bar, sidebar, drawer)
- Form elements (inputs, selects, checkboxes, radios, toggles, textareas)
- Feedback components (alerts, toasts, badges, empty states, skeletons, progress)
- Overlay components (modals, sheets, tooltips, dropdown menus)
- Data display components (tables, list items, metric cards, tags)
- Product-specific components (e.g. session cards, score displays)

Keep this inventory in working memory (component name → key map). Every
element placed on a screen must be an **instance** of a library component
imported via its key. No freehand drawing of elements that already exist
as components. No local component copies.

---

### Step 3: Build Screen List from UX Spec

Read `docs/ux-designer/ux-spec.md`, section **Screen Inventory**. Extract every
screen marked for MVP, grouped by:
1. Global screens (auth, errors, landing)
2. Feature screens (grouped by feature)
3. Post-MVP screens (reference only — do not design)

Cross-reference with each feature's `description.md` (user stories and
acceptance criteria) and the wireframe inventory to confirm screens, flows,
and edge cases. The UX spec body is the canonical screen list; feature
descriptions clarify behavior; the wireframe inventory locates the
existing wireframes to compose against.

Build the master screen list. Each entry needs:
- Screen name (must match UX spec naming exactly)
- Platform (desktop / mobile / both)
- Feature group
- Key components needed (from Step 2 inventory)
- Content requirements (realistic copy from PRD)

---

### Step 4: Prepare the Figma Screens File

The Screens file already exists (identified by the Screens file key from
Step 1). **Do not create a new file.** Work inside the existing file.

Ensure the page structure matches the UX spec screen grouping. For each
required group:
- If a matching page exists, reuse it.
- If not, create it via `figma.createPage()` and name it accordingly.

Target structure (adapt to the UX spec's actual feature groups):

```
Page 1  — Global: Auth & Onboarding
Page 2  — Global: Errors & System
Page 3  — [Feature Group 1 name]
Page 4  — [Feature Group 2 name]
...
Page N  — Mobile Variants (if in scope)
Page N+1 — Flows
```

If the Screens file already contains content from a previous run, do not
delete it silently. Diff against the target screen list and report to the
user before destructive changes:

```
Screens file already has [N] populated pages.
- Pages to keep/update: [...]
- Pages to add:         [...]
- Pages with stale content (not in current UX spec): [...]

How should I proceed? (update in place / clean and rebuild / abort)
```

---

### Step 5: Compose Each Screen

For each screen in the master list:

#### 5a. Set up the frame
- Desktop: 1440×900 frame (or height as needed for scrollable content)
- Mobile: 375×812 frame (iPhone-standard, or height as needed)
- Name the frame: `[Screen Name]` — matching UX spec exactly
- Set frame background to `surface-page` from the Design System variables

#### 5b. Instance navigation
- Import the correct navigation component from the DS library using
  `importComponentByKeyAsync(<key>)` (or `importComponentSetByKeyAsync` for
  variant sets), then call `.createInstance()` and append to the frame.
- Never duplicate an existing local instance — every instance originates from
  a fresh library import so it stays linked to the library.
- Set the active state to match the current screen's nav destination via the
  variant properties (`instance.setProperties({...})`).
- Bind variables using `importVariableByKeyAsync` + `setBoundVariable` (capture
  the returned paint and reassign — see figma-use skill rules).

#### 5c. Compose the content area
- Use **only instances** of DS library components (imported by key).
- Arrange using auto-layout matching the Design System's spacing tokens.
- Use realistic product copy (from PRD personas and features, not lorem ipsum)
- Handle all states visible on initial load:
  - Default state (data present)
  - Include empty state variant if this is the first screen a new user sees
- Apply the personality/archetype from `concept-brief.md` for editorial moments
  (headings, empty states, onboarding copy — use the display typeface where
  the design system defines editorial usage)

#### 5d. Name all layers
Follow the convention from the Design System file:
```
ScreenName / Section / ComponentInstance
```
Example: `Dashboard / Header / MetricCard-Score`

Every layer must be named. No "Frame 47" or "Group 12" in the final output.

#### 5e. Link variables
- All color fills reference Design System semantic variables
- All text styles reference Design System type tokens
- Theme switching (light/dark) must work by switching the variable collection
  mode — not by manual recoloring

---

### Step 6: Handle Edge-Case Screens

For each screen, derive edge cases from the feature's acceptance criteria
and the global empty/loading/error patterns in `ux-spec.md`. Design
these as additional frames on the same page:

- Empty state (no data yet)
- Error state (API failure, form validation)
- Loading state (skeleton or spinner — use Design System skeleton components)
- Limit reached (quota, plan restriction)
- Permission denied (if applicable)

Name edge-case frames: `[Screen Name] — [State]`
Example: `Dashboard — Empty`, `Dashboard — Loading`

Not every screen needs all edge cases. Design only those implied by the
feature's acceptance criteria and the global UX patterns in `ux-spec.md`.

---

### Step 7: Design Screen Flows

For critical user journeys identified in the UX spec (auth flow, onboarding,
primary task flow, payment flow), create a **flow page** that lays out the
screens in sequence with connector arrows showing the path.

Flow pages are reference compositions — they use the same screen frames but
arranged horizontally with annotations showing:
- User action that triggers the transition
- Any branching paths (success / error / cancel)

One flow page per critical journey. Name: `Flow — [Journey Name]`

---

### Step 8: Mobile Variants (if in scope)

If the UX spec defines mobile as in-scope for MVP:
- Create 375×812 frames for each mobile screen
- Instance the mobile navigation component (bottom tab bar)
- Adapt layouts: single-column, full-width cards, stacked forms
- Verify all touch targets ≥ 44×44px
- Tables → card lists on mobile
- Modals → bottom sheets on mobile (use Design System sheet component)
- Ensure skeletons match mobile layout, not desktop

If mobile is deferred, skip this step and note in `screen-inventory.md`.

---

### Step 9: Component Gap Check

After composing all screens, review whether any screen required a visual
element that does not exist in the Design System file.

If gaps exist:
- **Do not create ad-hoc components.** Flag each gap in the conversation:

```
⚠️ Component gap detected:
- [Screen Name] needs a [description] component not in the Design System
- Suggestion: [what it should look like / which existing component to extend]

Pause: should I create this in the Design System file first,
or proceed with a placeholder?
```

Wait for user direction before continuing.

---

### Step 10: Write docs/design/screen-inventory.md with frontmatter

Populate the template at `${CLAUDE_SKILL_DIR}/templates/screen-inventory.md`.
**Prepend the YAML frontmatter schema declared in `## Outputs`** (copy
`sources` and `user_brief` verbatim from design-system.md frontmatter;
append any `$ARGUMENTS` text to `user_brief`; copy
`figma.wireframes_file_key`, `figma.concept_file_key`, `figma.ds_file_key`;
add `figma.screens_file_key`).

The document body must include:
- Figma Screens file link and key
- Reference to the Design System file (link + key)
- Complete screen list with Figma links per screen/page
- Edge-case screens listed per feature
- Flow pages listed with journey names
- Mobile variant status (designed / deferred)
- Component gaps (if any were flagged and resolved)
- Responsive notes from UX spec carried forward

Then run the Self-Review Checklist + Iteration Loop below.

---

## Self-Review Checklist

Before declaring done, verify each item. Mark ✅/❌ inline.

**Structural:**
- [ ] `docs/design/screen-inventory.md` exists with valid frontmatter
- [ ] All inherited fields copied verbatim from design-system.md frontmatter
- [ ] `frontmatter.figma.screens_file_key` set (created this run or carried forward)
- [ ] Output structure matches the template

**UX best practices:**
- [ ] Every screen in the UX spec inventory has a corresponding Figma frame
- [ ] Screen names in Figma match UX spec exactly
- [ ] Edge-case states designed for screens where acceptance criteria or `ux-spec.md` patterns imply them
- [ ] Flow pages exist for each critical user journey in the UX spec
- [ ] Mobile frames use 375×812 and mobile navigation components (if mobile in scope)
- [ ] All mobile touch targets ≥ 44×44px (if mobile in scope)
- [ ] Realistic product copy used (no lorem ipsum, no "Button 1")
- [ ] Empty / error / loading states present where applicable
- [ ] Active navigation state correct on every screen

**UI best practices:**
- [ ] Every visual element on every screen is an **instance** of a DS component — no freehand for things that exist as components
- [ ] Every instance was imported from the library by key (no local component copies, no detached instances)
- [ ] All component instances use linked Figma Variables (semantic tokens)
- [ ] Light/dark theme switching works via variable mode toggle on every screen (if dark theme in scope)
- [ ] All layers named (`ScreenName / Section / ComponentInstance`); no "Frame N" or "Group N"
- [ ] Auto-layout used throughout
- [ ] No hardcoded color values anywhere
- [ ] Spacing matches DS spacing tokens (no orphan paddings)
- [ ] DS file untouched — only the Screens file written

**Visual screenshot review:**
- [ ] Render a representative sample of screens (≥ 1 per feature group;
      desktop + mobile if both in scope; light + dark if both in scope)
      via `mcp__plugin_figma_figma__get_screenshot`
- [ ] **Look at each rendered image.** Confirm:
      - Visual hierarchy is clear; primary actions are visually dominant
      - Spacing rhythm is consistent across screens
      - Empty / error variants visually distinct from default
      - Mobile renders fit within 375 px without truncation
      - Dark theme renders are genuinely dark-mode (no halation, no muddy contrast)
      - Nav active state is correct on every rendered screen
      - No broken / overlapping layers, no off-canvas content

**$ARGUMENTS handling:**
- [ ] Every directive applied OR noted under User Input Notes
- [ ] No directive silently ignored

**Component gaps:**
- [ ] No component gaps remain unresolved (gaps either resolved by user
      adding to DS, or explicitly accepted-as-placeholder with note)

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
    APPEND a "## ⚠ Outstanding Issues" section to docs/design/screen-inventory.md
      listing each unresolved checklist item with a one-line reason
    PRINT to console (verbatim):
      ⚠ /forge-design:ui completed with N unresolved issues — see docs/design/screen-inventory.md
    exit
```

---

## Quality Bar (pre-condition gate)

Do not begin the iteration loop until all of these pass.

- [ ] design-system.md frontmatter parsed; all upstream Figma file keys resolved
- [ ] wireframe-inventory.md body read; screen-to-wireframe mapping understood
- [ ] DS file opened via Figma MCP and component inventory built (name → key map)
- [ ] Screens file opened (or created on first run) via Figma MCP
- [ ] Screens file confirmed to have DS library enabled
- [ ] `$ARGUMENTS` parsed
- [ ] All MVP screens composed in Figma; edge-case states and flow pages added

---

## Chain

After the iteration loop exits successfully, tell the user:
> "All screens designed in Figma using the Design System components.
> Screen inventory written to `docs/design/screen-inventory.md`. Frontmatter
> includes the screens_file_key for re-runs.
>
> The forge-design pipeline is complete. Review the screens, then proceed
> to architecture (`/forge:architect`) and specification (Spec Kit)."

If the iteration loop exited with outstanding issues, additionally print:
> "⚠ N issues remain unresolved — see `## ⚠ Outstanding Issues` in
> `docs/design/screen-inventory.md`. Resolve before proceeding, or accept and continue."
