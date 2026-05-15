---
description: Rename a thread
allowed-tools: Bash, Read, Edit, AskUserQuestion
---

Rename a thread from `~/.claude/_thread/<old>/` to `~/.claude/_thread/<new>/` and update the title inside `thread.html`.

## Step 1 — Resolve old name

If an old name was passed as the first argument, use it. Otherwise:

1. List available threads:
   ```bash
   ls -1 "${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread" 2>/dev/null || echo "NONE"
   ```
2. If none exist, tell the user and stop.
3. Otherwise ask which thread to rename (use AskUserQuestion with thread names as options).

## Step 2 — Resolve new name

If a new name was passed as the second argument, use it. Otherwise ask:

```
AskUserQuestion:
  question: "What should the new name be?"
  header: "New name"
  options:
    - label: "Other"  description: "I'll type a name"
  multiSelect: false
```

Validate the new name: only letters, digits, hyphens, and underscores. No spaces. If invalid, tell the user and stop.

## Step 3 — Check for conflicts

```bash
THREAD_ROOT="${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread"
[ -d "$THREAD_ROOT/<new>" ] && echo "EXISTS" || echo "OK"
```

If the destination already exists, tell the user and stop.

## Step 4 — Rename the folder

```bash
mv "${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread/<old>" \
   "${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread/<new>"
```

## Step 5 — Update thread.html

Read `~/.claude/_thread/<new>/thread.html` with the Read tool, then use Edit to update:

1. `<title>` — change `Thread: <old>` to `Thread: <new>`
2. `<h1><span>` — change the displayed thread name from `<old>` to `<new>`

## Step 6 — Confirm

Tell the user:

> Thread renamed: **`<old>`** → **`<new>`**
