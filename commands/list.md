---
description: List all available threads
allowed-tools: Bash
---

List all threads stored in `~/.claude/_thread/`.

## Step 1 — Find threads

```bash
THREAD_ROOT="${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread"
if [ ! -d "$THREAD_ROOT" ]; then
  echo "NO_THREADS_DIR"
else
  for dir in "$THREAD_ROOT"/*/; do
    [ -d "$dir" ] || continue
    name=$(basename "$dir")
    html="$dir/thread.html"
    if [ -f "$html" ]; then
      updated=$(stat -f "%Sm" -t "%Y-%m-%d %H:%M" "$html" 2>/dev/null || stat -c "%y" "$html" 2>/dev/null | cut -c1-16)
      size=$(wc -c < "$html" | tr -d ' ')
      extras=$(find "$dir" -type f -not -name "thread.html" | wc -l | tr -d ' ')
      echo "THREAD|$name|$updated|$size|$extras"
    else
      echo "THREAD_NO_HTML|$name"
    fi
  done
fi
```

## Step 2 — Display

If no threads directory or no results, tell the user no threads exist yet and suggest `/thread:create`.

Otherwise present a clean table:

```
Threads (n total)
─────────────────────────────────────────────────────
  NAME            LAST SAVED          FILES   SIZE
  my-project      2026-05-14 09:31    1       4.2 KB
  research        2026-05-10 17:05    3       8.1 KB
─────────────────────────────────────────────────────
Use /thread:load <name> to load a thread into context.
```

Count the `extras` column (non-thread.html files) and add 1 for thread.html to get total FILES. Format `size` in human-readable KB (divide bytes by 1024, one decimal place).
