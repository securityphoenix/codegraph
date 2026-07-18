# Repository Map

Parent document: /CLAUDE.md
Related documents:
- /docs/architecture/SYSTEM_OVERVIEW.md
- /docs/architecture/MODULE_OWNERSHIP.md
- /docs/development/SAFE_CHANGE_ZONES.md

Read this when:
- You need to find the owner module for a behavior.
- You are deciding where a change belongs.

Purpose:
- Map source areas by runtime responsibility and interaction.

Scope:
- Includes repository modules that participate in the shipped system.
- Excludes folder-only summaries with no runtime relationship.

Primary runtime path:

```text
src/bin/codegraph.ts
  -> src/index.ts
  -> src/extraction/*
  -> src/db/*
  -> src/resolution/*
  -> src/graph/* and src/context/*
  -> src/mcp/* or CLI output
```

Key modules:

- `src/index.ts`: `CodeGraph` facade. Owns lifecycle, locking, index/sync orchestration, resolver setup, graph query API, watcher lifecycle.
- `src/bin/`: CLI entrypoints, node-version guard, fatal handler, command supervision, uninstall.
- `src/directory.ts`: `.codegraph/` location, root discovery, safe index-root decisions, initialization checks.
- `src/project-config.ts`: `codegraph.json` include/exclude/custom extension behavior.
- `src/extraction/`: scanning, language detection, grammar loading, tree-sitter extraction, parse workers, store worker, custom template/resource extractors.
- `src/extraction/languages/`: per-language `LanguageExtractor` configs and hooks consumed by the shared extractor.
- `src/extraction/kernel/`: TypeScript loader/decoder/router for native extraction buffers.
- `codegraph-kernel/`: Rust napi extraction kernel and routed language walkers.
- `src/db/`: `node:sqlite` adapter, schema, migrations, prepared query layer, WAL valve.
- `src/resolution/`: reference resolver, import resolver, name matcher, framework resolvers, dynamic-dispatch synthesizers, resolver worker/pool.
- `src/graph/`: graph traversal and high-level graph queries.
- `src/search/`: query parsing, identifier segmentation, ranking helpers.
- `src/context/`: source/context formatting for agent consumption.
- `src/mcp/`: MCP server, transport, daemon/proxy/session model, tool handlers, liveness/watchdog behavior.
- `src/installer/`: supported agent target registry and config writers.
- `src/sync/`: watcher and worktree-change detection.
- `src/telemetry/` and `telemetry-worker/`: anonymous opt-in client events and Cloudflare Worker ingest.
- `site/`: documentation website source, not core runtime.
- `docs/design/`: historical and current design records; verify against code before relying on old status.

Start reading by task:

- Parser language support: `src/extraction/grammars.ts`, `src/extraction/tree-sitter-types.ts`, `src/extraction/tree-sitter.ts`, `src/extraction/languages/<language>.ts`.
- Kernel language support: `src/extraction/kernel/index.ts`, `src/extraction/kernel/layout.ts`, `codegraph-kernel/src/<language>.rs`.
- Resolver behavior: `src/resolution/index.ts`, `src/resolution/name-matcher.ts`, `src/resolution/import-resolver.ts`, `src/resolution/frameworks/`.
- MCP tool behavior: `src/mcp/tools.ts`, `src/mcp/engine.ts`, `src/mcp/index.ts`.
- SQLite contract: `src/db/schema.sql`, `src/db/queries.ts`, `src/db/migrations.ts`.

Known gaps / uncertainties:
- Generated `dist/` is not in source control; runtime package behavior depends on build copying SQL and WASM assets correctly.
