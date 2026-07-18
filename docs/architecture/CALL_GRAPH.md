# Call Graph

Parent document: /CLAUDE.md
Related documents:
- /docs/architecture/AST_PARSER_AND_GRAPH_BUILD.md
- /docs/architecture/RUNTIME_FLOWS.md
- /docs/architecture/DEPENDENCY_GRAPH.md
- /docs/architecture/CHANGE_BLAST_RADIUS.md

Read this when:
- You need important runtime call paths from entrypoints to core modules.
- You are changing orchestration rather than a leaf helper.

Purpose:
- Capture the important call graph without dumping every function.

Scope:
- Includes primary call paths for indexing, parsing, resolution, and MCP.
- Excludes exhaustive per-command CLI paths.

## Full Index

```mermaid
sequenceDiagram
  participant CLI as codegraph CLI
  participant CG as CodeGraph
  participant EO as ExtractionOrchestrator
  participant PW as ParseWorkerPool
  participant EX as extractFromSource
  participant SW as StoreWriter/DB
  participant RR as ReferenceResolver

  CLI->>CG: CodeGraph.init/open + indexAll()
  CG->>EO: indexAll(progress, signal, verbose, walBackpressure, storeWriterOpts)
  EO->>PW: requestParse(file)
  PW->>EX: parse-worker message
  EX->>EX: kernel or tree-sitter extraction
  PW-->>EO: ExtractionResult or kernel buffers
  EO->>SW: finalized store bundle
  CG->>RR: initialize + runPostExtract()
  CG->>RR: resolveAndPersistBatched()
  RR->>RR: resolve refs + synthesizeCallbackEdges()
  RR->>SW: insert edges / settle refs
```

## Parser Dispatch

```text
extractFromSource
  -> custom extractors for SFC/template/resource formats
  -> file-level-only empty extraction for yaml/twig/properties
  -> tryKernelExtract / tryKernelExtractRaw for routed languages
  -> TreeSitterExtractor fallback
       -> getParser
       -> language preParse
       -> visitNode
       -> extractFunction/class/method/import/variable/call
       -> flushFnRefCandidates
       -> flushValueRefs
  -> framework extract hooks
```

## MCP Tool Call

```text
MCPServer.start
  -> direct/proxy/daemon mode
  -> MCPEngine / MCPSession
  -> ToolHandler.execute
  -> CodeGraph open/reuse
  -> search/graph/context methods
  -> bounded MCP response
```

## Resolution Call Path

```text
ReferenceResolver.resolveAndPersistBatched
  -> warmCachesYielding
  -> read pending unresolved refs
  -> resolveBatchYielding / ResolverPool
  -> resolveOneTimed
  -> import resolver / name matcher / framework resolver
  -> insert resolved edges
  -> delete resolved refs
  -> mark unresolved refs failed
  -> synthesizeCallbackEdges
  -> resolveChainedCallsViaConformance
  -> resolveDeferredThisMemberRefs
```

Known gaps / uncertainties:
- CLI command call graph is intentionally summarized; inspect `src/bin/codegraph.ts` before changing command UX.
