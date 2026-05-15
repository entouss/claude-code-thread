---
name: text-to-diagram
description: This skill should be used when the user asks to add a diagram, create a diagram, or visualize something inside a thread. It provides exact patterns for embedding Mermaid diagrams in thread.html files that open via file:// URLs, based on lessons learned from debugging rendering failures.
version: 1.0.0
---

# Text-to-Diagram for Threads

Guidance for generating Mermaid diagrams and embedding them correctly in `thread.html` files.

## When This Skill Applies

- User asks to "add a diagram", "create a diagram", or "visualize X" in a thread
- User asks to update or replace an existing diagram in a thread
- Any time Mermaid needs to be embedded in a `thread.html`

## Mermaid Setup — Required Patterns

`thread.html` files are opened via `file://` URLs. CDN ES module imports fail in this context. The only reliable pattern is:

### Script placement

Scripts must go at the **end of `<body>`**, not in `<head>`:

```html
  <script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
  <script>
    mermaid.initialize({ theme: 'dark' });
    mermaid.run({ querySelector: '.mermaid' });
  </script>
</body>
```

**Why v10:** Mermaid v11 dropped UMD — `mermaid.min.js` from v11 is an ESM bundle that does not expose a global `mermaid`. v10 ships a proper UMD bundle.

**Why end of body:** When scripts are in `<head>`, `mermaid.run()` fires before the `.mermaid` element is in the DOM.

**Why `mermaid.run()` explicitly:** `startOnLoad: true` relies on the `DOMContentLoaded` event which may already have fired by the time the script executes from a local file.

### Diagram element

Use `<div class="mermaid">`, not `<pre class="mermaid">` (v10 scans for `div.mermaid`):

```html
<div class="mermaid">
flowchart LR
    ...
</div>
```

### Check before adding scripts

Before injecting scripts, check if `mermaid.min.js` is already referenced in `thread.html`. Only add the script block if it's absent.

## Source viewer — Required UI Pattern

Always pair the rendered diagram with a collapsible source block and copy button:

```html
<div class="mermaid" id="diagram-NAME">
DIAGRAM SOURCE HERE
</div>
<details style="margin-top:1rem">
  <summary style="cursor:pointer;color:#718096;font-size:0.85rem;user-select:none">Show source</summary>
  <div style="position:relative;margin-top:0.5rem">
    <button onclick="navigator.clipboard.writeText(document.getElementById('diagram-src-NAME').textContent).then(()=>{this.textContent='Copied!';setTimeout(()=>this.textContent='Copy',1500)})"
      style="position:absolute;top:0.5rem;right:0.5rem;background:#2d3748;color:#e2e8f0;border:none;border-radius:4px;padding:0.25rem 0.6rem;font-size:0.75rem;cursor:pointer">Copy</button>
    <pre id="diagram-src-NAME" style="background:#1a202c;border:1px solid #2d3748;border-radius:6px;padding:1rem;font-size:0.8rem;color:#a0aec0;overflow-x:auto;white-space:pre">DIAGRAM SOURCE HERE</pre>
  </div>
</details>
```

Replace `NAME` with a short unique slug for the diagram. The diagram source appears twice: once in the `<div class="mermaid">` (rendered) and once in the `<pre>` (copyable plain text). Keep them in sync.

## Diagram Generation

Generate a Mermaid `flowchart LR` diagram from the user's description. Prefer:
- `flowchart LR` for workflows and data flows
- `flowchart TD` for hierarchies
- `sequenceDiagram` for interactions between components

Avoid angle brackets (`<`, `>`) in node labels — they confuse the HTML parser inside the div. Use plain text or quotes.

## Workflow

1. Resolve the active thread (read `~/.claude/_thread/.active`)
2. Read `thread.html` to understand existing content and check if Mermaid scripts are already present
3. Generate the Mermaid diagram source from the user's description
4. Inject the diagram section and (if not already present) the Mermaid script block using the Edit tool
5. Confirm and tell the user to refresh the browser
