---
description: >
  Builds the full design system in Figma from an approved concept direction:
  expands the minimal concept tokens into a complete semantic token set, then
  constructs the full component library (light + dark themes, mobile variants).
  Inherits inputs from concept-brief.md frontmatter; accepts optional steering
  via $ARGUMENTS. Runs fifth in the forge-design pipeline (ux → layout →
  wireframe → concept → ds → ui).
disable-model-invocation: true
---

# ds

## Usage

```
/forge-design:ds                                          # inherits everything from concept-brief
/forge-design:ds "skip dark theme for MVP — light only"
/forge-design:ds "tighten density one notch from concept; primary scale must include a -950 step"
```

No `@`-paths needed — `sources` and the upstream Figma file keys are read
from `docs/design/concept-brief.md` frontmatter. Pass freeform `$ARGUMENTS`
for design-system-specific directives.

## Role
Execute the approved visual direction into a production-ready design system.
The thinking is done — `concept-designer` produced the personality brief,
archetype, deviation, and token rationale. This skill takes that spec and
builds with precision and completeness.

Every token has a rationale (inherited from the concept brief). Every component
uses semantic tokens. Light and dark themes are first-class from token zero.
Mobile-friendliness is a constraint, not an afterthought.

---

## Inputs

- `docs/design/concept-brief.md` — **required**. Read its YAML frontmatter
  to inherit `sources.prd`, `sources.features`, `user_brief`, and the
  `figma.wireframes_file_key` + `figma.concept_file_key`. Read its body for
  the approved direction, token decisions, personality brief, archetype.
- The PRD at `frontmatter.sources.prd` (md/pdf) — brand tone, product type, personas.
- The features at `frontmatter.sources.features` — used to build the component inventory.
- `docs/design/ux-spec.md` body — navigation structure, interaction patterns, screen inventory.
- `$ARGUMENTS` — optional freeform design-system directives (see below).

**Required-input gate:** if `docs/design/concept-brief.md` does not exist
or its frontmatter lacks `figma.concept_file_key` → STOP and tell the user:
> "No upstream concept-brief with frontmatter found. Run `/forge-design:concept`
> first and approve a direction."

---

## Outputs

- Figma file: **[Product Name] — Design System** — full component library,
  light + dark, mobile variants.
- `docs/design/design-system.md` — populated from
  `${CLAUDE_SKILL_DIR}/templates/design-system.md`. **Opens with YAML
  frontmatter** that copies inherited fields from concept-brief and adds
  the new ds_file_key:

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
  ds_file_key: <created in this run>
