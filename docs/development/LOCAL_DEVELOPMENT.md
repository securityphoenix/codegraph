# Local Development

Parent document: /CLAUDE.md
Related documents:
- /docs/development/TESTING_STRATEGY.md
- /docs/operations/RUNBOOK.md
- /BUNDLING.md

Read this when:
- You need to build, test, or run the repo locally.

Purpose:
- Provide local setup and common pitfalls.

Scope:
- Includes source checkout development.
- Excludes release workflow details beyond local build implications.

Basic commands:

```bash
npm ci
npm run build
npm test
npm run cli
```

Useful targeted commands:

```bash
npx vitest run __tests__/extraction.test.ts
npx vitest run __tests__/mcp-daemon.test.ts
npx vitest run __tests__/security.test.ts
npm run build:kernel
```

Local run:

```bash
npm run build
node dist/bin/codegraph.js init <project>
node dist/bin/codegraph.js status <project>
node dist/bin/codegraph.js serve --mcp --path <project>
```

Pitfalls:

- Source runs need a Node version compatible with `node:sqlite`; CLI supports `>=20 <25`, but source DB backend needs modern Node.
- `npm run build` must copy schema and WASM assets into `dist/`.
- Native kernel is optional; missing prebuild should fall back to WASM.
- Mac `node_modules` must not be reused for Linux/Windows validation because native packages differ.
- Windows path and file-locking behavior needs real Windows validation for platform-sensitive changes.

Known gaps / uncertainties:
- Some validation scripts referenced by design docs may require local paths or external corpora not present in this checkout.
