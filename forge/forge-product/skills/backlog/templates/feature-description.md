# Feature: [Feature Name]
_Folder: `docs/features/NNN-feature-name/`_
_Date: [YYYY-MM-DD]_
_Release: v1 / v2 / v3_
_Source: [idea-validation / user input via $ARGUMENTS]_

---

## Overview
[2–3 sentences: what this feature does, why it exists, which persona it serves]

## RICE Score
| Reach | Impact | Confidence | Effort | Score |
|-------|--------|------------|--------|-------|
| X/10  | X/10   | X/10       | X/10   | X.X   |

_Note: [any modifier applied — e.g. "−2 Reach applied per user deprioritisation input"]_

---

## User Stories

- As a [persona], I want to [action] so that [outcome].
- As a [persona], I want to [action] so that [outcome].

---

## Acceptance Criteria

- [ ] [specific, testable criterion]
- [ ] [specific, testable criterion]
- [ ] [specific, testable criterion]

---

## Edge Cases

- [edge case and expected behaviour]
- [edge case and expected behaviour]

---

## Open Questions

- [question — from risky assumption, user directive, or unresolved detail]

---

## Vertical Slices

Each slice is a thin end-to-end cut through all layers (DB → API → UI where needed)
that delivers something independently shippable and testable. Slice 1 is the
**walking skeleton** — the thinnest possible implementation that proves the feature
works end-to-end.

Think: thin end-to-end slice → richer slice → full capability.
Not: DB schema → API → UI (nothing runs until the end).

Each slice is labelled with the release version it ships in.

| # | Slice | Release | Delivers | Testable via |
|---|-------|---------|----------|--------------|
| 1 | Walking skeleton: [name] | v1 / v2 / v3 | [what a user/API caller can do after this slice] | API test / e2e |
| 2 | [name] | v2 / v3 | [what a user/API caller can do after this slice] | API test / e2e |
| 3 | [name] | v3 | [what a user/API caller can do after this slice] | API test / e2e |

_Add or remove rows as needed. Every slice must be independently shippable and testable._

---

## UX Notes