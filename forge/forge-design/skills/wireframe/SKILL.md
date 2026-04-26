---
description: >
  Fills every screen from the UX spec with lo-fi wireframe content in Figma.
  Duplicates the approved layout shells, then populates content zones with
  gray-box elements — tables, cards, forms, stats, CTAs. Desktop + mobile
  for every screen. Inherits inputs from layout-spec.md frontmatter; works
  in the same Figma file created by /forge-design:layout. Runs third in the
  forge-design pipeline (ux → layout → wireframe → concept → ds → ui).
disable-model-invocation: true
---

# wireframe

## Usage

```
/forge-design:wireframe                                   # inherits everything from layout-spec
/forge-design:wireframe "skip post-MVP screens"
/forge-design:wireframe "use cards instead of tables on mobile; wireframe empty states for all data screens"
```

No `@`-paths needed — `sources` and `figma.wireframes_file_key` are read from
`docs/design/layout-spec.md` frontmatter. Pass freeform `$ARGUMENTS` for
wireframe-specific directives.

## Role
Take the approved layout shells and produce a complete set of wireframe screens.
For every screen in the UX spec, duplicate the correct layout variant and fill
its content zones with structural elements: tables, cards, form fields, stat
blocks, action buttons, lists, empty states.

Output is lo-fi: gray boxes with short text labels for identification. No color,
no icons (use text labels like `[search]`), no images (use crossed boxes). The
decisions made here are **what elements each screen contains** and **how they are
arranged within the approved layout**.

---

## Inputs

- `docs/design/layout-spec.md` — **required**. Read its YAML frontmatter to
  inherit `sources.prd`, `sources.features`, `user_brief`, and
  `figma.wireframes_file_key`. Read its body for layout variants and the
  screen-to-variant mapping.
