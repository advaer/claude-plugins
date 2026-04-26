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

| Name | Description |
|------|-------------|
| [`forge-product`](forge/forge-product) | AI Startup Studio — Product Management plugin (PRD + RICE-scored backlog with vertical slices). |
| [`forge-design`](forge/forge-design) | AI Startup Studio — design skills (`ux` → `layout` → `wireframe` → `concept` → `ds` → `ui`). Depends on the official [`figma`](https://claude.com/plugins) plugin (auto-installed). Requires `/mcp` authentication for the Figma MCP server. |

Both plugins belong to the **[`forge`](forge)** family — see [`forge/README.md`](forge/README.md) for the family-wide output-path convention and skill contract.

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

CLAUDE.md                   # guidance for Claude Code working in this repo
LICENSE                     # MIT
```

To add a new plugin, drop it under a family directory (e.g. `forge/forge-architect/`), give it a `.claude-plugin/plugin.json`, and append an entry to `plugins[]` in `.claude-plugin/marketplace.json` with a `source` pointing at the new path. See [`forge/README.md`](forge/README.md) for the family-wide checklist.

## Versioning

Plugin versions live in each plugin's `plugin.json`. Releases are tagged in the form `<plugin-name>--v<version>` (e.g. `forge-design--v0.1.0`) so consumers can pin against semver ranges via the `dependencies` field in their own `plugin.json`.

## License

[MIT](LICENSE)
