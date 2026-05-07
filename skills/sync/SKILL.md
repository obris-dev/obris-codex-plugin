---
name: obris-sync
description: Sync an Obris topic to a local directory. Pulls remote items down, pushes local edits back. Use when you need topic files available on disk.
---

# Sync Obris Topic

## Prerequisites

Requires the `obris` CLI. PyPI package is `obris-cli` (the installed
command is `obris`):

```bash
pip install obris-cli
```

Authentication is browser-based, two-step. See the `@obris-auth` skill
— short version: `obris auth login --no-wait`, show the user the URL,
then `obris auth complete` once they say they've authorized.

## First sync — link a topic to a directory

Find the topic with the MCP `list_topics` tool, then run:

```bash
obris sync -p <directory> -t <topic-id>
```

Pulls every item in the topic (and its subtopics) down into the
directory. Subtopics become subdirectories under it.

### When the directory already has files the user wants in the topic

Use `--add-all` on the first sync. The CLI walks the dir, uploads
every untracked file in one bulk request, and creates subtopics on
the server to mirror local subdir structure:

```bash
obris sync -p <directory> -t <topic-id> --add-all
```

`--add-all` skips a sensible default set of noise paths (`.git/`,
`.venv/`, `node_modules/`, `.env`, `.env.*`, `.ssh/`, `.aws/`,
`.gnupg/`, `.DS_Store`, etc.) so secrets and tooling cruft don't
land in the knowledge base. Common AI tool config dirs (`.claude/`,
`.cursor/`, `.github/`) are *not* skipped — they're treated as
content. If something the user wants synced is being skipped, see
the "Including a skipped file" section below.

## Ongoing sync — directory already linked

```bash
obris sync -p <directory>
```

The CLI auto-detects the linked topic from the directory's hidden
`.obris/` state dir. Picks up remote changes (pulls), pushes local
edits, and reports new untracked files without uploading them.

## Adding new files after the initial sync

`--add-all` is idempotent — re-running picks up newly-added local
files without touching ones already tracked:

```bash
obris sync -p <directory> --add-all
```

For a single file, the per-file path is also fine (see
`@obris-add`).

## Unlinking a file

When the user wants to stop syncing a file but keep both copies
intact, use unlink:

```bash
obris sync unlink <file-or-knowledge-id>
obris sync unlink notes.md draft.md           # multiple at once
```

Both the local file and the remote item stay where they are — the
CLI just removes the link so subsequent `obris sync` calls won't
re-pull or push the item. Re-link later with `@obris-link` or
`@obris-add`.

This is different from `obris knowledge delete <id>`, which
permanently deletes the remote item.

## Including a skipped file

If `obris sync` shows a file as untracked but `--add-all` skips it,
it's matching one of the default exclusions or one the user added.
Override it explicitly:

```bash
obris sync include <pattern> -p <directory>
```

`include` overrides defaults *and* prior `obris sync exclude` calls.
Last call wins. Examples:

```bash
obris sync include .env.example   # sync the example even though .env.* is excluded
obris sync include .DS_Store      # override a built-in default
obris sync include scratch/draft  # un-exclude a folder pattern previously excluded
```

To skip a file going forward:

```bash
obris sync exclude <pattern> -p <directory>
```

To see what's currently configured:

```bash
obris sync exclude --list -p <directory>
```

## Dry-run preview

```bash
obris sync -p <directory> --dry-run             # preview pulls/pushes only
obris sync -p <directory> --dry-run --add-all   # also previews "Would add ..."
```

The `--dry-run` flag never mutates server or disk state. Combine
with `--add-all` to see exactly which files would be uploaded
before committing. If the directory has no linked topic yet, the
dry-run shows what topic name and subtopic structure *would* be
created without actually creating them.

## Selective sync

Sync specific items only:

```bash
obris sync -p <directory> -i <item-id> -i <item-id>
```

Sync specific subtopic name paths only (glob-style):

```bash
obris sync -p <directory> --include 'Projects/**' --include '**/skill-*'
```

`--include` scopes **every operation** in the sync — pulls of new
items, push and pull of tracked items, conflict detection,
remote-deleted detection, and `--add-all` bulk uploads. Files outside
the include scope are surfaced separately so the user knows they
were skipped on purpose:

```
Skipped 2 untracked file(s) outside --include scope:
  Misc/random.md
  README.md
```

Patterns OR together; supports `**` (zero or more segments), `*`,
`?`, `[abc]`. Case-insensitive.

Use the MCP `get_topic_knowledge` tool to find item IDs by name.

## Conflicts

When the same item changed on both sides between syncs, the CLI
flags it conflicted and skips it for both push and pull. Other items
sync normally; the conflict is opt-in to resolve:

```bash
obris sync conflicts list -p <directory>
obris sync conflicts resolve <filename> --keep-local -p <directory>
obris sync conflicts resolve <filename> --keep-remote -p <directory>
```

`--keep-local` pushes the local copy up. `--keep-remote` pulls the
remote copy down and saves the local copy as a sibling
`<name> (conflict YYYY-MM-DD).<ext>` file so nothing is lost.

## Where state lives

The CLI stores its tracking metadata at `<directory>/.obris/` —
one JSON file per linked topic, plus a `.gitignore` so the dir
self-excludes from any git/svn checkout the user's project might
be inside. The engine ignores `.obris/` when scanning for files to
sync (it's in the default exclusion list).

`rm -rf <directory>` cleans up tracking state along with the
local files. No global state is left orphaned.

## Common flags

- `-p, --path` — local directory (defaults to current directory)
- `-t, --topic` — topic ID (required for first sync)
- `-i, --item` — specific item IDs (repeatable)
- `--include` — subtopic name-path glob (repeatable). Supports `**`,
  `*`, `?`, `[abc]`. Examples: `Projects/**`, `**/skill-*`,
  `Archive/2024`. Case-insensitive.
- `--add-all` — upload every untracked file
- `--no-subtopics` — with `--add-all`, skip subdir files instead of creating subtopics
- `--no-create` — error instead of creating a root topic when the dir has no state
- `--dry-run` — preview without making changes
- `-v, --verbose` — list every untracked / excluded / symlink path

### Machine-readable output

The top-level `--json` flag on `obris` itself (not the sync subcommand)
makes the sync command emit a structured JSON summary instead of the
human-readable lines:

```bash
obris --json sync -p <directory>
```

JSON fields: ``path``, ``pulled``, ``pushed``, ``conflicts``,
``errors``, ``untracked``, ``excluded_count``, ``symlinks``,
``conflicts_pending``, ``missing_local``. Use this when relaying
sync outcomes to other tools or when you need to count exactly
how many files moved.

## After syncing

Tell the user what was pulled, pushed, or skipped. If conflicts were
flagged, list them and explain the `conflicts resolve` flow. If
files were excluded by default, mention `obris sync include
<pattern>` as the override path.
