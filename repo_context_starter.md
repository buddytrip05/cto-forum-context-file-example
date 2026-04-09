# Repo-Scoped LLM Context for Claude Code

This starter explains a practical pattern: keep persistent project guidance in-repo so an LLM can discover structure and pull the right context without you pasting every markdown file into chat. Claude Code officially supports repo-level `CLAUDE.md`, nested `CLAUDE.md`, imports via `@path`, and path-scoped rules in `.claude/rules/`. [web:4][page:1]

## What the concept is

At the beginner level, the pattern is simple: add a concise `CLAUDE.md` at the repo root that tells Claude Code what the project is, where things live, how to build/test, and which docs matter most. Claude Code loads project `CLAUDE.md` files at session start, and it can also load more specific files from parent/child directories depending on where it is working. [page:1]

At the more advanced level, you organize context into smaller files and use structure rather than one giant prompt. Claude Code supports `@path/to/file` imports inside `CLAUDE.md`, directory-specific `CLAUDE.md`, and path-scoped rules in `.claude/rules/`, which means context can be layered and triggered when matching files are read. [page:1]

A related but distinct pattern exists in Aider: it builds a repository map containing files, key symbols, and signatures, then lets the model use that map to decide which files to inspect next. That is evidence that repo-navigation plus selective context loading is an established agent pattern, even though Claude Code implements it differently. [page:2]

## What is confirmed vs unknown

### Confirmed

- Claude Code reads `CLAUDE.md` files at the start of sessions as persistent project instructions. [web:15][page:1]
- Claude Code supports `CLAUDE.md` in the repo root, `.claude/CLAUDE.md`, user-level `~/.claude/CLAUDE.md`, and local `CLAUDE.local.md`. [page:1]
- Claude Code supports imports using `@path/to/file` syntax inside `CLAUDE.md`. [page:1]
- Claude Code supports path-specific rules with YAML frontmatter under `.claude/rules/`. [page:1]
- Subdirectory `CLAUDE.md` files can load on demand when Claude reads files in those directories. [page:1]
- Aider uses a repo map so the model can see the codebase shape and request more specific files when needed. [page:2]
- There are public examples of people organizing `.context/` or `CLAUDE.md` files to avoid repeating project conventions. [web:1][web:6][web:9]

### Unknown or not directly verified

- Whether Claude Code natively supports arbitrary custom metadata tags inside your own markdown files and automatically uses those tags as a first-class retrieval/routing system. Unknown from the sources reviewed. 
- Whether Claude Code automatically builds a symbolic repo map exactly like Aider’s internal repo map. Not confirmed by the sources reviewed. 
- Whether Claude Code automatically indexes all small markdown files in a custom context directory without referencing them from `CLAUDE.md`, nested files, rules, or direct file reads. Unknown from the sources reviewed.

## How to execute effectively

1. Keep the root `CLAUDE.md` short, ideally under 200 lines, because Anthropic recommends concise files for better adherence. [page:1]
2. Put only always-needed guidance in the root file: project map, commands, architecture boundaries, and high-signal conventions. [page:1]
3. Move specific guidance into imported docs or `.claude/rules/` so the system stays modular instead of bloated. [page:1]
4. Use directory-local `CLAUDE.md` files where a subsystem has unique rules, such as `frontend/CLAUDE.md` or `backend/CLAUDE.md`. [page:1]
5. Treat metadata as human-designed scaffolding unless your tool explicitly documents automatic metadata routing. The verified Claude Code primitives are file hierarchy, imports, and path-scoped rules. [page:1]

## Beginner version

Start with one repo file and one optional docs folder.

```text
my-repo/
├─ CLAUDE.md
├─ README.md
├─ docs/
│  ├─ architecture.md
│  ├─ project-map.md
│  └─ decisions.md
├─ src/
└─ tests/
```

Example root `CLAUDE.md`:

