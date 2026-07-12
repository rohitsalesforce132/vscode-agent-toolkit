# VS Code Agent Toolkit

> **Reusable AI agents for any codebase.** Drop `.github/` and `.vscode/` into any repo. GitHub Copilot Chat gains three agents that build code graphs, analyze logs, and triage bugs — all powered by VS Code's built-in tools (`search/*`, `read/*`, `lsp/*`, `graph/*`, `debug/*`, `test/*`). No scripts. No dependencies. Works with Python, TypeScript, Go, Java, Rust, C#, and more.

**Live repo:** https://github.com/rohitsalesforce132/vscode-agent-toolkit

---

## Table of Contents

- [What This Does](#what-this-does)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
  - [Option A: Quick Install (Copy Files)](#option-a-quick-install-copy-files)
  - [Option B: Git Submodule](#option-b-git-submodule)
  - [Option C: Clone and Explore the Demo](#option-c-clone-and-explore-the-demo)
- [How to Use](#how-to-use)
  - [1. Build a Code Graph](#1-build-a-code-graph)
  - [2. Analyze Logs](#2-analyze-logs)
  - [3. Triage a Bug](#3-triage-a-bug)
- [The Two-Phase Workflow](#the-two-phase-workflow)
- [What Each File Does](#what-each-file-does)
- [How It Works Under the Hood](#how-it-works-under-the-hood)
- [Supported Languages](#supported-languages)
- [Configuration Reference](#configuration-reference)
- [Troubleshooting](#troubleshooting)
- [FAQ](#faq)

---

## What This Does

Three AI agents that run inside GitHub Copilot Chat in VS Code. Each agent is a markdown prompt file that instructs Copilot which tools to use, in what order, and how to report results.

| Agent | What It Does | Trigger Phrase | Output |
|---|---|---|---|
| **Code Graph Builder** | Scans your entire repo — dependencies, call chains, data flows, API surface, blast radius — and writes a structured map. | `build code graph` | `CODE_GRAPH.md` |
| **Log Analysis** | Reads your application logs, correlates each error/warning with the code graph, traces root causes through source code. | `analyze logs` | `LOG_ANALYSIS.md` |
| **Bug Triage** | Investigates a specific bug: reads diagnostics, recent git changes, runs failing tests, inspects debugger state. | `investigate bug` | Root cause report |

**Key idea:** The agents use VS Code's built-in tools (Language Server Protocol, semantic search, call graph analysis, debug adapter) — not external scripts. This means they work on any language VS Code supports, with zero configuration.

---

## Prerequisites

1. **VS Code** (version 1.99+ — needs Copilot Chat with Agent Mode)
2. **GitHub Copilot** subscription (Pro, Business, or Enterprise)
3. **Copilot Chat** extension installed and signed in
4. Your project open in VS Code

> **Don't have Copilot?** The `.github/instructions/` files are still useful as reference for how to use VS Code's tool API — they document every tool family with decision tables and token-cost estimates.

---

## Installation

### Option A: Quick Install (Copy Files)

This is the fastest method. Copy two folders into your existing project:

```bash
# Navigate to your project
cd /path/to/your/project

# Copy the agent configuration
cp -r /path/to/vscode-agent-toolkit/.github .
cp -r /path/to/vscode-agent-toolkit/.vscode .

# Create a logs directory (optional, only if you want log analysis)
mkdir -p logs
```

**That's it.** Open your project in VS Code. Copilot automatically loads:
- `.github/copilot-instructions.md` → master tool instructions
- `.github/chatmodes/*.chatmode.md` → agent definitions
- `.github/instructions/*.instructions.md` → reusable patterns
- `.vscode/settings.json` → tool auto-approve rules
- `.vscode/mcp.json` → MCP server config

### Option B: Git Submodule

If you want to keep the toolkit as a dependency that updates independently:

```bash
cd /path/to/your/project
git submodule add https://github.com/rohitsalesforce132/vscode-agent-toolkit.git .agent-toolkit

# Symlink so VS Code picks it up (run once)
ln -s .agent-toolkit/.github .github
ln -s .agent-toolkit/.vscode .vscode
```

### Option C: Clone and Explore the Demo

Want to see it work before installing? The repo ships with a sample log file:

```bash
git clone https://github.com/rohitsalesforce132/vscode-agent-toolkit.git
code vscode-agent-toolkit
```

Then in Copilot Chat, type `analyze logs` — the agent will analyze the bundled `logs/app.log`.

---

## How to Use

Open Copilot Chat in VS Code (`Ctrl+Shift+I` or `Cmd+Shift+I`). Type any of the trigger phrases below. The agent runs automatically — no manual tool invocation needed.

### 1. Build a Code Graph

**Type in Copilot Chat:**
```
build code graph
```

**What happens:**
1. Agent scans all files (`search/files` with `**/*`)
2. Detects language and framework from file extensions
3. Traces all imports/dependencies (`search/text`)
4. Maps symbols and structure (`lsp/documentSymbols`)
5. Traces call chains (`graph/callgraph`)
6. Identifies blast radius for every module
7. Scans for diagnostics (`read/problems`)

**Output:** `CODE_GRAPH.md` at your repo root, containing:
- Architecture overview (file tree with annotations)
- Dependency graph (layered, cycle detection)
- Call graph (critical chains with `file:line` references)
- API surface table (every endpoint/function)
- Data flow paths (source → sink)
- Blast radius table ("if you change X, N files break")
- Diagnostics (errors, TODOs, security concerns)

**Example output section:**
```markdown
### Blast Radius Table
| If you change... | Files affected | Why |
|---|---|---|
| models/partner.py | 12 files | Nearly the entire codebase imports Partner |
| database.py | 11 files | Base class for all models |
```

### 2. Analyze Logs

**Type in Copilot Chat:**
```
analyze logs
```

**What happens:**
1. Agent checks if `CODE_GRAPH.md` exists (if not, suggests building it first)
2. Reads all files in `logs/` (`read/file`)
3. Finds all errors, warnings, and critical events (`search/text`)
4. Groups entries into incidents (by timestamp, entity ID, component)
5. For each incident:
   - Extracts the function name and error message from the log
   - Finds that function in source code (`search/text`)
   - Reads the function body (`read/symbol`)
   - Traces who called it (`graph/callgraph`)
   - Traces how data reached it (`graph/dataflow`)
6. Writes structured incident reports

**Output:** `LOG_ANALYSIS.md` at your repo root, containing:
- One incident report per error cluster
- Event timeline (timestamped, chronological)
- Root cause paragraph (the causal chain)
- Code path trace (handler → service → failure point, with `file:line`)
- Recommended actions (with checkboxes)

**Example incident report:**
```markdown
## Incident 1: Payment Gateway Timeout

### Event Timeline
| Time | Event | Source | Detail |
|---|---|---|---|
| 08:15:00 | ERROR | payment_service.js:142 | Connection timeout after 30000ms |
| 08:15:02 | ERROR | db | Transaction rollback for order_id=o-5001 |

### Root Cause
Payment gateway call at payment_service.js:142 has a 30s timeout.
The gateway was unreachable, causing a 500 error. The DB transaction
was rolled back correctly.

### Code Path
POST /api/payments/process → PaymentService.process() → PaymentGateway.charge()
  → TIMEOUT at payment_service.js:142
  → DB rollback at order_repository.js:88

### Recommended Action
- [ ] Add circuit breaker around payment gateway calls
- [ ] Implement retry with exponential backoff
```

### 3. Triage a Bug

**Type in Copilot Chat:**
```
investigate why [describe the bug]
```

**Examples:**
```
investigate why the payment endpoint returns 500
investigate why tests are failing in test_trust_engine.py
investigate why user login is broken after the last commit
```

**What happens:**
1. Reads current diagnostics (`read/problems`)
2. Checks what changed recently (`search/changes`, `git/diff`)
3. Reads the failing function (`read/symbol`)
4. Traces the call chain (`graph/callgraph`)
5. Runs failing tests to reproduce (`test/runFailed`)
6. Sets breakpoints and inspects state (`debug/variables`)
7. Reports root cause with evidence

**Output:** A structured root cause report in the chat (not a file):
```markdown
**Root Cause:** processOrder() in order_controller.js:45 accesses req.user.id,
but req.user is undefined when the auth middleware doesn't run.

**Evidence:**
- test failure: "TypeError: Cannot read property 'id' of undefined"
- debug variables: req.user = undefined at order_controller.js:45
- git diff: commit abc123 removed the auth middleware from this route

**Fix Direction:** Re-add auth middleware to the /orders route, or add a
null check at order_controller.js:45.
```

---

## The Two-Phase Workflow

The agents are designed to work in sequence. The code graph is the foundation; log analysis consumes it.

```
 ┌──────────────────┐
 │  Your Codebase   │
 └────────┬─────────┘
          │
          │ "build code graph"
          ▼
 ┌──────────────────┐     Uses: search/files, lsp/*, graph/*,
 │  Code Graph      │     read/problems, workspace/*
 │  Builder Agent   │──────────────────────────────────────┐
 └────────┬─────────┘                                      │
          │                                                │
          │ Generates                                      │
          ▼                                                │
 ┌──────────────────┐                                      │
 │  CODE_GRAPH.md   │  ← Dependencies, call chains,       │
 │                  │    blast radius, API surface,        │
 │                  │    data flow, diagnostics            │
 └────────┬─────────┘                                      │
          │                                                │
          │ "analyze logs"                                  │
          ▼                                                ▼
 ┌──────────────────┐               ┌──────────────────┐
 │  Log Analysis    │──────────────►│  logs/*.log      │
 │  Agent           │  reads logs   │  (your app logs) │
 └────────┬─────────┘               └──────────────────┘
          │
          │ Generates
          ▼
 ┌──────────────────┐
 │ LOG_ANALYSIS.md  │  ← Incident timeline, root cause,
 │                  │    code path trace, recommendations
 └──────────────────┘
```

**Why this order matters:** The log analysis agent reads `CODE_GRAPH.md` first to get the call chain and blast radius. Without it, the agent would need to re-discover the architecture for every log entry — burning tokens and time. With the graph pre-built, log analysis is fast and precise.

---

## What Each File Does

### `.github/copilot-instructions.md`
**Purpose:** Master instructions that Copilot loads automatically on every session.
**Contains:**
- Full tool catalog (which tool for which job)
- Golden rules (cheapest tool first, read-before-write, verify-before-done)
- Token cost table for each tool
- Workflow wiring (how code graph connects to log analysis)

### `.github/chatmodes/code-graph-builder.chatmode.md`
**Purpose:** Agent definition for the code graph builder.
**Contains:**
- 7-phase analysis pipeline (orient → map → trace deps → trace calls → data flow → diagnose → generate)
- Language-agnostic search patterns (auto-detects Python, TS, Go, Java, Rust, C#)
- Output template for `CODE_GRAPH.md`
- Rules: no scripts, always cite `file:line`, token discipline

### `.github/chatmodes/log-analysis.chatmode.md`
**Purpose:** Agent definition for log analysis.
**Contains:**
- 5-phase pipeline (read logs → classify incidents → load code graph → trace to code → generate report)
- Cross-log correlation methodology (merge timelines from multiple log files)
- Incident report template
- Rules: no fabrication, every claim cites a log line AND a code line

### `.github/chatmodes/bug-triage.chatmode.md`
**Purpose:** Agent definition for bug investigation.
**Contains:**
- 4-phase scientific method (scope → investigate → verify → report)
- Platform-specific bug patterns
- Rules: evidence not speculation, debugger is ground truth

### `.github/instructions/tool-selection-guide.instructions.md`
**Purpose:** Decision tables that tell the agent which tool to use for each type of question.
**Contains:**
- "Finding Code" table (semantic vs. literal vs. symbol search)
- "Understanding Structure" table (workspace vs. graph tools)
- "Fixing Code" table (problems → edit → verify)
- Token cost ranking (hover < symbol < file)

### `.github/instructions/log-to-code-patterns.instructions.md`
**Purpose:** Reusable correlation patterns for log analysis in any language.
**Contains:**
- Pattern 1: Error Trace (single log → source code)
- Pattern 2: Cascading Failure (multiple logs → incident)
- Pattern 3: Performance Degradation (latency → bottleneck)
- Pattern 4: State Corruption (unexpected value → setter)
- Pattern 5: Security Incident (auth fail → validation gap)
- Language-specific log format reference table

### `.github/instructions/code-graph-template.instructions.md`
**Purpose:** Defines the output structure for `CODE_GRAPH.md`.
**Contains:**
- 8-section template (architecture, dependencies, call graph, API surface, data flow, blast radius, diagnostics, state machines)
- Language adaptation guide (how the template changes for Python vs. Go vs. Rust)

### `.vscode/settings.json`
**Purpose:** Controls which tools agents can use and how approvals work.
**Contains:**
- Auto-approve rules: read/search/lsp/graph/test tools are auto-approved (safe, read-only)
- Terminal allowlist: `pytest`, `npm test`, `go test`, `cargo test`, `dotnet test`, `git status/diff/log`
- Edit tools require diff preview
- Debug tools require explicit approval

### `.vscode/mcp.json`
**Purpose:** MCP (Model Context Protocol) servers that extend agent capabilities.
**Contains:**
- `log-reader`: gives the agent read access to `logs/` directory
- `context7`: gives the agent access to library documentation (FastAPI, SQLAlchemy, Express, etc.)

### `logs/app.log`
**Purpose:** Sample log file with 5 incident types for testing.
**Contains:** Payment timeout, validation error, rate limiting, auth failure, and a crash (TypeError) — spread across 20+ log lines.

---

## How It Works Under the Hood

Every agent is a markdown file that tells Copilot: **which tools to use, in what order, and how to report.** No Python scripts. No external dependencies. Just prompts that leverage tools VS Code already has.

### The Tool Chain

VS Code Copilot has built-in tools organized in families:

```
read/*        →  read a file, read a symbol, read the Problems panel
search/*      →  semantic search, text search, file search, usage search
lsp/*         →  go-to-definition, find references, hover tooltips
graph/*       →  dependency graph, call graph, data flow analysis
workspace/*   →  directory tree, file listing, editor state
git/*         →  status, diff, log
test/*        →  run tests, read results, coverage
debug/*       →  set breakpoints, inspect variables, read callstack
edit/*        →  replace text, create files, rename symbols
terminal/*    →  run commands, start servers
```

### How the Code Graph Agent Uses Them

```
Step 1: search/files  "**/*.py"           → "What files exist?"
Step 2: search/text   "^from |^import "    → "What depends on what?"
Step 3: lsp/documentSymbols on entry point  → "What's the structure?"
Step 4: graph/callgraph  root="main()"     → "Who calls what?"
Step 5: graph/dataflow  from input→sink    → "How does data flow?"
Step 6: read/problems                      → "What's broken?"
Step 7: edit/create  "CODE_GRAPH.md"       → "Write the report"
```

### How the Log Analysis Agent Uses Them

```
Step 1: read/file  "logs/app.log"          → "What errors exist?"
Step 2: read/file  "CODE_GRAPH.md"         → "What's the architecture?"
Step 3: search/text  "[error message]"     → "Where is this error in code?"
Step 4: read/symbol  [error function]      → "What does this function do?"
Step 5: graph/callgraph  root="[function]" → "What called this?"
Step 6: edit/create  "LOG_ANALYSIS.md"     → "Write the incident report"
```

---

## Supported Languages

The agents auto-detect the language from file extensions and adapt their search patterns, entry point detection, and import tracing:

| Language | File Pattern | Import Syntax | Entry Point | Test Command |
|---|---|---|---|---|
| **Python** | `*.py` | `from X import Y` / `import X` | `main.py`, `app.py` | `python3 -m pytest` |
| **TypeScript** | `*.ts` | `import { X } from "Y"` | `index.ts`, `server.ts` | `npm test` |
| **JavaScript** | `*.js` | `require("X")` / `import` | `index.js`, `app.js` | `npm test` |
| **Go** | `*.go` | `import "pkg/path"` | `main.go` | `go test ./...` |
| **Java** | `*.java` | `import pkg.Class;` | `Main.java`, `Application.java` | `mvn test` / `gradle test` |
| **Rust** | `*.rs` | `use crate::X;` / `use pkg::X;` | `main.rs` | `cargo test` |
| **C#** | `*.cs` | `using Namespace;` | `Program.cs` | `dotnet test` |

**Adding a new language:** The agent will still work — it uses semantic search (`search/codebase`) as a fallback when it doesn't know the exact import syntax. The patterns above make it faster, not required.

---

## Configuration Reference

### Tool Auto-Approve Rules (`.vscode/settings.json`)

| Setting | Value | Effect |
|---|---|---|---|
| `chat.tools.read.autoApprove` | `true` | `read/file`, `read/symbol`, `read/problems` run without asking |
| `chat.tools.search.autoApprove` | `true` | All `search/*` tools auto-run |
| `chat.tools.lsp.autoApprove` | `true` | All `lsp/*` tools auto-run |
| `chat.tools.graph.autoApprove` | `true` | All `graph/*` tools auto-run |
| `chat.tools.test.autoApprove` | `true` | `test/run`, `test/runFailed` auto-run |
| `chat.tools.edit.requireDiffPreview` | `true` | Edits show a diff before applying |
| `chat.tools.debug.requireApproval` | `true` | Debug sessions require explicit OK |
| `chat.tools.terminal.autoApprove.allow` | `[...]` | Only allowlisted commands auto-run |

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
| `context7` | Library/framework documentation lookup | Optional, for quick API checks |

---

## Troubleshooting

### Agent doesn't appear in Copilot Chat dropdown
1. Make sure `.github/chatmodes/*.chatmode.md` files are in the **workspace root** (not nested deeper)
2. Run `Ctrl+Shift+P` → **Developer: Reload Window**
3. Check that your VS Code version supports custom chat modes (1.99+)
4. Verify Copilot extension is up to date

### Agent runs but doesn't produce CODE_GRAPH.md
1. Check if Copilot has **Agent Mode** enabled (not just "Ask" mode)
2. In Copilot Chat panel, look for a tools icon — make sure tools are enabled
3. The agent needs write permission to create the file. Try: **Bypass Approvals** mode

### Log analysis says "CODE_GRAPH.md not found"
Build the code graph first:
```
# In Copilot Chat:
build code graph
```
Then run log analysis. The log agent reads the graph for context.

### Agent is slow or hits token limits
This happens on large repos (>500 files). Solutions:
1. **Scope the analysis:** Instead of `build code graph`, say `build code graph for the src/payments/ module only`
2. **Use narrower tools:** The instructions already prioritize `lsp/hover` over `read/file`, but you can reinforce: "Use lsp/documentSymbols, not read/file"
3. **Run in phases:** Build one section at a time: "map dependencies only", then "trace call chains only"

### Terminal commands are blocked
Check `.vscode/settings.json` — the allowlist might not include your test runner. Add it:
```json
"allow": [
    "... existing entries ...",
    "your-command-here *"
]
```

### MCP servers fail to start
1. Ensure Node.js is installed: `node --version`
2. Try installing the package globally first: `npm install -g @modelcontextprotocol/server-filesystem`
3. Check the VS Code Output panel → select "MCP" from the dropdown → read error messages

---

## FAQ

**Q: Do I need GitHub Copilot to use this?**
A: The chat mode files are Copilot-specific. But the instructions files (`.github/instructions/`) are valuable reference documents for any AI coding assistant. Cline, Continue, and Cursor all support similar tool-use patterns.

**Q: Does this work with Cursor, Cline, or Continue?**
A: The concepts transfer but the file format differs. Cursor uses `.cursorrules`, Cline uses `.clinerules`. You'd need to adapt the prompt content. The tool families (`search/*`, `read/*`, `lsp/*`) are semantically equivalent across tools.

**Q: Can I use this on a monorepo?**
A: Yes. The agent detects the structure automatically. For very large monorepos, scope your request: "build code graph for packages/auth-service only".

**Q: How much does it cost in Copilot tokens/requests?**
A: A full code graph build on a 50-file repo typically uses 15–25 tool calls. Log analysis uses 10–20 calls per incident. With Copilot Pro's unlimited agent mode, this is well within limits.

**Q: Can the agents modify code?**
A: The code graph and log analysis agents are read-only (they use `read/*`, `search/*`, `lsp/*`, `graph/*`). The bug triage agent can suggest edits but requires diff preview approval. Terminal and debug tools require explicit approval.

**Q: What if my logs aren't in a `logs/` directory?**
A: Tell the agent: "analyze logs in /var/log/myapp/" or "analyze logs in ./output/logs/". The agent adapts to any path.

**Q: How do I add a new agent?**
A: Create a new `.github/chatmodes/my-agent.chatmode.md` file. Follow the same structure: Role → Mission → Tools → Rules → Output Format. Copilot picks it up automatically.

**Q: Can I customize the output format of CODE_GRAPH.md?**
A: Yes. Edit `.github/instructions/code-graph-template.instructions.md` to change the sections, ordering, or format. The agent reads this template and conforms.

---

## License

MIT — use freely in any project, commercial or otherwise.
