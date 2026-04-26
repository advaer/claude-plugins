# forge — AI Startup Studio

A family of Claude Code plugins for the founder-operator workflow: validate an idea, write a PRD, design the product, plan the build. Each plugin is independently usable; together they form a pipeline.

## Plugins in this family

| Plugin | Slash command | Purpose |
|--------|---------------|---------|
| [`forge-product`](forge-product) | `/forge-product:<skill>` | PRD + backlog. RICE-scored features, vertical slices. |
| [`forge-design`](forge-design) | `/forge-design:<skill>` | Design pipeline (ux → layout → wireframe → concept → ds → ui) backed by Figma. |

## Output convention

Every plugin in this family writes its outputs to **`docs/<plugin-suffix>/`**, where the suffix is whatever follows `forge-` in the plugin name:

| Plugin | Outputs land in |
|--------|-----------------|
| `forge-product` | `docs/product/` (e.g. `docs/product/prd.md`) |
| `forge-design` | `docs/design/` (e.g. `docs/design/ux-spec.md`, `design-system.md`, `screen-inventory.md`) |
| Future `forge-<area>` | `docs/<area>/` |

**Exception:** `/forge-product:backlog` writes to `docs/backlog/NNN-feature-name/description.md` — at the docs root, not under `docs/product/`. Backlog items are first-class artifacts that downstream plugins (`forge-design`, future `forge-architect`, etc.) consume independently of the PRD.

## Pipeline data flow

Each skill's output document opens with YAML frontmatter (`sources`, `user_brief`, plus domain-specific keys). Downstream skills read that frontmatter to inherit upstream context — so once you've run the entry-point skill of a plugin with explicit `@`-paths, every following skill in the chain runs argument-free.

```
forge-product:prd      → docs/product/prd.md
forge-product:backlog  → docs/backlog/NNN-*/description.md
        ↓ (frontmatter cascades)
forge-design:ux        → docs/design/ux-spec.md
forge-design:layout    → docs/design/layout-spec.md
forge-design:wireframe → docs/design/wireframe-inventory.md
forge-design:concept   → docs/design/concept-brief.md
forge-design:ds        → docs/design/design-system.md
forge-design:ui        → docs/design/screen-inventory.md
```

## Adding a new `forge-*` plugin

1. Create `forge/forge-<area>/` with `.claude-plugin/plugin.json`, a `README.md`, and `skills/<skill>/SKILL.md` per skill.
2. Default outputs to `docs/<area>/...`. If the plugin produces a stream of independent artifacts (like `backlog`), consider a top-level `docs/<artifact-type>/` instead and document the exception in this README.
3. Append a `plugins[]` entry to `../.claude-plugin/marketplace.json` with `source: "./forge/forge-<area>"`.
4. Validate via `claude plugin validate /path/to/repo` and `claude plugin validate forge/forge-<area>`.
5. Tag the first release: `claude plugin tag forge/forge-<area> --push -m "forge-<area> %s"`.

## Skill contract (inherited)

Every skill in this family:

- Accepts optional freeform `$ARGUMENTS`. Stops and asks if it cannot identify required inputs.
- Writes its output artifact with frontmatter so the next skill in the chain (within or across plugins) can pick up upstream context without re-passing arguments.
- Runs a self-review checklist at end-of-work, iterates up to 2 times to fix flagged issues, and surfaces remaining issues under a `## ⚠ Outstanding Issues` section plus a console warning.
