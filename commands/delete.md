---
description: Delete a thread and all its files
allowed-tools: Bash, AskUserQuestion
---

Permanently delete a thread from `~/.claude/_thread/<name>/`.

## Step 1 — Resolve thread name

If a name was passed as an argument, use it. Otherwise:

1. List available threads:
   ```bash
   ls -1 "${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread" 2>/dev/null || echo "NONE"
   ```
2. If none exist, tell the user and stop.
3. Otherwise ask the user which thread to delete (use AskUserQuestion with thread names as options).

## Step 2 — Count files and confirm

```bash
THREAD_DIR="${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread/<name>"
[ -d "$THREAD_DIR" ] && find "$THREAD_DIR" -type f | wc -l | tr -d ' ' || echo "MISSING"
```

If the directory is missing, tell the user and stop.

Ask for confirmation before deleting — this is irreversible:

```
AskUserQuestion:
  question: "Permanently delete thread '<name>' and all N file(s)? This cannot be undone."
  header: "Confirm delete"
  options:
    - label: "Yes, delete it"   description: "Remove the thread directory and all its files"
    - label: "Cancel"           description: "Keep the thread"
  multiSelect: false
```

If the user picks Cancel (or types anything other than confirm), stop without deleting.

## Step 3 — Delete

```bash
rm -rf "${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread/<name>"
```

## Step 4 — Confirm

Tell the user:

> Thread **`<name>`** deleted.
