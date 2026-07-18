# Observability

Parent document: /CLAUDE.md
Related documents:
- /docs/operations/RUNBOOK.md
- /docs/operations/FAILURE_MODES.md
- /TELEMETRY.md

Read this when:
- You need logs, debug hooks, metrics, or troubleshooting signals.

Purpose:
- List observable signals and debugging controls.

Scope:
- Includes local runtime observability and optional telemetry.
- Excludes hosted dashboards unless present outside this repo.

Signals:

- CLI progress phases: scanning, parsing, storing, resolving, linking.
- `codegraph status`: index status, backend, file/node/edge counts.
- `.codegraph/daemon.log`: detached daemon stdout/stderr.
- SQLite row counts and metadata in `.codegraph/codegraph.db`.
- Test fixtures and probes under `__tests__/` and selected `scripts/`.

Useful env vars:

- `CODEGRAPH_KERNEL=0`: disable native kernel.
- `CODEGRAPH_KERNEL_LANGS=<langs|all>`: override routed kernel languages.
- `CODEGRAPH_KERNEL_DEBUG=1`: explain kernel loading/routing failures.
- `CODEGRAPH_PARSE_WORKERS=N`: parser worker pool size.
- `CODEGRAPH_PARSE_TIMEOUT_MS=N`: parser timeout.
- `CODEGRAPH_NO_STORE_WORKER=1`: disable store worker.
- `CODEGRAPH_NO_WAL_DEFER=1`: disable WAL defer.
- `CODEGRAPH_NO_FAST_INIT=1`: disable fast-init DB mode.
- `CODEGRAPH_NO_DAEMON=1`: force direct MCP.
- `CODEGRAPH_SYNTH_TIMINGS=1`: print resolution/synthesis timings.

Telemetry:

- Client telemetry is anonymous and consent-gated.
- `TELEMETRY.md` is the user-facing policy.
- `docs/design/telemetry.md` and `telemetry-worker/README.md` contain implementation details.

Known gaps / uncertainties:
- No repository-defined production dashboard or alerting configuration was found.
