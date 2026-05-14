---
description: Load a thread into context and output a summary of its memory
allowed-tools: Bash, Read, AskUserQuestion
---

Load all files from `~/.claude/_thread/<name>/` into context and summarize the thread's memory.

## Step 1 — Resolve thread name

If a name was passed as an argument, use it. Otherwise:

1. List available threads:
   ```bash
   THREAD_ROOT="${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread"
   ls -1 "$THREAD_ROOT" 2>/dev/null || echo "NONE"
   ```
2. If none exist, tell the user to create one with `/thread:create` and stop.
3. If one or more exist, ask the user which to load:
   ```
   AskUserQuestion:
     question: "Which thread do you want to load?"
     header: "Thread"
     options: [one option per thread name found]
     multiSelect: false
   ```

## Step 2 — Verify the thread exists

```bash
THREAD_DIR="${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread/<name>"
[ -d "$THREAD_DIR" ] && echo "OK" || echo "MISSING"
```

If missing, tell the user and suggest `/thread:list`.

## Step 3 — Read all files

Find every file in the thread directory:

```bash
find "$THREAD_DIR" -type f | sort
```

Read each file using the Read tool. For `thread.html`, parse the HTML content to extract human-readable text from each `<section>`. For other files, read them as-is.

## Step 4 — Update last-loaded timestamp in thread.html

In `thread.html`, update the `<time id="updated">` element's text content to today's date (YYYY-MM-DD). Use the Edit tool to do this in-place — only change that one value.

## Step 5 — Output summary

Print a structured summary to the user:

```
─────────────────────────────────────────────
  Thread loaded: <name>
  Files read: N
─────────────────────────────────────────────

**Summary**
<content of #summary section, or "empty">

**Goals & Objectives**
<bulleted list from #goals, or "none">

**Key Context**
<bulleted list from #context, or "none">

**Decisions**
<content of #decisions, or "none">

**Session Entries** (most recent first)
<last 3 .entry divs, or "none">

**Notes**
<content of #notes, or "none">

**Other files**
<list any non-thread.html files with a one-line description>
─────────────────────────────────────────────
All thread content is now in context.
Use /thread:save <name> to write new context back to this thread.
```

If a section is empty (contains only the `.empty` placeholder text), show "none recorded" for that section rather than the placeholder text.
