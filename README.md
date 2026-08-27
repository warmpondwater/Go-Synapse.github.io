# Go-Synapse

**Go-Synapse** is a local, non-LLM Model Context Protocol (MCP) server. Think of it as an **`mcp-graph-generator-for-llm`** that also acts as a **`graph-AGENT-USER-Interface`**. It bridges the gap between your LLM Client (like Claude Desktop, Cursor, or AntiGravity) and complex repositories by providing deterministic static-to-graph analysis.

## What Go-Synapse IS:
- **The "Shovel":** We do the heavy mathematical lifting (AST parsing, single static assignment, and taint propagation) and package it as a standard MCP agent. You plug it into your existing LLM.
- **A Visual Graph Bridge:** It projects a real-time Graph Theory UI directly into your browser, allowing you to visually track the execution paths that your AI is auditing.
- **Strictly Local & Secure:** It executes locally. It does not handle, proxy, or require API keys. Your LLM client handles authorization. 

## What Go-Synapse IS NOT:
- **It is NOT an LLM orchestrator:** We do not route tokens, manage chat interfaces, or pay inference costs.
- **It is NOT an IDE replacement:** It is built for targeted structural analysis, not general-purpose coding.
- **It is NOT a chat interface:** We do not try to build a custom chat UI. Your interaction with the LLM happens natively in your tool of choice (Cursor, VS Code, Neovim, IntelliJ, or terminal). Go-Synapse is strictly the companion web visualizer that projects the architecture graph and visual models alongside your editor.

> [!IMPORTANT]
> **🔒 The Separation of Read vs. Write (Zero Mutation Risk):**
> * **Go-Synapse (Read-Only Code Intelligence)**: Parses code, runs graph math, enforces a strict read-only jail on your target repo, and provides 2ms SQL queries via MCP. It **never** mutates or overwrites your source files.
> * **User Agent Harness (Code Mutation)**: Your AI tool (Cursor, Claude Code, AntiGravity, VS Code) does the thinking and performs all code edits/writes directly in your editor.
> * **Live Bridge**: The instant your editor saves changes to disk, Go-Synapse's file watcher hot-reloads the 2D visual architecture in real time.

> [!TIP]
> For the comprehensive handbook, operational procedures, and relational AST schema reference, see the **[Go-Synapse Master Handbook & User Guide (docs/Go-Synapse.md)](docs/Go-Synapse.md)**.

---

## Architecture

1. **Polyglot AST Parser** ([parser/parser.go](parser/parser.go))
   Leverages the C-based Tree-sitter engine to parse source code into a language-agnostic node graph.
   - **Supported Languages:** Go, Python, JavaScript, TypeScript, Java, C++, C, Rust, Ruby, PHP, C#.
   - **Node Typology:** Maps constructs into visual types: Functions, Methods, Structs/Interfaces, Variables, and Packages.
   - **Fully-Qualified Lexical Scoping:** Variables and execution blocks are strictly isolated by their file, scope, and line constraints to resolve ambiguity during reference lookups. *Note: Top-level scope traversal creates `global_execution_block` nodes; identifiers matched at this root level (such as package declarations or declaration headers matching global symbols) will resolve as reference/calls from the file's global execution block.*
   - **Edge Typology:** Maps memory references into visual flow edges: `calls`, `writes`, `reads`, `returns`, `instantiates`, `may_send`, `may_receive`, and `may_spawn` (supporting cross-goroutine static channel and thread dataflow tracing).
   - **Language-Specific Analysis & Helpers:** Native Control Flow Graph (CFG) and Single Static Assignment (SSA) extraction via `golang.org/x/tools/go/ssa` for Go, with structural boundaries and heuristics fallback via Tree-sitter for others (split into [parser/ssa.go](parser/ssa.go), [parser/heuristics.go](parser/heuristics.go), [parser/utils.go](parser/utils.go), and [parser/semantic.go](parser/semantic.go)).
   - **Database Storage & Splitting:** SQLite syncing via [db/db.go](db/db.go), with specialized splits for database locking [db/lock.go](db/lock.go), SSA recording [db/ssa.go](db/ssa.go), and global resolution & kill zones [db/analysis.go](db/analysis.go).

