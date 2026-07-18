# Security Model

Parent document: /CLAUDE.md
Related documents:
- /docs/security/TRUST_BOUNDARIES.md
- /docs/ai/MODEL_GUARDRAILS.md
- /docs/integrations/EXTERNAL_INTEGRATIONS.md

Read this when:
- You are reviewing auth, path handling, secrets, telemetry, or MCP exposure.

Purpose:
- Document security assumptions and sensitive flows.

Scope:
- Includes local CLI/MCP security.
- Excludes hosted product auth not implemented in this repository.

Auth/authz:

- Local CLI operations rely on local OS user permissions.
- MCP server has no application user accounts; the MCP host launches it as the local user.
- Authorization boundary is therefore filesystem/project-root validation, not multi-user auth.

Sensitive assets:

- User source code.
- `.codegraph/codegraph.db`, which contains symbol names, file paths, signatures, docstrings, edges, and errors.
- Agent configuration files modified by installer.
- Optional telemetry consent/config state.

Controls:

- MCP path validation rejects sensitive/out-of-root paths.
- Input lengths are bounded in `src/mcp/tools.ts`.
- Indexing respects ignore rules plus default dependency/build/cache excludes.
- Telemetry policy excludes source, prompts, file paths, and secrets.
- Installer edits are target-specific and tested for idempotency/preservation.

Key risks:

- MCP host compromise equals local user process execution; CodeGraph cannot sandbox the host.
- Returning source to an external agent exposes that source to the agent/model provider selected by the user.
- SQLite graph can contain sensitive symbol names and docstrings; treat `.codegraph/` as project-sensitive.
- False graph edges can mislead security reviewers; resolver precision is a security quality concern.

Known gaps / uncertainties:
- No remote auth layer exists in this repo to review.
