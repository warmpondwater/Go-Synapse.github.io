# Bi-Directional Canvas Driving: Supercharging AI Agents via MCP & SQLite CTEs

**Category**: Artificial Intelligence / Model Context Protocol (MCP) / AI Agent Tooling / Software Architecture  
**Target Keywords**: Model Context Protocol, MCP server Go-Synapse, AI agent codebase navigation, SQLite CTE 2ms query, bi-directional canvas driving  
**Reading Time**: 8 Minutes  

---

## Introduction: Why AI Assistants Need More Than Text Prompting

Standard AI coding assistants (like Cursor, Claude Code, or AntiGravity) suffer from two fundamental bottlenecks when analyzing complex codebases:

1. **Context Window Exhaustion**: Feeding raw source files into LLM context windows causes high token bills, slow latency, and token truncation.
2. **Lack of Visual & Structural Interaction**: AI agents cannot "see" the visual canvas or focus the human developer's attention on specific architectural components.

**Go-Synapse** solves both bottlenecks by implementing a bi-directional **Model Context Protocol (MCP) Server (`mcp/server.go`)** backed by a **2ms Relational SQLite Data Surface (`db/analysis.go`)**.

---

## 1. 2ms Relational SQL Intelligence vs. Raw File Uploads

Instead of dumping thousands of lines of text into an LLM context window, Go-Synapse exposes a high-performance SQLite 3.x database (`synapse.db`).

### Recursive Common Table Expressions (CTEs)
Go-Synapse pre-compiles recursive CTE queries (`db/analysis.go`) to answer complex architectural questions in 2 milliseconds:

```sql
-- Example Recursive CTE: 2ms Upstream Blast Radius Query
WITH RECURSIVE CallTree AS (
    SELECT source, target, 1 AS depth
    FROM edges WHERE target = 'node_Go_auth_go_ValidateToken'
    UNION ALL
    SELECT e.source, e.target, ct.depth + 1
    FROM edges e
    INNER JOIN CallTree ct ON e.target = ct.source
    WHERE ct.depth < 10
)
SELECT DISTINCT source FROM CallTree;
```

---

## 2. Model Context Protocol (MCP) Tools (`mcp/server.go`)

Go-Synapse implements a native JSON-RPC 2.0 MCP server over `stdio`, giving AI agents a rich set of backend and visual tools:

| MCP Tool Name | What It Does | Why It Matters |
| :--- | :--- | :--- |
| **`execute_sql`** | Runs 2ms relational CTE queries against `synapse.db`. | Allows AI agents to query exact dependencies without reading raw files. |
| **`focus_dashboard_node`** | Pans and zooms the browser camera to specific 2D coordinates. | Directs the human developer's attention to exact code locations. |
| **`annotate_node`** | Paints specific nodes with badge (`INJECTION_RISK`) and hex color. | Visualizes security risks directly on the 2D canvas during model audits. |
| **`render_ui_template`** | Projects Mermaid diagrams or markdown into the Telemetry Drawer. | Gives AI agents a visual canvas to present architecture refactoring plans. |
| **`open_file_in_editor`** | Opens target source files at exact line numbers in VS Code. | Seamlessly transitions the developer from visual exploration to coding. |
| **`freeze`** | Locks/unlocks database state (`lock: true\|false`). | Guarantees deterministic database benchmarks during AI evaluation runs. |

---

## 3. Bi-Directional Canvas Driving Workflow

Through MCP, human developers and AI agents collaborate on a shared visual canvas:

```
┌───────────────────────────┐                            ┌───────────────────────────┐
│     AI AGENT (AntiGravity)│ ─── MCP JSON-RPC stdio ──► │     GO-SYNAPSE MCP SERVER │
│  Runs SQL queries         │ ◄── 2ms Relational Results │  (mcp/server.go)          │
└─────────────┬─────────────┘                            └─────────────┬─────────────┘
              │                                                        │
              │  focus_dashboard_node & annotate_node                  │ WebSocket (ws://)
              └────────────────────────────────────────────────────────┴───────► 🌐 Cytoscape 2D UI
```

### The Collaboration Sequence:
1. **Developer Request**: *"Check if refactoring `auth.go` breaks any downstream payment handlers."*
2. **AI SQL Query**: The AI agent calls `execute_sql` to run a 2ms CTE query against `synapse.db`.
3. **Visual Focus**: The agent calls `focus_dashboard_node(node_id="auth_Validate")`, automatically panning the developer's browser view to the exact node coordinates.
4. **Telemetry Report**: The agent projects a Mermaid dependency diagram into the **Agent Telemetry Drawer** using `render_ui_template`.

---

## Conclusion

Bi-directional canvas driving turns AI assistants from passive chat boxes into active architectural co-pilots.

Explore the complete master article directory in [Story Vault](index.html#articles).
