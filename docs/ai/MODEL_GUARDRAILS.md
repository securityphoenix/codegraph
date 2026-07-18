# Model Guardrails

Parent document: /CLAUDE.md
Related documents:
- /docs/ai/AI_SYSTEM_MAP.md
- /docs/security/TRUST_BOUNDARIES.md
- /docs/architecture/DATA_CONTRACTS.md

Read this when:
- You are reviewing AI-facing safety without expecting internal model calls.
- You are changing MCP output, path validation, or source disclosure behavior.

Purpose:
- Document guardrails around external model consumption of CodeGraph output.

Scope:
- Includes deterministic guardrails CodeGraph enforces before output reaches an external model.
- Excludes model-provider safety controls because CodeGraph does not call a model.

Guardrails present:

- Path validation rejects sensitive or out-of-root paths in MCP tool handling.
- Free-form string and path-like inputs are length bounded.
- Recoverable errors are success-shaped to avoid agent abandonment; security refusals remain hard errors.
- Output size budgets cap context bloat.
- Generated/minified files and low-value files are filtered/deprioritized in relevant paths.
- Telemetry excludes source code, prompts, file paths, and secrets by design.
- MCP server exposes tools even for unindexed roots but guides the agent to pass a valid `projectPath`.

Prompt-injection posture:

- CodeGraph does not execute instructions found in source files.
- Source content is returned to the external agent as context; the external agent is responsible for treating it as untrusted data.
- Server instructions should not claim source is trusted instruction text.

Known gaps / uncertainties:
- External model prompt-injection defenses depend on each MCP host/model; this repository can only constrain tool inputs/outputs.
