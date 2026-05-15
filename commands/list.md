---
description: List and select a thread to act on
allowed-tools: Bash, AskUserQuestion
---

Show all threads as an interactive picker and perform an action on the selected one.

## Step 1 — Gather threads

```bash
THREAD_ROOT="${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread"
ACTIVE=$(cat "$THREAD_ROOT/.active" 2>/dev/null || echo "")
for dir in "$THREAD_ROOT"/*/; do
  [ -d "$dir" ] || continue
  name=$(basename "$dir")
  [ "$name" = ".active" ] && continue
  updated=$(stat -f "%Sm" -t "%Y-%m-%d" "$dir/thread.html" 2>/dev/null \
            || stat -c "%y" "$dir/thread.html" 2>/dev/null | cut -c1-10 \
            || echo "unknown")
  active_marker=""
  [ "$name" = "$ACTIVE" ] && active_marker=" (active)"
  echo "$name|$updated|$active_marker"
done
```

If the `_thread` directory doesn't exist or no threads are found, tell the user no threads exist yet and suggest `/thread:create`. Stop.

## Step 2 — Present picker

Build one option per thread. Label: thread name (append ` ·  active` if it matches `.active`). Description: `Last saved: <date>`.

Add a final option: `Cancel` — description: `Do nothing`.

```
AskUserQuestion:
  question: "Select a thread:"
  header: "Threads"
  options: [one per thread, plus Cancel]
  multiSelect: false
```

## Step 3 — Act on selection

If the user picks **Cancel** (or dismisses), stop silently.

Otherwise ask what to do with the selected thread:

```
AskUserQuestion:
  question: "What do you want to do with '<name>'?"
  header: "Action"
  options:
    - label: "Load"    description: "Read all files into context and summarize"
    - label: "Save"    description: "Write current session context into this thread"
    - label: "Browse"  description: "Open thread.html in the browser"
    - label: "Folder"  description: "Open the thread folder in Finder"
  multiSelect: false
```

Execute the chosen action exactly as the corresponding command would:
- **Load** → follow all steps in `load.md`, using `<name>` as the resolved thread
- **Save** → follow all steps in `save.md`, using `<name>` as the resolved thread
- **Browse** → follow all steps in `browse.md`, using `<name>` as the resolved thread
- **Folder** → follow all steps in `folder.md`, using `<name>` as the resolved thread
