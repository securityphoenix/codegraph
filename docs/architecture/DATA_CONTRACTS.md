# Data Contracts

Parent document: /CLAUDE.md
Related documents:
- /docs/architecture/AST_PARSER_AND_GRAPH_BUILD.md
- /docs/architecture/DATA_LINEAGE.md
- /docs/architecture/CHANGE_BLAST_RADIUS.md
- /docs/development/TESTING_STRATEGY.md

Read this when:
- You are changing AST extraction, language support, schema, or resolver outputs.
- You need exact parser/language contracts.

Purpose:
- Define the stable contracts between parser, resolver, database, kernel, and MCP output.

Scope:
- Includes graph types, parser contracts, kernel ABI, and critical payload shapes.
- Excludes user-facing CLI help text.

## Core Graph Contract

`src/types.ts` owns `NODE_KINDS`, `EDGE_KINDS`, and `LANGUAGES`. These arrays are runtime contracts. `NODE_KINDS` and `EDGE_KINDS` are indexed by the native kernel wire format, so append only; never reorder.

Critical node fields:

- `id`: generated from file path, kind, name, and position conventions.
- `kind`: one of `NODE_KINDS`.
- `name`: simple lookup/display name.
- `qualifiedName`: path/scope-aware name used by search and disambiguation.
- `filePath`: project-root-relative path.
- `language`: one of `LANGUAGES`.
- `signature`, `docstring`, visibility/static/export/decorator flags.
- `returnType`: used by receiver inference, especially C/C++ chained calls.

Critical edge fields:

- `source`, `target`, `kind`.
- `metadata`: JSON for synthesized/wiring details.
- `line`, `column`: call/reference site.
- `provenance`: `tree-sitter`, `scip`, or `heuristic`.

## SQLite Contract

`src/db/schema.sql` stores:

- `nodes`: symbol graph.
- `edges`: graph relationships with unique `(source,target,kind,line,col)`.
- `files`: content hash, language, size, modification/index timestamps, node count, errors.
- `unresolved_refs`: pending/failed references awaiting resolution or retry.
- `nodes_fts`: FTS5 index over symbol names, qualified names, docstrings, signatures.
- `name_segment_vocab`: prompt/natural-language term to symbol-name helper table.
- `project_metadata`: index state, extraction version, and advisory metadata.

Schema risk: migrations must preserve existing `.codegraph/` indexes. FTS triggers and bulk-load modes are performance-sensitive and correctness-sensitive.

## AST Parser Contract

Every ordinary parser path returns `ExtractionResult`:

- `nodes`: file/symbol nodes.
- `edges`: directly known relationships such as `contains` or same-file direct edges.
- `unresolvedReferences`: references that need whole-project resolution.
- `errors`: warnings/errors for the file.
- `kernelBuffers`: optional raw native-kernel buffers for deferred decode.

`src/extraction/tree-sitter-types.ts` defines `LanguageExtractor`. A language config declares:

- AST node types for functions, classes, methods, interfaces, structs, enums, imports, calls, variables, fields, properties.
- Field names for identifier, body, parameters, and return type.
- Hooks for `preParse`, name/signature/visibility/export/static/async extraction, import parsing, variables, receiver type, return type, inheritance, custom visiting, synthesized members, and classification.

The shared extractor expects language hooks to preserve offsets when using `preParse`; replacing source with spaces and keeping newlines is required so line/column/source slicing stays valid.

## Per-Language Parser Routing

`src/extraction/grammars.ts` is the routing authority:

- Extension map controls `Language`.
- `.h` files use content sniffing for C/C++/Objective-C.
- Svelte/Vue/Astro delegate script/frontmatter extraction.
- Liquid/Razor/DFM/MyBatis/CFML use custom extractors.
- YAML/Twig/properties are file-level-only unless framework extraction adds nodes/refs.
- Vendored WASM grammars are used where `tree-sitter-wasms` is missing, too old, or must match the native kernel grammar source.

Supported language enum includes TypeScript, JavaScript, TSX, JSX, ArkTS, Python, Go, Rust, Java, C, C++, C#, Razor, PHP, Ruby, Swift, Kotlin, Dart, Svelte, Vue, Astro, Liquid, Pascal, Scala, Lua, Luau, Objective-C, R, Solidity, Nix, YAML, Twig, XML, properties, CFML, CFScript, CFQuery, COBOL, VB.NET, Erlang, Terraform, and unknown.

## Native Kernel Contract

`src/extraction/kernel/index.ts` currently routes these languages by default:

- `typescript`
- `tsx`
- `javascript`
- `jsx`
- `java`
- `python`
- `go`

Fallback rules:

- `CODEGRAPH_KERNEL=0` disables all kernel routing.
- `CODEGRAPH_KERNEL_LANGS=<langs|all>` replaces the default routing set.
- Missing/stale/broken native module falls back to WASM.
- Kernel `defer:` falls back to WASM for files whose parse-error recovery must remain canonical.

`src/extraction/kernel/layout.ts` mirrors `codegraph-kernel/src/buffers.rs`:

- Buffers: `meta`, `nodes`, `edges`, `refs`, `arena`.
- Strings are arena offsets/lengths.
- Kinds/provenance/visibility are numeric table indexes.
- ABI version mismatch means fallback to WASM.

## Resolver Contract

Extractors should not eagerly guess cross-file targets. They emit unresolved refs with:

- `fromNodeId`
- `referenceName`
- `referenceKind`
- `line`
- `column`

The store layer denormalizes `filePath` and `language`. The resolver then applies built-in filters, import resolution, language-family matching, framework resolvers, method/receiver matching, chain/deferred passes, and failed-ref parking.

## MCP Output Contract

`src/mcp/tools.ts` bounds input lengths and output sizes. Recoverable states such as not indexed must return success-shaped guidance. `isError` is reserved for security refusals and genuine malfunctions because early tool errors cause agents to abandon CodeGraph.

Known gaps / uncertainties:
- Some language extractor behavior is embedded in large `tree-sitter.ts` branches rather than per-language files; inspect both before changing a language.
