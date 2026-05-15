---
description: Save current session context into a thread's thread.html
allowed-tools: Bash, Read, Edit, AskUserQuestion
---

Distill key information from the current conversation and persist it into `~/.claude/_thread/<name>/thread.html`.

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

## Step 4 — Synthesize new content

Review the current conversation context and extract:

- **Summary** — 2–4 sentence description of what this project/thread is about and current status.
- **Goals** — The top objectives being worked toward (bullet points).
- **Key Context** — Critical facts, constraints, file paths, conventions, decisions that future sessions must know (bullet points, concise).
- **Decisions** — Specific architectural or implementation decisions made, with brief rationale.
- **Session Entry** — A new timestamped entry summarizing what happened in this session: what was built, changed, or discovered.
- **Notes** — Any miscellaneous information worth preserving.

Keep each item concise. Merge new content with existing content — do not erase prior entries. Session entries are appended (newest at top); all other sections are updated/merged.

## Step 5 — Write to thread.html

Use the Edit tool to update `thread.html` in place. Apply these changes:

1. **`<time id="updated">`** — set to today's date (YYYY-MM-DD).
2. **`#summary p`** — replace with the new summary paragraph(s).
3. **`#goals ul`** — replace `<li>` items with new goal items (preserve prior goals unless superseded).
4. **`#context ul`** — replace `<li>` items with merged key context points.
5. **`#decisions div`** — append new decision items (each as a `<p>`), keep prior decisions.
6. **`#entries div`** — prepend a new `.entry` block at the top:
   ```html
   <div class="entry">
     <div class="entry-date">YYYY-MM-DD</div>
     <div class="entry-body"><p>Session summary goes here.</p></div>
   </div>
   ```
7. **`#notes div`** — merge with any new notes.

If a section had `.empty` placeholder content, replace it entirely rather than appending.

## Step 6 — Confirm

Tell the user:

> Thread **`<name>`** updated — `thread.html` saved.
> View it with `/thread:browse <name>` or load it next session with `/thread:load <name>`.
