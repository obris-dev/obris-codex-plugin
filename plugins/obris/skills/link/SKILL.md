---
name: obris-link
description: Relink a renamed local file to its remote Obris item. Use after renaming a synced file to restore tracking.
---

# Link File to Obris Item

## Prerequisites

Requires the `obris` CLI (`pip install obris-cli` — note the package
name is `obris-cli`, not `obris`) and authentication via the
`@obris-auth` skill (two-step `obris auth login --no-wait` + `obris auth
complete` flow).

## Instructions

Relink a local file to an existing remote item after a rename.

```bash
obris sync link <filepath> -i <item-id> -t <topic-id>
```

### When to use

When a user renames a synced file, the next sync unlinks it and shows:

```
Missing: old-name.md (was synced to "old-name.md")
  This file will no longer sync with its remote item.
  To relink after a rename: obris sync link <new-filename> -i <item-id>
```

### Steps

1. Identify the file and the item ID to link to
2. If the user doesn't have the item ID, use MCP `get_topic_knowledge` to find the item by name
3. Run the link command
4. Confirm the link was established
5. Suggest running `obris sync` to verify everything is connected