2. **Live Bridge (`bridge/server.go`) & File Watcher (`watcher/watcher.go`)**
   A real-time WebSocket protocol linking the Go backend and the Cytoscape frontend, backed by an active file watcher.
   - Emits `GRAPH_DATA` on initialization, merging SQLite-cached properties (`color`, `badge`, `reconciled`, degrees) onto memory-parsed elements.
   - Pushes a comprehensive Markdown dependency (SCA) and test coverage (DAST) health summary to the Telemetry Drawer upon handshake.
   - Actively monitors the target directory for file changes and hot-reloads the AST graph to the UI on-the-fly, preserving audit highlights.

3. **MCP Worker Server (`mcp/server.go`)**
   Natively integrates with AI Agents via the Model Context Protocol (MCP). Instead of providing brittle, single-purpose endpoints, we follow a zero-tooling "Data Warehouse" philosophy:
   - `execute_sql`: The Agent is given direct SQL access to the underlying parsed AST database (`synapse.db`), letting it autonomously interrogate the repo architecture.
   - `render_ui_template`: The Agent can push rich visual models (Markdown, Mermaid, Tables) or raw `html` directly into the Telemetry Drawer to present its findings as interactive ad-hoc dashboards.
   - `annotate_node`: Paint the graph dynamically with threat levels based on the LLM's analysis.
   - `open_file_in_editor`: Instantly pop open a specific physical file in the human's local VSCode instance.
   - `get_visual_state`: Queries what visual node the human is currently selecting in the browser.
   - `get_symbol_context`: Retrieves raw code block, parent file path, visibility, package namespace, direct incoming callers, and direct outgoing callees for a given node ID, offering highly optimized context-fetching to AI agents.
   - `focus_dashboard_node`: Instantly forces the browser UI to highlight and center on a node.
   - `freeze`: Programmatically locks/unlocks the database graph (Audit Integrity Freeze) to prevent file-watching writes during an active audit.

4. **Visual Dashboard (`public/index.html`)**
   A custom Cytoscape.js interface featuring:
   - **5-Tier Hierarchical Set Layout Modes (Logical Abstraction down to Physical AST):**
      1. `1. Domain Set (60,000-Foot Macro Abstraction)`: Logical pseudo-allocation that groups packages into top-level bounded contexts (e.g. `Parser Engine`, `APM & Telemetry`, `Data Storage`, `CLI Tools`, `Server System`, `Core Engine`), decluttering massive graphs into clean macro domain boxes.
      2. `2. Package Set (100% Deterministic Package Namespaces)`: Expands Domains to show exact physical directory & module package containers (`package main`, `package db`, `mod rust_src`).
      3. `3. File Set (100% Deterministic Filesystem)`: Expands Packages to show exact physical source file nodes on disk (`parser/scanner.go`, `auth/jwt.py`).
      4. `4. Working Code - Clean Symbols (100% Deterministic AST)`: Expands files to show exact Tree-Sitter symbol declarations (functions, structs, interfaces, methods) with zero visual edge noise.
      5. `5. Working Code - Active Call Graph (100% Deterministic Edges)`: Expands all symbols with active call-graph, reference, channel, and taint-flow dataflow edges.
   - **Multi-Domain Inference Engine:** Automatically categorizes packages into bounded context Domains using dynamic import analysis and path heuristics across Go, Python, TypeScript, Java, Rust, and C++.
   - **Hub Portal Edge Reduction:** Automatically identifies high-indegree "Hub" nodes (nodes with in-degree > 3, such as logging, database connections, or common utilities) and collapses their incoming edge lines into compact `📞` portal badges on caller nodes. Hovering over any caller node dynamically illuminates and reveals the glowing edge line on-the-fly to prevent visual clutter.
   - **Include External / Stdlib Filter:** Allows filtering out third-party/standard library dependencies (`sys_external`, etc.) at the click of a button to declutter active code relationships.
   - **Taint Tracer:** Breadth-First-Search taint trace visualizer.
   - **Code Inspector:** Safe code popup displaying raw source blocks via `.textContent` (handling generics and HTML operators without escaping errors) with Highlight.js.
   - **APM Execution Trace Overlay:** Ingests exported Jaeger/OpenTelemetry JSON traces, matching spans to AST nodes, mapping execution durations, and visually painting dynamic runtime flows (using thicker glowing orange borders/lines and dimming non-participating elements).
   - **Agent Telemetry Drawer**: An interactive collapsible Telemetry Drawer for rendering agent dashboards and Mermaid diagrams.
   - **Live Markdown Editor:** A toggleable text editor for instantly writing and saving local markdown files to disk without leaving the graph view.

