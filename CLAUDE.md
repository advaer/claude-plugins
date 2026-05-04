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

`<plugin-path>` below is the relative path to the plugin from the repo root — `forge/forge-design` for forge-family plugins, `docs-hq` for standalone plugins.

```bash
# Validate the marketplace manifest before committing changes to it
claude plugin validate .

# Validate an individual plugin's plugin.json
claude plugin validate <plugin-path>

# Test-install locally without pushing
claude plugin marketplace add /Users/advaer/dev/pet/claude-plugins
claude plugin install <plugin-name>@advaer-plugins

# Cut a release for a plugin (after bumping version in plugin.json + committing)
claude plugin tag <plugin-path> --dry-run -m "<plugin-name> %s"
claude plugin tag <plugin-path> --push -m "<plugin-name> %s"
```

`claude plugin tag` validates that `plugin.json` and the marketplace entry agree on the version, then creates an annotated git tag in the form `<plugin-name>--v<version>` and pushes it. Skipping this step means semver-pinned consumers can't resolve dependencies on the plugin.

## Architecture

```
.claude-plugin/marketplace.json    # marketplace manifest — single source of truth
                                   # for which plugins this repo publishes

forge/                             # "AI Startup Studio" plugin family
└── <plugin-name>/
    ├── .claude-plugin/plugin.json
    ├── README.md
    └── skills/<skill>/SKILL.md

docs-hq/                           # standalone plugin (no family)
├── .claude-plugin/plugin.json
├── README.md
├── agents/<agent>.md              # subagents shipped by the plugin
└── skills/<skill>/SKILL.md
```

**Family directory (`forge/`)** is purely organizational — Claude Code doesn't interpret it. The `source` field in each marketplace entry resolves the actual path (`./forge/<plugin-name>` for forge-family, `./<plugin-name>` for standalone). Adding a sibling family later (e.g. `data/`) is just a directory and another `plugins[]` entry; standalone plugins live at the repo root.

**Output-path convention for `forge-*` plugins**: each plugin writes its outputs to `docs/<plugin-suffix>/` where suffix is whatever follows `forge-` (e.g. `forge-product` → `docs/product/`, `forge-design` → `docs/design/`). The single documented exception is `/forge-product:backlog`, which writes to `docs/backlog/NNN-feature-name/description.md` at the docs root because backlog items are first-class artifacts consumed by multiple downstream plugins. See [`forge/README.md`](forge/README.md). This convention is family-scoped — standalone plugins (e.g. `docs-hq`) define their own output behavior or have none (dispatch-only plugins write nothing).

**Plugin name vs. directory path** are decoupled. The slash-command namespace is the plugin's `name` field in `plugin.json` (e.g. `/forge-design:<skill>`, `/docs-hq:<skill>`), not the directory. Renaming the directory does not break invocations; renaming the plugin does.

**Subagents** live in `agents/<name>.md` at the plugin root. The agent's `name` frontmatter field is what other plugins/skills reference via `subagent_type:` — keep filename and `name` in sync. Agents have their own `tools:` allow-list and `model:` field; setting `model: sonnet` (or `haiku`) on a worker agent isolates context and saves tokens versus running on the parent's Opus.

### Cross-marketplace dependencies

Both current cases pull from `claude-plugins-official`:
- `forge-design` → `figma@claude-plugins-official`
- `docs-hq` → `context7@claude-plugins-official` + `playwright@claude-plugins-official`

Two pieces wire each up and **both are required** — Claude Code blocks cross-marketplace deps by default:

1. The plugin's `plugin.json` declares it via `dependencies: [{name: "<dep>", marketplace: "claude-plugins-official"}]`.
2. `.claude-plugin/marketplace.json` allow-lists the source via `allowCrossMarketplaceDependenciesOn: ["claude-plugins-official"]`.

Removing either silently breaks installs for new users. The allow-list is shared across all plugins in this marketplace — adding a third dep on the same source needs no further marketplace edit; adding a dep on a *new* source does.

**Depending on a plugin that ships an MCP server auto-installs that server**, but the user still runs `/mcp` to connect (and authenticate if the server requires it). Auth varies by MCP:
- Figma MCP → OAuth, user authenticates via `/mcp`.
- Context7 MCP → no auth required for the free tier; users wanting higher rate limits set `CONTEXT7_API_KEY`.
- Playwright MCP → no auth, just spawns a local browser via `npx`.

Always document the `/mcp` step (and any optional API keys) in the plugin's README.

### Versioning

- Stay on `0.x.x` while iterating; bump minor on contract changes (skill input/output frontmatter, slash-command names), patch on internal edits. Internal prompt rewrites that don't change inputs/outputs are not breaking.
- `version` lives in `plugin.json` only, never in the marketplace entry — the validator allows duplication but `plugin.json` silently wins, so duplication just rots.
- Release flow: edit `plugin.json` version → commit → push `main` → `claude plugin tag <plugin-path> --push`.

## Plugin authoring conventions

The marketplace currently has two plugin patterns. Apply the conventions for whichever pattern a plugin fits.

### Common to all plugins

- **Skill contract**: each skill accepts a freeform `$ARGUMENTS`; if it can't identify required inputs, it stops and asks. Pipeline skills (see below) write output artifacts with frontmatter so the next skill can pick up upstream context without re-passing arguments.
- **README required**: each plugin has a `README.md` covering setup (including `/mcp` if it depends on an MCP server), usage, and any optional configuration.
- **Tool allow-lists** on agents: keep the `tools:` list minimal — only what the agent actually uses. Don't include `Skill` on worker agents (they shouldn't load skills into their own context).

### Pipeline plugins (forge-*)

A multi-skill pipeline where each skill produces an artifact consumed by the next.

- **Self-review with iteration cap**: every skill runs a self-review checklist at end-of-work and iterates up to 2 times to fix flagged issues. Remaining issues land in a `## ⚠ Outstanding Issues` section in the output document plus a console warning — they are *not* silently dropped.
- **Frontmatter `disable-model-invocation: true`** on entry-point skills prevents the model from auto-invoking them; they're meant to be triggered explicitly via slash command.
- Outputs follow the `docs/<plugin-suffix>/` path convention (see Architecture).

### Dispatch plugins (docs-hq)

A plugin whose skill exists primarily to *dispatch* work to a subagent rather than do the work in the main thread.

- **Frontmatter `disable-model-invocation: false`** on the skill — auto-trigger by description matching is the whole point. Skill descriptions should be specific enough to avoid over-firing on tangential mentions (see `docs-hq:explore`'s "skip when mentioned in passing" clause).
- **Skill body is thin**: it tells the parent thread to invoke the worker agent via the `Agent` tool with the right `subagent_type`, then summarize the agent's output rather than paste it raw. The skill does not do the work itself.
- **Worker agent runs on a cheaper model** (`model: sonnet` or `model: haiku`) in its own context — that's the cost+context-isolation lever this pattern exists for.
- Don't write project artifacts; the dispatch pattern returns answers to the user, not files.

## Commit conventions

Conventional Commits — `feat`, `fix`, `docs`, `chore`, `refactor`. **Scope is the skill name** for plugin-internal changes (e.g. `feat(ux): ...`, `feat(explore): ...`), the plugin name for plugin-wide changes (`chore(forge-design): ...`, `chore(docs-hq): ...`), and omitted for marketplace-level changes.
