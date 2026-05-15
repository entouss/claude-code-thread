---
description: Create a new thread for long-term memory storage
allowed-tools: Bash, AskUserQuestion
---

Create a new named thread at `~/.claude/_thread/<name>/`.

## Step 1 — Resolve thread name

If the user passed a name as an argument to this command, use it directly.
Otherwise ask:

```
AskUserQuestion:
  question: "What should this thread be named?"
  header: "Thread name"
  options:
    - label: "project"       description: "General project thread"
    - label: "research"      description: "Research & notes thread"
    - label: "decisions"     description: "Architecture & decision log"
    - label: "Other"         description: "I'll type a custom name"
  multiSelect: false
```

Validate the name: only letters, digits, hyphens, and underscores allowed. No spaces. If invalid, tell the user and stop.

## Step 2 — Check for conflicts

```bash
THREAD_DIR="${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread/<name>"
[ -d "$THREAD_DIR" ] && echo "EXISTS" || echo "NEW"
```

If the thread already exists, tell the user and ask whether to open it instead (`/thread:load <name>`). Stop unless they confirm overwrite.

## Step 3 — Create directory and thread.html

```bash
mkdir -p "$THREAD_DIR"
```

Then write `$THREAD_DIR/thread.html` with this exact content (substituting `<name>` and today's ISO date):

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Thread: <name></title>
  <link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><text y='.9em' font-size='90'>🧵</text></svg>">
  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      background: #0f1117;
      color: #e2e8f0;
      line-height: 1.6;
      padding: 2rem;
      max-width: 860px;
      margin: 0 auto;
    }
    header {
      border-bottom: 1px solid #2d3748;
      padding-bottom: 1rem;
      margin-bottom: 2rem;
    }
    h1 { font-size: 1.8rem; color: #f7fafc; }
    h1 span { color: #63b3ed; }
    .meta { font-size: 0.8rem; color: #718096; margin-top: 0.4rem; }
    section { margin-bottom: 2rem; }
    h2 {
      font-size: 1rem;
      text-transform: uppercase;
      letter-spacing: 0.08em;
      color: #63b3ed;
      border-bottom: 1px solid #2d3748;
      padding-bottom: 0.4rem;
      margin-bottom: 1rem;
    }
    .empty { color: #4a5568; font-style: italic; font-size: 0.9rem; }
    ul { padding-left: 1.2rem; }
    li { margin-bottom: 0.4rem; }
    .entry {
      background: #1a202c;
      border: 1px solid #2d3748;
      border-radius: 6px;
      padding: 0.8rem 1rem;
      margin-bottom: 0.8rem;
    }
    .entry-date { font-size: 0.75rem; color: #4a5568; margin-bottom: 0.3rem; }
    .entry-body { font-size: 0.9rem; }
    p { margin-bottom: 0.6rem; }
  </style>
</head>
<body>
  <header>
    <h1>Thread: <span><name></span></h1>
    <div class="meta">
      Created: <time id="created"><created-date></time>
      &nbsp;·&nbsp;
      Last saved: <time id="updated"><created-date></time>
    </div>
  </header>
</body>
</html>
```

Use the Bash tool to write this file (use a heredoc or printf). Substitute `<name>` with the actual thread name and `<created-date>` with today's date in ISO 8601 format (YYYY-MM-DD). The logo and favicon are embedded as base64 data URIs — no external files needed.

## Step 4 — Set active thread

Write the new thread name to `~/.claude/_thread/.active` so other commands treat it as the current thread:

```bash
echo "<name>" > "${CLAUDE_CONFIG_DIR:-$HOME/.claude}/_thread/.active"
```

## Step 5 — Confirm

Tell the user:

> Thread **`<name>`** created at `~/.claude/_thread/<name>/`.
>
> Load it in any future session with `/thread:load <name>`.
> To save context into this thread, use `/thread:save <name>`.
> To view it in a browser, use `/thread:browse <name>`.