---
```

---

## Operator Profile (fixed)

Solo developer. No dedicated designer. Visual output must look like it was made
by a product designer, not assembled from component library defaults. The design
system is used directly by `/forge-design:ui` and then implemented by developers.

---

## Handling $ARGUMENTS

`$ARGUMENTS` may contain design-system directives. Examples:

- "skip dark theme for MVP — light only"
- "tighten density one notch from concept; primary scale must include a -950 step"
- "no shadow system — keep flat"
- "add a brand-secondary token alongside primary; use it for accent CTAs"
- "use Inter Tight for display, not the concept's choice"

### Parsing rules

1. Read `$ARGUMENTS` in full before building anything.
2. Classify each directive: `theme-scope`, `density-shift`, `token-add`,
   `token-override`, `scale-extend`, `system-feature` (e.g. shadow on/off),
   `other`.
3. Apply directives:
   - `theme-scope` → drives whether dark mode pages exist; if light-only,
     skip Pages 4 and 11.
   - `density-shift` → adjusts the spacing rhythm uniformly.
   - `token-add` / `token-override` → modifies the semantic token list.
   - `scale-extend` → extends primitive scales.
   - `system-feature` → enables/disables shadow system, etc.
4. Conflicts with the concept brief (which represents an approved decision)
   are resolved in favor of the new `$ARGUMENTS` and noted under
   User Input Notes in design-system.md.
5. Never silently ignore a directive.
6. The freeform `$ARGUMENTS` is **appended** to the inherited `user_brief`
   when written to design-system.md frontmatter.

---

## Process

---

### Step 1: Load the Approved Concept

1. Read `docs/design/concept-brief.md`. Parse YAML frontmatter:
   - `sources.prd`, `sources.features`, `user_brief`
   - `figma.wireframes_file_key`, `figma.concept_file_key`
2. Read concept-brief.md body. Extract:
   - Approved concept token decisions (primary hue, surfaces, type pairing, radius, shadow, density)
   - Personality brief (emotional target, audience, clichés to avoid)
   - Archetype and deviation
3. Read the PRD at `sources.prd` and feature files under `sources.features`
   for personas, brand tone, and the component inventory.
4. Read `docs/design/ux-spec.md` body for navigation structure and
   interaction patterns.

Do not re-derive concept decisions. They are already made. This skill executes them.

Confirm in the conversation:
```
Loading approved concept: [philosophy name]
Archetype: [name] | Deviation: [summary]
Building full token set from concept brief...
```

---

### Step 2: Expand the Full Token Set

Using the concept's minimal token decisions, expand to the complete semantic token
architecture.

#### Token Architecture (3 layers)

**Layer 1 — Primitive tokens** (raw values, theme-agnostic)
Named by value, not intent. Never referenced directly in components.
Examples: `gray-50`, `gray-900`, `violet-500`, `green-400`

Derive a full primitive palette from the approved hue:
- Neutral scale: 12 steps (50 through 950), warm or cool to match brand hue
- Primary scale: 9 steps (50 through 900)
- Semantic status scales: success (green), warning (amber), error (red), info (blue)
  — each 3 steps minimum (subtle tint, mid, strong)

**Layer 2 — Semantic tokens** (intent-mapped, theme-aware)
Different primitives per theme. This is the layer components reference.

Required semantic tokens:

*Surface:*
- `surface-page` — main background
- `surface-raised` — cards, panels, modals
- `surface-overlay` — tooltips, popovers
- `surface-sunken` — input backgrounds, code blocks

*Text:*
- `text-primary` — main body text
- `text-secondary` — supporting text, labels
- `text-muted` — placeholders, disabled
- `text-inverse` — text on colored backgrounds
- `text-link` — interactive text

*Border:*
- `border-subtle` — dividers, low-emphasis borders
- `border-default` — standard input/card borders
- `border-strong` — focus rings, selected states

*Action:*
- `action-primary` — primary button bg
- `action-primary-hover`
- `action-primary-text` — text on primary button
- `action-secondary` — secondary button bg
- `action-secondary-hover`
- `action-destructive` — delete/danger actions

*Semantic status:*
- `status-success`, `status-success-subtle`
- `status-warning`, `status-warning-subtle`
- `status-error`, `status-error-subtle`
- `status-info`, `status-info-subtle`

**Dark mode mapping rules:**
- Do not invert light theme — map independently
- `surface-page` dark: dark-but-not-black (avoid #000000 and #111111 — causes halation on OLED)
- Neutrals: warm or cool to match brand hue — not pure gray
- Shadows become less visible on dark — compensate with `border-subtle` tokens
- Re-check WCAG AA contrast for all text/surface combinations independently

**Layer 3 — Component tokens** (optional)
Add only if a component needs a value that doesn't fit the semantic layer cleanly.

#### Typography System

Inherit typeface decisions from concept brief. Expand to full scale:

```
display    — hero text, major numbers
heading-1  — page titles
heading-2  — section titles
heading-3  — card titles, subsections
body       — standard reading text (≥ 16px / 1rem)
body-sm    — secondary body, captions
label      — UI labels, form labels (≥ 12px)
label-sm   — badges, micro labels
```

Define per step: size (rem), line-height, letter-spacing, weight, typeface.
Mobile check: verify `body` ≥ 16px and `label` ≥ 12px.

#### Spacing System

4px base grid:
```
space-1:  4px    space-6:  24px
space-2:  8px    space-8:  32px
space-3:  12px   space-10: 40px
space-4:  16px   space-12: 48px
space-5:  20px   space-16: 64px
```

Apply density from approved concept (compact / balanced / spacious) as the
default spatial rhythm for components.

#### Radius and Shadow

Inherit from concept brief. Define full token scales:
- `radius-none`, `radius-sm`, `radius-md`, `radius-lg`, `radius-full`
- Shadow: 3–4 elevation levels if using shadow system; none if flat

---

### Step 3: Build the Figma Design System File

Create a new Figma file: **"[Product Name] — Design System"**

**Page 1 — Token Reference**
- All primitive tokens as color swatches (labeled name + hex)
- All semantic tokens in two-column layout: Light theme | Dark theme
- Typography scale: each step as live text with specs annotated
- Spacing scale: visual ruler with pixel values
- Radius tokens: 4 boxes showing each value
- Shadow tokens: 4 boxes showing each elevation (light and dark variants)

**Page 2 — Core Components (light theme)**
Three foundational components:
1. **Button** — primary filled, secondary outlined, destructive, disabled;
   all min 44px height
2. **Input field** — default, focused (focus ring), filled, error, disabled;
   with label and helper text
3. **Card** — with title, body text, secondary text element, primary action button

All components: semantic tokens only (no hardcoded values), named layers,
Figma Variables defined and linked.

**Page 3 — Mood Frame (light theme)**
1440×900 artboard — the approved concept moodframe, rebuilt using the now-complete
token system and proper reusable components. This replaces the inline-styled
concept moodframe. Use realistic product copy from the PRD.

**Page 4 — Core Components (dark theme)**
Pages 2 components with dark theme variable mode applied.
Dark mood frame: Page 3 with dark theme applied.
Verify each component independently — do not assume light mappings translate.

**Page 5 — Navigation**
- Top navigation bar (desktop): logo area, nav items (default, active, hover), CTA button
- Mobile navigation: bottom tab bar (4–5 items, active state) + hamburger/sheet pattern
- Sidebar variant if UX spec indicates sidebar navigation

**Page 6 — Form Elements**
- Text input (all states: default, focused, filled, error, disabled)
- Textarea
- Select / dropdown trigger
- Checkbox (default, checked, indeterminate, disabled)
- Radio button group
- Toggle / switch
- Form section: label + inputs + helper text + error message (assembled as realistic fragment)

**Page 7 — Feedback & Status**
- Alert / banner (success, warning, error, info)
- Toast notification (success, error, neutral)
- Badge (semantic colors + neutral)
- Empty state (icon + heading + body + CTA)
- Loading: spinner, skeleton (card skeleton, list item skeleton)
- Progress bar

**Page 8 — Overlay**
- Modal (header, body, footer with actions; small + large)
- Sheet / drawer (mobile bottom sheet)
- Tooltip
- Popover / dropdown menu (items, divider, destructive item)

**Page 9 — Data Display**
- Table (header row, data rows, hover, selected row)
- List item (with icon, title, secondary text, trailing action)
- Stats / metric card (number, label, trend indicator)
- Tag / pill list

**Page 10 — Mobile Variants**
Key components at 375px:
- Navigation (bottom tab bar)
- Card (full-width)
- Form fragment (stacked, full-width inputs)
- Modal → bottom sheet adaptation
- Verify all interactive elements ≥ 44×44px touch targets

**Page 11 — Dark Theme Gallery**
Pages 5–10 with dark theme variable mode applied.
Review each page for readability failures. Correct token definitions if needed.

**Component conventions (apply to all pages):**
- Semantic tokens only — zero hardcoded values
- Named layers: `ComponentName/Variant/State`
- Figma Variables linked throughout
- Auto-layout on all components

---

### Step 4: Write docs/design/design-system.md with frontmatter

Populate the template at `${CLAUDE_SKILL_DIR}/templates/design-system.md`.
**Prepend the YAML frontmatter schema declared in `## Outputs`** (copy
`sources` and `user_brief` verbatim from concept-brief.md frontmatter; append
any `$ARGUMENTS` text to `user_brief`; copy `figma.wireframes_file_key` and
`figma.concept_file_key`; add `figma.ds_file_key`).

