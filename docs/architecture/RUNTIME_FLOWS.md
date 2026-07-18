# Runtime Flows

Parent document: /CLAUDE.md
Related documents:
- /docs/architecture/AST_PARSER_AND_GRAPH_BUILD.md
- /docs/architecture/CALL_GRAPH.md
- /docs/architecture/RUNTIME_DEPENDENCY_TREE.md
- /docs/operations/FAILURE_MODES.md

Read this when:
- You are debugging startup, indexing, sync, extraction, resolution, or MCP tool output.
- You need the step order before editing code.

Purpose:
- Document core runtime flows step by step.

Scope:
- Includes CLI/SDK/MCP flows and parser/resolver lifecycle.
- Excludes detailed schema fields; see /docs/architecture/DATA_CONTRACTS.md.

## Fresh Init And Full Index

1. `codegraph init` enters through `src/bin/codegraph.ts`.
2. CLI loads `CodeGraph` lazily and calls `CodeGraph.init(projectRoot, { index })`.
3. `CodeGraph.init` calls `initGrammars()`, creates `.codegraph/`, initializes SQLite, and optionally calls `indexAll`.
4. `CodeGraph.indexAll` acquires an in-process mutex and cross-process `.codegraph/codegraph.lock`.
5. Bulk-index settings are applied: metadata `index_state=indexing`, FTS bulk load, WAL/fast-init tuning, segment vocab clear.
6. `ExtractionOrchestrator.indexAll` scans files using git-visible files when possible, falling back to filesystem scan.
7. Scanner applies defaults, `.gitignore`, embedded repo rules, `codegraph.json` include/exclude/custom extension rules, max file size, UTF-8 validity, generated-file handling.
8. Languages are detected from extension/content, then required tree-sitter WASM grammars are loaded lazily.
9. `ParseWorkerPool` parses files in worker threads. Routed kernel languages may return raw buffers instead of decoded objects.
10. `StoreWriter` may write fresh-DB bundles in a store worker; otherwise the orchestrator stores finalized bundles on the main thread.
11. `CodeGraph.indexAll` reinitializes the resolver after files exist, runs framework `postExtract`, resolves pending refs, then runs chained-call and deferred inherited-member passes.
12. Resolution also invokes synthesis passes such as callback/dynamic-dispatch edges.
13. Maintenance checkpoints/analyze run, FTS is rebuilt, metadata marks completion.

## Per-File Parse Flow

1. File language is detected in `src/extraction/grammars.ts`.
2. `extractFromSource` routes special formats first: Svelte, Vue, Astro, Liquid, Razor, XML/MyBatis, CFML/CFScript, DFM/FMX, file-level-only YAML/Twig/properties.
3. For ordinary supported languages, `tryKernelExtract` or `tryKernelExtractRaw` runs if the language is routed and the kernel is available.
4. Kernel `defer:` or failure falls back to `TreeSitterExtractor`.
5. `TreeSitterExtractor.extract` applies language `preParse`, parses, creates a file node, optional package namespace, walks AST nodes, emits nodes/edges/unresolved refs, then flushes function-as-value and value-reference candidates.
6. Framework `extract()` hooks may append route nodes and references after parser extraction.

## Sync Flow

1. `CodeGraph.sync` acquires the same lock discipline as full index.
2. Change detection uses git status when possible, otherwise filesystem reconciliation.
3. Removed files are deleted from DB; added/modified files are re-extracted and stored.
4. New unresolved refs are resolved.
5. Failed refs whose tail now matches changed symbols are retried.
6. Synthesis and maintenance run as needed.

## MCP Startup Flow

1. Agent launches `codegraph serve --mcp`.
2. `MCPServer.start` chooses direct, proxy, or daemon mode.
3. If daemon is enabled and a `.codegraph/` root exists, clients share a detached daemon over Unix socket or Windows named pipe.
4. The proxy handles stdio-to-socket transport and host PPID supervision.
5. The daemon owns long-lived CodeGraph, watcher, SQLite connection, and query pool state.
6. If daemon setup fails or no index root exists, direct stdio mode is used.

## MCP Tool Flow

1. Tool input is length/path validated in `src/mcp/tools.ts`.
2. Project root is resolved from `projectPath` or nearest `.codegraph/`.
3. Recoverable conditions such as not indexed return success-shaped guidance, not `isError`.
4. Query methods read SQLite through `CodeGraph`/`QueryBuilder`.
5. Results are ranked, budgeted, and formatted to prevent context bloat.

Known gaps / uncertainties:
- Exact CLI command branches are numerous; inspect `src/bin/codegraph.ts` for command-specific output semantics.
