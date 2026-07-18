# Runtime Dependency Tree

Parent document: /CLAUDE.md
Related documents:
- /docs/architecture/RUNTIME_FLOWS.md
- /docs/operations/FAILURE_MODES.md
- /docs/operations/OPERATIONAL_FAILURE_GRAPH.md

Read this when:
- A runtime flow fails and you need to know which dependencies must be healthy.
- You are changing startup order, workers, kernel, or SQLite behavior.

Purpose:
- List startup, runtime, and downstream dependencies for critical flows.

Scope:
- Includes dependency requirements for CLI, index, sync, MCP, and parser flows.
- Excludes third-party service details; see /docs/integrations/EXTERNAL_INTEGRATIONS.md.

## CLI Startup

Required:

- Supported Node version: `>=20 <25`, with hard blocks in `src/bin/node-version-check.ts`.
- V8 WASM runtime flags through `relaunchWithWasmRuntimeFlagsIfNeeded`.
- Package assets copied to `dist/`: `src/db/schema.sql` and `src/extraction/wasm/*.wasm`.

Optional:

- `git` for fast tracked/untracked scans and change detection.
- Rust/native kernel prebuild for routed language speed.

## Full Index

Must be healthy:

- Project root writable enough to create/update `.codegraph/`.
- SQLite DB openable via `node:sqlite`.
- File lock available.
- File scanner can enumerate files.
- Required grammar WASM can load, or language files will emit parser errors.
- Parse workers can spawn.
- Store path can write DB.

Best-effort/fallback:

- Native kernel absent or failing falls back to WASM.
- Git scan failure falls back to filesystem scan.
- Store worker unavailable falls back to main-thread store.
- Some malformed files produce per-file errors without aborting whole index.

## Sync

Must be healthy:

- Existing `.codegraph/codegraph.db`.
- File watcher or manual sync trigger.
- DB delete/reinsert behavior for changed files.
- Resolver retry path for changed refs.

Risk:

- Interrupted sync can leave pending refs; resolver sweep and failed-ref retry logic are intended to recover.

## MCP

Must be healthy:

- MCP stdio transport or daemon socket/pipe.
- Project root resolution.
- SQLite read connection.
- Tool input validation.

Best-effort/fallback:

- Daemon failure falls back to direct mode.
- Not-indexed roots return guidance instead of hiding tools.
- Liveness/PPID watchdogs kill stuck/orphaned processes.

## Parser/Kernel

Must be healthy:

- `web-tree-sitter` initialized for WASM path.
- Language grammar loaded before parser use.
- Kernel ABI and kind tables match if native path is used.

Fallback:

- Kernel `defer:` and kernel failures return null to use the WASM extractor.

Known gaps / uncertainties:
- Performance envelopes vary heavily by storage, CPU count, cgroup memory, and repository language mix.
