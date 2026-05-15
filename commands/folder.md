---
description: Open a thread's folder in the system file manager (Finder on macOS)
allowed-tools: Bash, AskUserQuestion
---

Open `~/.claude/_thread/<name>/` in the system file manager.

## Step 1 — Resolve thread name

If a name was passed as an argument, use it. Otherwise:

1. Check for an active thread:
   ```bash
   cat "${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread/.active" 2>/dev/null || echo "NONE"
   ```
2. If an active thread name is found and its directory exists, use it automatically without prompting.
3. Otherwise list available threads and ask the user which to open (use AskUserQuestion with thread names as options).
4. If no threads exist at all, tell the user to create one with `/thread:create` and stop.

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
