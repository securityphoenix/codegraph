# Failure Modes

Parent document: /CLAUDE.md
Related documents:
- /docs/operations/RUNBOOK.md
- /docs/operations/OPERATIONAL_FAILURE_GRAPH.md
- /docs/architecture/RUNTIME_DEPENDENCY_TREE.md

Read this when:
- A command, parser, index, DB, or MCP server is failing.

Purpose:
- Map symptoms to likely causes and containment.

Scope:
- Includes common local operational failures.
- Excludes external agent model behavior except MCP usage symptoms.

| Symptom | Probable cause | Containment / recovery |
|---|---|---|
| `CodeGraph not initialized` | No `.codegraph/codegraph.db` at resolved root | Run `codegraph init`, or pass `projectPath` to initialized subproject |
| Empty or tiny index | Project root is gitignored parent, excludes, unsupported extensions, or wrong root | Check `codegraph status`, `codegraph.json`, `.gitignore`, and root path |
| Parser crashes/OOM | Large WASM grammar heap, bad file, Node/V8 issue | Kernel fallback if available, `CODEGRAPH_PARSE_WORKERS=1`, timeout tuning, inspect errors |
| Kernel silently unused | Missing `.node`, ABI mismatch, unsupported language, `defer:` parse errors | Use `CODEGRAPH_KERNEL_DEBUG=1`; fallback is expected |
| `database is locked` | Concurrent writer or stuck daemon | Stop daemon/agent sessions, retry; lock file protects writes |
| MCP no tools/slow attach | Daemon startup race or host launch quirk | Pre-warm daemon for evals; use direct mode for isolation |
| Agent stops using CodeGraph | Tool returned hard errors or insufficient context | Preserve success-shaped recoverable errors and source sufficiency |
| High WAL/disk IO | Bulk writes/checkpoint behavior | WAL valve/fast-init should contain; inspect env kill switches |
| Wrong flow answer | Missing/false resolver or synthesized edge | Probe graph, inspect unresolved refs/heuristic edges, validate on control repo |
| Installer corrupts config | Target writer failed preservation/idempotency | Run installer target tests; restore from user config backup if available |

Known gaps / uncertainties:
- Some failures require inspecting local `.codegraph/` DB state; this doc does not enumerate every SQL probe.