5. **Graph & Environment Verification Engine**
   A rigorous validation pipeline to guarantee the semantic and cryptographic integrity of the graph:
   - **Symbol Reconciliation** ([verifier/reconciler.go](verifier/reconciler.go)): Reconciles AST nodes and calls/references at coordinate level against physical source files.
   - **Link Pruning & Checks** ([verifier/integrity.go](verifier/integrity.go)): Scans and deletes dangling edges pointing to non-existent target symbols, computes deterministic hashes, and calculates node degrees.
   - **Environment Drift Checks** ([verifier/audit_cert.go](verifier/audit_cert.go)): Audits Git dirty tree status, physical file hash drift, new/untracked files post-freeze, and AST parse density metrics.
   - **Cryptographic Signature** ([verifier/audit_cert.go](verifier/audit_cert.go)): Generates RSA-2048 key pairs to sign a validation certificate containing repository hubs, checksums, exclusions, and warnings.

---

## Installation

### Standard Installation
If you have a C compiler (GCC/Clang) and Go installed locally:
```bash
go mod tidy
go build
```

**Global Installation (Optional):**
Because Go-Synapse uses the `--dir` flag to dynamically parse external repositories, it is fully portable. You can register it in your `$PATH` to use it system-wide:
```bash
sudo mv ./Go-Synapse /usr/local/bin/go-synapse
```

### Compilerless Enterprise/Compliance Installation (Staging & Bundling)
If deploying to locked-down or air-gapped terminals where compilers are missing, you can use the separate compiler and staging tool `synapse-pkg`:
1. Build the packager utility:
   ```bash
   go build -o bin/synapse-pkg cmd/synapse-pkg/main.go
   ```
2. Build the core and fetch all pre-compiled LSP binaries into a single archive (e.g. for macOS ARM64):
   ```bash
   ./bin/synapse-pkg bundle -os darwin -arch arm64 -format tar.gz
   ```
3. Distribute the generated `synapse-darwin-arm64.tar.gz` to the locked user terminal. Once unpacked, Go-Synapse is ready to run without external dependencies or internet access.
4. **Batch License Provisioning**: `synapse-pkg` is non-interactive and scriptable (`synapse-pkg provision-license --key KEY --machine-id ID`), allowing IT admins to batch-generate signed `license.json` files across multi-user fleets.

### Remote Build Server Compilation (Linux/Windows)
If you are developing on macOS and want to compile Linux and Windows binaries natively on a remote Linux builder:
1. Ensure you have configured SSH access (e.g. passwordless SSH key pair) to your build server.
2. Run the deployment script:
   ```bash
   ./deploy.sh
   ```
This automatically synchronizes your local repository changes to the remote builder, triggers the compilation (`./build.sh`), and fetches the signed artifacts.

---

## Configuration

Go-Synapse first checks for a workspace-specific configuration at `./config.json` in the current working directory. If not found, it falls back to the protected system directory:
* **System Path:** `~/.go-synapse/config.json` (created automatically on first run with default values).

> [!NOTE]
> If a workspace `./config.json` is loaded but lacks a `"license_key"` field, the configuration loader automatically falls back and checks the global `~/.go-synapse/config.json` to retrieve the license key.

