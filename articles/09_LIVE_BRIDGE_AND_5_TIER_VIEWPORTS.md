# Eliminating Spaghetti Graphs: The Live WebSocket Bridge & 5-Tier Set Viewport Engine

**Category**: UI/UX Engineering / Cytoscape.js & Graph Visualization / Systems Architecture  
**Target Keywords**: Cytoscape.js visualizer, 5-tier viewport set theory, live websocket bridge, Hub Portal badges, Go-Synapse UI  
**Reading Time**: 7 Minutes  

---

## Introduction: The "Visual Spaghetti" Problem in Architecture Tools

Graph visualizers are infamous for turning large codebases into unreadable "spaghetti line tangles". When a codebase grows past 10,000 nodes, traditional UML tools and dependency visualizers overlap thousands of lines over a screen, rendering the graph useless.

**Go-Synapse** solves visual spaghetti through two specialized UI/UX innovations: a **5-Tier Compound Set Viewport System** and **Hub Portal Badges (📞)**, connected in real time via a **Live WebSocket Bridge**.

---

## 1. The Live WebSocket Communication Bridge (`bridge/server.go`)

Go-Synapse links its high-speed Go backend daemon directly to the browser visualizer (`http://127.0.0.1:8080`) over a secure WebSocket connection (`ws://127.0.0.1:8080/live`).

```
┌───────────────────────────┐   WebSocket (ws://127.0.0.1:8080/live)   ┌───────────────────────────┐
│     GO-SYNAPSE DAEMON     │ ───────────────────────────────────────► │   CYTOSCAPE 2D DASHBOARD  │
│  File Watcher (watcher/)  │ ◄─────────────────────────────────────── │   (public/index.html)     │
│  Incremental re-parses    │         Live Event JSON Patches          │   5-Tier Viewport & Drawer│
└───────────────────────────┘                                          └───────────────────────────┘
```

### Real-Time Event Protocols:
- **`GRAPH_DATA`**: On startup, pushes warm SQLite database graph patches directly to Cytoscape.js.
- **`FILE_SAVE_DELTA`**: The hot-reload watcher (`watcher/watcher.go`) listens for workspace file saves, triggering incremental re-parses in under 50ms and updating changed nodes while preserving user camera pan/zoom coordinates.
- **`NODE_HIGHLIGHT` & `RENDER_DASHBOARD`**: Exposes real-time visual control to AI agents via Model Context Protocol (MCP).

---

## 2. The 5-Tier Compound Set Viewport System (`public/index.html`)

To prevent visual clutter, Go-Synapse organizes source code into 5 distinct abstraction viewports based on set theory:

| Viewport Tier | Name | Visual Abstraction Level | What You See |
| :--- | :--- | :--- | :--- |
| **Tier 1** | **Domain Set** | Macro Architecture | System domain containers (e.g. `Auth`, `Payments`, `Database`). |
| **Tier 2** | **Package Set** | Folder Level | Package containers showing directory physical distributions. |
| **Tier 3** | **File Set** | File Boundaries | Source files (`.go`, `.py`, `.ts`, `.cpp`) populating packages. |
| **Tier 4** | **Clean Symbols** | Uncluttered Code | Active function and struct nodes without connection line clutter. |
| **Tier 5** | **Working Code** | Full Call Graph | Complete function call execution lines, channel dataflows, and taint paths. |

---

## 3. Spaghetti Prevention: Hub Portal Badges (📞)

High-indegree utility functions (such as logging functions, database connection handles, or error reporters) are called by hundreds of files. Rendering lines for these functions creates visual spaghetti.

### How Hub Portal Badges Work:
- When a node's in-degree exceeds a configurable threshold (e.g. `in_degree > 3`), Go-Synapse automatically **collapses the connecting lines**.
- The utility node is converted into a compact **Hub Portal Badge (📞)** in the 2D canvas.
- Clicking a portal badge opens the **Code Inspector Drawer**, listing callers in a clean side panel without drawing hundreds of crossing lines across the visual viewport.

---

## 4. Visual Taint Tracing & 5% Opacity Isolation

When inspecting security vulnerabilities (such as SQL injection risks or prompt injections):
- Go-Synapse runs a static taint analysis pass.
- It dims **95% of the codebase to 5% opacity**.
- The exact untrusted execution path — from HTTP request input to dangerous database sink — lights up in **glowing red (`#ff1744`)**.

---

## Conclusion

With the Live WebSocket Bridge, 5-Tier Set Viewports, and Hub Portal Badges, Go-Synapse delivers a clean, responsive 2D visual canvas that scales effortlessly to repositories with over 100,000 lines of code.

Explore the complete master article directory in [Story Vault](index.html#articles).
