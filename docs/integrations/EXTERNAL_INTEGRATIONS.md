# External Integrations

Parent document: /CLAUDE.md
Related documents:
- /docs/architecture/SERVICE_TOPOLOGY.md
- /docs/operations/DEPLOYMENT.md
- /docs/security/TRUST_BOUNDARIES.md

Read this when:
- You are changing dependencies, agent integrations, release packaging, or telemetry.

Purpose:
- Inventory external systems and contract risks.

Scope:
- Includes package dependencies, agent config targets, git, SQLite, tree-sitter, native kernel, and telemetry.

Runtime/package dependencies:

- Node.js: CLI/runtime, worker threads, built-in `node:sqlite`.
- SQLite via `node:sqlite`: local graph storage.
- `web-tree-sitter`: WASM parser runtime.
- `tree-sitter-wasms` plus vendored WASM files: grammar sources.
- Rust/napi native module: optional extraction kernel prebuilds.
- `commander`: CLI parsing.
- `@clack/prompts`: installer prompts.
- `jsonc-parser`: comment-preserving config edits.
- `ignore` and `picomatch`: scan/watch scope matching.

Local tool integrations:

- Git: file enumeration, embedded repository handling, change detection.
- Filesystem watchers: auto-sync.
- Agent MCP configs: Claude, Cursor, Codex, Gemini, Antigravity, Kiro, opencode/Hermes where target modules exist.

Network/cloud integrations:

- npm/GitHub Releases: package distribution.
- Cloudflare Worker telemetry ingest: optional anonymous event upload.

Contract risks:

- Tree-sitter grammar version drift between WASM and native kernel can break parity.
- Agent MCP config formats differ; target writers must preserve comments/user settings.
- Git behavior differs across submodules, worktrees, ignored embedded repos, and platforms.
- `node:sqlite` availability depends on runtime Node version; bundled runtime is expected to satisfy it.
- Telemetry schema changes must remain backward-compatible with buffered local events.

Known gaps / uncertainties:
- Exact installed agent list can change; inspect `src/installer/targets/registry.ts`.