### Example `config.json`
```json
{
  "editor": "code",
  "terminal": "open -a Terminal",
  "max_nodes": 1000000,
  "allow_external_paths": false,
  "languages": ["Go", "Python", "JavaScript", "TypeScript", "C", "C++", "Rust", "Java", "Ruby", "PHP", "C#"],
  "lsp_servers": {
    "Rust": {
      "command": "rust-analyzer",
      "download_url": "https://artifactory.internal.mycompany.com/binaries/rust-analyzer"
    },
    "Java": {
      "command": "jdtls",
      "download_url": "https://artifactory.internal.mycompany.com/binaries/jdtls"
    }
  },
  "tree_sitter": {
    "Rust": {
      "download_url": "https://artifactory.internal.mycompany.com/binaries/tree-sitter-rust.so"
    },
    "Java": {
      "download_url": "https://artifactory.internal.mycompany.com/binaries/tree-sitter-java.so"
    }
  }
}
```

* **`max_nodes` (integer):** The Node Circuit Breaker limit (defaults to `1000000`). If a codebase has more than this number of structural nodes, parsing will halt with a `413 Payload Too Large` error to prevent browser OOM crashes.
* **`allow_external_paths` (boolean):** If `false` (default), the server and MCP tool block any file-open or read requests outside the target directory boundary. If `true`, external paths are allowed but a warning is logged.
* **`editor` / `terminal` (string):** Commands used by the `open_file_in_editor` and OS integration hooks.
* **`languages` (array of strings):** Enabled tree-sitter language parsers.
* **`lsp_servers` (object):** Enterprise/compliance overrides mapping languages to custom LSP start commands, arguments, and local mirror download URLs.
* **`tree_sitter` (object):** Compliance overrides mapping languages to local mirror download URLs for Tree-sitter grammars and parser shared objects.

### Protected System Files
All execution state is isolated in `~/.go-synapse/`:
1. `~/.go-synapse/config.json`: System configuration (detailed above).
2. `~/.go-synapse/synapse.db`: The SQLite AST database containing parsed node metadata.
3. `~/.go-synapse/signatures.json`: Customizable threat signatures used by the pre-flight injection scanner.

### Pre-Flight Threat & Code Smell Signatures

Go-Synapse runs an automated **Pre-Flight Scan** over the parsed AST database at the end of every sync cycle. It matches node labels and source code chunks against a list of regular expressions defined in `~/.go-synapse/signatures.json` (initialized automatically with defaults on first run). 

Flagged nodes are assigned the badge `PROBABILITY_ZONE:INJECTION_RISK`, colored red (`#ff1744`), and can be instantly focused using the **Highlight Threat Nodes** filter.

#### Default Signatures:
* **Indirect Prompt Injection:** Detects LLM instructions embedded in comments or strings (e.g., `ignore previous instructions`, `system prompt`).
* **SQL Injection:** Detects common SQLi keywords (e.g., `union select`, `drop table`).
* **XSS / HTML Injection:** Detects inline HTML script tags and event handlers (e.g., `<script>`, `onerror=`, `onload=`).
* **Unsecured HTTP Endpoint:** Detects unencrypted `http://` or `ws://` URL string literals.
* **Potential Hardcoded Secret:** Flags variable names containing `passwd`, `password`, `secret`, `token`, or `api_key` assigned to a string.
* **Unresolved TODO/FIXME:** Flags developer comments like `TODO:` or `FIXME:`.
* **Block/Loop-Scoped Defer:** Flags Go `defer` statements defined inside `if`, `for`, or `switch` blocks (which can cause transaction leaks or connection pool exhaustion).
* **SQL Wildcard Query:** Flags unoptimized `SELECT * FROM` wildcard queries.
* **External Command Execution:** Flags calls to OS process execution APIs (e.g., `exec.Command`, `os.StartProcess`, `subprocess.run`, `os.system`).

#### Customizing Signatures:
You can extend this file to inject your own custom architectural rules, security policies, or code smell detectors.

