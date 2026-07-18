# Deployment

Parent document: /CLAUDE.md
Related documents:
- /BUNDLING.md
- /docs/integrations/EXTERNAL_INTEGRATIONS.md
- /telemetry-worker/README.md

Read this when:
- You are changing packaging, release, installation, or telemetry-worker deployment.

Purpose:
- Summarize deployment and distribution assumptions.

Scope:
- Includes npm/bundle distribution and optional telemetry worker.
- Excludes hosted CodeGraph platform operations not represented here.

Distribution:

- Package name: `@colbymchenry/codegraph`.
- Build command: `npm run build`.
- Build copies `src/db/schema.sql` and `src/extraction/wasm/*.wasm` into `dist/`.
- CLI binary: `dist/bin/codegraph.js`.
- `BUNDLING.md` is canonical for self-contained bundle details.
- `CHANGELOG.md` is the release-note source of truth.

Runtime environments:

- Local user machine: CLI, MCP server/proxy/daemon, SQLite graph, parser workers.
- Optional Cloudflare Worker: telemetry ingest under `telemetry-worker/`.

Deployment risks:

- Forgetting to copy schema or WASM breaks installed parsing/storage.
- Native kernel prebuild absence should degrade to WASM, not break indexing.
- Manual npm publish path can ship wrong artifact; release workflow is canonical.
- Node version support is enforced at CLI runtime.

Known gaps / uncertainties:
- GitHub Actions workflow files were not deeply audited in this pass; verify before release-flow edits.
