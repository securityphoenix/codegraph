# CLAUDE.md

Lean engineering map for AI assistants. Keep deep analysis in `/docs/**`; use `/DOC_INDEX.md` as the compressed router.

## 1. System Overview

CodeGraph is a local-first code intelligence system distributed as `@colbymchenry/codegraph`. The same package provides a CLI, SDK, installer, SQLite-backed indexer, and MCP server for AI coding agents. It builds a deterministic graph from source code: files become graph nodes, tree-sitter/native extraction emits symbols and unresolved references, resolution turns references into edges, and MCP tools return compact source/context.

It does not invoke an LLM internally. AI behavior is external: Claude, Cursor, Codex, Gemini, Kiro, and similar agents call CodeGraph over MCP.

## 2. Repository Ownership and Boundaries

- Core SDK facade: `src/index.ts`.
- CLI entrypoint and lifecycle: `src/bin/codegraph.ts`.
- Extraction and parser ownership: `src/extraction/`, `src/extraction/languages/`, `codegraph-kernel/`.
- Persistence ownership: `src/db/`.
- Resolution and synthesized flow ownership: `src/resolution/`.
- Agent/MCP ownership: `src/mcp/`.
- Installer ownership: `src/installer/`.
- Public documentation site: `site/`.
- Telemetry ingest worker: `telemetry-worker/`.

## 3. High-Level Architecture

```text
CLI/MCP/SDK
  -> CodeGraph facade
  -> extraction orchestrator
  -> tree-sitter WASM or native kernel parser
  -> SQLite graph store
  -> reference resolver and synthesis passes
  -> graph/search/context APIs
  -> MCP tool output for agents
```

Canonical detail: `/docs/architecture/SYSTEM_OVERVIEW.md`.

## 4. Entry Points

- `src/bin/codegraph.ts`: user CLI, installer fallback on no args, `init`, `index`, `sync`, `serve --mcp`, `status`, query commands, upgrade/uninstall.
- `src/index.ts`: exported SDK class `CodeGraph` plus lower-level exports.
- `src/mcp/index.ts`: MCP direct/proxy/daemon startup.
- `telemetry-worker/src/index.ts`: Cloudflare Worker telemetry ingest.

## 5. Core Runtime Flow Summaries

- Fresh index: initialize `.codegraph/`, scan git/filesystem-visible files, detect language, load required grammars, parse in workers, store SQLite rows, resolve refs, synthesize dynamic edges, checkpoint/mark complete.
- Sync: detect git/file changes, re-extract changed files, delete removed files, retry affected failed refs, resolve new refs.
- MCP query: resolve project root, open or proxy to daemon, validate paths/input, query graph, format bounded context.
- Parser fallback: routed kernel languages use native buffers when available; parse-error or kernel failure falls back to WASM tree-sitter.

Canonical detail: `/docs/architecture/RUNTIME_FLOWS.md`.

## 6. High-Risk Areas

- Parser contracts: `src/extraction/tree-sitter.ts`, per-language extractors, `src/extraction/grammars.ts`, and kernel routing.
- Wire contracts: `src/types.ts`, `src/extraction/kernel/layout.ts`, `codegraph-kernel/src/buffers.rs`.
- SQLite schema and bulk-store behavior.
- Resolver precision and dynamic-dispatch synthesis.
- MCP output budgets and success-shaped error handling.
- Cross-platform filesystem, watcher, path, process, and daemon behavior.

## 7. Documentation Map

- Architecture: `/docs/architecture/`.
- AI/MCP behavior: `/docs/ai/`.
- Operations: `/docs/operations/`.
- Security: `/docs/security/`.
- Integrations: `/docs/integrations/`.
- Development: `/docs/development/`.
- Existing design history: `/docs/design/`.

## 8. Documentation Loading Guide

- General understanding: `/DOC_INDEX.md`, `/CLAUDE.md`, `/docs/architecture/SYSTEM_OVERVIEW.md`.
- Architecture tracing: add `/docs/architecture/REPOSITORY_MAP.md`, `/docs/architecture/RUNTIME_FLOWS.md`, `/docs/architecture/DEPENDENCY_GRAPH.md`.
- Parser/language work: add `/docs/architecture/AST_PARSER_AND_GRAPH_BUILD.md`, `/docs/architecture/DATA_CONTRACTS.md`, `/docs/development/TESTING_STRATEGY.md`.
- Runtime debugging: add `/docs/operations/OBSERVABILITY.md`, `/docs/operations/FAILURE_MODES.md`.
- Schema or contract changes: add `/docs/architecture/DATA_CONTRACTS.md`, `/docs/architecture/CHANGE_BLAST_RADIUS.md`.
- Integration changes: add `/docs/integrations/EXTERNAL_INTEGRATIONS.md`, `/docs/architecture/SERVICE_TOPOLOGY.md`.
- Security review: add `/docs/security/SECURITY_MODEL.md`, `/docs/security/TRUST_BOUNDARIES.md`.
- MCP/agent behavior: add `/docs/ai/AI_SYSTEM_MAP.md`, `/docs/ai/AGENT_WORKFLOWS.md`.
- Production incident response: add `/docs/operations/RUNBOOK.md`, `/docs/operations/OPERATIONAL_FAILURE_GRAPH.md`.

## 9. Recommended Reading Order

1. `/DOC_INDEX.md`
2. `/CLAUDE.md`
3. `/docs/architecture/SYSTEM_OVERVIEW.md`
4. Task-specific canonical doc from the loading guide
5. Relevant source files named by that doc

## 10. Rules for Safe Changes

- Trace the runtime path before editing; do not patch only the symptom.
- Reuse existing parser hooks, resolver helpers, and tests before adding new abstractions.
- For a new language, update detection, grammar routing, per-language extractor, parser/kernel tests, and search-quality validation together.
- For kernel changes, keep TS layout and Rust buffers in lockstep.
- For MCP changes, preserve bounded output and success-shaped recoverable errors.
- For schema changes, include migrations and verify old indexes open.

## 11. Known Unknowns

- Some `docs/design/*` files are historical and may lag current code; prefer code and this hierarchy for current behavior.
- Full per-language extraction parity is only documented where code/tests reveal it; unvalidated language/framework claims need real repo probes.
- Production deployment of the optional telemetry worker depends on Cloudflare state not present in this checkout.