Format of `~/.go-synapse/signatures.json`:
```json
[
  {
    "name": "Custom Deprecated API Warning",
    "regex": "(?i)(legacyApiCall|deprecatedMethodName)"
  }
]
```

### Security & Network Architecture
Go-Synapse is built with strict local-only security boundaries out of the box:
* **Loopback Binding:** The HTTP and WebSocket server explicitly binds exclusively to `127.0.0.1` (never `0.0.0.0`), preventing external network access to your source code or editor hooks.
* **Transport Encryption:** Auto-generates local self-signed TLS certificates (`https://127.0.0.1:8080`) when keypairs are present in `~/.go-synapse/`, falling back to HTTP if missing.
* **CSWSH Protection:** WebSocket connections enforce strict `Origin` header validation, accepting requests only from local loopback origins (`http://127.0.0.1:8080`, `http://localhost:8080`, `https://127.0.0.1:8080`, `https://localhost:8080`).
* **CSRF Mitigation:** Mutating endpoints (such as telemetry sync) require `X-CSRF-Token` headers injected at template startup.

---

## Running the Engine

**1. Standalone / Human Mode**
```bash
./Go-Synapse
```
Navigate to `http://localhost:8080`. The engine will parse the target directory (defaulting to current directory or specified via `-dir`) and render the graph.

**Port Customization (`-port`):**
To run multiple instances simultaneously or avoid port collisions, specify a custom port:
```bash
./Go-Synapse -dir /path/to/project1 -port 8080
./Go-Synapse -dir /path/to/project2 -port 8081
```

**2. MCP / Agent Mode**
Instead of manually configuring an `mcp.json` file, simply instruct your AI Agent to start the MCP server directly. The Agent should execute the raw binary as a child process:
```bash
/path/to/Go-Synapse -dir /path/to/target/project -mcp
```
*Note: The Standalone server must be running on `:8080` (or specified `-port`) simultaneously so the MCP worker can bridge HTTP annotations.*

**3. Verification & Auditing Mode**
To run a synchronous verification pass of the graph database against the physical file tree and sign an integrity certificate:
```bash
./Go-Synapse -audit=verify -dir /path/to/target/project
```
*(Legacy syntax `./Go-Synapse -verify -dir ...` is also supported and automatically mapped to `-audit=verify`)*

This commands:
1. Runs node and edge coordinate reconciliation.
2. Prunes dangling links.
3. Recalculates node degrees (hubs) and computes a deterministic database checksum.
4. Audits the environment (Git status dirty flags, post-freeze file drift, low AST density warnings).
5. Generates or loads `./audit_private_key.pem` and `./audit_public_key.pem`, then signs the resulting statistics into `./audit_certificate.json`.

**4. Additional Auditing Commands**
Running these commands populates the database and automatically updates the **SCA & Coverage Audit Summary** panel (Telemetry Drawer) in the visualizer dashboard:

* **Static Ingestion Audit:** Perform only the static parsing and semantic mapping phase:
  ```bash
  ./Go-Synapse -audit=static -dir /path/to/target/project
  ```
* **Software Composition Analysis (SCA):** Check third-party dependencies and query security advisories:
  ```bash
  ./Go-Synapse -audit=sca -dir /path/to/target/project
  ```
  *Use the `-offline` compliance flag to disable outbound network requests and perform offline queries only:*
  ```bash
  ./Go-Synapse -audit=sca -dir /path/to/target/project -offline
  ```
* **Dynamic Analysis Audit (DAST):** Run test-execution dynamic analysis and map statement coverage to AST nodes:
  ```bash
  ./Go-Synapse -audit=dynamic -dir /path/to/target/project
  ```

**5. OKF Export Mode**
To export the parsed codebase representation as Git-diffable, portable Open Knowledge Format (OKF) Markdown files with YAML frontmatter:
```bash
./Go-Synapse -export=okf -dir /path/to/target/project
# or using the direct boolean flag with custom output path:
./Go-Synapse -export-okf -dir /path/to/target/project -okf-out ./custom_okf_export
```
* **`-export=okf` / `-export-okf`:** Enables the OKF export run. Exports are saved to `~/.go-synapse/exports/<project_name>/` by default.
* **`-okf-out`:** (Optional) Specifies a custom output directory for the exported OKF bundle.

