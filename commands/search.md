---
description: Search across all threads for a keyword or phrase
allowed-tools: Bash
---

Search all files in `~/.claude/_thread/` for a query string and display matching results grouped by thread.

## Step 1 — Resolve query

If a query was passed as an argument, use it directly. Otherwise tell the user to provide a search term, e.g. `/thread:search <query>`, and stop.

## Step 2 — Check threads exist

```bash
THREAD_ROOT="${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread"
[ -d "$THREAD_ROOT" ] && ls -1 "$THREAD_ROOT" || echo "NONE"
```

If no threads exist, tell the user and stop.

## Step 3 — Search

```bash
grep -ril "<query>" "${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread" 2>/dev/null
```

For each matching file, get context lines:

```bash
grep -in "<query>" "<file>" 2>/dev/null
```

Use case-insensitive (`-i`) matching. For `thread.html` files, strip HTML tags from match lines before displaying so the output is readable:

```bash
grep -in "<query>" "<file>" | sed 's/<[^>]*>//g' | sed 's/^[[:space:]]*//'
```

## Step 4 — Display results

Group results by thread name. Format:

```
Search: "<query>" — N match(es) across M thread(s)
─────────────────────────────────────────────────
  thread: my-project  (thread.html)
    line 42: This decision was made because of the API rate limits
    line 87: rate limits also affect the background sync job

  thread: research  (notes.md)
    line 3: Papers on rate limiting strategies
─────────────────────────────────────────────────
Use /thread:load <name> to load a matching thread.
```

If no matches found:

```
No matches for "<query>" across N thread(s).
```

Limit displayed lines to 5 per file to keep output scannable. If a file has more than 5 matches, show a count: `… and N more match(es)`.
