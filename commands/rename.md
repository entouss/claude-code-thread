---
description: Rename a thread
allowed-tools: Bash, Read, Edit, AskUserQuestion
---

Rename a thread from `~/.claude/_thread/<old>/` to `~/.claude/_thread/<new>/` and update the title inside `thread.html`.

## Step 1 — Resolve old name

Run this script to resolve the old thread name:

```bash
THREAD_ROOT="${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread"
REQUESTED=""  # replace with first argument if provided

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
