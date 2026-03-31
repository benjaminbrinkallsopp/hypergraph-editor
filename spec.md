# Hypergraph Editor — Spec

A browser-based interactive network editor where **arcs can connect to other arcs**, not just nodes. This makes it a hypergraph editor — standard graph tools only support node-to-node edges.

## Core concept

The editor has two element types:

- **Nodes** — labelled circles, freely positioned on a canvas
- **Arcs** — directed curved edges whose source and target can each be either a node or another arc

An arc that connects to another arc attaches at that arc's **midpoint** — a visible, clickable dot rendered on every arc's curve.

## Architecture

Single self-contained HTML file. No external dependencies. All rendering is done on a `<canvas>` element for full control over hit detection and drawing. State is held in plain arrays (`nodes[]`, `arcs[]`) with no framework.

## Features

### Canvas rendering

- HiDPI-aware canvas that scales with `devicePixelRatio`
- Responsive — fills available space and re-renders on window resize
- Subtle dark grid background

### Nodes

- Rendered as filled circles (radius 22px) with a border and centered label
- Auto-labelled alphabetically: A, B, C, ... Z, N27, N28, ...
- Draggable — click and drag to reposition
- Created by double-clicking empty canvas space, or via the "+ Node" sidebar button

### Arcs

- Rendered as **quadratic bezier curves** with a perpendicular offset (45px), so every arc has a visible curve and a distinct midpoint
- Directed — each arc has an **arrowhead** drawn near the target end, oriented along the curve's tangent
- Each arc displays an **orange midpoint dot** — this is the connection point that other arcs can attach to
- Four connection types supported:
  - Node → Node
  - Node → Arc
  - Arc → Node
  - Arc → Arc
- Created by selecting a source element, then **Shift+clicking** a target element
- Self-loops and duplicate arcs are prevented
- Recursive position resolution with cycle detection — if arcs form a reference cycle, a fallback position is used instead of infinite recursion

### Color coding

- **Cyan** (`#00d4ff`) — direct arcs (node → node)
- **Pink** (`#ff6b9d`) — meta-arcs (any arc where at least one endpoint is another arc)
- **Orange** (`#ffa94d`) — arc midpoint dots

### Selection and hover

- Click any node or arc to select it — shown with a **yellow glow** effect
- Hover over nodes or arcs to see a **white highlight**
- Cursor changes to pointer on hover, crosshair when Shift is held over a valid target
- Holding Shift with a selected element shows a **dashed preview line** from the selection to the cursor

### Deletion

- **Delete** or **Backspace** removes the selected element
- Deleting a node cascades: all arcs connected to that node are also removed
- Deleting an arc cascades: all arcs that reference it (as source or target) are also removed
- Orphaned arc cleanup runs iteratively until no dangling references remain

### Hit detection

- Nodes: point-in-circle test with a small tolerance
- Arc midpoints: point-in-circle test on the midpoint dot (priority over curve hit)
- Arc curves: distance-to-bezier computed by sampling 50 points along the curve, with a 10px threshold
- Hit priority: nodes (topmost) → arc midpoints → arc curves

### Sidebar panel

- **Actions**: "+ Node" button and "Clear All" button
- **Selected**: info panel showing details of the currently selected node or arc (type, label/ID, connections, whether it's a meta-arc)
- **Legend**: color key for all visual elements
- **Controls**: keyboard/mouse shortcut reference
- **Stats**: live count of nodes, arcs, and meta-arcs

### Prompt output

- Bottom bar generates a **natural-language description** of the current graph
- Lists all nodes, direct arcs, and meta-arcs in readable form
- "Copy" button copies the description to clipboard with brief feedback

### Demo content

- Loads with a starter graph: 3 nodes (A, B, C), 2 direct arcs (A→B, B→C), and 1 meta-arc (C→ the A→B arc)
- Demonstrates the core feature immediately without requiring user interaction

## Data model

```
Node: { id: number, x: number, y: number, label: string }

Arc:  { id: number,
        sourceType: 'node' | 'arc', sourceId: number,
        targetType: 'node' | 'arc', targetId: number }
```

Arc endpoints are polymorphic — `sourceType`/`targetType` determine whether the ID refers to a node or another arc. Position resolution is recursive: an arc's visual position depends on its endpoints, which may themselves be arcs whose positions depend on their endpoints.
