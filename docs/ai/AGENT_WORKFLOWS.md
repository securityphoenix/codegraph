# Agent Workflows

Parent document: /CLAUDE.md
Related documents:
- /docs/ai/AI_SYSTEM_MAP.md
- /docs/architecture/RUNTIME_FLOWS.md
- /docs/operations/OBSERVABILITY.md

Read this when:
- You are changing how agents connect to or use CodeGraph.
- You are debugging why an agent fell back to Read/Grep.

Purpose:
- Document MCP workflows and human approval boundaries.

Scope:
- Includes external agent connection, tool use, and indexing expectations.
- Excludes prompts/model provider internals because this repo does not invoke models.

## Install Workflow

1. User runs `codegraph install`.
2. Installer detects supported agent targets from `src/installer/targets/registry.ts`.
3. Target writer updates the appropriate MCP config file.
4. Installer does not index projects; the user still runs `codegraph init` per project.
5. Installer strips legacy duplicated instruction blocks and relies on MCP initialize instructions.

Supported target modules include Claude, Cursor, Codex, Gemini, Antigravity, Kiro, and other targets represented in `src/installer/targets/`.

## Project Initialization Workflow

1. User runs `codegraph init` in a project.
2. `.codegraph/` is created.
3. Full graph index is built.
4. Auto-sync/watch can keep it fresh.

Agents should not silently initialize arbitrary projects; indexing is a user action because it writes local state and can be expensive.

## MCP Query Workflow

1. Agent calls a CodeGraph MCP tool.
2. Tool resolves `projectPath` or nearest initialized root.
3. Tool validates path and input length.
4. Tool opens/reuses a CodeGraph instance.
5. Tool queries graph/search/context APIs.
6. Tool returns bounded source and relationship context.

Important behavior:

- `codegraph_explore` is the primary broad context tool.
- `codegraph_node` is the secondary depth tool when a specific symbol body is needed.
- Expected recoverable problems return guidance as successful tool content.
- Security refusals remain hard errors.

## Human Approval Requirements

- Installing into agent config is explicit user action.
- Indexing a project is explicit user action.
- Uninstall removes agent config and CLI install locations only when invoked.
- Telemetry requires consent and must never include source code or paths.

Known gaps / uncertainties:
- Exact tool list can drift; inspect `src/mcp/tools.ts` for current schemas.
