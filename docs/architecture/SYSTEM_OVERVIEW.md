# System Overview

Parent document: /CLAUDE.md
Related documents:
- /docs/architecture/REPOSITORY_MAP.md
- /docs/architecture/RUNTIME_FLOWS.md
- /docs/architecture/SERVICE_TOPOLOGY.md

Read this when:
- You need the system purpose and major runtime components.
- You are orienting before changing extraction, resolution, MCP, or storage.

Purpose:
- Explain what CodeGraph is and how the major pieces cooperate.

Scope:
- Includes core product architecture and runtime role.
- Excludes per-language parser mechanics; see /docs/architecture/DATA_CONTRACTS.md.

CodeGraph builds a local semantic graph from source code so AI coding agents can ask for symbols, flows, impact radius, and source context without crawling files manually. The system is deterministic: graph data comes from parser output, resolver heuristics, framework extractors, and SQLite queries, not from an internal LLM.

The same package serves four surfaces:

- CLI: project initialization, full indexing, sync, status, query, installer, upgrade, uninstall.
- SDK: `CodeGraph` class in `src/index.ts`.
- MCP server: tools for external agents, with direct/proxy/daemon runtime modes.
- Installer: writes MCP config for supported agent clients.

Architecture style:

- Local-first: per-project `.codegraph/` data, no hosted dependency for graph operation.
- Parse-store-resolve pipeline: extraction emits raw graph facts; resolution and synthesis enrich them after all files are present.
- Worker-isolated parsing: tree-sitter WASM and native-kernel work run off the main thread for bulk indexing.
- SQLite as source of runtime truth: nodes, edges, files, unresolved refs, FTS, and metadata are stored in one local database.
- Agent-facing output discipline: MCP tools return bounded, high-signal source windows and success-shaped guidance for recoverable states.

Major capabilities:

- Multi-language AST extraction through tree-sitter WASM plus custom template/resource extractors.
- Native extraction kernel for routed languages: TypeScript, TSX, JavaScript, JSX, Java, Python, and Go at this checkout.
- Framework-aware route/reference extraction.
- Cross-file import/name/reference resolution.
- Dynamic-dispatch synthesis for callback, observer, event, closure-collection, React render, and framework-specific flows.
- Search, callers/callees, impact, affected tests, file summaries, and AI context formatting.

Known gaps / uncertainties:
- Some public/design docs predate shipped kernel work; verify current behavior in source.
- Hosted product claims in README are outside this local repository's runtime.
