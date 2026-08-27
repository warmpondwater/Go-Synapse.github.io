# Smart Incremental Testing: Run Only the Unit Tests That Matter with Go-Synapse

**Category**: DevOps / Test Automation / CI/CD Performance / Software Quality  
**Target Keywords**: incremental unit testing, AST call graph test selection, Go-Synapse test runner, CI/CD speedup, git diff test filter  
**Reading Time**: 7 Minutes  

---

## Introduction: The CI/CD Bottleneck in Large Monoliths

As software repositories grow beyond 100,000 lines of code, test suites inevitably slow down. In large microservices or enterprise monoliths, running full unit test suites on every pull request can take 15 to 45 minutes.

Developers face a frustrating trade-off:
1. **Run All Tests**: Burn compute hours, delay PR merges, and slow down engineering velocity.
2. **Run Tests Manually**: Risk missing an obscure downstream test failure, allowing regressions into production.

**Go-Synapse** eliminates this trade-off with **AST Call-Graph Incremental Testing** (`./Go-Synapse -incremental-test`).

---

## 1. How AST Call-Graph Selection Works

Traditional incremental test runners rely on simple file-level matching: if `auth.go` changes, they run `auth_test.go`. However, if `auth.go` modifies a struct method called by `payment.go`, file-level runners fail to execute `payment_test.go`.

Go-Synapse operates at the **AST symbol level**:

```
┌────────────────────────────────────────────────────────────────────────┐
│                   INCREMENTAL TEST SELECTION ENGINE                    │
├───────────────────┬───────────────────────┬────────────────────────────┤
│ 1. GIT DIFF SCAN  │ 2. AST GRAPH TRAVERSAL│ 3. SMART SUITE EXECUTION   │
│ Detects modified  │ Traverses `edges` in  │ Executes strictly the      │
│ lines & symbols.  │ synapse.db upstream.  │ affected unit test suites. │
└───────────────────┴───────────────────────┴────────────────────────────┘
```

### The 3-Step Selection Mechanics:
1. **Git Line Inspection**: Go-Synapse inspects modified lines between your current branch and the base ref (e.g. `origin/main`).
2. **Upstream Call Graph Traversal**: Using recursive CTE queries against `synapse.db`, it identifies all functions, methods, and packages that depend on the modified symbols.
3. **Targeted Test Execution**: It maps the affected call tree directly to unit test functions, triggering only the test suites that exercise the changed logic.

---

## 2. Command Line Execution

Running incremental test selection requires a single command:

```bash
./Go-Synapse -incremental-test -blast-base=origin/main -dir /path/to/repo
```

### Terminal Output Example:
```
CLI: Starting synchronous parser pass to update AST databases...
CLI: Running node and edge reconciliation...
CLI: Calculating AST call-graph blast radius against base 'origin/main'...
[Incremental Test] Modified symbols detected: 2 (AuthToken.Validate, UserSession.Clear)
[Incremental Test] Upstream caller test suites identified: 3 of 42 suites.
[Incremental Test] Executing target test suites:
  ✓ go test ./pkg/auth -run TestValidateToken
  ✓ go test ./pkg/session -run TestClearSession
  ✓ go test ./pkg/payment -run TestProcessWithSession

[Incremental Test] 3 suites passed in 1.4s (39 suites skipped, saved 14m 22s).
```

---

## 3. Comparison: Full Suite vs. Incremental AST Selection

| Metric / Feature | Traditional Full Test Suite | File-Level Incremental Runner | Go-Synapse AST Incremental Runner |
| :--- | :--- | :--- | :--- |
| **Execution Time** | 15 – 45 Minutes | 3 – 8 Minutes | **1 – 3 Seconds** |
| **Cross-Package Safety** | High | Low (Misses cross-package callers) | **100% High (Traverses AST relational graph)** |
| **CI/CD Compute Cost** | High ($$$) | Medium ($$) | **Ultra-Low ($)** |
| **Developer Feedback Loop** | Slow (Context Switching) | Partial | **Instant (Real-Time)** |

---

## Conclusion: Slashing CI Pipeline Times

By combining git diff line analysis with deep AST call graph reachability, Go-Synapse ensures developers get instant test feedback without waiting on 45-minute CI pipelines.

Explore the full article catalog in [Story Vault](index.html#articles).
