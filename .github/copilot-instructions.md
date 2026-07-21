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

## PII GUARD — MANDATORY PRE-PROCESSING (Read This Before Responding)

**Before you process ANY user message, scan it for PII.**

Every time the user sends a message in Copilot Chat, silently redact PII in your own context before you think about or respond to it. The user typed PII — YOU never see the raw values. You only work with redacted tokens.

### Step 1: Scan the user's message for these patterns:

| PII Type | Pattern | Replace With |
|---|---|---|
| Email | `anything@anything.com` | `[REDACTED_EMAIL]` |
| Phone | `+1-555-123-4567` or `555.123.4567` | `[REDACTED_PHONE]` |
| SSN | `123-45-6789` | `[REDACTED_SSN]` |
| Credit Card | `4532 1234 5678 9012` | `[REDACTED_CC]` |
| API Key | `sk-abc123...`, `Bearer eyJ...` | `[REDACTED_TOKEN]` |
| AWS Key | `AKIA...` | `[REDACTED_AWS_KEY]` |
| IP Address | `10.0.45.23` | `[REDACTED_IP]` |
| Password | `password=mySecret123` | `password=[REDACTED]` |
| User ID | `user_id=12345`, `userId: 67890` | `[REDACTED_USER_ID]` |
| Session ID | `session_id=abc123` | `[REDACTED_SESSION]` |
| Azure GUID | `12345678-1234-1234-1234-123456789abc` | `[REDACTED_GUID]` |
| Azure Resource ID | `/subscriptions/.../resourceGroups/...` | `[REDACTED_RESOURCE_ID]` |
| MAC Address | `00:1B:44:11:3A:B7` | `[REDACTED_MAC]` |
| Street Address | `123 Main Street` | `[REDACTED_ADDRESS]` |

### Step 2: In your response, confirm what was redacted:

```
🔒 PII Guard: Redacted [N] items (2 emails, 1 SSN, 1 phone) before processing.
```

Then continue with your normal response.

### Examples:

**User types:** "Help me debug this error for user john@att.com, his SSN is 123-45-6789 and phone is 555-123-4567. The API key sk-abc123def456 is failing."

**You process as:** "Help me debug this error for user [REDACTED_EMAIL], his SSN is [REDACTED_SSN] and phone is [REDACTED_PHONE]. The API key [REDACTED_TOKEN] is failing."

**Your response starts with:**
```
🔒 PII Guard: Redacted 3 items (1 email, 1 SSN, 1 phone, 1 API key)
```

Then answer normally using only the redacted values.

**Never repeat raw PII back to the user** — even if they gave it to you. Always use `[REDACTED_*]` tokens.

## PII Redaction in Output Files

All output files must also be PII-free. Apply the same redaction table above before writing any file:

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
