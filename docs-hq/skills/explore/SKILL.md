---
name: explore
description: Off-thread documentation exploration. Use when the user asks a question *about* a library, framework, SDK, API, or CLI tool — including well-known ones (React, Vue, Next.js, Prisma, Tailwind, Django, Express, Spring, Supabase, LangGraph, etc.). Covers setup/configuration, version migration, library-specific debugging, API reference, and code generation that involves an external library. Dispatches the work to the `docs-explorer` subagent (Sonnet, fresh context, Context7 → web → Playwright fallback) so docs content does not pollute the main thread. Skip when the user only mentions a library in passing without asking about it, and for refactoring, business-logic debugging, or general programming concepts.
disable-model-invocation: false
---

Documentation requests should be handled by the `docs-explorer` subagent. Do not answer from training data inline. Do not paraphrase Context7/web content into the main thread yourself — that defeats the purpose of this skill (saving main-thread context and using a cheaper model).

## Input

If `$ARGUMENTS` is non-empty, treat it as the research question — verbatim, no rephrasing. Otherwise use the user's most recent message as the question. Edge case: if `$ARGUMENTS` is just a short clarifier (one phrase, not a full sentence) and the conversation already has a clear docs question, append `$ARGUMENTS` to that question.

Examples:
- `/docs-hq:explore Does LangGraph have a Java SDK?` → research question is the full string after the verb.
- User message "How do I configure Next.js middleware?" with no slash invocation → research question is that message.

## What to do — every time

1. **Invoke the `Agent` tool with `subagent_type: docs-explorer`.** Pass the
   question verbatim (from `$ARGUMENTS` or conversation), plus any version
   hints the user mentioned ("React 19", "Next.js 15.2"). For multiple
   libraries in one question, list them all in a single subagent call so the
   agent can parallelize internally — do not spawn one subagent per library.

2. **Wait for the subagent to return.** It will reply with a structured
   per-library block (source, key info, code examples).

3. **Summarize for the user — do not paste the raw subagent output.** Pull
   out the answer to the user's specific question, keep one or two short code
   snippets if they're load-bearing, and cite the library version + source
   (Context7 vs. URL) when relevant. The raw block is for your reference;
   the user wants the answer.

## Trigger conditions (auto-invocation)

Use this skill when the user asks about:
- A specific library or framework (React, Vue, Next.js, Prisma, Supabase, Tailwind, Django, Spring, Express, LangGraph, …) — even ones you "know".
- Setup or configuration ("How do I configure X").
- Version migration ("React 18 → 19", "Next.js app router migration").
- Library-specific debugging ("Why does Prisma throw …").
- API reference ("What auth methods does Supabase expose").
- Code generation that uses an external library.

## When NOT to use

- Refactoring existing code in the project.
- Debugging the user's own business logic.
- General programming concepts (algorithms, design patterns, language semantics).
- Anything answerable purely from the project's own files.
