# claude-code-thread

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
    thread.html      ← structured HTML memory store (human-readable)
    notes.md         ← any extra files you add manually
```

`thread.html` is a styled dark-mode HTML file with sections for Summary, Goals, Key Context, Decisions, Session Entries, and Notes. Open it in any browser via `/thread:browse`.

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

### Local (development)

The plugin is registered by pointing Claude Code's plugin registry at this directory. See `~/.claude/plugins/installed_plugins.json`.

### From GitHub (future)

```
/plugin install thread@<marketplace>
```
