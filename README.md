# VS Code Agent Toolkit

> Two slash commands for VS Code Copilot Chat. `/analyze-code` builds a code graph, `/analyze-logs` uses it to investigate incidents. No scripts, no dependencies — pure agent tools.

## How It Works

```
/analyze-code              /analyze-logs <logfile>
      ↓                           ↓
  codegraph.md    ←── read ──  log-analysis.md
  (the map)                     (the report)
```

1. Run `/analyze-code` → agent maps your codebase → writes `codegraph.md`
2. Run `/analyze-logs app/logs/error.log` → agent reads `codegraph.md` + your log file → writes `log-analysis.md`
3. All PII is automatically redacted in both output files

## Quick Start

```bash
git clone https://github.com/rohitsalesforce132/vscode-agent-toolkit.git
code vscode-agent-toolkit
```

Open Copilot Chat (`Ctrl+Shift+I`), type `/analyze-code`.

## Slash Commands

| Command | Input | Output | Description |
|---|---|---|---|
| `/analyze-code` | Current workspace | `codegraph.md` | Maps architecture, dependencies, call chains, blast radius, data flow |
| `/analyze-logs` | Log file path | `log-analysis.md` | Correlates log errors with code paths using `codegraph.md` |

## File Structure

```
.vscode/
├── prompts/
│   ├── analyze-code.prompt.md    ← /analyze-code agent
│   └── analyze-logs.prompt.md    ← /analyze-logs agent
├── mcp.json                      ← MCP server config (optional)
└── settings.json                 ← Auto-approve rules
.github/
├── copilot-instructions.md       ← Global rules (PII redaction, tool tiers)
├── codegraph.md                  ← Code graph agent context
└── loganalysis.md                ← Log analysis agent context
logs/
└── app.log                       ← Sample log (for testing)
```

## What `/analyze-code` Produces

Writes `codegraph.md` at repo root with 7 sections:

1. **Architecture Overview** — annotated file tree, language, framework
2. **Dependency Graph** — layered modules, circular dependency detection
3. **Call Graph** — critical chains with file:line + **blast radius table**
4. **API Surface** — endpoints, handlers, service dependencies
5. **Data Flow** — source → sink paths
6. **Diagnostics** — linter errors, TODO/FIXME debt
7. **Risk Areas** — high coupling, dead code, security concerns

## What `/analyze-logs` Produces

Writes `log-analysis.md` at repo root:

1. **Incident Classification** — severity, entity (redacted), subsystem, window
2. **Event Timeline** — merged and ordered by timestamp
3. **Root Cause** — causal chain from user action to error
4. **Code Path** — traced using `codegraph.md` call graph (handler → service → failure)
5. **Recommended Actions** — specific fixes with file:line
6. **Recent Changes** — git commits correlated with incidents

## PII Redaction

Both agents redact PII before writing output files:

| Type | Redacted To |
|---|---|
| Emails | `[REDACTED_EMAIL]` |
| IP addresses | `[REDACTED_IP]` |
| Phone numbers | `[REDACTED_PHONE]` |
| API keys / tokens | `[REDACTED_TOKEN]` |
| User IDs | `[REDACTED_USER_ID]` |
| Session IDs | `[REDACTED_SESSION]` |
| Credit cards | `[REDACTED_CC]` |
| AWS keys | `[REDACTED_AWS_KEY]` |
| SSN | `[REDACTED_SSN]` |
| Passwords | `password = [REDACTED]` |

## Tool Surface

| Category | Tools |
|---|---|
| Search | `codebase`, `text`, `regex`, `files`, `usages`, `changes`, `symbols` |
| Reading | `file`, `symbol`, `selection`, `problems` |
| Workspace | `tree`, `files`, `openEditors`, `settings` |
| LSP | `definition`, `references`, `hover`, `implementation`, `documentSymbols` |
| Graph | `dependencies`, `callgraph`, `dataflow`, `context` |
| Git (read) | `status`, `diff`, `log` |
| Edit (Phase 7 only) | `create`, `file` — **only** for `codegraph.md` / `log-analysis.md` |

**Blocked:** `terminal/*`, `debug/*`, `test/*`, `edit/delete`, `edit/rename`, `git/commit`, `git/checkout`

## Language Support

Python, TypeScript, JavaScript, Go, Java, Rust, C#, Ruby, PHP, Kotlin, C, C++

## License

MIT
