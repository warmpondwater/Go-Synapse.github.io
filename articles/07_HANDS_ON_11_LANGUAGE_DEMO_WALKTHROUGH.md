# Hands-On with Go-Synapse: A Walkthrough of the 11-Language Demo Repository

**Category**: Developer Tools / Polyglot Engineering / Software Architecture / Live Demo  
**Target Keywords**: polyglot code visualizer, Go-Synapse demo, 11 languages static analysis, multi-language call graph, 2D codebase visualizer  
**Reading Time**: 8 Minutes  

---

## Introduction: The Challenge of Multi-Language Repositories

In modern polyglot microservices and enterprise applications, developers rarely work in a single programming language. A typical stack might use **Go** for high-concurrency microservices, **Python** for machine learning pipelines, **TypeScript/JavaScript** for web frontends, **Rust** or **C++** for performance-critical engines, and **Java** or **C#** for legacy enterprise backend systems.

Visualizing how these languages interact across a single workspace has historically required 5 or 6 separate developer tools.

**Go-Synapse** solves polyglot fragmentation by indexing and rendering **11 core programming languages in a single, unified 2D spatial graph**.

---

## 1. Inside the 11-Language Demo Repository

The official Go-Synapse demo repository (`demo/`) is designed to showcase Go-Synapse's full polyglot capabilities.

### Language Directory Breakdown

| # | Language | Demo Subdirectory | Sample Code Functionality | Active LSP / Parser Engine |
| :--- | :--- | :--- | :--- | :--- |
| **1** | **Go** | `go_src/` | Dataflow pipeline, string concatenation, proto serializers | 🟢 `gopls` |
| **2** | **Python** | `py_src/` | SQLite event analytics tracking engine | 🟢 `pyright-langserver` |
| **3** | **JavaScript** | `js_src/` | Client-side user management & DOM handlers | 🟢 `typescript-language-server` |
| **4** | **TypeScript** | `ts_src/` | Strongly typed user profile manager & email notifier | 🟢 `typescript-language-server` |
| **5** | **C** | `c_src/` | Low-level C memory allocation & header routines | 🟢 `clangd` |
| **6** | **C++** | `cpp_src/` | Object-oriented `PaymentService` & account validator | 🟢 `clangd` |
| **7** | **Rust** | `rust_src/` | Memory-safe system utility routines | 🟢 `rust-analyzer` |
| **8** | **Java** | `java_src/` | Enterprise OOP class structures | 🟢 `jdtls` |
| **9** | **PHP** | `php_src/` | Notification management service | 🟢 `intelephense` |
| **10** | **Ruby** | `ruby_src/` | `OrderProcessor` class & fulfillment workflow | 🟡 Syntax Heuristics Engine |
| **11** | **C#** | `cs_src/` | `SecurityAuditor` access logging class | 🟡 Syntax Heuristics Engine |

---

## 2. Launching and Indexing the Demo in 3 Seconds

Launching Go-Synapse against the demo repository requires a single terminal command:

```bash
./Go-Synapse -dir /path/to/go-synapse-demo -port 8080
```

### What Happens When You Press Enter:

1. **Simultaneous LSP Booting**: Go-Synapse boots `gopls`, `clangd`, `pyright`, `jdtls`, `intelephense`, and `typescript-language-server` concurrently.
2. **Relational AST Database Creation**: In under 3 seconds, Go-Synapse parses **1,853 code elements**, segmenting them into **1,682 AST nodes** and **171 relational edges** inside `synapse.db`.
3. **Static Single Assignment (SSA) Extraction**: For Go code, the native Go SSA engine extracts **17,703 SSA Basic Blocks** and **20,008 Control Flow Edges**.
4. **Pre-Flight Threat Scanning**: The security scanner evaluates code nodes against `signatures.json` regex patterns on save.

```
2026/08/09 13:38:41 DB Sync: parsed 1853 elements. Segmented into 1682 nodes and 171 edges.
2026/08/09 13:38:41 Starting native SSA extraction for /path/to/go-synapse-demo
2026/08/09 13:38:41 Extracted 17703 SSA Blocks, 20008 CFG Edges, 91032 SSA Values
2026/08/09 13:38:42 Scanner: Loaded 9 threat signatures from ~/.go-synapse/signatures.json
```

---

## 3. Exploring the Live Demo Features

Opening `https://127.0.0.1:8080` in your browser reveals the interactive 2D spatial canvas:

### Feature 1: The 5-Tier Viewport Zoom
- **Tier 1 (Domain Set)**: Displays high-level language containers (`Go`, `Python`, `C++`, `Java`, `TypeScript`).
- **Tier 2 & 3 (Package & File Sets)**: Zooms into individual folders (`go_src`, `py_src`, `cpp_src`) and source files (`test.cpp`, `test.py`).
- **Tier 5 (Working Code)**: Displays exact function nodes, struct methods, and call lines.

### Feature 2: Live Threat Illumination (`INJECTION_RISK`)
- In `go_src/dataflow.go`, the function `GetUserInput` reads un-sanitized HTTP parameters concatenated into a SQL string.
- Go-Synapse automatically flags `GetUserInput` with a `PROBABILITY_ZONE:INJECTION_RISK` badge and paints the node in **glowing red (`#ff1744`)**.

### Feature 3: Spaghetti Prevention via Hub Portal Badges (📞)
- High-indegree utility nodes (like standard library loggers or database connection handles) are collapsed into **Hub Portal Badges (📞)**.
- Clicking a portal badge expands its callers in a clean side drawer without cluttering the 2D canvas with hundreds of overlapping lines.

---

## 4. Generating the Signed Audit Certificate

To complete the demo walkthrough, run the audit verification command:

```bash
./Go-Synapse -audit=verify -dir /path/to/go-synapse-demo
```

Go-Synapse reconciles 100% of nodes and edges, prunes dangling links, and writes a cryptographically signed audit certificate to `demo/logs/audit_certificate.json`.

---

## Conclusion: Try the Demo Yourself

Whether you are auditing legacy C++ and Java systems, building Go microservices, or training developers on modern architecture, the Go-Synapse demo provides a 2D visual playground over real multi-language code.

Explore the complete master story directory in [Story Vault](index.html#articles).