**6. Demo Target Mode**
We maintain a pre-packaged, clean sibling demo codebase containing curated polyglot source files, isolated dead code, and high fan-out generated files to showcase Go-Synapse's capabilities.
To verify the demo target:
```bash
./Go-Synapse -verify -dir ./demo
```
This produces a successful verification certificate with **zero warnings** and a verified golden baseline hash.

**7. Blast Radius & CI/CD Operations Mode**
To analyze the impact / blast radius of local git modifications (staged or unstaged) or changes compared to a base commit or branch:
```bash
# Compare local unstaged/staged changes (falls back to latest commit if clean)
./Go-Synapse -blast-radius -dir /path/to/target/project

# Compare local changes against a specific base ref (e.g. main or origin/main)
./Go-Synapse -blast-radius -blast-base=origin/main -dir /path/to/target/project
```
This command:
1. Performs a synchronous parser and reconciliation pass to ensure the local SQLite database matches your current workspace files.
2. Extracts changed lines from `git diff <base>`.
3. Maps changed lines to physical AST functions, methods, or structs.
4. Traverses backwards along `calls` edges to identify all upstream callers (transitive blast radius).
5. Outputs a Markdown report and a formatted Mermaid impact diagram.

**8. Incremental Test Selection Mode**
To automatically detect and run *only* the specific test functions connected to your local changes via the call graph:
```bash
# Detect and execute target tests matching local staged/unstaged changes
./Go-Synapse -incremental-test -dir /path/to/target/project

# Run tests affected since origin/main
./Go-Synapse -incremental-test -blast-base=origin/main -dir /path/to/target/project
```
This command:
1. Re-parses the repository and reconciles the SQLite database to align with active source files.
2. Sweeps git diffs to detect changed line coordinates.
3. Recursively traces the call path from modified nodes to any Go test functions (`Test*` in `*_test.go`).
4. Groups the target tests by package and executes them sequentially using `go test -v -run "^(TestName1|TestName2)$" ./package`.

**9. AST Semantic Diff Mode**
To perform structural AST diffing between two codebase states or database snapshots:
```bash
# Compare the current project database against a prior snapshot
./Go-Synapse -diff=backup_revision.db -dir /path/to/target/project

# Compare two database snapshots explicitly
./Go-Synapse -diff=old_revision.db,new_revision.db
```
This command:
1. Attaches both SQLite AST databases via native relational SQL (`ATTACH DATABASE`).
2. Computes added, removed, and modified AST structural nodes (functions, structs, classes) across revisions.
3. Computes added and removed dependency call-graph edges.
4. Outputs a structured, human-readable terminal report detailing all structural modifications.

**10. Structured Integration & Conformance Test Packages**
The core integration and verification tests are divided into two distinct packages under the `tests/` directory:
* **Completeness Verification (`tests/completeness` package):** Validates the **completeness and conformance of generated outputs** (e.g. ensuring exported OKF documents contain valid YAML frontmatter, strict ISO timestamps, and unbroken internal relative links).
  Run with:
  ```bash
  go test -v ./tests/completeness -run TestOKFConformance
  ```
* **Functionality Verification (`tests/functionality` package):** Validates the **working functionality of engine processes** (e.g. database schema mapping, verifier coordinate repairs, taint tracer pathfinding, and reachability-BFS dead-code quarantine logic).
  Run with:
  ```bash
  go test -v ./tests/functionality
  ```

To run the complete test suite concurrently:
```bash
go test -v ./...
```

**11. Antigravity AI Agent Skill Setup**
If you use Antigravity, you can create a zero-config global wrapper and an auto-loaded skill to instruct your AI on exactly how to boot and use the tool.

