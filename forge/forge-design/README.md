# forge-design

Design skills for AI Startup Studio. Standalone Claude Code plugin — invoke as `/forge-design:<skill>`.

The entry-point skill (`ux`) accepts arbitrary input file references and a freeform brief; downstream skills inherit those references via frontmatter on the entry-point's output. So you can use these skills against any project — the only contract is the input format.

## Plugin structure

```
forge-design/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   ├── ux/
│   │   ├── SKILL.md
│   │   └── templates/
│   │       └── ux-spec.md
│   ├── layout/
│   │   ├── SKILL.md
│   │   └── templates/
│   │       └── layout-spec.md
│   ├── wireframe/
│   │   ├── SKILL.md
│   │   └── templates/
│   │       └── wireframe-inventory.md
│   ├── concept/
│   │   ├── SKILL.md
│   │   └── templates/
│   │       └── concept-brief.md
│   ├── ds/
│   │   ├── SKILL.md
│   │   └── templates/
│   │       └── design-system.md
│   └── ui/
│       ├── SKILL.md
│       └── templates/
│           └── screen-inventory.md
└── README.md
```

## Setup

Requires the [Figma MCP](https://mcp.figma.com) for all skills except `ux`. Authenticate with `/mcp`.

## Usage

The entry-point skill is `ux`. Pass file references with `@` and any freeform brief as plain arguments:

```
/forge-design:ux @path/to/prd.{md|pdf} @path/to/features "freeform brief"
/forge-design:ux "freeform brief only — no PRD yet"
```

`features` may be a folder, a single file, or multiple files. Supported formats: `.md`, `.pdf` (convert `.docx`/`.xlsx` to PDF first).

Downstream skills inherit the PRD path, features path, and freeform brief from the upstream artifact's frontmatter — no need to re-pass them:

```
/forge-design:layout
/forge-design:wireframe
/forge-design:concept
/forge-design:ds
/forge-design:ui
```

Any downstream skill also accepts its own freeform `$ARGUMENTS` for per-step overrides or additions.

## Skill contract

Every skill in this plugin:

- Accepts optional freeform user input as `$ARGUMENTS`. Stops and asks if it cannot identify required inputs.
- Writes its output artifact with frontmatter (`sources`, `user_brief`, `figma`) so the next skill in the chain can pick up where it left off.
- Runs a self-review checklist at the end of its work covering UX best practices, UI best practices, and a real visual screenshot review (for Figma-producing skills — not just node-tree inspection).
- Iterates up to 2 times to fix flagged issues. If issues remain, the output document includes a `## ⚠ Outstanding Issues` section and a console warning is printed.

## Contributing

Conventional Commits — `feat`, `fix`, `docs`, `chore`, `refactor`. Scope is the skill name where applicable.