- `docs/ux-designer/ux-spec.md` — **required**. Body: screen inventory,
  interaction patterns. (Frontmatter not re-read — it's already in layout-spec.)
- The PRD at `frontmatter.sources.prd` (md/pdf) — personas, feature names for realistic labels.
- The features at `frontmatter.sources.features` — each feature's
  description (user stories + acceptance criteria). Feature files are NOT
  enriched with UX notes; derive content from the description plus
  `docs/ux-designer/ux-spec.md` (screen inventory, interaction patterns).
- Figma file at `frontmatter.figma.wireframes_file_key` — opened to add wireframe pages.
- `$ARGUMENTS` — optional freeform wireframe directives (see below).

**Required-input gate:** if `docs/design/layout-spec.md` does not exist or
its frontmatter lacks `figma.wireframes_file_key` → STOP and tell the user:
> "No upstream layout-spec with frontmatter found. Run `/forge-design:layout`
> first."

## Outputs

- Figma file: **same file** as `/forge-design:layout` — new pages added with
  wireframed screens.
- `docs/design/wireframe-inventory.md` — populated from
  `${CLAUDE_SKILL_DIR}/templates/wireframe-inventory.md`. **Opens with
  YAML frontmatter** copied verbatim from layout-spec.md frontmatter, with
  any `$ARGUMENTS` text appended to `user_brief`:

```yaml
---
sources:
  prd: <copied>
  features: <copied>
user_brief: |
  <copied + appended $ARGUMENTS>
figma:
  wireframes_file_key: <copied>
---
```

---

## Operator Profile (fixed)

Solo developer. No dedicated designer. Wireframes exist to verify screen
completeness and content layout before expensive visual design. Speed and
coverage matter more than aesthetics.

---

## Handling $ARGUMENTS

`$ARGUMENTS` may contain directives. Examples:

- "skip post-MVP screens entirely"
- "dashboard needs a chart placeholder above the table"
- "use cards instead of table rows for session list"
- "add a quick-action floating button on mobile screens"
- "wireframe empty states for all data screens"

### Parsing rules

1. Read `$ARGUMENTS` in full before wireframing.
2. Classify each directive: `content-change`, `element-swap`, `screen-skip`,
   `state-variant`, `mobile-specific`, `other`.
3. Apply to relevant screens. If a directive conflicts with the UX spec or
   the inherited `user_brief`, follow the directive and note the conflict
   in wireframe-inventory.md under User Input Notes.
4. Never silently ignore a directive.
5. The freeform `$ARGUMENTS` is **appended** to the inherited `user_brief`
   when written to the wireframe-inventory frontmatter.

---

## Process

---

### Step 1: Load layout context

1. Read `docs/design/layout-spec.md`. Parse YAML frontmatter:
   - `sources.prd`, `sources.features`, `user_brief`
   - `figma.wireframes_file_key`
2. Read layout-spec.md body. Extract layout variants with zone definitions
   and the screen-to-variant mapping.
3. Read the PRD at `sources.prd` and every feature file under `sources.features`.
4. Read `docs/ux-designer/ux-spec.md` body for screen inventory and
   interaction patterns.

Build the full screen list with:
- Screen name
- Feature it belongs to
- Layout variant to use
- Key elements per screen — derive from the screen's entry in
  `ux-spec.md`'s screen inventory (Purpose, Key actions) and the
  feature's user stories + acceptance criteria
- State variants needed (empty, loading, error — only if they affect layout)
- MVP vs. post-MVP status

---

### Step 2: Open the Figma file and verify layout frames

Open the Figma file using the file key from layout-spec.md.
Verify the Layout Shells page exists with the expected variant frames.

If layout frames are missing or don't match the spec, stop and tell the user.

---

### Step 3: Wireframe all screens

**Visual rules (enforce strictly):**
- Background: white (`#FFFFFF`)
- All UI elements: light gray fill (`#E5E7EB`) with `#9CA3AF` border, 1px
- Text: black (`#111827`), Inter, two sizes only:
  - 14px for labels and body text
  - 18px bold for page titles and section headings
- Placeholder images: gray box with diagonal cross (`×`) and label (e.g. `[Avatar]`, `[Chart]`)
- Placeholder icons: text label in brackets (e.g. `[search]`, `[bell]`, `[menu]`)
- No color — no brand colors, no status colors, no tinted fills
- No rounded corners except pills and avatars
- Auto-layout on all frames
- Named layers: `Content/[SectionName]/[ElementName]`

**Page structure:**

Create new pages in the existing Figma file:

- **Page: Global Screens** — auth (login, signup, password reset), error (404, system)
- **Page: Feature NNN — [Name]** — one page per feature, all screens for that feature
- **Page: Post-MVP** _(if applicable)_ — deferred screens, labeled `[POST-MVP]`

**For each screen:**

1. Duplicate the correct layout variant frame (desktop + mobile) from the
   Layout Shells page.

2. Set the navigation active state — highlight the correct nav item for this
   screen (bold label or underline — keep it lo-fi).

3. Fill the content zone with elements derived from `ux-spec.md` (the
   screen's entry in the screen inventory + relevant flows) and the
   feature's user stories / acceptance criteria:
   - **Page title** — realistic name from the feature description
   - **Data displays** — tables, card grids, stat blocks, lists
   - **Forms** — labeled input fields, dropdowns, textareas, submit buttons
   - **Actions** — primary CTA button, secondary actions, destructive actions
   - **Filters / toolbars** — search bars, filter dropdowns, sort controls
   - **Empty states** — message + CTA for screens with dynamic data
   - Use realistic labels from the PRD and feature descriptions — no lorem ipsum

4. Name the frames: `[ScreenName] — Desktop`, `[ScreenName] — Mobile`

5. For state variants that change layout (empty state, error state), create
   additional frames: `[ScreenName] — Desktop — Empty`,
   `[ScreenName] — Mobile — Empty`. Skip states that are purely visual
   (color changes, animations).

**Mobile-specific adjustments:**
- Tables become card lists or horizontally scrollable
- Multi-column content stacks vertically
- Desktop side panels become separate screens or bottom sheets
  (label: `[→ drawer]` or `[→ separate screen]`)
- Touch targets: all interactive elements at least 44px height
- Bottom tab bar shows correct active state

---

### Step 4: Completeness check

After all screens are wireframed, verify:

- [ ] Every screen in the UX spec's screen inventory has a desktop wireframe
- [ ] Every screen has a mobile wireframe
- [ ] Every feature has at least one screen
- [ ] Navigation active state is correct on every screen
- [ ] Empty states wireframed for screens that display dynamic data
- [ ] Auth flow screens present (login, signup, reset)
- [ ] Error screens present (404, system error)
- [ ] No screen references a nav item that doesn't exist in the layout shell
- [ ] Post-MVP screens are on a separate page and labeled
- [ ] All frames use auto-layout
- [ ] All layers named (no "Frame 47", "Rectangle 12")
- [ ] Screen-to-variant mapping from layout-spec.md is respected — no screen
      uses the wrong layout variant

Fix any gaps before presenting.

---

### Step 5: Present for review

```
## ⏸ Wireframes — Ready for Review

**Figma file:** [link]

**Coverage:**
- [N] screens wireframed (desktop + mobile)
- [N] state variants (empty, error)
- [N] post-MVP screens (flagged, separate page)

**Screens by feature:**
- [Feature name]: [screen names]
- ...

**Flagged issues:**
- [any missing content, ambiguous flows, or UX spec gaps discovered]

Reply with:
- **approved** — I'll write the wireframe inventory
- **changes: [list]** — I'll update specific screens and re-present
```

Do not write wireframe-inventory.md until the human approves.

---

### Step 6: Write docs/design/wireframe-inventory.md with frontmatter

After approval, populate the template at
`${CLAUDE_SKILL_DIR}/templates/wireframe-inventory.md`. **Prepend the YAML
frontmatter schema declared in `## Outputs`** (copy `sources`, `user_brief`,
and `figma.wireframes_file_key` verbatim from layout-spec.md frontmatter;
append any `$ARGUMENTS` text to `user_brief`). Write to
`docs/design/wireframe-inventory.md`.

Then run the Self-Review Checklist + Iteration Loop below.

---

## Self-Review Checklist

Before declaring done, verify each item. Mark ✅/❌ inline.

**Structural:**
- [ ] `docs/design/wireframe-inventory.md` exists with valid frontmatter
- [ ] `frontmatter.sources` and `frontmatter.figma.wireframes_file_key` copied verbatim
- [ ] `frontmatter.user_brief` includes inherited brief + appended $ARGUMENTS
- [ ] Output structure matches the template

**UX best practices:**
- [ ] Every screen in the UX spec has a desktop wireframe AND a mobile wireframe
- [ ] Every feature has at least one screen
- [ ] Navigation active state is correct on every screen
- [ ] Empty states wireframed for screens that display dynamic data
- [ ] Loading and error states wireframed where layout differs from default
- [ ] Auth flow screens present (login, signup, reset)
- [ ] Error screens present (404, system error)
- [ ] Mobile touch targets ≥ 44px on every interactive element
- [ ] Realistic labels — no lorem ipsum, no "Button 1"
- [ ] Post-MVP screens separated and labeled `[POST-MVP]`

**UI best practices (lo-fi structural):**
- [ ] Visual hierarchy clear within each screen (titles distinct from body)
- [ ] Spacing rhythm consistent (no orphan paddings)
- [ ] Type scale: only the two declared sizes (14 / 18 bold)
- [ ] No color used (grayscale only)
- [ ] All frames use auto-layout
- [ ] All layers named (`Content/[SectionName]/[ElementName]`)

**Visual screenshot review:**
- [ ] Render a representative sample (≥ 1 per feature page) via `mcp__plugin_figma_figma__get_screenshot`
- [ ] **Look at the rendered images.** Confirm: nothing overlaps, text is
      legible, mobile and desktop look like the same product, nav state is right
- [ ] Confirm empty/error variants visually distinct from default

**$ARGUMENTS handling:**
- [ ] Every directive applied OR noted under User Input Notes with reason
- [ ] No directive silently ignored
- [ ] Correct layout variant used for every screen (per layout-spec mapping)

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
    APPEND a "## ⚠ Outstanding Issues" section to docs/design/wireframe-inventory.md
      listing each unresolved checklist item with a one-line reason
    PRINT to console (verbatim):
      ⚠ /forge-design:wireframe completed with N unresolved issues — see docs/design/wireframe-inventory.md
    exit
```

---

## Quality Bar (pre-condition gate)

Do not begin the iteration loop until all of these pass.

- [ ] layout-spec.md exists; frontmatter parsed; Figma file key resolved
- [ ] UX spec body read
- [ ] PRD and feature files at the inherited paths read
- [ ] `$ARGUMENTS` parsed
- [ ] Step 4 (Completeness check) passes
- [ ] Human approval received before writing wireframe-inventory.md

---

## Chain

After the iteration loop exits successfully, tell the user:
> "Wireframes approved and inventory written to
> `docs/design/wireframe-inventory.md`. Frontmatter cascaded from layout-spec.
>
> Next: run `/forge-design:concept` to define the visual direction
> (no arguments needed — concepts will render against these wireframes)."

If the iteration loop exited with outstanding issues, additionally print:
> "⚠ N issues remain unresolved — see `## ⚠ Outstanding Issues` in
> `docs/design/wireframe-inventory.md`. Resolve before proceeding, or accept and continue."
