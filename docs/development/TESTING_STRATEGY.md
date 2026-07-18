# Testing Strategy

Parent document: /CLAUDE.md
Related documents:
- /docs/development/LOCAL_DEVELOPMENT.md
- /docs/architecture/DATA_CONTRACTS.md
- /docs/development/SAFE_CHANGE_ZONES.md

Read this when:
- You need to choose checks for a change.

Purpose:
- Map test suites and validation methods to system risk areas.

Scope:
- Includes local automated tests and parser/agent validation strategy.
- Excludes full CI matrix details.

Baseline:

```bash
npm test
```

Test areas:

- Extraction/parser: `__tests__/extraction.test.ts` and language/framework-specific tests.
- Kernel: `__tests__/kernel-*`, `npm run build:kernel`, parity scripts referenced in `docs/design/rust-kernel-migration-plan.md`.
- Resolver/frameworks: framework integration, dynamic-dispatch, callback/value-reference tests.
- DB: node-sqlite backend, migrations, WAL, perf tests.
- MCP: daemon, roots, catch-up, bind failure, liveness, tool behavior tests.
- Installer: target contract tests for idempotency, preservation, uninstall.
- Security: path refusal, symlink/session-marker resistance, config redaction.
- Telemetry: consent/event serialization tests.

Parser/language change bar:

1. Add/update extractor tests for the syntax shape.
2. Verify node kinds, qualified names, signatures, and unresolved refs.
3. Run affected resolver/search tests.
4. For framework or dynamic-flow support, run deterministic graph probes and spot-check edges.
5. For product claims, run small/medium/large real-repo validation; do not rely on a synthetic fixture only.

Kernel language change bar:

1. Keep grammar source parity.
2. Keep `layout.ts` and Rust buffers ABI-compatible.
3. Prove fallback path still works.
4. Pass parity/dump gate before default routing.

Known gaps / uncertainties:
- Agent A/B tests are expensive/noisy and not part of ordinary `npm test`; use when behavior affects agent sufficiency.
