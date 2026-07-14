# Copilot Agent Instructions

## Three Slash Commands

| Command | What It Does | Output |
|---|---|---|
| `/analyze-code` | Maps any codebase: dependencies, call chains, blast radius, data flow | Writes **`codegraph.md`** |
| `/analyze-logs` | Analyzes application logs using `codegraph.md` as the map | Writes **`log-analysis.md`** |
| `/analyze-aks` | Diagnoses AKS infrastructure: pods, nodes, networking, OOM, ingress | Writes **`aks-analysis.md`** |

## Workflow

```
/analyze-code  →  codegraph.md
                      ↓
/analyze-logs <logfile>  →  log-analysis.md

/analyze-aks <aks-logfile>  →  aks-analysis.md
```

`/analyze-logs` requires `codegraph.md`. If missing, stops and tells user to run `/analyze-code` first.
`/analyze-aks` works standalone or with `codegraph.md` for deeper tracing.

## Allowed Tools

| Family | Tools | Auto-Approved |
|---|---|---|
| `search/*` | codebase, text, regex, files, usages, changes, symbols | ✅ |
| `read/*` | file, symbol, selection, problems | ✅ |
| `workspace/*` | tree, files, openEditors, settings | ✅ |
| `lsp/*` | definition, declaration, typeDefinition, references, hover, implementation, documentSymbols, workspaceSymbols | ✅ |
| `graph/*` | dependencies, callgraph, controlflow, dataflow, context, knowledge | ✅ |
| `git` (read) | status, diff, log | ✅ |
| `edit/create` | Write output `.md` files (Phase 7 only) | ✅ |
| `edit/file` | Overwrite output `.md` files if they exist | ✅ |

## Blocked Tools

| Family | Tools | Status |
|---|---|---|
| `edit/file` | On any file OTHER than `codegraph.md` / `log-analysis.md` / `aks-analysis.md` | ❌ BLOCKED |
| `edit/delete` | Any file | ❌ BLOCKED |
| `edit/rename` | Any file | ❌ BLOCKED |
| `terminal/*` | run, background, kill | ❌ BLOCKED |
| `debug/*` | start, continue, step, variables, callstack, breakpoints | ❌ BLOCKED |
| `test/*` | run, runFailed, results, coverage | ❌ BLOCKED |
| `git` (write) | commit, checkout | ❌ BLOCKED |

## PII Redaction (Mandatory)

All output files must be PII-free. Scan and redact before writing:

| PII Type | Redact To |
|---|---|
| Email addresses | `[REDACTED_EMAIL]` |
| IP addresses | `[REDACTED_IP]` |
| Phone numbers | `[REDACTED_PHONE]` |
| Credit card numbers | `[REDACTED_CC]` |
| API keys / tokens / passwords | `[REDACTED_TOKEN]` / `password = [REDACTED]` |
| User IDs | `[REDACTED_USER_ID]` |
| Session IDs | `[REDACTED_SESSION]` |
| AWS keys (AKIA...) | `[REDACTED_AWS_KEY]` |
| SSN (xxx-xx-xxxx) | `[REDACTED_SSN]` |
| Azure Resource IDs | `[REDACTED_RESOURCE_ID]` |
| Azure GUIDs / tenant IDs | `[REDACTED_GUID]` |
| ACR login servers | `[REDACTED_ACR]` |
| MAC addresses | `[REDACTED_MAC]` |

## Tool Selection Priority

```
lsp/hover           → type signature         (~50 tokens)
lsp/documentSymbols → file outline            (~100 tokens)
read/symbol         → one function body       (~200 tokens)
search/usages       → all call sites          (~300 tokens)
read/file           → entire file             (~500+ — AVOID on big files)
graph/callgraph     → transitive call chain   (~500 tokens)
graph/dependencies  → module relationships    (~300 tokens)
```

## Golden Rules

1. **Write to files, not chat.** Each command writes its own `.md` file at repo root.
2. **Read-only until Phase 7.** Use only search/read/lsp/graph tools during analysis.
3. **`codegraph.md` is the bridge.** `/analyze-logs` reads it. If missing, stop.
4. **Redact all PII.** No emails, keys, tokens, IPs, GUIDs, resource IDs in output.
5. **Cheapest tool first.** `lsp/hover` → `read/symbol` → `read/file`.
6. **Always cite file:line.** Every claim must have a verifiable source.
7. **No fabrication.** If a tool returns empty, say "not found."
8. **Check YAML for AKS.** 80% of AKS issues are in deployment manifests, not application code.
