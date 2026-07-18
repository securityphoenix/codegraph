# Module Ownership

Parent document: /CLAUDE.md
Related documents:
- /docs/architecture/REPOSITORY_MAP.md
- /docs/development/SAFE_CHANGE_ZONES.md

Read this when:
- You need to identify the canonical owner for behavior before editing.

Purpose:
- Prevent changes from landing in the wrong layer.

Scope:
- Includes ownership boundaries for API handling, parsing, storage, resolution, integrations, workers, config, and observability.

| Concern | Owner |
|---|---|
| Public SDK API | `src/index.ts` |
| CLI command UX | `src/bin/codegraph.ts` |
| Project root and `.codegraph/` lifecycle | `src/directory.ts` |
| Project config | `src/project-config.ts` |
| Language detection and grammar loading | `src/extraction/grammars.ts` |
| Shared AST walking | `src/extraction/tree-sitter.ts` |
| Per-language syntax hooks | `src/extraction/languages/<language>.ts` |
| Template/resource extractors | `src/extraction/*-extractor.ts` |
| Native kernel routing/decoding | `src/extraction/kernel/` |
| Native extraction implementation | `codegraph-kernel/src/` |
| Parse worker pool | `src/extraction/parse-pool.ts`, `src/extraction/parse-worker.ts` |
| Store worker | `src/extraction/store-writer.ts`, `src/extraction/store-worker.ts` |
| SQLite schema/queries | `src/db/` |
| Reference resolution | `src/resolution/index.ts` |
| Import resolution | `src/resolution/import-resolver.ts` |
| Name matching | `src/resolution/name-matcher.ts` |
| Framework extraction/resolution | `src/resolution/frameworks/` |
| Dynamic-dispatch synthesis | `src/resolution/callback-synthesizer.ts` |
| Graph traversal | `src/graph/` |
| Search query parsing/ranking | `src/search/` |
| AI context formatting | `src/context/` |
| MCP protocol/process lifecycle | `src/mcp/` |
| Agent installer config | `src/installer/` |
| File watching/sync support | `src/sync/` |
| Telemetry client | `src/telemetry/` |
| Telemetry ingest | `telemetry-worker/` |
| Release/user docs | `README.md`, `BUNDLING.md`, `CHANGELOG.md`, `site/` |

Known gaps / uncertainties:
- Some older design docs mix ownership proposals with shipped implementation; verify in source.
