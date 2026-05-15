---
description: Delete a thread and all its files
allowed-tools: Bash, AskUserQuestion
---

Permanently delete a thread from `~/.claude/_thread/<name>/`.

## Step 1 — Resolve thread name

Run this script to resolve the thread name:

```bash
THREAD_ROOT="${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread"
REQUESTED=""  # replace with argument if provided

THREADS=$(find "$THREAD_ROOT" -mindepth 1 -maxdepth 1 -type d 2>/dev/null \
  | xargs -I{} basename {} | grep -v '^\.' | sort)

if [ -n "$REQUESTED" ]; then
  if [ -d "$THREAD_ROOT/$REQUESTED" ]; then
    echo "RESOLVED=$REQUESTED"
  else
    MATCHES=$(echo "$THREADS" | grep -i "$REQUESTED" 2>/dev/null || true)
    COUNT=$(echo "$MATCHES" | grep -c . 2>/dev/null || echo 0)
    if [ "$COUNT" -eq 1 ]; then echo "RESOLVED=$MATCHES"
    elif [ "$COUNT" -gt 1 ]; then echo "RESOLVED=AMBIGUOUS"; echo "$MATCHES"
    else echo "RESOLVED=MISSING"
    fi
  fi
else
  echo "RESOLVED=PROMPT"; echo "$THREADS"
fi
```

- `RESOLVED=<name>` — use it directly.
- `RESOLVED=AMBIGUOUS` + list — show `AskUserQuestion` with only the listed matches.
- `RESOLVED=PROMPT` + list — show `AskUserQuestion` with all threads.
- `RESOLVED=MISSING` — tell the user no match was found and suggest `/thread:list`. Stop.
- Empty `THREADS` — tell the user no threads exist and stop.

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
