# VS Code Agent Toolkit — Code Graph + Log Analysis

> A reusable, language-agnostic agent system for VS Code Copilot. Drop into any repo and get AI agents that build code graphs and analyze logs using built-in tools.

## What This Gives You

| Agent | What It Does | Trigger |
|---|---|---|
| **Code Graph Builder** | Maps any codebase: dependencies, call chains, data flow, blast radius | "build code graph" |
| **Log Analysis** | Correlates logs with code paths to find root causes | "analyze logs" |
| **Bug Triage** | Investigates bugs via diagnostics, debug, tests | "investigate bug" |

## Quick Start

### Option A: Use as-is (demo with sample logs)
1. Open this folder in VS Code
2. Open Copilot Chat (`Ctrl+Shift+I`)
3. Type: **"build code graph"** → agent generates `CODE_GRAPH.md`
4. Type: **"analyze logs"** → agent reads `logs/app.log`, correlates with code, generates `LOG_ANALYSIS.md`

### Option B: Drop into your own repo
```bash
# Copy the .github/ and .vscode/ folders into your project
cp -r .github /path/to/your/project/
cp -r .vscode /path/to/your/project/
mkdir -p /path/to/your/project/logs
# Add your log files to logs/
# Open in VS Code → Copilot Chat → "build code graph"
```

### Option C: Git submodule
```bash
git submodule add https://github.com/rohitsalesforce132/vscode-agent-toolkit.git .agent-toolkit
# Reference the chat modes and instructions from your project's copilot-instructions.md
```

## How It Works

```
┌─────────────────────────────────────────────────────────────┐
│                    VS Code Copilot Chat                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  User: "build code graph"                                    │
│    │                                                         │
│    ▼                                                         │
│  ┌──────────────────────┐                                   │
│  │ Code Graph Builder   │  Uses: search/files, lsp/*,       │
│  │ Agent                │  graph/*, read/problems           │
│  └──────────┬───────────┘                                   │
│             │                                               │
│             ▼                                               │
│  ┌──────────────────────┐                                   │
│  │ CODE_GRAPH.md        │  ← Architecture, dependencies,   │
│  │ (generated)          │    call chains, blast radius      │
│  └──────────┬───────────┘                                   │
│             │                                               │
│             │ User: "analyze logs"                           │
│             ▼                                               │
│  ┌──────────────────────┐    ┌──────────────────────┐       │
│  │ Log Analysis Agent   │───►│ logs/*.log           │       │
│  │                      │    │ (your app logs)      │       │
│  │ Reads CODE_GRAPH.md  │    └──────────────────────┘       │
│  │ + log files          │                                   │
│  │ → traces errors      │                                   │
│  │   to code paths      │                                   │
│  └──────────┬───────────┘                                   │
│             │                                               │
│             ▼                                               │
│  ┌──────────────────────┐                                   │
│  │ LOG_ANALYSIS.md      │  ← Incident reports with          │
│  │ (generated)          │    root cause + code path         │
│  └──────────────────────┘                                   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## File Structure

```
vscode-agent-toolkit/
├── .github/
│   ├── copilot-instructions.md              ← Master agent instructions (auto-loaded by Copilot)
│   ├── chatmodes/
│   │   ├── code-graph-builder.chatmode.md   ← Agent: builds CODE_GRAPH.md from any repo
│   │   ├── log-analysis.chatmode.md         ← Agent: correlates logs with code graph
│   │   └── bug-triage.chatmode.md           ← Agent: root cause investigation
│   └── instructions/
│       ├── tool-selection-guide.md          ← Decision table: which tool for which question
│       ├── log-to-code-patterns.md          ← 5 correlation patterns for any language
│       └── code-graph-template.md           ← Output template for CODE_GRAPH.md
├── .vscode/
│   ├── mcp.json                             ← MCP servers (log reader, Context7 docs)
│   └── settings.json                        ← Tool auto-approve rules, terminal allowlist
├── logs/
│   └── app.log                              ← Sample log with 5 incident types
└── README.md                                ← This file
```

## Tools Used (from VS Code Agent Tools Reference)

| Tool Family | Specific Tools | Purpose |
|---|---|---|
| `search/*` | codebase, text, files, usages, changes | Find code, symbols, recent changes |
| `read/*` | file, symbol, problems | Read source + diagnostics |
| `workspace/*` | tree, files | Project structure |
| `lsp/*` | definition, references, hover, documentSymbols | Compiler-grade navigation |
| `graph/*` | dependencies, callgraph, dataflow | Structural relationships |
| `git/*` | status, diff, log | Version control state |
| `test/*` | run, runFailed, results | Verification |
| `debug/*` | start, variables, callstack | Runtime inspection |
| `edit/*` | replace, create, rename | Code changes |
| `terminal/*` | run, background | Shell execution |

## Works With Any Language

The agents detect the language from file extensions and adapt:

| Language | Import Pattern | Entry Point | Test Command |
|---|---|---|---|
| Python | `from X import Y` | `main.py` | `python3 -m pytest` |
| TypeScript | `import { X } from Y` | `index.ts` | `npm test` |
| Go | `import "pkg"` | `main.go` | `go test ./...` |
| Java | `import pkg.Class` | `Main.java` | `mvn test` |
| Rust | `use crate::X` | `main.rs` | `cargo test` |
| C# | `using Namespace` | `Program.cs` | `dotnet test` |

## License

MIT — use freely in any project.
