# claude-code-thread

🧵

A Claude Code plugin for long-term memory across sessions. Store context, decisions, and notes in named **threads** that persist in `~/.claude/_thread/`.

## Commands

| Command | Description |
|---|---|
| `/thread:create [name]` | Create a new thread |
| `/thread:list` | List all threads |
| `/thread:load [name]` | Load a thread into context and summarize it |
| `/thread:save [name]` | Save current session context into a thread |
| `/thread:browse [name]` | Open `thread.html` in the browser |
| `/thread:delete [name]` | Delete a thread |

## Storage

Each thread lives at `~/.claude/_thread/<name>/`:

```
~/.claude/_thread/
  my-project/
    thread.html      ← HTML memory store (human-readable, dark-mode)
    notes.md         ← any extra files you add manually
```

`thread.html` is a styled dark-mode HTML file. Claude decides what to store and how to organize it — no fixed sections. Open it in any browser via `/thread:browse`.

## Workflow

```
# Start a new project thread
/thread:create my-project

# … work with Claude …

# Save what you learned to the thread
/thread:save my-project

# Next session — pick up where you left off
/thread:load my-project

# Review it visually
/thread:browse my-project
```

## Installation

```
/plugin marketplace add entouss/claude-code-thread
/plugin install thread@claude-code-thread
```
