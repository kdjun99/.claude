---
name: context-layer-protocol
description: "Shared protocol defining 4-layer codebase context system for spec skills. Defines layers, tools, fallback chains, context budgets, and agent constraints. Referenced by spec-analysis, spec-design, spec-decomposition-patterns, and spec-feature-pipeline."
---

# Context Layer Protocol

Shared reference for the hybrid layered codebase context system used across all spec skills.

## When to Use

- Referenced by `spec-analysis`, `spec-design`, `spec-decomposition-patterns`, `spec-feature-pipeline`
- Not invoked directly by users — this is a shared protocol document

## Layer Definitions

### Layer 1: Cached Conventions (project-memory)

- **Source**: project-memory MCP tools (`project_memory_read`, `project_memory_write`)
- **Contains**: techStack, build commands, conventions, module structure, domain entity notes
- **Freshness**: Updated by `codebase-learn`; persists across sessions
- **Fallback**: If empty or stale (>7 days), fall back to repo-structure skill file. Optionally suggest running `codebase-learn` first.
- **When to use**: Session start / Phase 0, before any codebase queries
- **Consumers**: spec-analysis (Phase 1B-pre), spec-decomposition (Per-Repo Adjustment), spec-feature-pipeline (Phase 0)

### Layer 2: Architecture Context (DeepWiki) [DEFERRED — stub only]

- **Source**: DeepWiki MCP (`read_wiki_structure`, `read_wiki_contents`, `ask_question`)
- **Status**: DEFERRED. Primary user repos are private; DeepWiki cannot index them.
- **Availability check**:
  ```
  deepwiki_available = false
  # Full DeepWiki integration deferred until private repo indexing confirmed via Devin MCP.
  # When enabled: try read_wiki_structure(repo: "{org}/{repo}"); set true on success.
  ```
- **Future**: Full integration when Devin MCP private repo indexing is confirmed working.
- **Consumers**: spec-analysis (Phase 1B-pre stub), spec-feature-pipeline (Phase 0 stub)

### Layer 3: Feature-Specific Exploration (explore + AST grep)

- **Source**: explore agent (haiku) + `ast_grep_search` (PRIMARY tool)
- **Contains**: Related modules, service method patterns, DI patterns, decorator usage, resolver/mutation signatures
- **Freshness**: Real-time (reads live codebase)
- **Primary approach**: `ast_grep_search` for structural pattern matching:
  - Service classes: `@Injectable() class $SERVICE { constructor($DEPS) { $$$BODY } }`
  - Resolver queries: `@Query(() => $TYPE) async $NAME($ARGS) { $$$BODY }`
  - Resolver mutations: `@Mutation(() => $TYPE) async $NAME($ARGS) { $$$BODY }`
  - Guard usage: `@UseGuards($GUARD) $METHOD`
  - ObjectType definitions: `@ObjectType() class $TYPE { $$$FIELDS }`
  - InputType definitions: `@InputType() class $TYPE { $$$FIELDS }`
- **Supplementary**: explore agent uses `Grep` + `Glob` for file discovery and content search
- **Fallback**: AST grep is always available (core tool). No external dependency.
- **When to use**: Per feature-request, during spec-design Step A
- **Consumers**: spec-design (Step A)

#### Future Enhancement: LSP Tools

When an `explore-high` agent is created with `lsp_find_references`, `lsp_goto_definition`, and `lsp_hover` capabilities, Layer 3 can be upgraded:
- `lsp_document_symbols`: full type information for method signatures
- `lsp_find_references`: caller graph and dependency direction mapping
- `lsp_hover`: complete type definitions for complex GraphQL types

Until then, AST grep provides sufficient structural pattern matching.

### Layer 4: Implementation Precision (AST grep + Read)

- **Source**: `ast_grep_search`, targeted file `Read`
- **Contains**: Exact implementation patterns, decorator usage, test conventions, error handling patterns, validation patterns
- **Freshness**: Real-time
- **Fallback**: Always available (core tools)
- **When to use**: During spec-design Step B, before analyst delegation
- **Consumers**: spec-design (Step B — analyst opus only)

## Context Budget

Each context variable has a maximum token budget. The orchestrator reads source data, applies filtering, and interpolates the result into `Task()` prompt parameters (same mechanism as `$CODING_GUIDE`).

