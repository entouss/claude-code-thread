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
    MATCHES=$(echo "$THREADS" | grep -i "$REQUESTED" 2>/dev/null || true)
    COUNT=$(echo "$MATCHES" | grep -c . 2>/dev/null || echo 0)
    if [ "$COUNT" -eq 1 ]; then
      echo "RESOLVED=$MATCHES"
      find "$THREAD_ROOT/$MATCHES" -type f | sort | sed "s|$THREAD_ROOT/$MATCHES/||"
    elif [ "$COUNT" -gt 1 ]; then
      echo "RESOLVED=AMBIGUOUS"
      echo "$MATCHES"
    else
      echo "RESOLVED=MISSING"
    fi
  fi
else
  echo "RESOLVED=PROMPT"
fi
```

Parse the output:
- If `RESOLVED=<name>`: exact or single partial match confirmed — remaining lines are file paths, proceed to Step 3.
- If `RESOLVED=AMBIGUOUS`: multiple partial matches — show `AskUserQuestion` with only the listed matches, then run minimal verification on the chosen name.
- If `RESOLVED=PROMPT`: no argument — show `AskUserQuestion` with the full `THREADS` list, then run minimal verification.
- If `RESOLVED=MISSING`: no match found — tell the user and suggest `/thread:list`. Stop.
- If `THREADS` is empty: tell the user no threads exist yet and suggest `/thread:create`. Stop.

Minimal verification (run after picker):
```bash
THREAD_ROOT="${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread"
TARGET="<chosen>"
[ -d "$THREAD_ROOT/$TARGET" ] && find "$THREAD_ROOT/$TARGET" -type f | sort | sed "s|$THREAD_ROOT/$TARGET/||" || echo "MISSING"
```

## Step 2 — AskUserQuestion (only if RESOLVED=PROMPT or AMBIGUOUS)

```
AskUserQuestion:
  question: "Which thread do you want to load?"
  header: "Thread"
  options: [matched threads if AMBIGUOUS, all threads if PROMPT]
  multiSelect: false
```

Then run the minimal verification script above with the chosen name.

## Step 3 — Read all files

Using the file list from Step 1, read each file with the Read tool. For `thread.html`, extract the human-readable content from the body. For other files, read as-is.

## Step 4 — Finalise in one script

Update the timestamp and set the active thread in a single call:

```bash
THREAD_ROOT="${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread"
TARGET="<name>"
HTML="$THREAD_ROOT/$TARGET/thread.html"
TODAY=$(date +%Y-%m-%d)

sed -i.bak "s|<time id=\"updated\">[^<]*</time>|<time id=\"updated\">$TODAY</time>|" "$HTML" \
  && rm -f "$HTML.bak"
echo "$TARGET" > "$THREAD_ROOT/.active"
echo "DONE"
```

## Step 5 — Output summary

Present the thread content to the user in a clear, natural way — following whatever structure is already in the thread. Don't impose a fixed format; let the thread's own organization guide the output. Note any other files alongside `thread.html`. End with:

```
All thread content is now in context.
Use /thread:save to write new context back to this thread.
```
