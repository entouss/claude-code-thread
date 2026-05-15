---
description: Save current session context into a thread's thread.html
allowed-tools: Bash, Read, Edit, AskUserQuestion
---

Distill relevant context from the current conversation and persist it into `~/.claude/_thread/<name>/thread.html`.

## Step 1 — Resolve thread name

Run this script to resolve the thread name:

```bash
THREAD_ROOT="${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread"
REQUESTED=""  # replace with argument if provided

THREADS=$(find "$THREAD_ROOT" -mindepth 1 -maxdepth 1 -type d 2>/dev/null \
  | xargs -I{} basename {} | grep -v '^\.' | sort)
ACTIVE=$(cat "$THREAD_ROOT/.active" 2>/dev/null || true)

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
elif [ -n "$ACTIVE" ] && [ -f "$THREAD_ROOT/$ACTIVE/thread.html" ]; then
  echo "RESOLVED=$ACTIVE"
else
  echo "RESOLVED=PROMPT"; echo "$THREADS"
fi
```

- `RESOLVED=<name>` — use it directly.
- `RESOLVED=AMBIGUOUS` + list — show `AskUserQuestion` with only the listed matches.
- `RESOLVED=PROMPT` + list — show `AskUserQuestion` with all threads.
- `RESOLVED=MISSING` — tell the user no match was found and suggest `/thread:list`. Stop.
- Empty `THREADS` — suggest `/thread:create` and stop.

## Step 3 — Read current thread.html

Read `$THREAD_DIR/thread.html` using the Read tool to understand what is already stored.

## Step 4 — Enhance the thread

Review the current conversation and use your judgment to decide what context would be valuable for a future session. Don't follow a fixed template — structure the content however makes sense for this thread. Things that are often worth capturing: what was worked on, what was learned or decided, open questions, key facts a future session would otherwise have to re-derive. Omit anything ephemeral or obvious.

Merge new content with what's already in the thread. Don't erase prior content unless it's been superseded. Use `<section>` elements with `<h2>` headings to organize content. Update the `<time id="updated">` timestamp to today's date (YYYY-MM-DD).

## Step 5 — Confirm

Tell the user:

> Thread **`<name>`** updated — `thread.html` saved.
> View it with `/thread:browse <name>` or load it next session with `/thread:load <name>`.
