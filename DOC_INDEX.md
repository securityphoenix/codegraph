# CodeGraph Documentation Index

Repository identity: `@colbymchenry/codegraph`, a local-first TypeScript/Node code intelligence CLI, SDK, and MCP server. It parses source with tree-sitter WASM plus a native Rust extraction kernel for routed languages, stores a SQLite graph under `.codegraph/`, resolves references and framework/dynamic-dispatch edges, then serves compact context to AI coding agents.

## Authoritative Domains

| Domain | Load |
|---|---|
| System map | `/CLAUDE.md`, `/docs/architecture/SYSTEM_OVERVIEW.md` |
| Repository layout | `/docs/architecture/REPOSITORY_MAP.md` |
| Runtime and indexing flows | `/docs/architecture/RUNTIME_FLOWS.md`, `/docs/architecture/CALL_GRAPH.md` |
| AST, parser, and graph build | `/docs/architecture/AST_PARSER_AND_GRAPH_BUILD.md`, `/docs/architecture/DATA_CONTRACTS.md` |
| Native extraction kernel | `/docs/architecture/RUNTIME_DEPENDENCY_TREE.md`, `/docs/design/rust-kernel-migration-plan.md` |
| SQLite graph contract | `/docs/architecture/DATA_CONTRACTS.md`, `/docs/architecture/DATA_LINEAGE.md` |
| MCP and agent workflows | `/docs/ai/AI_SYSTEM_MAP.md`, `/docs/ai/AGENT_WORKFLOWS.md` |
| Operations and incidents | `/docs/operations/RUNBOOK.md`, `/docs/operations/FAILURE_MODES.md`, `/docs/operations/OPERATIONAL_FAILURE_GRAPH.md` |
| Security and trust boundaries | `/docs/security/SECURITY_MODEL.md`, `/docs/security/TRUST_BOUNDARIES.md` |
| External dependencies | `/docs/integrations/EXTERNAL_INTEGRATIONS.md` |
| Local development and tests | `/docs/development/LOCAL_DEVELOPMENT.md`, `/docs/development/TESTING_STRATEGY.md` |
| Safe changes and blast radius | `/docs/development/SAFE_CHANGE_ZONES.md`, `/docs/architecture/CHANGE_BLAST_RADIUS.md` |

## Task Routing

- General repo understanding: load `/DOC_INDEX.md`, `/CLAUDE.md`, then `/docs/architecture/SYSTEM_OVERVIEW.md`.
- Parser or language coverage work: load `/docs/architecture/AST_PARSER_AND_GRAPH_BUILD.md`, `/docs/architecture/DATA_CONTRACTS.md`, `/docs/architecture/CHANGE_BLAST_RADIUS.md`, then the specific `src/extraction/languages/<language>.ts`.
- Native-kernel changes: load `/docs/architecture/RUNTIME_DEPENDENCY_TREE.md`, `/docs/design/rust-kernel-migration-plan.md`, `src/extraction/kernel/`, and `codegraph-kernel/src/`.
- MCP behavior/debugging: load `/docs/ai/AGENT_WORKFLOWS.md`, `/docs/architecture/SERVICE_TOPOLOGY.md`, and `src/mcp/`.
- Runtime incident: load `/docs/operations/RUNBOOK.md`, `/docs/operations/FAILURE_MODES.md`, `/docs/operations/OBSERVABILITY.md`.
- Schema or graph contract change: load `/docs/architecture/DATA_CONTRACTS.md`, `/docs/architecture/DATA_LINEAGE.md`, `/docs/architecture/CHANGE_BLAST_RADIUS.md`.
- Installer or release change: load `/docs/integrations/EXTERNAL_INTEGRATIONS.md`, `/docs/operations/DEPLOYMENT.md`, `BUNDLING.md`, and `CHANGELOG.md`.
- Security review: load `/docs/security/SECURITY_MODEL.md`, `/docs/security/TRUST_BOUNDARIES.md`, and `src/mcp/tools.ts`.

## High-Risk Areas

- `src/types.ts` node/edge/language arrays are wire contracts for the native kernel; append only.
- `src/extraction/tree-sitter.ts` is the fallback parser and holds cross-language special cases; small edits can affect many languages.
- `src/extraction/kernel/layout.ts` must match `codegraph-kernel/src/buffers.rs` byte-for-byte.
- `src/db/schema.sql` and `src/db/queries.ts` define graph compatibility and FTS behavior.
- `src/resolution/index.ts`, `src/resolution/name-matcher.ts`, and `src/resolution/callback-synthesizer.ts` decide whether flows connect or silently disappear.
- `src/mcp/tools.ts` controls context size, success-shaped errors, and agent behavior.

## Do Not Load Unless Needed

- `CHANGELOG.md`: release history only; large.
- `README.md`: user-facing product docs; useful for claims, not runtime truth.
- `docs/design/*`: deep historical design records; load the one named by a canonical doc.
- `site/src/content/docs/*`: public website source; load for published user docs only.
