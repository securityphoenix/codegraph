# Change Blast Radius

Parent document: /CLAUDE.md
Related documents:
- /docs/development/SAFE_CHANGE_ZONES.md
- /docs/architecture/DATA_CONTRACTS.md
- /docs/development/TESTING_STRATEGY.md

Read this when:
- You need to estimate what a change can break.
- You are touching parser, schema, resolver, MCP, or installer behavior.

Purpose:
- Identify high-impact modules and likely direct/indirect failures.

Scope:
- Includes change-risk guidance for core modules.
- Excludes PR-specific review findings.

High blast radius:

- `src/types.ts`: changing kind/language arrays can corrupt kernel decode, query validation, schema semantics, tests, and public API compatibility.
- `src/extraction/grammars.ts`: extension/language routing changes can silently move files to the wrong extractor or remove them from indexing.
- `src/extraction/tree-sitter.ts`: shared AST walk and special cases affect most languages and graph density.
- `src/extraction/kernel/layout.ts` and `codegraph-kernel/src/buffers.rs`: ABI mismatch causes fallback or misdecode.
- `src/db/schema.sql`, `src/db/queries.ts`, `src/db/migrations.ts`: can break existing indexes, FTS, resolution persistence, and MCP reads.
- `src/resolution/index.ts`: affects every cross-file edge and unresolved-ref lifecycle.
- `src/resolution/name-matcher.ts`: false positives poison flow answers; false negatives force agents back to Read/Grep.
- `src/resolution/callback-synthesizer.ts`: edge explosion or partial flow bridging can degrade agent behavior.
- `src/mcp/tools.ts`: output budget/shape controls whether agents trust and keep using CodeGraph.
- `src/mcp/index.ts` daemon/proxy: process lifecycle regressions can orphan daemons or break multi-client sharing.
- `src/installer/targets/*`: can corrupt user agent config; must preserve comments/format/idempotency.

Lower blast radius:

- Public docs under `site/` when not changing generated package behavior.
- Single-target installer copy if registry/contracts are untouched.
- Telemetry worker schema additions if backward-compatible.
- New tests/fixtures.

Mandatory checks by area:

- Parser language change: targeted extractor tests, `__tests__/extraction.test.ts`, language-specific tests, search-quality probes if behavior affects flows.
- Kernel change: `npm run build:kernel`, kernel parity tests, layout parity, fallback behavior.
- Schema change: migration test/open old DB path, FTS behavior, node/edge count sanity.
- Resolver/synthesizer change: deterministic graph probes, edge count spot-check, control repo regression.
- MCP change: mcp/tool tests, not-indexed success shape, path refusal behavior, output cap behavior.
- Installer change: target contract tests and uninstall/idempotency coverage.

Known gaps / uncertainties:
- Agent A/B validation is inherently noisy; do not conclude from one run.
