# Copilot Agent Instructions

## You Are a Tool-First Agent

You do NOT read files manually or guess at code structure. You use VS Code's built-in tools for everything. These tools are your eyes and hands.

## Two Agents, Two Files

This workspace has two self-contained agent definitions:

| File | Agent | Trigger | Output |
|---|---|---|---|
| `.github/codegraph.md` | Code Graph Builder | "build code graph" | `CODE_GRAPH.md` |
| `.github/loganalysis.md` | Log Analysis | "analyze logs" | `LOG_ANALYSIS.md` |

When a user triggers a task, read the corresponding agent file and follow its pipeline exactly.

## Tool Catalog (memorize this)

### Read-Only Tools (auto-approved — use freely)
| Tool | When to Use | Token Cost |
|---|---|---|
| `search/codebase` | "Where is X?" / "How does X work?" — semantic search | Medium |
| `search/text` | Find exact strings, error messages, literals | Low |
| `search/files` | Find files by name/glob pattern | Low |
| `search/usages` | Who calls this function/variable? | Medium |
| `search/changes` | What changed recently? (incident triage) | Low |
| `read/file` | Read an entire file (avoid for files >500 lines) | High |
| `read/symbol` | Read one function/class definition (PREFERRED) | Low |
| `read/problems` | What errors/warnings exist right now? | Low |
| `workspace/tree` | Directory tree | Low |
| `lsp/definition` | Jump to where a symbol is defined | Low |
| `lsp/references` | All reference sites (compiler-precise) | Medium |
| `lsp/hover` | Type signature + docs (CHEAPEST type info) | Very Low |
| `lsp/implementation` | Find concrete impls of an interface | Medium |
| `lsp/documentSymbols` | File outline / table of contents | Low |
| `graph/dependencies` | Module dependency graph | Medium |
| `graph/callgraph` | Caller/callee graph (blast radius) | Medium |
| `graph/dataflow` | Data propagation paths (source→sink) | Medium |

### Write Tools (require diff preview)
| Tool | When to Use |
|---|---|
| `edit/replace` | Change a specific string/range (PREFERRED — surgical) |
| `edit/file` | Apply multiple edits to a file |
| `edit/create` | Create a new file |

### Execute Tools (allowlisted)
| Tool | When to Use |
|---|---|
| `terminal/run` | Run tests, builds, one-shot commands |
| `terminal/background` | Start dev server, watcher (non-blocking) |
| `test/run` | Run all tests |
| `test/runFailed` | Re-run only failing tests (FAST iteration) |
| `test/results` | Read last results without re-running (FREE) |
| `debug/*` | Set breakpoint, inspect variables, read callstack |

## Golden Rules

### 1. Tool Selection Priority
```
lsp/hover → read/symbol → read/file
```
Always start with the cheapest tool. Only escalate if the cheaper one doesn't answer the question.

### 2. Read Before Write
```
read/symbol → edit/replace → read/problems → test/runFailed
```
NEVER edit a file without reading it first. The edit anchor must be against current content.

### 3. Verify Before Done
A task is not complete until:
- `read/problems` shows zero new errors
- `test/run` or `test/runFailed` passes
- `git/diff` has been reviewed

### 4. Token Discipline
- Files >500 lines: NEVER `read/file`. Use `lsp/documentSymbols` → `read/symbol`.
- Don't re-read files you've already seen in this session.
- Prefer `search/codebase` (semantic) over reading 5 files to find something.

### 5. No Fabrication
Every claim must cite a `file:line`. If a tool returns empty results, say "not found" — don't guess.

## The Two-Phase Workflow

```
"build code graph" → CODE_GRAPH.md (architecture, deps, call chains, blast radius)
        ↓
"analyze logs"     → LOG_ANALYSIS.md (incidents traced to code root causes)
```

The log analysis agent reads `CODE_GRAPH.md` first — it needs the call graph and blast radius to trace errors efficiently. Build the graph first.
