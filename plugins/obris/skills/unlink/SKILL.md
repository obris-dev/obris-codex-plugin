---
name: obris-unlink
description: Stop syncing a local file without deleting either side. Use when the user wants to keep both the local file and the remote item but break the sync link.
---

# Unlink File from Obris Item

## Prerequisites

Requires authentication via the `@obris-auth` skill (two-step
`obris auth login --no-wait` + `obris auth complete` flow).

## Instructions

Break the local-to-remote sync link without deleting either side.

```bash
obris sync unlink <file-or-knowledge-id>
obris sync unlink notes.md draft.md           # multiple at once
```

The local file and the remote item stay where they are. The CLI just
removes the tracking link so subsequent `obris sync` calls won't
re-pull or push that item.

### When to use

- The user wants to stop syncing a specific file but keep both copies
  intact.
- Different from `obris knowledge delete <id>`, which permanently
  deletes the remote item.
- Different from removing a file from disk, which the sync engine
  reports as missing without unlinking.

### Steps

1. Identify the file (basename or knowledge ID) the user wants to
   unlink. If the user only knows the file's name and there could be
   ambiguity, use the MCP `get_topic_knowledge` tool to find the
   exact knowledge ID.
2. Run the unlink command.
3. If the CLI reports the target as ambiguous (matches multiple
   tracked items), it prints the candidates. Re-run with the specific
   knowledge ID from the list.
4. Confirm the link was broken. The local file and remote item are
   unchanged.
5. If the user wants to reconnect the file later, point them at
   `@obris-link` (after a rename) or `@obris-add` (to re-add fresh).
