# Generic Code Graph Template

## Purpose
This template defines the output structure for `CODE_GRAPH.md`. 
The Code Graph Builder agent generates this file for ANY codebase.

## How to Use
1. The **Code Graph Builder** agent reads the entire repo and fills in this template.
2. The **Log Analysis** agent reads the completed `CODE_GRAPH.md` to trace log errors to code.
3. The **Bug Triage** agent reads it to understand blast radius before debugging.
4. Humans read it to understand the architecture.

## Template Sections

### 1. Architecture Overview
```
project-name/
├── entry-point.ext          ← application bootstrap
├── config/                  ← configuration files
├── module-a/                ← what this module does
│   ├── file-a.ext              key class/function
│   └── file-b.ext              key class/function
├── module-b/                ← what this module does
└── tests/                   ← test suite
```
Metadata: total files, total classes, total functions, total endpoints, language, framework.

### 2. Dependency Graph
Layer-by-layer from foundation (no deps) to surface (most deps).
For each layer, list: module → [internal dependencies].
Flag any circular dependencies.

### 3. Call Graph (Critical Chains)
Trace the most important call chains with file:line references:
- Entry point → main workflow
- API endpoint → handler → service → data layer
- Error handling paths
- Background job / scheduled task chains

### 4. API Surface
Table: Method | Path/Name | Handler | File:Line | Services Called | Auth Required

### 5. Data Flow
Source → Sink paths for critical data:
- User input → database
- External API response → internal model
- Config values → runtime behavior

### 6. Blast Radius Table
| If you change... | Files affected | Why |
|---|---|---|
| [core module] | N files | [reason] |

### 7. Diagnostics
| Severity | File | Line | Issue |
|---|---|---|---|
| ⚠️ | file.ext | N | description |

### 8. State Machines (if applicable)
If the codebase has enums, state machines, or workflow patterns, document the valid transitions.

## Language Adaptation
The agent detects the language and adapts:
- **Python:** `def`, `class`, `import`, FastAPI/Flask/Django decorators
- **TypeScript:** `export`, `interface`, `import`, Express/Nest decorators
- **Go:** `func`, `type struct`, `import`, net/http handlers
- **Java:** `public class`, `@RequestMapping`, Spring beans
- **Rust:** `fn`, `impl`, `use`, `pub struct`
- **C#:** `public class`, `[HttpPost]`, ASP.NET controllers
