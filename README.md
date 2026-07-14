# VS Code Agent Toolkit

> Three slash commands for VS Code Copilot Chat. Build code graphs, analyze application logs, and diagnose AKS infrastructure — no scripts, no dependencies, pure agent tools.

## How It Works

```
/analyze-code              /analyze-logs <logfile>          /analyze-aks <aks-logfile>
      ↓                           ↓                                ↓
  codegraph.md    ←── read ──  log-analysis.md               aks-analysis.md
  (the map)                    (app incidents)               (infra incidents)
```

## Quick Start

```bash
git clone https://github.com/rohitsalesforce132/vscode-agent-toolkit.git
code vscode-agent-toolkit
```

Open Copilot Chat (`Ctrl+Shift+I`), type `/` to see all three commands.

## Slash Commands

| Command | Input | Output | Description |
|---|---|---|---|
| `/analyze-code` | Current workspace | `codegraph.md` | Maps architecture, dependencies, call chains, blast radius |
| `/analyze-logs` | App log file path | `log-analysis.md` | Correlates app log errors with code paths |
| `/analyze-aks` | AKS log file path | `aks-analysis.md` | Diagnoses K8s pods, nodes, networking, OOM, ingress |

## File Structure

```
.github/
├── prompts/
│   ├── analyze-code.prompt.md     ← /analyze-code
│   ├── analyze-logs.prompt.md     ← /analyze-logs
│   └── analyze-aks.prompt.md      ← /analyze-aks
├── copilot-instructions.md        ← Global rules (PII redaction, tool tiers)
├── codegraph.md                   ← Code graph agent context
└── loganalysis.md                 ← Log analysis agent context
.vscode/
├── mcp.json                       ← MCP server config (optional)
└── settings.json                  ← Auto-approve rules
logs/
├── app.log                        ← Sample app log
└── aks-infra.log                  ← Sample AKS infrastructure log
```

## `/analyze-aks` — AKS Infrastructure Analyzer

Diagnoses across 6 Kubernetes layers:

| Layer | What It Detects |
|---|---|
| **Control Plane** | etcd timeouts, API server issues, scheduler failures |
| **Node / kubelet** | MemoryPressure, DiskPressure, PIDPressure, evictions, node NotReady |
| **Network** | DNS failures, SNAT exhaustion, NetworkPolicy blocks, connection refused |
| **Workload / Pod** | CrashLoopBackOff, OOMKilled, ImagePullBackOff, FailedScheduling |
| **Storage** | FailedMount, PVC pending, CSI driver errors |
| **Ingress** | NGINX/Traefik/App Gateway 502/503/504, upstream timeouts, TLS errors |

Recognizes Azure-specific patterns: ACR auth failures, managed identity errors, subnet exhaustion, load balancer stuck pending, VM reboots.

Includes KQL (Kusto) table reference for Azure Monitor / Log Analytics exports.

## PII Redaction

All agents redact PII before writing output files:

| Type | Redacted To |
|---|---|
| Emails | `[REDACTED_EMAIL]` |
| IP addresses | `[REDACTED_IP]` |
| Phone numbers | `[REDACTED_PHONE]` |
| API keys / tokens | `[REDACTED_TOKEN]` |
| User IDs | `[REDACTED_USER_ID]` |
| Azure Resource IDs | `[REDACTED_RESOURCE_ID]` |
| Azure GUIDs / tenant IDs | `[REDACTED_GUID]` |
| ACR login servers | `[REDACTED_ACR]` |
| MAC addresses | `[REDACTED_MAC]` |
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
| Edit (Phase 7 only) | `create`, `file` — **only** for output `.md` files |

**Blocked:** `terminal/*`, `debug/*`, `test/*`, `edit/delete`, `edit/rename`, `git/commit`, `git/checkout`

## Language Support

Python, TypeScript, JavaScript, Go, Java, Rust, C#, Ruby, PHP, Kotlin, C, C++

## License

MIT
