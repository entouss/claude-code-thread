---
description: Open a thread's folder in the system file manager (Finder on macOS)
allowed-tools: Bash, AskUserQuestion
---

Open `~/.claude/_thread/<name>/` in the system file manager.

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
elif [ -n "$ACTIVE" ] && [ -d "$THREAD_ROOT/$ACTIVE" ]; then
  echo "RESOLVED=$ACTIVE"
else
  echo "RESOLVED=PROMPT"; echo "$THREADS"
fi
```

- `RESOLVED=<name>` — use it directly.
- `RESOLVED=AMBIGUOUS` + list — show `AskUserQuestion` with only the listed matches.
- `RESOLVED=PROMPT` + list — show `AskUserQuestion` with all threads.
- `RESOLVED=MISSING` — tell the user no match was found and suggest `/thread:list`. Stop.
- Empty `THREADS` — tell the user no threads exist and suggest `/thread:create`. Stop.

## Step 2 — Verify the folder exists

```bash
THREAD_DIR="${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread/<name>"
[ -d "$THREAD_DIR" ] && echo "OK" || echo "MISSING"
```

If missing, tell the user and suggest `/thread:create <name>`.

## Step 3 — Open in file manager

Detect the platform and open the folder:

- **macOS** (Platform: `darwin`):
  ```bash
  open "${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread/<name>"
  ```
- **Linux** (Platform: `linux`):
  ```bash
  xdg-open "${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread/<name>"
  ```
- **Windows** (Platform: `win32`, Shell: `bash`):
  ```bash
  explorer "$(cygpath -w "${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread/<name>")"
  ```
- **Windows** (Platform: `win32`, Shell: `powershell`/`pwsh`):
  ```powershell
  explorer (Join-Path ($env:CLAUDE_CONFIG_DIR ?? "$HOME\.claude") "_thread\<name>")
  ```

## Step 4 — Confirm

Tell the user:

> Opening **`<name>`** in Finder.
> Path: `~/.claude/_thread/<name>/`
