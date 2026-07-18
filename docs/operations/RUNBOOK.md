# Runbook

Parent document: /CLAUDE.md
Related documents:
- /docs/operations/FAILURE_MODES.md
- /docs/operations/OBSERVABILITY.md
- /docs/development/LOCAL_DEVELOPMENT.md

Read this when:
- You need to operate, debug, or recover a local CodeGraph install.

Purpose:
- Provide common operational actions and recovery paths.

Scope:
- Includes local CLI/MCP operation.
- Excludes hosted product operations not present in this checkout.

Common commands:

```bash
npm run build
npm test
node dist/bin/codegraph.js init
node dist/bin/codegraph.js index
node dist/bin/codegraph.js sync
node dist/bin/codegraph.js status
node dist/bin/codegraph.js serve --mcp --path <project>
```

Project recovery:

- Bad or partial index: run `codegraph index <project>` to recreate the DB.
- Database in use on Windows: stop running MCP daemon/proxy sessions, then retry.
- Not indexed in MCP: run `codegraph init` in the target project or pass `projectPath` to an initialized subproject.
- Slow/hung parser: try `CODEGRAPH_PARSE_WORKERS=1` or increase `CODEGRAPH_PARSE_TIMEOUT_MS`.
- Suspected kernel issue: run with `CODEGRAPH_KERNEL=0` to force WASM fallback.
- Daemon issue: run with `CODEGRAPH_NO_DAEMON=1` to force direct MCP mode.

Operational files:

- `.codegraph/codegraph.db`: SQLite graph.
- `.codegraph/codegraph.lock`: write lock.
- `.codegraph/daemon.log`: detached daemon log when available.
- `.codegraph/daemon.sock` or Windows named pipe metadata: daemon connection.

Known gaps / uncertainties:
- Some operational behavior is host-specific; Windows file locks and macOS/Linux watcher behavior differ.
