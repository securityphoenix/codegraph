# Trust Boundaries

Parent document: /CLAUDE.md
Related documents:
- /docs/security/SECURITY_MODEL.md
- /docs/architecture/SERVICE_TOPOLOGY.md
- /docs/ai/MODEL_GUARDRAILS.md

Read this when:
- You need ingress/egress, identity, and secret boundaries.

Purpose:
- Define trust zones and privileged modules.

Scope:
- Includes local process, filesystem, MCP, installer, parser, and telemetry boundaries.

Trust zones:

- User shell/local OS: trusted to invoke CLI and grant filesystem access.
- Project source tree: untrusted data; parse it, do not execute it.
- MCP client/agent: untrusted input; validate path, length, and requested project.
- `.codegraph/` DB: trusted cache derived from source but must tolerate corruption/staleness.
- Installer target config files: sensitive user configuration; edit surgically.
- Telemetry endpoint: external egress; optional and source-free.

Privileged modules:

- `src/mcp/tools.ts`: enforces MCP input/path/security behavior.
- `src/directory.ts` and `src/utils.ts`: root/path validation.
- `src/installer/targets/*`: writes user config.
- `src/db/*`: opens and mutates local SQLite.
- `src/bin/codegraph.ts`: process startup, env behavior, Node version, command dispatch.

Sensitive ingress:

- MCP tool arguments.
- CLI path arguments.
- Project source files and config files.
- Git output used for scan/change detection.

Sensitive egress:

- MCP tool responses containing source snippets.
- Agent config writes.
- Optional telemetry events.

Known gaps / uncertainties:
- External agent/model privacy depends on the user's chosen agent; CodeGraph cannot enforce provider-side retention policy.
