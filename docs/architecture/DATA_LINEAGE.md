# Data Lineage

Parent document: /CLAUDE.md
Related documents:
- /docs/architecture/AST_PARSER_AND_GRAPH_BUILD.md
- /docs/architecture/DATA_CONTRACTS.md
- /docs/architecture/RUNTIME_FLOWS.md
- /docs/security/TRUST_BOUNDARIES.md

Read this when:
- You need to trace where graph data comes from and where it is consumed.
- You are changing schemas, extraction output, or MCP result formatting.

Purpose:
- Document the origin, transformations, storage, and consumption of key data.

Scope:
- Includes source files, graph rows, unresolved refs, synthesized edges, MCP output, and telemetry.
- Excludes every query method implementation.

## Source To Graph

```mermaid
flowchart TD
  Source[Project source files] --> Scan[Git/filesystem scan + ignore scope]
  Scan --> Detect[Language detection]
  Detect --> Parse[Kernel or tree-sitter/custom extractor]
  Parse --> Result[ExtractionResult]
  Result --> Store[finalizeStoreBundle/storeFileBundle]
  Store --> Nodes[(nodes)]
  Store --> Edges[(edges)]
  Store --> Files[(files)]
  Store --> Refs[(unresolved_refs)]
```

## References To Resolved Edges

```mermaid
flowchart TD
  Refs[(pending unresolved_refs)] --> Resolver[ReferenceResolver]
  Resolver --> Import[import resolver]
  Resolver --> Name[name matcher]
  Resolver --> Framework[framework resolvers]
  Resolver --> Edge[(edges)]
  Resolver --> Failed[(failed unresolved_refs)]
  Edge --> Synth[dynamic synthesis passes]
  Synth --> Edge
```

## Graph To Agent Output

```mermaid
flowchart TD
  Nodes[(nodes/FTS)] --> Query[Graph/search queries]
  Edges[(edges)] --> Query
  Files[(files + source on disk)] --> Context[ContextBuilder/MCP formatting]
  Query --> Context
  Context --> Agent[MCP tool response]
```

Key lineage rules:

- File records use content hash and modified time for staleness decisions.
- Parser-generated unresolved refs do not include final targets; target choice is resolver-owned.
- Framework extractors may add route nodes and refs after the parser pass.
- Synthesized dynamic-dispatch edges are stored as `provenance='heuristic'` with metadata explaining the synthesizer/wiring.
- MCP output includes source from disk/graph coordinates, not LLM-generated summaries.
- Telemetry records command/event metadata only after consent; it is not used to build graph answers.

Known gaps / uncertainties:
- Source files can change during an MCP response; watcher/sync/catch-up reduce staleness but do not make source reads transactional with SQLite.