1. **Wrapper Script:** Create `~/.local/bin/start-synapse`:
   ```bash
   #!/bin/bash
   TARGET_DIR="${1:-$PWD}"
   echo "Starting Go-Synapse for target: $TARGET_DIR"
   cd /path/to/Go-Synapse
   rm -f synapse.log
   nohup ./Go-Synapse -dir "$TARGET_DIR" > synapse.log 2>&1 &
   echo "Go-Synapse started. View the UI at http://127.0.0.1:8080"
   ```
   Make sure to make it executable (`chmod +x ~/.local/bin/start-synapse`).

2. **Antigravity Plugin & Skill:** Create `~/.gemini/config/plugins/go-synapse-plugin/plugin.json` and a skill file `~/.gemini/config/plugins/go-synapse-plugin/skills/go-synapse/SKILL.md`. Prepend the standard YAML frontmatter:
   ```yaml
   ---
   name: go-synapse
   description: Guide for starting and using the Go-Synapse visualization and AST analysis tool. Use this skill when asked to visualize a codebase, trace vulnerabilities, or map an AST.
   ---
   ```
   Now, any Antigravity agent entering your workspace will automatically know how to trigger and use the architecture graph seamlessly.

---

## Features

- **Unified Taint Propagation Tracer & Subgraph Engine:** 
  - **Unified All-Edge Reachability:** Click any `SOURCE` or target symbol in the UI to trace dataflow across ALL relational edge types (`calls`, `reads`, `writes`, `returns`, `references`, `instantiates`, `may_spawn`, `may_send`, `may_receive`). Non-participating elements fade to 5% opacity while active paths light up with sequence step badges (`[1] calls`, `[2] reads`, `[3] writes`).
  - **Bounded Start-to-Stop Taint Traversal:** When both a Source (Start) and Sink (Stop) node are selected, the engine executes a path-bounded recursive CTE to extract *only* the exact chain of nodes and edges connecting Start $\rightarrow$ Stop.
  - **Timestamped Subgraph Artifacts:** Extracted subgraphs are saved to `~/.go-synapse/extractions/` formatted as:  
    `<lead_node_name>-<direction>-<depth>-<time>-<date>.[html|db]`  
    *(e.g., `CalculateEdgeWeight-downstream-2-202059-20260803.html`)*.
  - **Standalone Interactive HTML Reports & Code Inspector:** Every extracted `.html` report contains a self-contained WebGL canvas with Cytoscape icons (`⚙️`, `⚡`, `🏛️`, `💎`), color-coded edges, edge deduplication (`GROUP BY source, target, label`), and an interactive **🕵️ CODE INSPECTOR** drawer that displays exact source code snippets, line ranges (`L62-L114`), package namespaces, and visibility when clicking any node.
- **Dead-code Quarantine Box:** Unreachable symbols and transitively isolated "dead code islands" (unused subgraphs that call each other but have no path to live entry points) are automatically identified via a **Mark-and-Sweep Reachability BFS** and grouped into an aggressive red boundary box in the UI. Note that standard library/external dependencies are excluded, and quarantined nodes can still render internal connections to show the dead code's structure.
- **Progressive Layout Modes:** Built-in visualization selector to switch between detail levels:
  - *Domain Mode (Domain Boxes Only):* Collapses all packages/modules to show only top-level system domain boundaries, drastically improving layout/render speeds on large repositories.
  - *Package Mode (No Edges):* Shows package/directory structures inside their domain containers without connection lines.
  - *Working Code (No Edges):* Displays all expanded classes/functions without connection lines.
  - *Working Code (With Edges):* Displays all expanded classes/functions along with their call, dataflow, and reference edges. **Note:** This layout automatically culls and hides all unconnected nodes from view, leaving only the active modules and connections currently in play.
