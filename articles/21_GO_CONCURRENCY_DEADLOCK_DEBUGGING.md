# Debugging Goroutine Deadlocks Visually: SSA Concurrency Primitives in 2D Space with Go-Synapse

**Category**: Go Engineering / Concurrency & Goroutines / Compiler SSA / Static Analysis  
**Target Keywords**: Go SSA concurrency visualizer, goroutine deadlock debugging, Go channel static analysis, Go-Synapse SSA, may_send may_receive may_spawn  
**Reading Time**: 8 Minutes  

---

## Introduction: The Invisibility of Go Concurrency Bugs

Go's concurrency model (goroutines and channels) is lightweight and powerful. However, debugging concurrent Go applications is notoriously difficult:

- **Deadlocks**: Goroutines waiting indefinitely on un-buffered channel sends or receives do not generate explicit compile errors.
- **Goroutine Leaks**: Asynchronous `go worker()` invocations spawned inside long-running functions pile up in memory without clear visual trace lines.
- **Race Conditions**: Shared memory access across channels is invisible in standard text editors.

**Go-Synapse** makes Go concurrency visible through native **Go Static Single Assignment (SSA) Concurrency Edge Primitives** (`parser/ssa.go`).

---

## 1. How Native Go SSA Extraction Works (`parser/ssa.go`)

Go-Synapse integrates the native Go SSA compiler package (`golang.org/x/tools/go/ssa`). It parses compiled Go code into linear basic blocks (`ssa_blocks`) and tracks explicit concurrency channel operations.

```
┌────────────────────────────────────────────────────────────────────────┐
│                     SSA CONCURRENCY EXTRACTION ENGINE                  │
├───────────────────┬───────────────────────┬────────────────────────────┤
│ ssa_blocks        │ ssa_edges             │ ssa_values                 │
│ Linear instruction│ CFG edges & channel   │ Explicit registers, values,│
│ basic blocks.     │ communication flows.  │ and channel operations.    │
└───────────────────┴───────────────────────┴────────────────────────────┘
```

### The 3 SSA Concurrency Edge Types:
1. **`may_send`**: Visualizes goroutines transmitting data over channels.
2. **`may_receive`**: Highlights receiver goroutines waiting to read channel messages.
3. **`may_spawn`**: Renders asynchronous `go func()` routine invocations as glowing branch lines.

---

## 2. Visual Deadlock & Channel Leak Detection

In the Cytoscape 2D visual canvas (`http://127.0.0.1:8080`), SSA concurrency primitives illuminate thread interactions:

```
[ ProducerRoutine ] ──(may_send)──► [ Unbuffered Channel ] ──(may_receive)──► [ ConsumerRoutine ]
  (Spawns via may_spawn)                                                       (Blocked / Waiting)
```

### How Go-Synapse Exposes Concurrency Anti-Patterns:
- **Un-matched `may_send` Without Receiver**: Flags channel send operations that lack corresponding `may_receive` edge targets, identifying potential goroutine blocking leaks.
- **Async Spawning Inside Loops**: Renders `may_spawn` edges originated inside loop blocks, warning developers of unbounded thread growth.
- **Dominance Frontier & Kill Zone Analysis**: Using recursive CTE queries on `ssa_blocks` (`CalculateKillZones` in `db/analysis.go`), Go-Synapse identifies the last-use block for channel variables, pin-pointing where channels should be closed.

---

## 3. Database Schema for SSA Concurrency (`synapse.db`)

For deep AI interrogation via MCP (`execute_sql`), Go-Synapse exposes 3 dedicated SSA tables in SQLite:

```sql
-- Query all channel send and receive edges in a function
SELECT e.source_block_id, e.target_block_id, e.edge_type 
FROM ssa_edges e
WHERE e.edge_type IN ('may_send', 'may_receive', 'may_spawn');
```

---

## Conclusion: Making Multi-Threaded Flow Intuitive

By converting compiler SSA registers and channel operations into explicit 2D visual edges, Go-Synapse turns invisible Go concurrency bugs into clear, actionable visual models.

Explore the full article catalog in [Story Vault](index.html#articles).
