# docs-hq

Headquarters for documentation operations. Off-thread, in a fresh **Sonnet** context — keeps the main thread focused and avoids burning the more expensive model on docs work.

The first skill ships now: `explore` (look up external library/framework docs). More skills (write, audit, generate `llms.txt`) are planned under the same plugin.

## Why

Doing docs lookup inline in the main thread:
- pollutes the parent context with raw docs content (often thousands of tokens),
- spends Opus tokens on a task Sonnet handles well.

`docs-hq` replaces inline docs answers with a subagent dispatch:
1. The `explore` skill (`/docs-hq:explore`, also auto-triggered) tells the main thread to invoke the `docs-explorer` agent rather than answer inline.
2. The agent runs in its own context on Sonnet, queries Context7 first, falls back to web search and (last resort) Playwright for JS-rendered docs.
3. The main thread receives a compact result and summarizes it for the user — the raw docs dump never lands in the parent context.

> Note: the `explore` skill ships with `disable-model-invocation: false` — i.e. the model auto-triggers it from conversation. This is intentional and a deliberate divergence from the `forge-*` plugins in this marketplace, which use `true` to keep their pipelines explicit. Auto-trigger is the whole point of `docs-hq:explore`.

## Setup

`docs-hq` depends on two plugins from `claude-plugins-official`, both of which ship only an MCP server (no extra skills):
- **context7** — Upstash Context7 MCP for LLM-optimized, version-specific docs.
- **playwright** — Microsoft Playwright MCP, used only as a deep fallback for JS-rendered docs.

Install:

```
/plugin marketplace add advaer/claude-plugins
/plugin install docs-hq@advaer-plugins
```

Claude Code will pull in the two dependencies automatically. After install, run `/mcp` to connect the two servers — Context7 works unauthenticated; Playwright spins up a browser on first use. Node is required (both MCPs run via `npx`).

### Overlap with a personal `context7-mcp` skill

If you have a standalone `context7-mcp` skill installed at `~/.claude/skills/context7-mcp/` (or anywhere else), **remove or disable it** — it has the same trigger surface as `docs-hq:explore` and will race with it. The whole point of `docs-hq:explore` is off-thread dispatch; the standalone skill runs Context7 inline in your main thread.

## Configuration

### Context7 API key (optional, recommended for heavy use)

Context7 has a free tier with rate limits. For higher limits, set a `CONTEXT7_API_KEY` env var so the `@upstash/context7-mcp` server picks it up at startup. You can either:
- export it in your shell profile before launching Claude Code, or
- add it to the `context7` server entry in your user-level `~/.claude.json` MCP config under `headers` / `env` (depending on transport).

The MCP itself is shipped by the official `context7@claude-plugins-official` plugin — `docs-hq` doesn't override its config.

## Usage

### Auto-trigger (no slash needed)

The `explore` skill auto-triggers when the user asks a question about a library, framework, SDK, API, or CLI tool:

> "How do I configure Next.js middleware?"
> "Does LangGraph have a Java SDK?"

It is *not* designed to fire when a library is mentioned in passing without a question (e.g. "we're on Prisma but the bug is in our auth code" → no trigger).

### Explicit invocation with a question

Pass the question directly after the slash command:

```
/docs-hq:explore Does LangGraph have a Java SDK?
/docs-hq:explore How do I configure Next.js middleware?
/docs-hq:explore Compare Prisma and Drizzle for Postgres
```

The full text after the slash command is taken as the research question and forwarded verbatim to the `docs-explorer` subagent. Multiple libraries in one question are looked up in parallel inside the subagent.

## Troubleshooting

- **`/mcp` shows context7 or playwright as disconnected** — confirm Node is on your PATH (`node --version`, `npx --version`). Both servers spawn via `npx`; without Node they fail silently at startup.
- **The agent says it found nothing** — Context7 may not have the library. The agent automatically falls back to web search and (if necessary) Playwright; if all three fail, you'll see an explicit "no docs available" answer instead of a hallucination. That's the intended behavior.
- **Context7 returns rate-limit errors** — set `CONTEXT7_API_KEY` (see Configuration).
- **Playwright launches a browser window** — expected on first use; subsequent runs reuse the same browser. Used only as a last-resort fallback.

## Plugin structure

```
docs-hq/
├── .claude-plugin/plugin.json
├── agents/docs-explorer.md    # the worker — runs on Sonnet, does the actual lookup
├── skills/explore/SKILL.md    # entry point — dispatches the main thread to the subagent
└── README.md
```

## Credits

The `docs-explorer` agent is based on work by **Maximilian Schwarzmüller** ([Academind](https://academind.com/)). Thanks!
