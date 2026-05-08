# create-project

A Claude Code skill that scaffolds a new project's documentation tree, git hooks, and seed files from a four-question intake.

The skill treats the project as the docs first: specs come before code, and the docs directory is the source of truth. Primary reader is [SilverBullet](https://silverbullet.md); Obsidian works as a secondary reader.

## What it does

1. **Intake** — asks four questions (what, who for, MVP, tech decisions) in a single message.
2. **Initializes git** if the directory isn't already a repo.
3. **Creates the docs tree**:
   ```
   docs/
   ├── README.md
   ├── GLOSSARY.md
   ├── LAUNCH.md
   ├── adr/ADR.md
   ├── devlog/DEVLOG.md
   ├── reading/READING.md
   ├── templates/{todo,adr,devlog}.md
   └── todos/{active,backlog,completed,icebox}/
   ```
4. **Populates seed files** from intake answers — glossary terms, launch stages, first devlog, first ADRs (only when a clear rationale was given), initial todos.
5. **Wires git hooks** — pre-commit regenerates the todos index from frontmatter; post-commit appends commit lines to the day's devlog entry.

## Conventions baked in

- Wiki-link targets drop the `.md` suffix (SilverBullet convention).
- Index files are `UPPERCASE.md` matching their directory; standalone docs are `SCREAMING_SNAKE_CASE.md`.
- Every non-index file gets YAML frontmatter (`type`, `status`, `description`, etc.).
- Todos move through `active → completed` (with commit references) or `active → icebox` (with a `reason:`).
- ADRs are numbered, append-only — superseded ADRs get a note, never deleted.
- `<!-- GENERATED:START -->` / `<!-- GENERATED:END -->` markers in index tables are load-bearing; `scripts/todos-index` rewrites only that region.

## Installation

### Via `npx skills add` (recommended)

```sh
npx skills add buzdyk/create-project
```

This uses [vercel-labs/skills](https://github.com/vercel-labs/skills) to install `SKILL.md` into your Claude Code skills directory.

### Manual

Copy `SKILL.md` into your Claude Code skills directory (e.g. `~/.claude/skills/create-project/SKILL.md`).

Invoke with `/create-project` from inside an empty (or new) project directory.
