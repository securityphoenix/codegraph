# Safe Change Zones

Parent document: /CLAUDE.md
Related documents:
- /docs/architecture/CHANGE_BLAST_RADIUS.md
- /docs/development/TESTING_STRATEGY.md
- /docs/architecture/MODULE_OWNERSHIP.md

Read this when:
- You need to know where changes are relatively safe or dangerous.

Purpose:
- Guide minimal, correctly scoped changes.

Scope:
- Includes repo-specific safe/risky zones and mandatory checks.

Safer zones:

- Docs that do not change release claims.
- Tests and fixtures.
- Single installer target edits with target contract tests.
- Public website copy under `site/` when not changing CLI/runtime behavior.
- Telemetry UI/policy text when event schema is unchanged.

Medium-risk zones:

- CLI output/command UX.
- Context formatting.
- Search ranking.
- Single framework resolver with narrow tests.
- One language extractor hook when grammar routing and shared extractor branches are untouched.

High-risk zones:

- Shared parser walk in `src/extraction/tree-sitter.ts`.
- `src/extraction/grammars.ts` extension/language routing.
- Native kernel ABI/routing/decoder.
- SQLite schema, migrations, and write/query layer.
- Resolver/name matcher/synthesizer core.
- MCP tool error shape and output budgets.
- Daemon/proxy lifecycle.
- Cross-platform path/watcher/process code.

Before editing high-risk zones:

- Identify every caller.
- Confirm whether a shared fix is smaller than caller-specific guards.
- Add one targeted regression check.
- Run the smallest relevant test first, then broader tests if the shared contract moved.

Known gaps / uncertainties:
- Some high-risk behavior is validated by historical real-repo corpora, not only local tests.
