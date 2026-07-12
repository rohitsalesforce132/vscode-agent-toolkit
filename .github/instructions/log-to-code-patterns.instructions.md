# Log-to-Code Correlation Patterns

## Pattern Library

These patterns apply to ANY language/framework. The log format changes but the methodology doesn't.

### Pattern 1: Error Trace (Single Log → Source Code)
```
read/file log → extract error message + function name + line number
→ search/text "[error message fragment]" in codebase
→ read/symbol [function that logged the error]
→ graph/callgraph root="[function]" direction="callers" depth=3
→ Report: "Error at file:line, caused by caller chain: A→B→C"
```

### Pattern 2: Cascading Failure (Multiple Logs → Incident)
```
search/text entity_id across ALL log files
→ merge by timestamp → build timeline
→ identify FIRST error in the cascade (root trigger)
→ read/symbol [first error function]
→ graph/dataflow → trace how bad data propagated
→ Report: "Root trigger was X at T1, which caused Y at T2, Z at T3"
```

### Pattern 3: Performance Degradation
```
search/text in logs pattern="latency|duration|slow|timeout|elapsed"
→ identify requests with latency > threshold
→ search/text "[endpoint path]" in codebase → find handler
→ read/symbol [handler] → identify DB queries, external calls
→ graph/callgraph root="[handler]" direction="callees" depth=2
→ Report: "Endpoint X takes Nms because it calls A (Nms) + B (Nms)"
```

### Pattern 4: State Corruption
```
search/text entity_id in logs → find unexpected state value
→ search/text "[state value]" in codebase → find where it's set
→ read/symbol [setter function]
→ graph/dataflow from [input source] to [state variable]
→ Report: "State X got value Y because input Z wasn't validated at file:line"
```

### Pattern 5: Security Incident
```
search/text in logs pattern="unauthorized|forbidden|invalid.*key|auth.*fail"
→ search/text "[auth function]" in codebase
→ read/symbol [auth function] → find the validation logic
→ graph/dataflow from request input → auth check
→ Report: "Auth bypass because function X at file:line doesn't check Y"
```

## Language-Specific Log Patterns

| Language | Typical Log Format | Error Indicators |
|---|---|---|
| Python | `LEVEL module:line — message` | `Traceback`, `Exception`, `ERROR` |
| TypeScript/JS | `[timestamp] LEVEL: message` | `TypeError`, `UnhandledPromiseRejection` |
| Go | `LEVEL file:line: message` | `panic:`, `goroutine`, `fatal` |
| Java | `LEVEL [thread] class - message` | `Exception`, `Stacktrace`, `Caused by` |
| Rust | `LEVEL [module] message` | `panic!`, `unwrap()`, `ERROR` |
| C# | `LEVEL [source] message` | `Exception`, `StackTrace` |

## Incident Classification
| Severity | Criteria | Action |
|---|---|---|
| EMERGENCY | System down, data loss, security breach | Page on-call immediately |
| CRITICAL | Core feature broken, no workaround | Investigate within 1 hour |
| WARNING | Degraded performance, intermittent failure | Investigate within 4 hours |
| INFO | Normal operation, context for incidents | No action — timeline context |