```md
# Project Instructions

## Purpose
- Internal product for fulfillment operations.

## Project map
- `src/` application code
- `tests/` automated tests
- `docs/architecture.md` system design
- `docs/project-map.md` directory guide

## Build and test
- Install: `pnpm install`
- Dev: `pnpm dev`
- Test: `pnpm test`

## Working norms
- Reuse existing patterns before adding abstractions.
- Keep functions small and typed.
- Update docs when changing architecture.

## Read first for large changes
- @docs/architecture.md
- @docs/project-map.md
```

This version is enough to let Claude Code start with a persistent project brief and pointers to the right documents. [page:1]

## Advanced version

Use layered context, scoped rules, and a directory map.

```text
my-repo/
├─ CLAUDE.md
├─ .claude/
│  ├─ CLAUDE.md
│  └─ rules/
│     ├─ testing.md
│     ├─ api.md
│     └─ ui.md
├─ docs/
│  ├─ context/
│  │  ├─ project-map.md
│  │  ├─ domain-glossary.md
│  │  ├─ architecture-overview.md
│  │  ├─ product-principles.md
│  │  └─ integration-notes.md
├─ apps/
│  ├─ web/
│  │  └─ CLAUDE.md
│  └─ api/
│     └─ CLAUDE.md
└─ packages/
```

Suggested pattern:

- `CLAUDE.md`: global orientation, commands, critical rules, imports. 
- `docs/context/project-map.md`: directory guide with “when to read this” notes.
- `apps/web/CLAUDE.md`: UI-specific patterns.
- `apps/api/CLAUDE.md`: API/data conventions.
- `.claude/rules/*.md`: file-type or path-specific rules that trigger only when relevant. [page:1]

### Optional metadata pattern

If you want meta tags, treat them as a convention you define for humans and for prompts, not as a documented Claude Code feature.

```md
---
tags: [payments, retries, external-api]
paths: ["apps/api/**"]
when_to_read: "Use for webhook, retry, or payment prompts"
---

# Payment Integration Notes
...
```

That can still be useful because you or the agent can search for tags and follow the document’s own routing hints, but automatic tag-based retrieval is not verified in the reviewed Claude Code docs. Unknown. [page:1]

## Can you point Claude Code at this effectively?

Yes, at the repo level: Claude Code is explicitly designed to read project files, run in a working directory, and load `CLAUDE.md` instructions from the repo hierarchy. [web:15][page:1]

For a simple start, open Claude Code in the repo and create `CLAUDE.md` plus a few small markdown docs referenced with `@imports`. That is the most direct, documented way to give it persistent context without attaching files manually each session. [page:1]

For a more advanced setup, combine root instructions, nested subsystem `CLAUDE.md` files, and `.claude/rules/` so context is progressively disclosed. That is close to the behavior you described, but the verified mechanism is file hierarchy and scoped loading, not a documented “meta-tag router.” [page:1]

## Simple commands to try in Claude Code

These are starter prompts, not shell commands:

1. `Read CLAUDE.md and summarize the project structure before making changes.`
2. `Check docs/context/project-map.md and tell me which files matter for adding retry logic to the webhook handler.`
3. `I need to update the checkout API. Read only the relevant files and explain your plan first.`
4. `Before coding, read the nearest CLAUDE.md files and any matching rules, then list the constraints you will follow.`

These prompts align with Claude Code’s documented behavior of using repo instructions and reading files as needed. [page:1]

## Are people actually doing this?

Yes, there is direct evidence of real usage, though quality and rigor vary by source. Anthropic documents the official `CLAUDE.md` workflow; Aider documents repo-map based context selection; and community writeups and posts describe people keeping persistent repo guidance in `CLAUDE.md` or `.context/` folders to avoid repeating conventions. [page:1][page:2][web:1][web:6][web:9]

The strongest proof is the official Claude Code memory documentation, because it confirms the product supports repo-scoped instructions, imports, nested files, and scoped rules as first-class features. Community posts are useful as examples of practice, but they should be treated as anecdotal rather than product guarantees. [page:1][web:1][web:6]
