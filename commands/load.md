---
description: Load a thread into context and output a summary of its memory
allowed-tools: Bash, Read, AskUserQuestion
---

Load all files from `~/.claude/_thread/<name>/` into context and summarize the thread's memory.

## Step 1 — Resolve and verify in one script

Run a single script that lists available threads and, if a name was passed as an argument, verifies it exists and collects its files immediately. If a name was passed as an argument, set `REQUESTED=<name>` in the script; otherwise leave it empty.

```bash
THREAD_ROOT="${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread"
REQUESTED=""  # replace with argument if provided

# Collect thread names (exclude hidden files/dirs)
THREADS=$(find "$THREAD_ROOT" -mindepth 1 -maxdepth 1 -type d 2>/dev/null \
  | xargs -I{} basename {} | grep -v '^\.' | sort)

echo "THREADS=$THREADS"

if [ -n "$REQUESTED" ]; then
  if [ -d "$THREAD_ROOT/$REQUESTED" ]; then
    echo "RESOLVED=$REQUESTED"
    find "$THREAD_ROOT/$REQUESTED" -type f | sort | sed "s|$THREAD_ROOT/$REQUESTED/||"
  else
    echo "RESOLVED=MISSING"
  fi
else
  echo "RESOLVED=PROMPT"
fi
```

Parse the output:
- If `RESOLVED=<name>`: thread is confirmed, the remaining lines are the relative file paths — proceed to Step 3.
- If `RESOLVED=PROMPT`: show `AskUserQuestion` with the `THREADS` list, then run a minimal verification:
  ```bash
  THREAD_ROOT="${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread"
  TARGET="<chosen>"
  [ -d "$THREAD_ROOT/$TARGET" ] && find "$THREAD_ROOT/$TARGET" -type f | sort | sed "s|$THREAD_ROOT/$TARGET/||" || echo "MISSING"
  ```
- If `RESOLVED=MISSING`: tell the user the thread was not found and suggest `/thread:list`. Stop.
- If `THREADS` is empty: tell the user no threads exist yet and suggest `/thread:create`. Stop.

## Step 2 — AskUserQuestion (only if RESOLVED=PROMPT)

```
AskUserQuestion:
  question: "Which thread do you want to load?"
  header: "Thread"
  options: [one option per name in THREADS]
  multiSelect: false
```

Then run the minimal verification script above with the chosen name.

## Step 3 — Read all files

Using the file list from Step 1, read each file with the Read tool. For `thread.html`, extract human-readable text from each `<section>`. For other files, read as-is.

## Step 4 — Finalise in one script

Update the timestamp and set the active thread in a single call:

```bash
THREAD_ROOT="${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread"
TARGET="<name>"
HTML="$THREAD_ROOT/$TARGET/thread.html"
TODAY=$(date +%Y-%m-%d)

# Update <time id="updated"> and write .active atomically
sed -i.bak "s|<time id=\"updated\">[^<]*</time>|<time id=\"updated\">$TODAY</time>|" "$HTML" \
  && rm -f "$HTML.bak"
echo "$TARGET" > "$THREAD_ROOT/.active"
echo "DONE"
```

## Step 5 — Output summary

Print a structured summary to the user:

```
─────────────────────────────────────────────
  Thread loaded: <name>
  Files read: N
─────────────────────────────────────────────

**Summary**
<content of #summary section, or "none recorded">

**Goals & Objectives**
<bulleted list from #goals, or "none recorded">

**Key Context**
<bulleted list from #context, or "none recorded">

**Decisions**
<content of #decisions, or "none recorded">

**Session Entries** (most recent 3)
<last 3 .entry divs, or "none recorded">

**Notes**
<content of #notes, or "none recorded">

**Other files**
<list any non-thread.html files with a one-line description>
─────────────────────────────────────────────
All thread content is now in context.
Use /thread:save to write new context back to this thread.
```

If a section contains only the `.empty` placeholder text, show "none recorded" instead.
