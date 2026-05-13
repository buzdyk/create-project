---
name: create-project
description: Scaffold a new project with docs/, git hooks, and seed files from intake questions
allowed-tools: Bash, Write, Read, Glob, Grep, AskUserQuestion
---

# Create Project

You are bootstrapping a new project's documentation and automation. Follow these steps exactly.

## Step 1: Intake

Ask the user these four questions **in a single message**. Wait for all answers before proceeding.

1. **What is this project?** (1-2 sentences)
2. **Who is it for? How does it apply to your life?**
3. **What does the MVP look like?**
4. **What tech decisions have you already made?**

In the same message, list the defaults you'll apply unless they override them, and invite changes:

> Defaults (say so if you'd like any of these changed before I scaffold):
> - **Naming**: files inside a folder are `SPEC_NAME.md` — the folder name conveys the type, so no `adr-`/`active-` prefixes (`adr/USER_AUTH.md`, not `adr/ADR_USER_AUTH.md`).
> - **Index location**: index files live as a **sibling** of the folder they index (`docs/ADR.md` indexes `docs/adr/`). Exception: `docs/README.md`.
> - **Wiki links**: targets drop the `.md` suffix (SilverBullet convention).
> - **Index style**: `UPPERCASE.md` matching the folder name.

If the user overrides any default, apply that override consistently across the scaffold (tree, index file contents, link generation in `scripts/todos-index`, devlog wiki-link path) — don't leave the new project half-converted.

## Step 2: Initialize git

If the current directory is not already a git repository, run `git init`.

## Step 3: Create seed directories and index files

Create these directories and their index files. Do not create any other directories — they grow with the project.

Index files live **one folder above** the folder they index — siblings of the folder, not inside it. The only exception is `docs/README.md`, which acts as the entry-point for the docs tree itself.

```
docs/
├── README.md            # entry point for docs/
├── ADR.md               # index of adr/
├── DEVLOG.md            # index of devlog/
├── READING.md           # index of reading/
├── TEMPLATES.md         # index of templates/
├── TODOS.md             # summary of todos/ buckets
├── adr/
├── devlog/
├── reading/
├── templates/
└── todos/
    ├── ACTIVE.md        # index of active/
    ├── BACKLOG.md       # index of backlog/
    ├── COMPLETED.md     # index of completed/
    ├── ICEBOX.md        # index of icebox/
    ├── active/
    ├── backlog/
    ├── completed/
    └── icebox/
```

### Index file content

Link targets drop the `.md` suffix (SilverBullet / Obsidian convention). Keep `.md` in prose when describing filenames on disk.

**docs/README.md** — the entry point. Carries conventions so readers know how the tree is organized:

````markdown
# Docs

The **project is the docs**. Specs come before code; code is generated from specs; docs are the source of truth and kept tidy.

Primary reader is [SilverBullet](https://silverbullet.md); Obsidian is a secondary reader. Wiki-link navigation assumes SilverBullet/Obsidian semantics.

## Index

- [GLOSSARY](./GLOSSARY) — shared vocabulary
- [LAUNCH](./LAUNCH) — launch plan and stages
- [ADR](./ADR) — architecture decision records (numbered, append-only) → `adr/`
- [DEVLOG](./DEVLOG) — daily dev log entries → `devlog/`
- [READING](./READING) — research notes → `reading/`
- [TEMPLATES](./TEMPLATES) — starter files → `templates/`
- [TODOS](./TODOS) — `active/`, `backlog/`, `completed/`, `icebox/` summary

## File Conventions

### Naming

- Index files live **one folder above** the folder they index, named `UPPERCASE.md` matching the folder (e.g., `ADR.md` is the sibling of `adr/`, `ACTIVE.md` is the sibling of `active/`). The only exception is this `README.md`, which sits inside `docs/`.
- **Don't repeat the folder name in the file.** The path already conveys the type. Default pattern is `folder-name/SPEC_NAME.md` — files inside `adr/` are named `USER_AUTH.md`, not `adr-user-auth.md` or `adr/ADR_USER_AUTH.md`.
- Standalone todos and docs: `SCREAMING_SNAKE_CASE.md`
- Numbered items inside epics: `01-SHORT_NAME.md`, `02-SHORT_NAME.md`
- Devlog entries: `YYYY/MM_MMM/DD.md` (e.g., `2026/04_APR/19.md`)
- Reading notes: `NN_TOPIC_NAME.md` with a global sequence number

### Frontmatter

Every file that isn't an index gets YAML frontmatter:

```yaml
---
type: todo | epic | devlog | feature | adr | reading
status: pending | in_progress | done | backlog | icebox
description: one-line summary shown in the index   # todos
reason: why it's frozen                             # icebox todos only
date: 2026-02-25                                    # devlog entries only
---
```

`description`, `status`, and `reason` on todo files feed the generated index tables (`scripts/todos-index`) — keep them current.

### Cross-references

- **Drop `.md` from link targets.** SilverBullet addresses pages by name, not path+extension.
  - Markdown links: `[Title](./FILE)` — no `.md` suffix on the target.
  - Wiki links: `[[FILE]]` or `[[FILE|Display]]` — no `.md` inside.
- Prefer `[[wiki]]` for cross-doc references. Use `[text](./path)` when display text differs from the target page name.
- Keep `.md` in prose/backticks when describing *filenames on disk* — those are not links.

## Todo Lifecycle

1. **New work** starts as a file in `todos/active/`.
2. **When done**, move to `todos/completed/`, set `status: done`, add a "What Was Done" section with commit references.
3. **Deferred work** moves to `todos/icebox/` with a `reason:` frontmatter field.
4. **Future ideas** go straight to `todos/backlog/`.

The indexes (`todos/ACTIVE.md`, `todos/BACKLOG.md`, `todos/ICEBOX.md`, `todos/COMPLETED.md`, and `TODOS.md`) are regenerated from frontmatter by `scripts/todos-index`, wired into the pre-commit hook.

## Devlog

Entries live at `devlog/YYYY/MM_MMM/DD.md` and are auto-generated by the post-commit hook from the day's commits. A row in `DEVLOG.md` is inserted the first time a date is written.

## ADRs

Numbered sequentially (`001-`, `002-`, ...) and append-only. Superseded ADRs get a "Superseded by ADR NNN" note — never deleted.
````

**ADR.md:**
```markdown
# Architecture Decision Records

Numbered sequentially. Append-only — superseded ADRs get a note, never deleted.

| ADR | Title | Status |
|-----|-------|--------|
```

**DEVLOG.md:**
```markdown
# Dev Log

Daily development logs, auto-generated by post-commit hook.

## Entries

| Date | Entry | Description |
|------|-------|-------------|
```

**READING.md:**
```markdown
# Reading

Reference material on technical topics relevant to the project.
```

**TEMPLATES.md:**
```markdown
# Templates

Starter files for common document types.

| Template | Usage |
|----------|-------|
| [todo](./templates/todo) | New task or work item |
| [adr](./templates/adr) | Architecture decision record |
| [devlog](./templates/devlog) | Daily dev log entry |
```

**TODOS.md** (at `docs/TODOS.md`, sibling of `todos/`) — bucket summary populated by `scripts/todos-index`:
```markdown
# Todos

<!-- GENERATED:START -->
| Bucket | Description | Count |
|--------|-------------|-------|
<!-- GENERATED:END -->

_Regenerate with `scripts/todos-index`._
```

**ACTIVE.md:**
```markdown
# Active Todos

Work currently in progress or queued.

<!-- GENERATED:START -->
| Todo | Description | Status |
|------|-------------|--------|
<!-- GENERATED:END -->

_Regenerate with `scripts/todos-index`._
```

**BACKLOG.md:**
```markdown
# Backlog

Future work, not yet scheduled.

<!-- GENERATED:START -->
| Todo | Description |
|------|-------------|
<!-- GENERATED:END -->

_Regenerate with `scripts/todos-index`._
```

**COMPLETED.md:**
```markdown
# Completed

Archive of finished work.

<!-- GENERATED:START -->
| Todo | Description |
|------|-------------|
<!-- GENERATED:END -->

_Regenerate with `scripts/todos-index`._
```

**ICEBOX.md:**
```markdown
# Icebox

Deferred indefinitely. Each item includes a reason.

<!-- GENERATED:START -->
| Todo | Description | Reason |
|------|-------------|--------|
<!-- GENERATED:END -->

_Regenerate with `scripts/todos-index`._
```

The `<!-- GENERATED:START -->` / `<!-- GENERATED:END -->` markers are load-bearing — `scripts/todos-index` (added in Step 6) rewrites only the region between them.

## Step 4: Populate docs from intake answers

Use the user's answers to create these files. Extract terms, services, entities, and infrastructure from all four answers — don't ask clarifying questions, just use what's there.

### GLOSSARY.md

```markdown
# Glossary

## Services

| Term | Definition |
|------|------------|

## Data Entities

| Term | Definition |
|------|------------|

## Infrastructure

| Term | Definition |
|------|------------|
```

Populate each table from the intake answers. If the user mentioned services, tech stack components, or domain concepts, they go here. Leave a table empty if there's nothing to fill it with yet.

### LAUNCH.md

```markdown
# Launch Plan

## Current State

{From answer #1 — what the project is and where it stands}

## Launch Stages

### Stage 1 — MVP
{From answer #3 — break the MVP description into checklist items}

## Launch Blockers

- [ ] {Any blockers implied by the answers}
```

### First devlog entry

Create `docs/devlog/YYYY/MM_MMM/DD.md` using today's date (e.g. `2026/03_MAR/13.md`):

```markdown
---
type: devlog
date: YYYY-MM-DD
---
# Mon DD - Dev Log

## Summary

**Project Setup**
- Initialized project documentation
- {Brief context from answers — what the project is, first priorities}

## Commits

- (auto-populated by post-commit hook)
```

Add a row to `docs/DEVLOG.md` using the nested wiki-link path (no `.md`), prefixed with `devlog/` since `DEVLOG.md` now lives one level above the `devlog/` folder:

```
| YYYY-MM-DD | [[devlog/YYYY/MM_MMM/DD]] | Initialized project docs |
```

### First ADR(s)

If the user's answer to question #4 (tech decisions) mentions specific choices with reasoning, create ADRs. Use this format:

```markdown
---
type: adr
status: accepted
date: YYYY-MM-DD
---
# ADR-001: Title

## Status
Accepted

## Context
{Why this decision was needed}

## Decision
{What was decided}

## Consequences
{Tradeoffs}
```

Only create ADRs for decisions that have clear rationale. If the user just listed a tech stack without reasoning, put the stack in the GLOSSARY infrastructure table instead.

### Initial todos

If the MVP answer implies concrete first tasks, create them in `todos/active/` with frontmatter (the `description:` field feeds the generated index):

```markdown
---
type: todo
status: pending
description: one-line summary shown in ACTIVE.md
---
# Task Title

## Problem

{What needs to be built or fixed}

## Approach

{Steps, if obvious from context}
```

Don't hand-edit `ACTIVE.md` — `scripts/todos-index` regenerates its table from frontmatter on the first commit.

## Step 5: Create template files

### templates/todo.md

```markdown
---
type: todo
status: pending
description: one-line summary shown in the index
---
# Title

## Problem

## Approach

## Related
```

### templates/adr.md

```markdown
---
type: adr
status: proposed
date: {{date}}
---
# ADR-XXX: Title

## Context

## Decision

## Consequences
```

### templates/devlog.md

```markdown
---
type: devlog
date: YYYY-MM-DD
---
# Mon DD - Dev Log

## Summary

**Category**
- What was done

## Commits

- `hash` Message
```

## Step 6: Install git hooks and automation

Create the following files. All AI-invoking hooks support two env vars:

| Env var | Purpose | Example |
|---------|---------|---------|
| `AI_CMD` | Override the AI CLI command. Default: `claude -p` | `export AI_CMD="ollama run llama3"` |
| `SKIP_AI_HOOKS` | Set to `1` to bypass all AI hooks for a single commit | `SKIP_AI_HOOKS=1 git commit` |

The `-m` flag already skips `prepare-commit-msg` (git doesn't invoke it when a message is provided). `SKIP_AI_HOOKS` is for skipping the `post-commit` devlog generation too — useful for bulk operations, rebases, or working offline.

### scripts/todos-index

Regenerates todo index tables from per-file YAML frontmatter. Each per-bucket index sits one folder above its bucket (`docs/todos/ACTIVE.md` indexes `docs/todos/active/`, etc.), and the overall summary sits one folder above `todos/` (`docs/TODOS.md`). The script rewrites only the region between `<!-- GENERATED:START -->` and `<!-- GENERATED:END -->`.

```bash
#!/bin/bash

# Regenerate todo index tables from per-file YAML frontmatter.
# Reads: docs/todos/{active,backlog,icebox,completed}/*.md
# Writes: the region between <!-- GENERATED:START --> and <!-- GENERATED:END -->
# inside each bucket's sibling index file (docs/todos/ACTIVE.md, etc.) and the
# top-level docs/TODOS.md summary.
#
# Each todo file's frontmatter should look like:
#   ---
#   type: todo
#   status: pending | icebox | done
#   description: one-line summary shown in the index
#   reason: why-it-is-frozen (icebox only)
#   ---

set -e

ROOT="$(git rev-parse --show-toplevel)"
DOCS_DIR="$ROOT/docs"
TODOS_DIR="$DOCS_DIR/todos"

extract_field() {
    local file="$1" field="$2"
    awk -v field="$field" '
        BEGIN { in_fm = 0 }
        /^---[[:space:]]*$/ {
            if (in_fm == 0) { in_fm = 1; next }
            else { exit }
        }
        in_fm == 1 {
            pos = index($0, ":")
            if (pos > 0) {
                key = substr($0, 1, pos-1)
                val = substr($0, pos+1)
                sub(/^[[:space:]]+/, "", key)
                sub(/[[:space:]]+$/, "", key)
                sub(/^[[:space:]]+/, "", val)
                sub(/[[:space:]]+$/, "", val)
                if (key == field) { print val; exit }
            }
        }
    ' "$file"
}

generate_index() {
    local dir="$1" index_file="$2" kind="$3"
    local bucket header rows="" f base type desc status reason
    bucket=$(basename "$dir")

    case "$kind" in
        active)
            header=$'| Todo | Description | Status |\n|------|-------------|--------|'
            ;;
        backlog)
            header=$'| Todo | Description |\n|------|-------------|'
            ;;
        icebox)
            header=$'| Todo | Description | Reason |\n|------|-------------|--------|'
            ;;
        completed)
            header=$'| Todo | Description |\n|------|-------------|'
            ;;
    esac

    shopt -s nullglob
    for f in "$dir"/*.md; do
        base=$(basename "$f" .md)
        type=$(extract_field "$f" "type")
        [ -z "$type" ] && continue

        desc=$(extract_field "$f" "description")
        case "$kind" in
            active)
                status=$(extract_field "$f" "status")
                rows+="| [$base](./$bucket/$base.md) | $desc | $status |"$'\n'
                ;;
            backlog|completed)
                rows+="| [$base](./$bucket/$base.md) | $desc |"$'\n'
                ;;
            icebox)
                reason=$(extract_field "$f" "reason")
                rows+="| [$base](./$bucket/$base.md) | $desc | $reason |"$'\n'
                ;;
        esac
    done

    if [ ! -f "$index_file" ]; then
        echo "warning: $index_file does not exist, skipping" >&2
        return
    fi

    if ! grep -q "<!-- GENERATED:START -->" "$index_file"; then
        echo "warning: $index_file has no GENERATED region, skipping" >&2
        return
    fi

    local tmp
    tmp=$(mktemp)
    TODOS_INDEX_BLOCK="${header}"$'\n'"${rows}" \
    awk '
        /<!-- GENERATED:START -->/ {
            print
            printf "%s", ENVIRON["TODOS_INDEX_BLOCK"]
            in_gen = 1
            next
        }
        /<!-- GENERATED:END -->/ {
            in_gen = 0
            print
            next
        }
        !in_gen { print }
    ' "$index_file" > "$tmp"
    mv "$tmp" "$index_file"
    echo "updated $index_file"
}

count_items() {
    local dir="$1" f type n=0
    shopt -s nullglob
    for f in "$dir"/*.md; do
        type=$(extract_field "$f" "type")
        [ -n "$type" ] && n=$((n + 1))
    done
    echo "$n"
}

generate_summary() {
    local index_file="$DOCS_DIR/TODOS.md"
    local active_n backlog_n icebox_n completed_n
    active_n=$(count_items "$TODOS_DIR/active")
    backlog_n=$(count_items "$TODOS_DIR/backlog")
    icebox_n=$(count_items "$TODOS_DIR/icebox")
    completed_n=$(count_items "$TODOS_DIR/completed")

    local header rows
    header=$'| Bucket | Description | Count |\n|--------|-------------|-------|'
    rows=""
    rows+="| [Active](./todos/ACTIVE.md) | Work in progress or queued for the current cycle | $active_n |"$'\n'
    rows+="| [Backlog](./todos/BACKLOG.md) | Future work, not yet scheduled | $backlog_n |"$'\n'
    rows+="| [Icebox](./todos/ICEBOX.md) | Deferred indefinitely | $icebox_n |"$'\n'
    rows+="| [Completed](./todos/COMPLETED.md) | Archive of finished work | $completed_n |"$'\n'

    if [ ! -f "$index_file" ]; then
        echo "warning: $index_file does not exist, skipping" >&2
        return
    fi
    if ! grep -q "<!-- GENERATED:START -->" "$index_file"; then
        echo "warning: $index_file has no GENERATED region, skipping" >&2
        return
    fi

    local tmp
    tmp=$(mktemp)
    TODOS_INDEX_BLOCK="${header}"$'\n'"${rows}" \
    awk '
        /<!-- GENERATED:START -->/ {
            print
            printf "%s", ENVIRON["TODOS_INDEX_BLOCK"]
            in_gen = 1
            next
        }
        /<!-- GENERATED:END -->/ {
            in_gen = 0
            print
            next
        }
        !in_gen { print }
    ' "$index_file" > "$tmp"
    mv "$tmp" "$index_file"
    echo "updated $index_file"
}

generate_index "$TODOS_DIR/active"    "$TODOS_DIR/ACTIVE.md"    active
generate_index "$TODOS_DIR/backlog"   "$TODOS_DIR/BACKLOG.md"   backlog
generate_index "$TODOS_DIR/icebox"    "$TODOS_DIR/ICEBOX.md"    icebox
generate_index "$TODOS_DIR/completed" "$TODOS_DIR/COMPLETED.md" completed
generate_summary
```

### scripts/hooks/pre-commit

```bash
#!/bin/bash

# Pre-commit hook - regenerate todo index tables if any todo file is staged
set -e

ROOT="$(git rev-parse --show-toplevel)"

if ! git diff --cached --name-only | grep -q '^docs/todos/.*\.md$'; then
    exit 0
fi

"$ROOT/scripts/todos-index" > /dev/null

for f in \
    docs/TODOS.md \
    docs/todos/ACTIVE.md \
    docs/todos/BACKLOG.md \
    docs/todos/ICEBOX.md \
    docs/todos/COMPLETED.md
do
    if [ -f "$ROOT/$f" ] && ! git diff --quiet -- "$ROOT/$f"; then
        git add "$ROOT/$f"
    fi
done
```

### scripts/hooks/post-commit

```bash
#!/bin/bash

# Post-commit hook - generate daily devlog summary
# Skip: SKIP_AI_HOOKS=1 git commit
[ "${SKIP_AI_HOOKS:-}" = "1" ] && exit 0
exec "$(git rev-parse --show-toplevel)/scripts/devlog-summary"
```

### scripts/hooks/prepare-commit-msg

```bash
#!/bin/bash

# Auto-generate commit message using Claude
# Runs after `git commit` but before editor opens
# User can still edit the message before finalizing
# Skip: SKIP_AI_HOOKS=1 git commit

[ "${SKIP_AI_HOOKS:-}" = "1" ] && exit 0

COMMIT_MSG_FILE=$1
COMMIT_SOURCE=$2

# Only generate for regular commits (not merge, squash, etc.)
if [ -n "$COMMIT_SOURCE" ]; then
    exit 0
fi

# Skip if message already provided via -m flag
if [ -s "$COMMIT_MSG_FILE" ] && ! grep -q "^#" "$COMMIT_MSG_FILE"; then
    exit 0
fi

AI_CMD="${AI_CMD:-claude -p}"

# Get staged diff
STAGED_DIFF=$(git diff --cached --stat)
STAGED_DIFF_FULL=$(git diff --cached)

if [ -z "$STAGED_DIFF" ]; then
    exit 0
fi

# Generate commit message
MESSAGE=$($AI_CMD "Generate a concise git commit message for these changes.

Files changed:
${STAGED_DIFF}

Diff:
${STAGED_DIFF_FULL}

Rules:
- First line: imperative mood, max 50 chars, no period (e.g., 'Add user auth', 'Fix search bug')
- If needed, add blank line then brief body explaining why (not what)
- No conventional commit prefixes unless obvious (feat:, fix:, etc.)
- Be specific, not generic
- NEVER mention Claude, Sonnet, Opus, GPT, AI, LLM, or any AI tool
- Write as if a human developer wrote it

Output only the commit message, nothing else." 2>/dev/null)

if [ -n "$MESSAGE" ]; then
    # Prepend generated message, keep git's commented help text
    EXISTING=$(cat "$COMMIT_MSG_FILE")
    printf "%s\n\n%s" "$MESSAGE" "$EXISTING" > "$COMMIT_MSG_FILE"
fi
```

### scripts/devlog-summary

```bash
#!/bin/bash

# Generate/update today's devlog from commits
# Called by post-commit hook or run manually: ./scripts/devlog-summary

set -e

[ "${SKIP_AI_HOOKS:-}" = "1" ] && exit 0

AI_CMD="${AI_CMD:-claude -p}"
DOCS_DIR="$(git rev-parse --show-toplevel)/docs"
DEVLOG_DIR="${DOCS_DIR}/devlog"
DATE_FULL=$(date +"%Y-%m-%d")
DATE_DISPLAY=$(date +"%b %d")

# Build year/month directory path: YYYY/MM_MMM/DD.md
YEAR=$(date +"%Y")
MONTH_DIR=$(date +"%m_%b" | tr '[:lower:]' '[:upper:]')
DAY=$(date +"%d")
ENTRY_DIR="${DEVLOG_DIR}/${YEAR}/${MONTH_DIR}"
mkdir -p "$ENTRY_DIR"
FILE_PATH="${ENTRY_DIR}/${DAY}.md"

# Wiki link path for DEVLOG.md index (relative to docs/, since DEVLOG.md lives at docs/DEVLOG.md)
WIKI_LINK="devlog/${YEAR}/${MONTH_DIR}/${DAY}"

# Get today's commits
TODAYS_COMMITS=$(git log --since="midnight" --pretty=format:"- %h %s" --reverse)

if [ -z "$TODAYS_COMMITS" ]; then
    echo "No commits today"
    exit 0
fi

# Get changed files summary for context
FILES_CHANGED=$(git log --since="midnight" --pretty=format:"" --name-only | sort -u | grep -v '^$' | head -20)

# Generate summary
SUMMARY=$($AI_CMD "Summarize a developer's daily work for their personal log.

Commits:
${TODAYS_COMMITS}

Files:
${FILES_CHANGED}

Output as bullet points grouped by area. Example format:

**Auth**
- Added password reset flow
- Implemented magic links

**Docs**
- Moved daily logs to devlog/

Rules:
- Group related changes under bold headers
- Use short bullet points (5-10 words each)
- No preamble, just the formatted summary
- Skip trivial changes" 2>/dev/null || echo "_Summary generation failed - add manually_")

# Write the file
cat > "$FILE_PATH" << EOF
---
type: devlog
date: ${DATE_FULL}
---
# ${DATE_DISPLAY} - Dev Log

## Summary

${SUMMARY}

## Commits

${TODAYS_COMMITS}
EOF

# Update DEVLOG.md if this date isn't already listed
INDEX_FILE="${DOCS_DIR}/DEVLOG.md"
if [ -f "$INDEX_FILE" ] && ! grep -q "\[\[${WIKI_LINK}\]\]" "$INDEX_FILE"; then
    SHORT_DESC=$($AI_CMD "Summarize this in under 10 words for a table cell, no period at end:
${SUMMARY}" 2>/dev/null || echo "Daily work log")

    SHORT_DESC_ESCAPED=$(echo "$SHORT_DESC" | sed 's/[&/\]/\\&/g')
    NEW_ROW="| ${DATE_FULL} | [[${WIKI_LINK}]] | ${SHORT_DESC_ESCAPED} |"

    awk -v new_row="$NEW_ROW" '
        /^\| 20[0-9][0-9]-/ { last_entry_line = NR }
        /^\|---/ { separator_line = NR }
        { lines[NR] = $0 }
        END {
            insert_after = last_entry_line ? last_entry_line : separator_line
            for (i = 1; i <= NR; i++) {
                print lines[i]
                if (i == insert_after) print new_row
            }
        }
    ' "$INDEX_FILE" > "${INDEX_FILE}.tmp" && mv "${INDEX_FILE}.tmp" "$INDEX_FILE"
fi

# Print output if run manually (not from hook)
if [ -t 1 ]; then
    echo "Written to $FILE_PATH"
    echo ""
    cat "$FILE_PATH"
fi
```

After creating these files, run:

```bash
chmod +x scripts/hooks/post-commit scripts/hooks/prepare-commit-msg scripts/hooks/pre-commit scripts/devlog-summary scripts/todos-index
git config core.hooksPath scripts/hooks
```

`core.hooksPath` points git at the in-repo hooks directory — no symlinks into `.git/hooks/`, works under worktrees. It's local config, so every clone needs to rerun `git config core.hooksPath scripts/hooks` after cloning. Note this in the project README.

## Step 7: Summary

Print a summary of what was created:
- List all files created with their paths
- Mention env vars: `AI_CMD` to swap the AI backend (default: `claude -p`, e.g. `export AI_CMD="ollama run llama3"`), `SKIP_AI_HOOKS=1` to bypass all AI hooks for a single commit
- Mention that fresh clones must run `git config core.hooksPath scripts/hooks` to wire hooks
- Note that new directories (features/, runbooks/, workflows/, etc.) should be created when the first file needs a home — not before
