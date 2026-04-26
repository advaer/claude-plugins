# Layout Specification

> **Figma file:** <!-- link -->
> **Figma file key:** <!-- file key for downstream skills -->
> **Approved:** <!-- date -->

---

## Layout Variants

### Variant 1: <!-- e.g. Authenticated Default -->

**Used by:** <!-- list of screen names from UX spec -->

#### Desktop (1440×900)

| Zone | Position | Size | Contents |
|------|----------|------|----------|
| Navigation | <!-- e.g. Left, persistent --> | <!-- e.g. 240px wide, full height --> | <!-- e.g. Logo, nav items, user menu at bottom --> |
| Header | <!-- e.g. None --> | — | — |
| User menu | <!-- e.g. Bottom-left, inside sidebar --> | <!-- e.g. 240×56px --> | <!-- e.g. Avatar, name, dropdown trigger --> |
| Content area | <!-- e.g. Right of sidebar --> | <!-- e.g. flex, max-width 1200px --> | <!-- e.g. Page title + main content --> |

#### Mobile (375×812)

| Zone | Position | Size | Contents |
|------|----------|------|----------|
| Navigation | <!-- e.g. Bottom tab bar --> | <!-- e.g. 375×56px --> | <!-- e.g. 4 tabs: Home, Sessions, Progress, Settings --> |
| Header | <!-- e.g. Top bar --> | <!-- e.g. 375×48px --> | <!-- e.g. Back arrow, page title, action icon --> |
| User menu | <!-- e.g. Settings tab --> | — | <!-- e.g. Profile section at top of settings screen --> |
| Content area | <!-- e.g. Below header, above tab bar --> | <!-- e.g. flex --> | <!-- e.g. Single column, full width --> |

---

### Variant 2: <!-- e.g. Auth / Guest -->

**Used by:** <!-- e.g. Login, Signup, Password Reset -->

#### Desktop (1440×900)

| Zone | Position | Size | Contents |
|------|----------|------|----------|
| <!-- zones --> | | | |

#### Mobile (375×812)

| Zone | Position | Size | Contents |
|------|----------|------|----------|
| <!-- zones --> | | | |

---

<!-- repeat for additional variants -->

---

## Screen-to-Variant Mapping

| Screen Name | Variant |
|-------------|---------|
| <!-- every screen from UX spec --> | <!-- variant name --> |

---

## User Input Notes

**Directives received:**

| Directive | Classification | Applied to |
|-----------|----------------|------------|
| "<!-- directive text -->" | <!-- type --> | <!-- variant / zone --> |

**Conflicts or resolutions:**
- <!-- conflict description and resolution -->