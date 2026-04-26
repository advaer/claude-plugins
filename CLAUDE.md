# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A **Claude Code plugin marketplace** named `advaer-plugins`. It is *not* a single plugin — it's a manifest at `.claude-plugin/marketplace.json` that lists plugins living in subdirectories. End users add it via:

```
/plugin marketplace add advaer/claude-plugins
/plugin install <plugin-name>@advaer-plugins
```

The GitHub repo is named `claude-plugins`; the marketplace **identifier** is `advaer-plugins`. Both are intentional — the validator (`claude plugin validate`) rejects any marketplace name resembling Anthropic's official ones (anything starting `claude-*`), so the identifier had to be distinctive even though the repo name doesn't.

## Common commands

```bash
# Validate the marketplace manifest before committing changes to it
claude plugin validate .

# Validate an individual plugin's plugin.json
claude plugin validate forge/<plugin-name>

# Test-install locally without pushing
claude plugin marketplace add /Users/advaer/dev/pet/claude-plugins
claude plugin install <plugin-name>@advaer-plugins

# Cut a release for a plugin (after bumping version in plugin.json + committing)
claude plugin tag forge/<plugin-name> --dry-run -m "<plugin-name> %s"
claude plugin tag forge/<plugin-name> --push -m "<plugin-name> %s"
```

`claude plugin tag` validates that `plugin.json` and the marketplace entry agree on the version, then creates an annotated git tag in the form `<plugin-name>--v<version>` and pushes it. Skipping this step means semver-pinned consumers can't resolve dependencies on the plugin.

## Architecture

```
.claude-plugin/marketplace.json    # marketplace manifest — single source of truth
                                   # for which plugins this repo publishes
forge/                             # "AI Startup Studio" plugin family
└── <plugin-name>/
    ├── .claude-plugin/plugin.json # plugin manifest (name, version, deps)
    ├── README.md
    └── skills/<skill>/SKILL.md    # the actual capability units
```

**Family directory (`forge/`)** is purely organizational — Claude Code doesn't interpret it. The `source` field in each marketplace entry resolves the actual path (`./forge/<plugin-name>`). Adding a sibling family later (e.g. `data/`) is just a directory and another `plugins[]` entry.

**Output-path convention for `forge-*` plugins**: each plugin writes its outputs to `docs/<plugin-suffix>/` where suffix is whatever follows `forge-` (e.g. `forge-product` → `docs/product/`, `forge-design` → `docs/design/`). The single documented exception is `/forge-product:backlog`, which writes to `docs/backlog/NNN-feature-name/description.md` at the docs root because backlog items are first-class artifacts consumed by multiple downstream plugins. See [`forge/README.md`](forge/README.md).

**Plugin name vs. directory path** are decoupled. The slash-command namespace is the plugin's `name` field in `plugin.json` (e.g. `/forge-design:<skill>`), not the directory. Renaming the directory does not break invocations; renaming the plugin does.

### Cross-marketplace dependencies

`forge-design` depends on `figma@claude-plugins-official`. Two pieces wire this up and **both are required** — Claude Code blocks cross-marketplace deps by default:

1. `forge/forge-design/.claude-plugin/plugin.json` declares it via `dependencies: [{name: "figma", marketplace: "claude-plugins-official"}]`.
2. `.claude-plugin/marketplace.json` allow-lists the source via `allowCrossMarketplaceDependenciesOn: ["claude-plugins-official"]`.

Removing either silently breaks installs for new users. When adding a new plugin that depends on something outside this marketplace, both edits are needed.

**MCP servers are not auto-configured by plugin dependencies.** If a plugin needs an MCP server (e.g. Figma MCP), the user still authenticates via `/mcp` themselves — depend on the MCP-providing plugin and document the `/mcp` step in the plugin's README.

### Versioning

- Stay on `0.x.x` while iterating; bump minor on contract changes (skill input/output frontmatter, slash-command names), patch on internal edits. Internal prompt rewrites that don't change inputs/outputs are not breaking.
- `version` lives in `plugin.json` only, never in the marketplace entry — the validator allows duplication but `plugin.json` silently wins, so duplication just rots.
- Release flow: edit `plugin.json` version → commit → push `main` → `claude plugin tag <plugin-path> --push`.

## Plugin authoring conventions

These are shared across `forge-*` plugins and apply to any new plugin in the family unless the plugin's own README overrides:

- **Skill contract**: each skill accepts a freeform `$ARGUMENTS`; if it can't identify required inputs, it stops and asks. Pipeline skills write output artifacts with frontmatter (`sources`, `user_brief`, plus domain-specific keys like `figma`) so the next skill can pick up upstream context without re-passing arguments.
- **Self-review with iteration cap**: every skill runs a self-review checklist at end-of-work and iterates up to 2 times to fix flagged issues. Remaining issues land in a `## ⚠ Outstanding Issues` section in the output document plus a console warning — they are *not* silently dropped.
- **Frontmatter `disable-model-invocation: true`** on entry-point skills prevents the model from auto-invoking them; they're meant to be triggered explicitly via slash command.

## Commit conventions

Conventional Commits — `feat`, `fix`, `docs`, `chore`, `refactor`. **Scope is the skill name** for plugin-internal changes (e.g. `feat(ux): ...`), the plugin name for plugin-wide changes (`chore(forge-design): ...`), and omitted for marketplace-level changes.
