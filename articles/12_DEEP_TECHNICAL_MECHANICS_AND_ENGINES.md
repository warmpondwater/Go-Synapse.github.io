# Inside the Engine: The 7 Core Technical Mechanics Powering Go-Synapse's AST Data Surface

**Category**: Systems Engineering / Compiler Design / Static Analysis / Software Architecture  
**Target Keywords**: Go-Synapse architecture, AST data surface, static taint tracing, LSP semantic engine, Go SSA parser, compilerless enterprise packaging  
**Reading Time**: 10 Minutes  

---

## Introduction: Engineering a Zero-Latency AST Data Surface

Most software visualization tools operate as surface-level wrappers around basic string searches or simple regex parsers. They struggle with large repositories, fail to resolve true symbol definitions across file boundaries, and lack the performance required to serve real-time AI agents.

**Go-Synapse** was engineered from the ground up to solve these technical limitations. It combines 7 specialized backend engines into a unified, zero-latency AST data surface capable of parsing 1,000,000+ nodes into relational SQLite tables in milliseconds.

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        GO-SYNAPSE 7-ENGINE ARCHITECTURE                                │
├─────────────────┬─────────────────┬──────────────────┬─────────────────────────────────┤
│ 1. LSP Hybrid   │ 2. Native Go    │ 3. Pre-Flight    │ 4. Static Taint Tracing &       │
│    Engine       │    SSA Engine   │    AST Scanner   │    5% Opacity Isolation        │
├─────────────────┼─────────────────┼──────────────────┼─────────────────────────────────┤
│ 5. Compilerless │ 6. 5-Tier Set   │ 7. Bi-Directional│                                 │
│    synapse-pkg  │    Viewports    │    MCP AI Canvas │                                 │
└─────────────────┴─────────────────┴──────────────────┴─────────────────────────────────┘
```

---

## 1. LSP Hybrid Semantic Engine (`parser/semantic.go`)

Syntax parsing alone cannot distinguish between a function call and a variable identifier with the same name. Go-Synapse solves symbol ambiguity by booting native Language Server Protocol binaries (`gopls`, `rust-analyzer`, `pyright-langserver`, `clangd`, `typescript-language-server`).

### Read vs. Write Resolution
`parser/semantic.go` parses assignment operators (`=`, `+=`, `:=`, `++`, `--`, `&var`) to distinguish variable **reads** from variable **writes**:
- Variable **writes** create state-mutation AST nodes.
- Variable **reads** create data-flow dependency edges.

---

## 2. Native Go SSA Control Flow Engine (`parser/ssa.go`)

For Go repositories, Go-Synapse integrates the native Go Static Single Assignment package (`golang.org/x/tools/go/ssa`).

### Basic Blocks & Concurrency Primitives
It constructs explicit linear basic blocks (`ssa_blocks`) and tracks cross-goroutine static channel dataflows:
- **`may_send`**: Channels transmitting data.
- **`may_receive`**: Goroutines waiting on channel input.
- **`may_spawn`**: Asynchronous `go` routine invocations.

---

## 3. Pre-Flight AST Threat Scanner (`parser/scanner.go`)

On every file save, Go-Synapse automatically audits code symbols against regular expression pattern sets in `~/.go-synapse/signatures.json`.

### Threat Types Instantly Detected:
1. **Indirect Prompt Injection**: `ignore previous instructions`, `system prompt override`
2. **SQL Injection String Concatenation**: `DROP TABLE`, `UNION SELECT`, `select * from ... +`
3. **Cross-Site Scripting (XSS)**: `<script>`, `onerror=`, `onload=`
4. **Remote Code Execution (RCE)**: `exec.Command`, `os.StartProcess`, `subprocess.run`
5. **Hardcoded Secrets**: `api_key`, `passwd`, `secret`, `token` assignments
6. **Block/Loop Defer Resource Leaks**: `defer` statements inside `for` or `if` loops
7. **Unencrypted Endpoints**: `http://`, `ws://`

Flagged nodes are assigned badge `PROBABILITY_ZONE:INJECTION_RISK` and painted in glowing red (`#ff1744`).

---

## 4. Static Taint Tracing & 5% Opacity Isolation

To eliminate visual noise during security audits, Go-Synapse performs static taint analysis:
- **Input Tracking**: Traces untrusted HTTP request parameters (`req.URL.Query()`) through intermediate assignment variables down to execution sinks (`os.WriteFile`, `db.Exec`, `exec.Command`).
- **5% Opacity Canvas**: Automatically dims 95% of non-participating codebase nodes down to **5% opacity**, focusing the auditor's visual attention strictly on the active threat path.

---

## 5. Compilerless Enterprise Packaging (`cmd/synapse-pkg/main.go`)

High-security SCIF and defense environments operate in 100% air-gapped isolation without internet access or C compilers.

### How `synapse-pkg` Works:
- Pre-packages native LSP binaries (`gopls`, `clangd`, `rust-analyzer`) into standalone `.tar.gz` distribution archives.
- Allows enterprise teams to deploy Go-Synapse onto zero-client workstations without needing GCC, internet access, or external downloads.

---

## 6. 5-Tier Set Viewport Engine & Hub Portal Badges (📞)

To maintain 60 FPS graph rendering performance on large monoliths:
- **5 Viewport Tiers**: Zooms seamlessly from macro Domain Sets (Tier 1) down to Working Code (Tier 5).
- **Hub Portal Badges (📞)**: High-indegree utility nodes (like loggers or database connection handles with `in_degree > 3`) are collapsed into compact portal badges, eliminating visual line tangles.

---

## 7. Bi-Directional AI Canvas Driving via MCP (`mcp/server.go`)

Go-Synapse implements JSON-RPC 2.0 stdio tools exposing complete backend and visual control to AI agents:
- **`execute_sql`**: Runs 2ms relational CTE queries against `synapse.db`.
- **`focus_dashboard_node`**: Pans and zooms the browser camera to specific node coordinates.
- **`annotate_node`**: Paints vulnerability badges directly on the 2D canvas.
- **`render_ui_template`**: Projects Mermaid flowcharts into the Telemetry Drawer.
- **`freeze`**: Locks database state (`lock: true|false`) during AI agent benchmark runs.

---

## Conclusion

The 7 core backend engines of Go-Synapse turn complex source code repositories into a high-speed, queryable, and visually intuitive AST data surface.

Explore the complete master article directory in [Story Vault](index.html#articles).