- **Omni-Search Engine:** Case-insensitive search natively scanning visual labels, hidden node identifiers, node typology (e.g. `function`), file languages, and raw source code chunks concurrently.
- **Heuristic Edge Reconciliation:** A global linker pass that maps unresolvable dynamic AST edges to mathematically equivalent validated nodes in the graph to suppress UI ghost nodes.
- **Dynamic OS Hooks:** Both the human (via node clicks) and the AI (via MCP) can instantly open exact physical file locations in VSCode.
- **Simple Note Scratchpad:** Directly inside the dashboard, users can open a lightweight text area to quickly jot down audit notes and persist them straight to the local disk without context switching.
- **Agent Telemetry Drawer:** A resizable side panel that allows the AI Agent to project complex, non-text visual models (like Mermaid architectures, dynamic threat execution tables, or structured logs) directly into the Cytoscape UI. This unifies the visual workspace and prevents the user from constantly context-switching between the LLM client and the graph visualizer.

---

## Troubleshooting & FAQ

### Clearing Corrupted State / Database Locks
If the SQLite AST database locks or exhibits corrupted state after an ungraceful shutdown:
```bash
rm -f ~/.go-synapse/synapse.db*
```
Go-Synapse will automatically recreate the database schema and re-parse the workspace on the next run.

### Memory & System Sizing
* **Small to Medium Repos (<100k LOC):** ~150MB – 400MB RAM.
* **Large Repos (100k–500k LOC):** ~500MB – 1.2GB RAM.
* **Monorepos (>500k LOC):** Adjust `max_nodes` in `config.json` if needed (defaults to `1,000,000`). For smooth browser WebGL rendering on large codebases, switch to **Domain Set Layout Mode** to reduce the active DOM/canvas node count.

### Uninstallation & Cleanup
To completely remove Go-Synapse from your system:
```bash
# Remove global binaries
sudo rm -f /usr/local/bin/go-synapse /usr/local/bin/synapse-pkg

# Remove system state, SQLite database, and local configs
rm -rf ~/.go-synapse/
```

---

## 🤖 Agent Context (Architectural Directives)
*Notice to future AI agents interacting with this codebase:*
This architecture has undergone rigorous adversarial auditing (`marketing/agent_audit_logs.md`) and is bound by a strict **Local Non-LLM MCP Agent** business model. When proposing or implementing code changes, you **MUST** adhere to the following constraints defined in `marketing/export_hardening_plan.md`:
1. **Strict Directory Isolation:** Never merge the execution environment with the target environment. The binary/SQLite DB must live in a protected system directory, while the ingested code lives exclusively in the isolated `./target/` directory. Path traversal protection is paramount.
2. **OOM Boundaries (Node Circuit Breakers):** Cytoscape WebGL degrades heavily at scale. Do not attempt to natively project >1,000,000 nodes without pagination. Respect the `config.json` limit overrides.
3. **Zero Trust Local Execution:** Do not assume local execution means safe execution. Malicious source code ingested into the AST can trigger SQLi or XSS via the LLM (Indirect Prompt Injection). All `execute_sql` operations MUST be strictly parameterized, and all DOM rendering MUST be locally sanitized. We rely on **Pre-Flight Injection Scanning** over the AST database *before* LLM execution. For teams running audits against untrusted or unknown external repositories, it is highly recommended to implement server-side HTML/payload sanitization at the Go backend boundary to protect visualizer clients.

---

## 📄 License & Intellectual Property

Go-Synapse is proprietary commercial software protected under copyright law and distributed under the **[Go-Synapse End User License Agreement (EULA)](LICENSE)**:

`Copyright (c) 2026 Go-Synapse. All Rights Reserved.`

Third-party open-source components and their corresponding licenses (Tree-sitter, Cytoscape.js, SQLite, Highlight.js, etc.) are cataloged in **[THIRD_PARTY_LICENSES.txt](THIRD_PARTY_LICENSES.txt)**.

---

## Disclaimer

**Go-Synapse is provided "as is", without warranty of any kind, express or implied.** The developers and contributors hold no liability for the method of use, system errors, data corruption, or the security/validity of the reports generated. The issued `audit_certificate.json` represents a mathematical/heuristic state verification and does not guarantee application security, safety, or functional correctness. Users assume all risk associated with using this software.
