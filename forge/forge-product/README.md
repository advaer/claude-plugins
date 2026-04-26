# forge-product

Product Management plugin for AI Startup Studio. Standalone Claude Code plugin — invoke as `/forge-product:<skill>`.

The entry-point skill (`prd`) accepts arbitrary input file references and a freeform brief; the downstream skill (`backlog`) inherits those references via frontmatter on the PRD output. So you can use this plugin against any project — the only contract is the input format.

## Plugin structure

```
forge-product/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   ├── prd/
│   │   ├── SKILL.md
│   │   └── templates/
│   │       └── prd.md
│   └── backlog/
│       ├── SKILL.md
│       └── templates/
│           └── feature-description.md
└── README.md
```

## Usage

The entry-point skill is `prd`. Pass the validated-idea reference with `@` and any freeform brief as plain arguments:

```
/forge-product:prd @path/to/idea-validation.{md|pdf} "freeform brief"
/forge-product:prd "a Notion-like CRM for indie consultants — focus on freelancers, not agencies"
```

`idea-validation` may be a `.md` or `.pdf` file. (Convert `.docx`/`.xlsx` to PDF first.) If you don't pass an `@`-path, `prd` falls back to `docs/idea-validator/idea-validation.md` if it exists.

The downstream skill inherits the PRD path and freeform brief from `docs/product/prd.md` frontmatter — no need to re-pass:

```
/forge-product:backlog                                   # inherits everything from prd.md
/forge-product:backlog "regenerate 003-notifications"    # optional per-run steering
```

## Outputs

| Skill | Writes |
|---|---|
| `prd` | `docs/product/prd.md` (with YAML frontmatter: `sources.idea_validation`, `user_brief`) |
| `backlog` | `docs/backlog/NNN-feature-name/description.md` (one per feature; same frontmatter cascaded) |

## Skill contract

Every skill in this plugin:

- Accepts optional freeform user input as `$ARGUMENTS`. Stops and asks if it cannot identify required inputs.
- Writes its output artifact with frontmatter (`sources`, `user_brief`) so the next skill in the chain — or any sibling plugin like `forge-design` — can pick up where it left off.
- Runs a self-review checklist at the end of its work covering structural completeness and product/PM best practices.
- Iterates up to 2 times to fix flagged issues. If issues remain, the output document includes a `## ⚠ Outstanding Issues` section and a console warning is printed.

## Contributing

Conventional Commits — `feat`, `fix`, `docs`, `chore`, `refactor`. Scope is the skill name where applicable.
