# VS Code Agent Toolkit

> Minimal, self-contained VS Code Copilot agents for code graph analysis and log correlation. Open any workspace, type `/analyze-code` or `/analyze-logs`, and get structured intelligence — no scripts, no dependencies, no configuration.

## What It Does

| Command | Purpose | How |
|---|---|---|
| `/analyze-code` | Build a code graph from any codebase | Uses `search/*`, `lsp/*`, `graph/*` tools to map architecture, dependencies, call chains, blast radius, data flow |
| `/analyze-logs` | Analyze logs and correlate with code | Reads log files, traces errors back to source functions, cross-references with recent git changes |

Both are **read-only** — they never modify your files, run commands, or execute tests. Output goes directly into the Copilot Chat.

## How to Use

### 1. Clone into your workspace

```bash
git clone https://github.com/rohitsalesforce132/vscode-agent-toolkit.git
# Open the folder in VS Code
code vscode-agent-toolkit
```

### 2. Open Copilot Chat

Press `Ctrl+Shift+I` (or `Cmd+Shift+I` on Mac) to open GitHub Copilot Chat.

### 3. Use the slash commands

Type `/analyze-code` to map the current workspace's architecture.
Type `/analyze-logs` to investigate log files.

## File Structure

```
.vscode/
├── prompts/
│   ├── analyze-code.prompt.md    ← /analyze-code slash command
│   └── analyze-logs.prompt.md    ← /analyze-logs slash command
├── mcp.json                      ← MCP server config (optional)
└── settings.json                 ← Auto-approve rules for read-only tools
.github/
├── copilot-instructions.md       ← Global agent rules (read-only, tool priority)
├── codegraph.md                  ← Code graph agent context
└── loganalysis.md                ← Log analysis agent context
logs/
└── app.log                       ← Sample log file (for testing)
```

## Tool Surface

These agents use 56 tools across 11 categories — all read-only:

| Category | Tools Used |
|---|---|
| Code Reading | `read/file`, `read/symbol`, `read/selection`, `read/problems` |
| Search | `search/codebase`, `search/text`, `search/regex`, `search/files`, `search/usages`, `search/changes`, `search/symbols` |
| Workspace | `workspace/tree`, `workspace/files`, `workspace/openEditors`, `workspace/settings` |
| Language Server | `lsp/definition`, `lsp/references`, `lsp/hover`, `lsp/implementation`, `lsp/documentSymbols`, `lsp/workspaceSymbols` |
| Graph & Knowledge | `graph/dependencies`, `graph/callgraph`, `graph/controlflow`, `graph/dataflow`, `graph/context`, `graph/knowledge` |
| Git (read-only) | `git/status`, `git/diff`, `git/log` |

**Blocked:** `edit/*`, `terminal/*`, `debug/*`, `test/*`, `git/commit`, `git/checkout`

## `/analyze-code` — What It Produces

A structured code graph printed to chat with 7 sections:

1. **Architecture Overview** — annotated file tree, language/framework, file count
2. **Dependency Graph** — layered module dependencies, circular dependency detection
3. **Call Graph** — critical call chains with file:line references
4. **Blast Radius Table** — "if you change X, N files break because..."
5. **API Surface** — endpoints/handlers with their service dependencies
6. **Data Flow** — source → sink paths (input → validation → storage)
7. **Diagnostics** — existing linter/compiler errors + TODO/FIXME debt

## `/analyze-logs` — What It Produces

A structured incident report printed to chat:

1. **Incident Classification** — severity, entity, subsystem, time window
2. **Event Timeline** — merged across all log files, ordered by timestamp
3. **Root Cause** — causal chain from user action to error
4. **Code Path** — handler → service → failure point with file:line
5. **Recommended Actions** — specific fixes with file:line references
6. **Correlation Matrix** — cross-log entity tracking
7. **Change Correlation** — recent git changes that may have caused incidents

## Language Support

Works with any codebase — auto-detects language from file extensions:

Python, TypeScript, JavaScript, Go, Java, Rust, C#, Ruby, PHP, Kotlin, C, C++

## Testing the Agents

1. Open any project in VS Code
2. Type `/analyze-code` in Copilot Chat → get a full code graph
3. Drop log files in a `logs/` directory
4. Type `/analyze-logs` → get incident reports correlated with code

## How It Works

- `.vscode/prompts/*.prompt.md` — These register as **slash commands** in VS Code Copilot Chat. The YAML frontmatter `description` becomes the autocomplete tooltip.
- `.github/copilot-instructions.md` — Global rules injected into every agent session (read-only, tool priority)
- `.vscode/settings.json` — Auto-approve rules for read-only tools so the agent runs without permission prompts
- `.github/codegraph.md` + `.github/loganalysis.md` — Extended agent context for deeper analysis

## License

MIT
