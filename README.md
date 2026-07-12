# VS Code Agent Toolkit

> **Reusable AI agents for any codebase.** Drop `.github/` and `.vscode/` into any repo. GitHub Copilot Chat gains two agents that build code graphs and analyze logs — all powered by VS Code's built-in tools. No scripts. No dependencies. Works with Python, TypeScript, Go, Java, Rust, C#, and more.

**Live repo:** https://github.com/rohitsalesforce132/vscode-agent-toolkit

---

## Table of Contents

- [What This Does](#what-this-does)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [How to Use](#how-to-use)
  - [1. Build a Code Graph](#1-build-a-code-graph)
  - [2. Analyze Logs](#2-analyze-logs)
- [The Two-Phase Workflow](#the-two-phase-workflow)
- [File Structure](#file-structure)
- [How It Works Under the Hood](#how-it-works-under-the-hood)
- [Supported Languages](#supported-languages)
- [Configuration Reference](#configuration-reference)
- [Troubleshooting](#troubleshooting)
- [FAQ](#faq)

---

## What This Does

Two AI agents that run inside GitHub Copilot Chat in VS Code. Each agent is a single self-contained markdown file with all instructions, tool patterns, and output templates embedded.

| Agent | What It Does | Trigger | Output |
|---|---|---|---|
| **Code Graph Builder** (`codegraph.md`) | Scans your entire repo — dependencies, call chains, data flows, API surface, blast radius — and writes a structured map. | `build code graph` | `CODE_GRAPH.md` |
| **Log Analysis** (`loganalysis.md`) | Reads your application logs, correlates each error with the code graph, traces root causes through source code. | `analyze logs` | `LOG_ANALYSIS.md` |

**Key idea:** The agents use VS Code's built-in tools (Language Server Protocol, semantic search, call graph analysis) — not external scripts. This means they work on any language VS Code supports, with zero configuration.

---

## Prerequisites

1. **VS Code** (version 1.99+ — needs Copilot Chat with Agent Mode)
2. **GitHub Copilot** subscription (Pro, Business, or Enterprise)
3. **Copilot Chat** extension installed and signed in
4. Your project open in VS Code

---

## Installation

### Option A: Quick Install (Copy 3 Files)

```bash
cd /path/to/your/project

# Copy the agent files (just 3 files)
cp /path/to/vscode-agent-toolkit/.github/codegraph.md .github/codegraph.md
cp /path/to/vscode-agent-toolkit/.github/loganalysis.md .github/loganalysis.md
cp /path/to/vscode-agent-toolkit/.github/copilot-instructions.md .github/copilot-instructions.md

# Copy VS Code config (optional but recommended)
cp -r /path/to/vscode-agent-toolkit/.vscode .vscode

# Create a logs directory (for log analysis)
mkdir -p logs
```

Open your project in VS Code. Done.

### Option B: Clone and Explore

```bash
git clone https://github.com/rohitsalesforce132/vscode-agent-toolkit.git
code vscode-agent-toolkit
```

In Copilot Chat, type `analyze logs` to see it work on the bundled sample log.

---

## How to Use

Open Copilot Chat (`Ctrl+Shift+I`). Type a trigger phrase.

### 1. Build a Code Graph

**Type:**
```
build code graph
```

**What happens (7 phases):**
1. Scans all files → detects language and framework
2. Maps symbols and structure via Language Server
3. Traces all imports → builds dependency graph
4. Traces call chains → builds call graph
5. Identifies data flow paths (source → sink)
6. Scans for diagnostics and TODOs
7. Writes `CODE_GRAPH.md`

**Output: `CODE_GRAPH.md`**

```markdown
# Code Graph — [Your Project]

## 1. Architecture Overview
[Annotated file tree with metadata]

## 2. Dependency Graph
[Layered: Foundation → Business Logic → API Surface]
[Circular dependency check]

## 3. Call Graph (Critical Chains)
[Key call chains with file:line references]

### Blast Radius Table
| If you change... | Files affected | Why |
|---|---|---|
| models/user.py | 12 files | Nearly entire codebase imports User |
| database.py | 11 files | Base class for all models |

## 4. API Surface
| Method | Path | Handler | File:Line | Services Called |
|---|---|---|---|---|

## 5. Data Flow
[Source → Sink paths for critical data]

## 6. Diagnostics
| Severity | File | Line | Issue |
|---|---|---|---|
```

### 2. Analyze Logs

**Type:**
```
analyze logs
```

**Prerequisite:** `CODE_GRAPH.md` must exist. If missing, the agent tells you to build it first.

**What happens (6 phases):**
1. Reads all files in `logs/`
2. Finds all errors, warnings, criticals
3. Groups entries into incidents (by timestamp, entity ID, component)
4. Loads `CODE_GRAPH.md` for architecture context
5. For each incident: traces the log error → finds the function in code → traces the call chain → finds root cause
6. Writes `LOG_ANALYSIS.md`

**Output: `LOG_ANALYSIS.md`**

```markdown
# Log Analysis Report

## Incident 1: Payment Gateway Timeout

### Summary
| Field | Value |
|---|---|
| Entity | order_id=o-5001 |
| Severity | CRITICAL |
| Subsystem | payment_service |
| Window | 08:15:00 – 08:15:03 |

### Event Timeline
| Time | Event | Source | Detail |
|---|---|---|---|
| 08:15:00 | ERROR | payment_service.js:142 | Connection timeout after 30000ms |
| 08:15:02 | ERROR | db | Transaction rollback for order_id=o-5001 |

### Root Cause
Payment gateway call at payment_service.js:142 has a 30s timeout.
The gateway was unreachable, causing a 500 error...

### Code Path
POST /api/payments/process → PaymentService.process() → PaymentGateway.charge()
  → TIMEOUT at payment_service.js:142

### Recommended Action
- [ ] Add circuit breaker around payment gateway calls
- [ ] Implement retry with exponential backoff
```

---

## The Two-Phase Workflow

The agents work in sequence. The code graph is the foundation; log analysis consumes it.

```
 ┌──────────────────┐
 │  Your Codebase   │
 └────────┬─────────┘
          │
          │ "build code graph"
          ▼
 ┌──────────────────┐
 │ CODE_GRAPH.md    │ ← Dependencies, call chains, blast radius,
 │ (generated)      │   API surface, data flow, diagnostics
 └────────┬─────────┘
          │
          │ "analyze logs"
          ▼
 ┌──────────────────┐      ┌──────────────────┐
 │ Log Analysis     │─────►│ logs/*.log       │
 │ Agent            │      │ (your app logs)  │
 └────────┬─────────┘      └──────────────────┘
          │
          │ Generates
          ▼
 ┌──────────────────┐
 │ LOG_ANALYSIS.md  │ ← Incident timeline, root cause,
 │ (generated)      │   code path trace, recommendations
 └──────────────────┘
```

**Why this order matters:** The log analysis agent reads `CODE_GRAPH.md` first to get the call chain and blast radius. Without it, the agent re-discovers the architecture for every log entry — burning tokens and time. With the graph pre-built, log analysis is fast and precise.

---

## File Structure

```
vscode-agent-toolkit/
├── .github/
│   ├── copilot-instructions.md    ← Master tool instructions (auto-loaded by Copilot)
│   ├── codegraph.md               ← Agent 1: builds CODE_GRAPH.md from any repo
│   └── loganalysis.md             ← Agent 2: correlates logs with code graph
├── .vscode/
│   ├── mcp.json                   ← MCP servers (log reader, Context7 docs)
│   └── settings.json              ← Tool auto-approve rules, terminal allowlist
├── logs/
│   └── app.log                    ← Sample log with 5 incident types
└── README.md                      ← This file
```

**Just 3 agent files + 2 config files.** That's the entire toolkit.

| File | Self-Contained? | Contains |
|---|---|---|
| `codegraph.md` | ✅ | Tool catalog, language detection patterns, 7-phase pipeline, output template, rules |
| `loganalysis.md` | ✅ | Tool catalog, 5 correlation patterns, 6-phase pipeline, output template, rules |
| `copilot-instructions.md` | ✅ | Master tool reference, golden rules, workflow wiring |

---

## How It Works Under the Hood

Every agent is a single markdown file that tells Copilot: **which tools to use, in what order, and how to report.** No Python scripts. No external dependencies.

### How the Code Graph Agent Works

```
Step 1: search/files  "**/*.py"            → "What files exist?"
Step 2: search/text   "^from |^import "     → "What depends on what?"
Step 3: lsp/documentSymbols on entry point   → "What's the structure?"
Step 4: graph/callgraph  root="main()"      → "Who calls what?"
Step 5: graph/dataflow  from input→sink     → "How does data flow?"
Step 6: read/problems                        → "What's broken?"
Step 7: edit/create  "CODE_GRAPH.md"        → "Write the report"
```

### How the Log Analysis Agent Works

```
Step 1: read/file  "logs/app.log"           → "What errors exist?"
Step 2: read/file  "CODE_GRAPH.md"          → "What's the architecture?"
Step 3: search/text  "[error message]"      → "Where is this error in code?"
Step 4: read/symbol  [error function]       → "What does this function do?"
Step 5: graph/callgraph  root="[function]"  → "What called this?"
Step 6: edit/create  "LOG_ANALYSIS.md"      → "Write the incident report"
```

---

## Supported Languages

The agents auto-detect the language from file extensions and adapt their search patterns:

| Language | Import Pattern | Entry Point | Test Command |
|---|---|---|---|
| **Python** | `from X import Y` | `main.py`, `app.py` | `python3 -m pytest` |
| **TypeScript** | `import { X } from "Y"` | `index.ts`, `server.ts` | `npm test` |
| **JavaScript** | `require("X")` / `import` | `index.js`, `app.js` | `npm test` |
| **Go** | `import "pkg/path"` | `main.go` | `go test ./...` |
| **Java** | `import pkg.Class;` | `Main.java` | `mvn test` / `gradle test` |
| **Rust** | `use crate::X;` | `main.rs` | `cargo test` |
| **C#** | `using Namespace;` | `Program.cs` | `dotnet test` |

---

## Configuration Reference

### Tool Auto-Approve Rules (`.vscode/settings.json`)

| Setting | Value | Effect |
|---|---|---|
| `chat.tools.read.autoApprove` | `true` | `read/file`, `read/symbol`, `read/problems` auto-run |
| `chat.tools.search.autoApprove` | `true` | All `search/*` tools auto-run |
| `chat.tools.lsp.autoApprove` | `true` | All `lsp/*` tools auto-run |
| `chat.tools.graph.autoApprove` | `true` | All `graph/*` tools auto-run |
| `chat.tools.test.autoApprove` | `true` | `test/run`, `test/runFailed` auto-run |
| `chat.tools.edit.requireDiffPreview` | `true` | Edits show a diff before applying |
| `chat.tools.debug.requireApproval` | `true` | Debug sessions require explicit OK |

### Terminal Allowlist

These commands auto-approve (everything else requires manual approval):

```json
"allow": [
    "python3 -m pytest *",
    "npm test *",
    "npm run *",
    "go test *",
    "cargo test *",
    "dotnet test *",
    "git status",
    "git diff",
    "git log *"
]
```

### MCP Servers (`.vscode/mcp.json`)

| Server | Provides | Required? |
|---|---|---|
| `log-reader` | Filesystem read access to `logs/` | Only for log analysis |
| `context7` | Library/framework documentation lookup | Optional |

---

## Troubleshooting

### Agent doesn't appear in Copilot Chat
1. Ensure `.github/codegraph.md` and `.github/loganalysis.md` are in the **workspace root**
2. Run `Ctrl+Shift+P` → **Developer: Reload Window**
3. Check VS Code version supports custom chat modes (1.99+)
4. Verify Copilot extension is up to date

### Log analysis says "CODE_GRAPH.md not found"
Build the code graph first:
```
build code graph
```
Then run log analysis.

### Agent runs but doesn't produce output files
1. Check Copilot has **Agent Mode** enabled (not just "Ask" mode)
2. Try **Bypass Approvals** mode — the agent needs write permission to create files

### Agent is slow or hits token limits
On large repos (>500 files):
1. **Scope the request:** `build code graph for the src/payments/ module only`
2. **Run in phases:** `map dependencies only`, then `trace call chains only`

### Terminal commands are blocked
Add your command to the allowlist in `.vscode/settings.json`.

---

## FAQ

**Q: Do I need GitHub Copilot?**
A: The agent files are Copilot-specific prompts. But the tool patterns (search → lsp → graph → read) work with any AI assistant that has tool-use capability (Cline, Cursor, Continue).

**Q: Why only 2 agents now?**
A: Each agent file is fully self-contained — no shared instruction files to maintain. Two files cover the two main use cases: understanding code and investigating problems.

**Q: Can I use this on a monorepo?**
A: Yes. Scope your request: `build code graph for packages/auth-service only`.

**Q: Can I customize the output format?**
A: Edit the output template section inside `codegraph.md` or `loganalysis.md`. Each file contains its own template.

**Q: How do I add a new agent?**
A: Create `.github/your-agent.md`. Follow the same structure: Role → Tool Catalog → Execution Pipeline → Output Template → Rules.

**Q: What if my logs aren't in logs/?**
A: Tell the agent: `analyze logs in /var/log/myapp/`. It adapts to any path.

**Q: Works with Cursor/Cline/Continue?**
A: Concepts transfer but file format differs (`.cursorrules`, `.clinerules`). The tool families are semantically equivalent.

---

## License

MIT — use freely in any project.
