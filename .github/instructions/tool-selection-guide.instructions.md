# Tool Selection Guide

## Decision Table: Which Tool for Which Question

### Finding Code
| Situation | Use This Tool | Why |
|---|---|---|
| "Where is X implemented?" | `search/codebase` | Semantic — finds by meaning, not exact words |
| "Who calls X?" | `lsp/references` or `search/usages` | Symbol-aware, won't match comments |
| "Find every TODO" | `search/text` | Exact literal match |
| "Find all test files" | `search/files` with `**/*test*` | Filename glob |
| "What's the type of X?" | `lsp/hover` | Cheapest type info — one call |
| "Go to definition of X" | `lsp/definition` | Compiler-precise jump |
| "Find implementations of interface X" | `lsp/implementation` | For DI/plugin systems |
| "What's the outline of this file?" | `lsp/documentSymbols` | Table of contents |

### Understanding Structure
| Situation | Use This Tool | Why |
|---|---|---|
| "What's the repo layout?" | `search/files` with `**/*` | Flat file list |
| "Module dependencies" | `graph/dependencies` | Architecture-level graph |
| "Call chain for X" | `graph/callgraph` | Transitive callers/callees |
| "Data flow: input → sink" | `graph/dataflow` | Taint/value propagation |
| "What changed recently?" | `search/changes` or `git/diff` | Narrow suspect set |
| "Blast radius of changing X" | `graph/callgraph` + `lsp/references` | Who breaks if X changes |

### Fixing Code
| Situation | Use This Tool | Why |
|---|---|---|
| "Fix all errors" | `read/problems` → `edit/replace` | Problems panel is structured |
| "This test is failing" | `test/results` → `debug/*` | Read results before re-running |
| "Verify my fix worked" | `test/runFailed` | Only re-run failures, not full suite |
| "Start the dev server" | `terminal/background` | Non-blocking |

### Making Changes
| Situation | Use This Tool | Why |
|---|---|---|
| "Change this specific line" | `edit/replace` | Surgical — fails if anchor is ambiguous |
| "Rename X everywhere" | `edit/rename` (symbol mode) | LSP-backed, updates all refs |
| "Create a new module" | `edit/create` | New file |

## Tool Sequencing Rules

1. **Narrow before broad.** `lsp/hover` before `read/symbol` before `read/file`.
2. **Read before write.** Always `read/symbol` or `read/file` before `edit/*`.
3. **Verify before done.** `read/problems` clean + `test/run` green = finished.
4. **Search before read.** `search/codebase` to locate, then `read/symbol` to read.
5. **Semantic for concepts, literal for strings.** `search/codebase` for "how does auth work", `search/text` for "TODO".

## Token Discipline
| Tool | Context Cost | When to Use |
|---|---|---|
| `lsp/hover` | ~50 tokens | Type signature + docs |
| `lsp/documentSymbols` | ~100 tokens | File outline |
| `read/symbol` | ~200 tokens | One function body |
| `search/usages` | ~300 tokens | All call sites |
| `read/file` (small) | ~500 tokens | File < 100 lines |
| `read/file` (large) | ~2000+ tokens | Last resort — avoid on big files |
| `graph/callgraph` | ~500 tokens | Transitive call chain |
| `graph/dependencies` | ~300 tokens | Module relationships |

**Rule:** If a file is >500 lines, NEVER `read/file` it. Use `lsp/documentSymbols` → `read/symbol` on the specific function.
