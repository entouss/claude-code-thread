---
description: Open a thread's thread.html in the default browser
allowed-tools: Bash, AskUserQuestion
---

Open `~/.claude/_thread/<name>/thread.html` in the system's default browser.

## Step 1 — Resolve thread name

If a name was passed as an argument, use it. Otherwise:

1. List available threads:
   ```bash
   ls -1 "${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread" 2>/dev/null || echo "NONE"
   ```
2. If none exist, tell the user to create one with `/thread:create` and stop.
3. Otherwise ask the user which thread to browse (use AskUserQuestion with thread names as options).

## Step 2 — Verify file exists

```bash
THREAD_FILE="${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread/<name>/thread.html"
[ -f "$THREAD_FILE" ] && echo "OK" || echo "MISSING"
```

If missing, tell the user and suggest `/thread:create <name>`.

## Step 3 — Open in browser

Detect the platform and open the file:

- **macOS** (Platform: `darwin`):
  ```bash
  open "$THREAD_FILE"
  ```
- **Linux** (Platform: `linux`):
  ```bash
  xdg-open "$THREAD_FILE"
  ```
- **Windows** (Platform: `win32`, Shell: `bash`):
  ```bash
  start "$THREAD_FILE"
  ```
- **Windows** (Platform: `win32`, Shell: `powershell`/`pwsh`):
  ```powershell
  Start-Process "$THREAD_FILE"
  ```

## Step 4 — Confirm

Tell the user:

> Opening **`<name>`** in your browser.
> Path: `~/.claude/_thread/<name>/thread.html`
