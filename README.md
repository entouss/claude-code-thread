# claude-code-thread

🧵

A Claude Code plugin for long-term memory across sessions. Store context, decisions, and notes in named **threads** that persist in `~/.claude/_thread/`.

## Commands

| Command | Description |
|---|---|
| `/thread:create [name]` | Create a new thread |
| `/thread:list` | Interactive picker — select a thread and act on it |
| `/thread:load [name]` | Load a thread into context and summarize it |
| `/thread:save [name]` | Save current session context into a thread |
| `/thread:browse [name]` | Open `thread.html` in the browser |
| `/thread:folder [name]` | Open the thread folder in Finder |
| `/thread:search <query>` | Search across all threads |
| `/thread:rename <old> <new>` | Rename a thread |
| `/thread:delete [name]` | Delete a thread |

All commands accept a partial thread name — e.g. `/thread:load proj` will match `my-project`. If multiple threads match, a picker is shown.

## Storage

Each thread lives at `~/.claude/_thread/<name>/`:

```
~/.claude/_thread/
  my-project/
    thread.html      ← HTML memory store (human-readable, dark-mode)
    notes.md         ← any extra files you add manually
```

`thread.html` is a dark-mode HTML file with a 🧵 favicon. Claude decides what to store and how to organize it — no fixed sections. Open it in any browser via `/thread:browse`.

The last loaded thread is remembered as the active thread — `browse`, `save`, and `folder` will use it automatically when no name is given.

## Workflow

```
# Start a new project thread
/thread:create my-project

# … work with Claude …

# Save what you learned to the thread
/thread:save

# Next session — pick up where you left off
/thread:load my-project

# Review it visually
/thread:browse

# Open the folder in Finder
/thread:folder
```

## Installation

```
/plugin marketplace add entouss/claude-code-thread
/plugin install thread@claude-code-thread
```
