---
description: Save current session context into a thread's thread.html
allowed-tools: Bash, Read, Edit, AskUserQuestion
---

Distill relevant context from the current conversation and persist it into `~/.claude/_thread/<name>/thread.html`.

## Step 1 — Resolve thread name

If a name was passed as an argument, use it. Otherwise:

1. Check for an active thread:
   ```bash
   cat "${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread/.active" 2>/dev/null || echo "NONE"
   ```
2. If an active thread name is found and its `thread.html` exists, use it automatically without prompting.
3. Otherwise list available threads and ask the user which to save into (use AskUserQuestion with the discovered names as options).
4. If no threads exist at all, suggest `/thread:create` and stop.

## Step 2 — Verify thread exists

```bash
THREAD_DIR="${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread/<name>"
[ -f "$THREAD_DIR/thread.html" ] && echo "OK" || echo "MISSING"
```

If missing, suggest `/thread:create <name>` and stop.

## Step 3 — Read current thread.html

Read `$THREAD_DIR/thread.html` using the Read tool to understand what is already stored.

## Step 4 — Enhance the thread

Review the current conversation and use your judgment to decide what context would be valuable for a future session. Don't follow a fixed template — structure the content however makes sense for this thread. Things that are often worth capturing: what was worked on, what was learned or decided, open questions, key facts a future session would otherwise have to re-derive. Omit anything ephemeral or obvious.

Merge new content with what's already in the thread. Don't erase prior content unless it's been superseded. Use `<section>` elements with `<h2>` headings to organize content. Update the `<time id="updated">` timestamp to today's date (YYYY-MM-DD).

## Step 5 — Confirm

Tell the user:

> Thread **`<name>`** updated — `thread.html` saved.
> View it with `/thread:browse <name>` or load it next session with `/thread:load <name>`.
