# claude-plugins

Plugin marketplace for [Claude Code](https://www.anthropic.com/claude-code), by Rinat Advaer.

## Add this marketplace

```
/plugin marketplace add advaer/claude-plugins
```

## Install a plugin

```
/plugin install forge-design@advaer-plugins
```

## Plugins

### `forge` family — AI Startup Studio

| Name | Description |
|------|-------------|
| [`forge-product`](forge/forge-product) | AI Startup Studio — Product Management plugin (PRD + RICE-scored backlog with vertical slices). |
| [`forge-design`](forge/forge-design) | AI Startup Studio — design skills (`ux` → `layout` → `wireframe` → `concept` → `ds` → `ui`). Depends on the official [`figma`](https://claude.com/plugins) plugin (auto-installed). Requires `/mcp` authentication for the Figma MCP server. |

See [`forge/README.md`](forge/README.md) for the family-wide output-path convention and skill contract.

### Standalone

| Name | Description |
|------|-------------|
| [`docs-hq`](docs-hq) | Off-thread documentation operations. Dispatches docs lookup (the `explore` skill) to a Sonnet subagent — Context7 MCP first, web + Playwright fallback. Depends on the official `context7` and `playwright` plugins (auto-installed). |

## Updating

```
/plugin marketplace update advaer-plugins
```

## Repository layout

```
.claude-plugin/
└── marketplace.json        # marketplace manifest

forge/                      # AI Startup Studio plugin family
├── README.md               # family-wide convention + skill contract
├── forge-product/
│   ├── .claude-plugin/plugin.json
│   ├── README.md
│   └── skills/{prd,backlog}/
└── forge-design/
    ├── .claude-plugin/plugin.json
    ├── README.md
    └── skills/{ux,layout,wireframe,concept,ds,ui}/

docs-hq/                    # standalone — off-thread docs operations
├── .claude-plugin/plugin.json
├── README.md
├── agents/docs-explorer.md
└── skills/explore/

CLAUDE.md                   # guidance for Claude Code working in this repo
LICENSE                     # MIT
```

To add a new plugin, either drop it under an existing family directory (e.g. `forge/forge-architect/`) or place it at the repo root if it's standalone (e.g. `docs-hq/`). Give it a `.claude-plugin/plugin.json`, and append an entry to `plugins[]` in `.claude-plugin/marketplace.json` with a `source` pointing at the new path. For forge-family additions see [`forge/README.md`](forge/README.md) for the family-wide checklist.

## Versioning

Plugin versions live in each plugin's `plugin.json`. Releases are tagged in the form `<plugin-name>--v<version>` (e.g. `forge-design--v0.1.0`) so consumers can pin against semver ranges via the `dependencies` field in their own `plugin.json`.

## License

[MIT](LICENSE)