| Variable | Max Tokens | Contents | Consumer Agents |
|----------|-----------|----------|-----------------|
| `$CACHED_CONTEXT` | 2000 | techStack, build commands, conventions, top 20 domain entities | explore (haiku), analyst (opus) |
| `$ARCHITECTURE_CONTEXT` | 1500 | Module hierarchy summary, key domain boundaries | explore (haiku), analyst (opus) |
| `$IMPLEMENTATION_PATTERNS` | 1000 | 3-5 most relevant AST grep pattern matches | analyst (opus) ONLY |

### Filtering Strategy

#### $CACHED_CONTEXT (from project-memory, max 2000 tokens)

1. Always include: `techStack` (compact), `build` (compact), `conventions` (compact)
2. Include from `structure`: `mainDirectories` list only (omit nested details)
3. Include from `notes`: filter to "Domain:" prefixed notes, limit to top 20 by relevance to the current spec's entity list
4. If still over budget: omit `structure` section entirely (Layer 3 explore will rediscover it)

#### $ARCHITECTURE_CONTEXT (from DeepWiki — DEFERRED, max 1500 tokens)

1. Extract: top-level module list with one-line responsibility descriptions
2. Extract: domain boundary summary (which modules own which entities)
3. Omit: full wiki content, code examples, detailed explanations

#### $IMPLEMENTATION_PATTERNS (from AST grep, max 1000 tokens)

1. Include: at most 5 pattern matches, each truncated to first 10 lines of match context
2. Prioritize: patterns from the same module as the target FR's entities
3. Omit: duplicate patterns (same pattern in multiple files — keep one example)

### Haiku Agent Constraints

- Haiku has the smallest context window
- **Never** pass `$IMPLEMENTATION_PATTERNS` to haiku agents
- `$CACHED_CONTEXT` (max 2000 tokens) is the maximum context injection for haiku
- `$ARCHITECTURE_CONTEXT` is passed to haiku only when available AND fits within 1500 tokens

## Fallback Chain Summary

```
Layer 1 unavailable (no project-memory data)
  → Use repo-structure skill file (static)
  → If also missing: proceed without cached context (full codebase scan)

Layer 2 unavailable (DEFERRED / private repo / not indexed)
  → Skip entirely (stub: deepwiki_available = false)
  → Layer 3 explore compensates with broader file discovery

Layer 3 AST grep returns no matches (unusual framework / non-standard patterns)
  → Fall back to Grep + Glob text search
  → explore agent provides file-level discovery regardless

Layer 4 always available
  → ast_grep_search + Read are core tools with no external dependency
```

## Tool Availability Matrix

| Layer | Tool | Required? | Check | Fallback |
|-------|------|-----------|-------|----------|
| 1 | project-memory | No | `project_memory_read` returns data | repo-structure skill |
| 2 | DeepWiki MCP | No (DEFERRED) | Stub: always false | Skip |
| 3 | `ast_grep_search` | Yes | Always available | N/A (core tool) |
| 3 | explore agent | Yes | Always available | N/A (core agent) |
| 3 | LSP tools | No (FUTURE) | Requires explore-high agent | ast_grep_search |
| 4 | AST grep | Yes | Always available | N/A (core tool) |
| 4 | Read | Yes | Always available | N/A (core tool) |

**Minimum viable**: Layer 3 (explore + ast_grep) + Layer 4 (AST grep + Read) = enhanced current behavior
**Full capability (future)**: All 4 layers with DeepWiki + LSP = maximum context richness

## Domain Entity Notes Format

project-memory domain notes use the `project_memory_add_note` API with two parameters:

```
project_memory_add_note(
  category: "Domain",
  content: "{ModelName} -> table:{table_name}, graphql:{TypeName}, module:{module_path}"
)
```

### Deduplication Strategy

Before adding a domain note:
1. Read existing notes: `project_memory_read(sections: "notes")`
2. For each discovered entity `{ModelName}`:
   a. Search existing notes for prefix match `"{ModelName} ->"` in category "Domain"
   b. If found and content changed: delete old, add updated
   c. If found and unchanged: skip (no duplicate)
   d. If not found: add new

### Precedence Rules

- `domain-dictionary.md` is authoritative (human-verified Tier 1)
- `project-memory` domain notes are supplementary (machine-discovered)
- On conflict: `domain-dictionary.md` takes precedence
- project-memory notes serve as a discovery feed for human verification
