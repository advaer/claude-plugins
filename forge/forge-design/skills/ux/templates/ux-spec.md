# UX Specification
_Product: [working title]_
_Date: [YYYY-MM-DD]_
_Source: `docs/product/prd.md` + feature descriptions_
_User input applied: [Yes — see User Input Notes / No]_
_Platform scope: [Web / Mobile / Both — note MVP vs deferred]_

---

## Navigation Structure

**Pattern:** [e.g. Top navigation bar (web) + Bottom tab bar (mobile)]

| Destination | Level | Features Housed | Visible To |
|-------------|-------|-----------------|------------|
| [name] | Primary | [feature names] | [All / Authenticated / Role] |
| [name] | Primary | [feature names] | [All / Authenticated / Role] |
| [name] | Secondary | [feature names] | [All / Authenticated / Role] |

**Notes:**
- [Any context-sensitive navigation rules, e.g. pre-login vs post-login nav]
- [Role-based nav differences, if any]

---

## User Journey Maps

### Journey: [Persona 1 Name]

**Entry point:** [How this persona first encounters the product]

| Step | What they see | What they do | Outcome |
|------|---------------|--------------|---------|
| 1 | [screen / state] | [action] | [next state] |
| 2 | | | |
| … | | | |

**Activation moment:** [The step at which core value is first experienced]
**Recurring loop:** [What brings them back after the first session]
**Upgrade trigger:** [Where and why they convert to paid]

---

### Journey: [Persona 2 Name] _(if applicable)_

| Step | What they see | What they do | Outcome |
|------|---------------|--------------|---------|
| 1 | | | |

---

## Cross-Feature Flows

### Onboarding / First-Run

1. [Step — what the user sees and does]
2. [Step]
3. [Branch: if X → go to step N / if Y → go to step M]
4. [Step]
> **Activation endpoint:** [The screen or state that marks successful onboarding]

---

### Authentication

**Signup:**
1. [Step]
2. [Step]

**Login:**
1. [Step]
2. [Step]

**Password reset:**
1. [Step]
2. [Step]

**Social auth:** [Included / Deferred — reason]

---

### Payment and Upgrade Flow

> MoR platform: [Paddle / LemonSqueezy / TBD]
> No direct card data handled by the application.

1. [Trigger: what prompts the upgrade prompt]
2. [Step — e.g. upgrade modal or dedicated page]
3. [Redirect to MoR checkout]
4. [Return URL handling — success / failure]
5. [Post-payment state: what changes in the UI]

---

### Error Recovery

**Network / system errors:**
- [Pattern — e.g. toast notification with retry action]

**Validation errors:**
- [Pattern — e.g. inline, on blur, field-level]

**Destructive actions:**
- [Pattern — e.g. inline confirmation, not modal]

**Session expiry:**
- [Pattern — e.g. redirect to login with return URL preserved]

---

### [Additional Cross-Feature Flow — e.g. Invitation Flow]

1. [Step]
2. [Step]

---

## Global UX Patterns

### Empty States
- **First-use empty state:** [What the user sees when a list/view has no data on first visit — include a call to action]
- **Post-action empty state:** [e.g. after deleting all items]
- **Search / filter with no results:** [Pattern]

### Loading States
- [Pattern — e.g. skeleton screens for lists, spinner for async actions, optimistic UI where appropriate]

### Error Handling
- **Validation errors:** [inline / on-blur / on-submit]
- **System errors:** [toast / banner / dedicated error screen]
- **Destructive action confirmation:** [inline / dialog — specify which pattern and when]

### Success Feedback
- [Pattern — e.g. toast for async completions, inline state change for immediate actions]

### Forms
- **Field grouping:** [e.g. logical sections with dividers]
- **Required fields:** [e.g. marked with asterisk, or mark optional instead]
- **Validation timing:** [on blur / on submit / hybrid]
- **Submission state:** [disabled button + spinner while in flight]

### Responsive Behaviour
- **Breakpoints:** [e.g. mobile <768px, tablet 768–1024px, desktop >1024px]
- **Key layout changes at mobile:** [e.g. sidebar collapses to bottom tabs, tables become cards]
- **Deferred at mobile MVP:** [any interactions or screens not optimised for mobile yet]

---

## Screen Inventory

### Global Screens

| Screen Name | Platform | Purpose | Entry Points | Key Actions |
|-------------|----------|---------|--------------|-------------|
| Auth — Login | Web + Mobile | Authenticate returning user | Unauthenticated redirect, nav link | Submit credentials, go to signup, reset password |
| Auth — Signup | Web + Mobile | Register new account | CTA, login page link | Submit registration, go to login |
| Auth — Password Reset | Web + Mobile | Reset forgotten password | Login page link | Submit email, submit new password |
| Error — 404 | Web | Unknown route | Any bad URL | Go home |
| Error — System | Web + Mobile | Unrecoverable error | System error | Retry, go home |

### [Feature: NNN — Feature Name] Screens

| Screen Name | Platform | Purpose | Entry Points | Key Actions |
|-------------|----------|---------|--------------|-------------|
| [Feature] — [View] | [Web / Mobile / Both] | [one sentence] | [list] | [list] |
| [Feature] — [View] | | | | |

### [Feature: NNN — Feature Name] Screens

| Screen Name | Platform | Purpose | Entry Points | Key Actions |
|-------------|----------|---------|--------------|-------------|
| | | | | |

---

## Post-MVP Screens _(reference only — not designed at MVP)_

| Screen Name | Feature | Reason Deferred |
|-------------|---------|-----------------|
| [name] | [feature] | [e.g. low RICE score, post-MVP feature, user input] |

---

## Open Questions

- [UX question carried forward from PRD or surfaced during flow design]
- [question]

---

## User Input Notes

**Directives received:**
| Directive | Classification | Applied to |
|-----------|----------------|------------|
| "[directive text]" | [type] | [UX spec section or feature file] |

**Conflicts or resolutions:**
- [conflict description and how it was resolved, if any]