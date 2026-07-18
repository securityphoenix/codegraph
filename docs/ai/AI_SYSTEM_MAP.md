# AI System Map

Parent document: /CLAUDE.md
Related documents:
- /docs/ai/AGENT_WORKFLOWS.md
- /docs/architecture/SERVICE_TOPOLOGY.md
- /docs/security/TRUST_BOUNDARIES.md

Read this when:
- You need to understand how CodeGraph serves AI coding assistants.
- You are changing MCP tools, server instructions, or context formatting.

Purpose:
- Map deterministic CodeGraph behavior to external AI agent usage.

Scope:
- Includes MCP/tool/context behavior.
- Excludes nonexistent in-process LLM calls, model selection, or prompt execution.

CodeGraph does not call an LLM. It provides graph-backed tools to external AI agents over MCP. The AI-facing system is therefore:

```text
external agent prompt
  -> MCP host
  -> CodeGraph MCP server
  -> graph/search/context query
  -> bounded source-rich tool result
  -> external agent reasoning
```

AI touchpoints:

- `src/mcp/server-instructions.ts`: guidance sent during MCP initialize; single source of truth for tool-use instructions.
- `src/mcp/tools.ts`: tool schemas, input validation, result budgets, error-shape policy.
- `src/context/`: markdown/JSON source formatting.
- `src/search/`: natural-language-ish query parsing and identifier segmentation.
- `src/graph/`: callers/callees/impact traversal backing AI answers.
- `src/installer/targets/*`: configures external AI tools to launch CodeGraph MCP.

Deterministic vs AI-driven boundary:

- Deterministic: parsing, graph storage, resolution, ranking, source slicing, output caps.
- AI-driven: the external agent decides which tool to call, what question to ask, and how to interpret returned source.

Downstream dependency:

- If CodeGraph output is incomplete, external agents fall back to Read/Grep.
- If CodeGraph output is wrong, external agents may produce wrong explanations with high confidence.
- MCP response shape matters: recoverable errors must not train agents to abandon CodeGraph.

Known gaps / uncertainties:
- Agent tool-choice behavior changes by host/model and cannot be controlled reliably from this repo.
