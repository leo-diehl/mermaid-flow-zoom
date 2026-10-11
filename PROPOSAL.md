# Proposal: focused views of a shared Mermaid diagram

**Status:** parked (2026-10-11). Captured for later, not scheduled.

## Idea

A note keeps one main, full diagram. Subsections of a page (or other notes) embed a *view* of it: the same diagram, opened at a preset zoom and position that focuses on the part the subheading talks about.

````markdown
```mermaid-view
source: [[System Architecture#^main-flow]]
zoom: 2.1
center: 340, 120
locked: true
```
````

## Behaviour

- A view gets the existing toolbar (zoom out, zoom in, reset, fullscreen) plus a **lock** button.
- **Locked:** pan and zoom are disabled, so the mouse wheel scrolls the page instead of zooming.
- **Unlocked:** pan and zoom work as usual.
- **Reset** returns to the view's preset, not to fit-to-frame.
- **Re-locking** after moving saves the current zoom and position into the block as the new preset.

## Feasibility

Feasible with Obsidian's plugin API:

- `registerMarkdownCodeBlockProcessor("mermaid-view", ...)` renders the block.
- Resolve `source`, read the note, find that Mermaid block, render it with Obsidian's bundled Mermaid (`loadMermaid()`), and wrap it in the existing zoom container.
- On re-lock, `ctx.getSectionInfo(el)` gives the block's line range, and `app.vault.process()` rewrites its `zoom` and `center`.

## Hard parts

1. **Viewport-independent coordinates.** The plugin keeps pan and zoom as pixel offsets in the viewport (`x`, `y`, `scale`). Presets must instead be stored in diagram (SVG viewBox) units, as a center point plus a visible extent. Otherwise the same preset frames a different area at another pane width, on mobile, or in fullscreen. This is a redesign of how position is handled.
2. **Presets go stale when the main diagram changes.** Mermaid re-lays out the whole diagram when a node is added, so every saved `center` can silently point at the wrong area.
3. **Re-lock edits the note.** That re-renders the block and goes through iCloud sync. It should save only on re-lock, never on every pan.

## Alternatives to weigh first

- **Focus by node IDs instead of coordinates:** `focus: AuthService, TokenStore` fits the view to the bounding box of those nodes. It survives edits to the main diagram (solves hard part 2), but works best for flowcharts.
- **Split the diagram** into sub-diagrams per section, or use Mermaid `subgraph`s.
- **An existing plugin** may already do this. Not yet searched.

## Open questions

- **The problem:** what concrete notes would use this, and how often does the problem come up?
- **Coordinates or node IDs**, or both?
- **Where it lives:** this fork (which means a large drift from upstream, so every upstream update needs a manual merge and a security review), a proposal to upstream, or a separate plugin that depends on this one?
- **Priority** relative to other work.

## Done means

- Views render in Reading view and Live Preview, on desktop and mobile.
- The same preset frames the same region at any pane width.
- Lock, unlock, reset and re-lock behave as described.
- When the source block is missing, the view shows a clear error.
- It's documented in the README.