The document body must include:
- Personality brief and archetype rationale (from concept brief)
- Token reference: all semantic tokens with light + dark values and purpose
- Typography scale: all 8 steps with full specs
- Spacing scale
- Radius and shadow system with rationale
- Component inventory: one row per component with Figma link + usage note
- Do / Don't guidelines for the 5 most consequential decisions (color, type, radius, spacing, dark mode)

Then run the Self-Review Checklist + Iteration Loop below.

---

## Self-Review Checklist

Before declaring done, verify each item. Mark ✅/❌ inline.

**Structural:**
- [ ] `docs/design/design-system.md` exists with valid frontmatter
- [ ] `frontmatter.sources` and `frontmatter.user_brief` copied verbatim from concept-brief (with $ARGUMENTS appended)
- [ ] `frontmatter.figma.wireframes_file_key` and `frontmatter.figma.concept_file_key` carried forward
- [ ] `frontmatter.figma.ds_file_key` set
- [ ] Output structure matches the template
- [ ] A developer could implement the system from design-system.md without opening Figma

**UX best practices:**
- [ ] All interactive components meet 44px minimum touch target
- [ ] Body text ≥ 16px; no label below 12px
- [ ] Focus state defined and visible on every interactive component (input, button, link)
- [ ] Empty / loading / error tokens or component variants exist for data displays
- [ ] Mobile variants exist for navigation, card, form, modal-as-bottom-sheet
- [ ] Dark theme: `surface-page` is not pure black (#000 / #111)
- [ ] Component inventory covers every component implied by feature user stories + acceptance criteria and the wireframe inventory — no gaps

**UI best practices:**
- [ ] Every token has documented rationale (inherited from concept brief — not "looks good")
- [ ] Light and dark themes use independently mapped semantic tokens — not inversions
- [ ] Primary color passes WCAG AA contrast against both theme backgrounds
- [ ] All semantic-status tokens (success/warning/error/info) have subtle and strong variants
- [ ] Spacing tokens use the 4 px base grid consistently
- [ ] Type pairing matches the approved concept; no ad-hoc fallback fonts in components
- [ ] Components reference semantic tokens only — zero hardcoded color values
- [ ] Named layers everywhere (`ComponentName/Variant/State`); auto-layout on every component

**Visual screenshot review:**
- [ ] Render the Token Reference page, the Core Components pages (light + dark),
      the rebuilt Mood Frame, and one Mobile Variant page via
      `mcp__plugin_figma_figma__get_screenshot`
- [ ] **Look at the rendered images.** Confirm:
      - Tokens look consistent on the canvas (no mismatched neutrals/primaries)
      - Components feel like one family (consistent radius, shadow, type)
      - Dark theme is genuinely dark-mode (not just a tinted light)
      - Rebuilt mood frame is visually consistent with the approved concept moodframe
      - Mobile components fit within 375 px without truncation

**$ARGUMENTS handling:**
- [ ] Every directive applied OR noted under User Input Notes
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
    APPEND a "## ⚠ Outstanding Issues" section to docs/design/design-system.md
      listing each unresolved checklist item with a one-line reason
    PRINT to console (verbatim):
      ⚠ /forge-design:ds completed with N unresolved issues — see docs/design/design-system.md
    exit
```

---

## Quality Bar (pre-condition gate)

Do not begin the iteration loop until all of these pass.

- [ ] concept-brief.md exists; frontmatter parsed; concept_file_key resolved
- [ ] PRD and feature files at the inherited paths read
- [ ] `$ARGUMENTS` parsed
- [ ] Full token set expanded in working memory (primitives + semantic + typography + spacing)
- [ ] All required Figma pages built (Token Reference, Core Components, Mood Frame, Navigation, Forms, Feedback, Overlay, Data Display, Mobile, Dark Gallery — minus pages skipped per `theme-scope`)

---

## Chain

After the iteration loop exits successfully, tell the user:
> "Design system built and documented in `docs/design/design-system.md`.
> Frontmatter cascaded from concept-brief with ds Figma file key added.
>
> Next: run `/forge-design:ui` to compose every screen from the Design
> System components (no arguments needed — inputs are inherited)."

If the iteration loop exited with outstanding issues, additionally print:
> "⚠ N issues remain unresolved — see `## ⚠ Outstanding Issues` in
> `docs/design/design-system.md`. Resolve before proceeding, or accept and continue."