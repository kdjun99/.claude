---
name: spec-analysis
description: "Rules for analyzing PDF design specs (기획서): checklist extraction, feature grouping, domain term extraction, and 3-tier domain mapping auto-proposal. Use with ralph for persistent execution."
---

# Spec Analysis

Rules and patterns for analyzing PDF design specs into structured artifacts.

## When to Use

- "이 기획서를 spec-analysis로 분석해줘"
- PDF design spec을 구조화된 분석 아티팩트로 변환할 때
- 도메인 매핑 자동 제안이 필요할 때

## Prerequisites

- PDF spec file path
- Project root with `.claude/` directory
- (Optional) `{project-root}/.claude/skills/coding-guide/skill.md`
- (Optional) `{project-root}/.claude/skills/domain-dictionary.md`

## Workspace Setup

```
{project-root}/.claude/workspace/specs/{spec-id}/
├── _spec-analysis.md       ← output
├── _domain-mapping.md      ← output
└── _codebase-scan.md       ← output (from explore agent)
```

## PDF Execution Model

**CRITICAL**: The orchestrator must NEVER read PDF files directly.
PDF images accumulate in the orchestrator context and trigger API Error 400 (image dimension limit for many-image requests).

**Execution rule**: ALL PDF reading happens inside subagents. Each subagent gets an isolated context — images don't leak back to the orchestrator. The orchestrator only reads the TEXT output files that subagents produce.

**Multi-PDF strategy**: When multiple PDFs exist (APP/CMS/MO/PC), spawn one subagent PER PDF in parallel. Each subagent saves results to a separate text file. The orchestrator merges the text outputs.

## Phase 1A: PDF Analysis (analyst, opus)

Spawn `oh-my-claudecode:analyst` (opus) with the prompt template below.
**DO NOT read the PDF yourself** — pass the path to the subagent.

For multiple PDFs, spawn one analyst per PDF in parallel:

```
Task(subagent_type="oh-my-claudecode:analyst", model="opus", prompt="""
Read the PDF at {pdf-path} (use pages parameter for large PDFs, max 20 pages per request).
Extract the following and save ALL results to {workspace}/_spec-analysis-raw.md:

1. COVER PAGE: title, version, date, author

2. CHECKLIST (체크리스트):
   Look for columns: No, 기능/기능명, 반영여부, 비고
   Classify each item:
   - O, 반영, ✓ → Included
   - X, 미반영, ✗ → Excluded
   - Ambiguous → Uncertain (flag for human review)

3. DETAILED SPECS (for each Included item):
   - Screen/UI descriptions
   - Field definitions with types and validation rules
   - Business rules and state transitions
   - Access control requirements
   - Page references for traceability

4. FEATURE GROUPS (priority order for grouping):
   a. Same entity/model — features on same data entity
   b. Same user flow — sequential steps in user journey
   c. Same domain — same business domain
   d. Explicit spec grouping — respect spec's own section dividers
   Target: 2-6 items per group. Single-item groups OK if truly independent.

5. DOMAIN TERMS:
   Extract business-specific Korean terms EXACTLY as written in the spec.
   Categories: entities, metrics, states, actions, concepts.
   Rules: exact spec wording (no translation), brief description each, deduplicate, group by feature group.

Output format: structured markdown with clear sections for each of the 5 areas above.
""")
```

After subagent completes, the orchestrator reads `_spec-analysis-raw.md` (text only) and proceeds to Phase 1B/1C.

## Phase 1B: Codebase Scan (explore, haiku)

Delegate to `oh-my-claudecode:explore` (haiku):

1. Scan Prisma schema files (*.prisma) — extract ALL model names + @@map table names
2. Scan for @ObjectType() / @InputType() decorators — extract GraphQL type names
3. Scan for @Injectable() service classes — extract service names and module paths
4. Build entity inventory

Save to: `_codebase-scan.md`

## Phase 1C: 3-Tier Domain Mapping

After both 1A and 1B complete, auto-propose mappings:

### Tier 1 — Dictionary Lookup (TRUSTED)
- Search `domain-dictionary.md` for exact match on `기획서 용어` or `동의어`
- If found: mark `TRUSTED` — use dictionary's Prisma Model, DB Table, GraphQL Type
- TRUSTED items skip review (already verified)

### Tier 2 — Codebase Scan Fuzzy Match (PROPOSED [?])
- If not in dictionary: fuzzy-match against codebase scan results
- Match strategies: Korean-English translation similarity, entity name similarity, module path keywords
- If plausible match: mark `PROPOSED [?]` — developer must verify

### Tier 3 — No Match ([NEW])
- No match anywhere: mark `[NEW]`
- Developer must fill in or confirm as N/A

## Output Templates

### _spec-analysis.md

```markdown
# Spec Analysis: {spec-title}

## Metadata
- **Spec ID**: {spec-id}
- **Project**: {project}
- **PDF Path**: {pdf-path}
- **Spec Title**: {title}
- **Version**: {version}
- **Date**: {date}
- **Analyzed**: {current date}

## Checklist Summary
| Status | Count |
|--------|-------|
| Included | {n} |
| Excluded | {n} |
| Uncertain | {n} |

### Included Items
| # | Feature | Page Ref | Notes |

### Excluded Items
| # | Feature | Reason |

### Uncertain Items
| # | Feature | Issue |

## Feature Groups
### Group: {name}
- **Items**: #{n1}, #{n2}
- **Domain**: {description}
- **Description**: {1 sentence}
- **Key Entities**: {entity names}
- **Est. Feature-Requests**: {count} ({f} foundation + {d} domain)

## Domain Terms
| Term | Description | Used In Groups |

## Estimates
| Metric | Count |
|--------|-------|
| Feature Groups | {n} |
| Est. Foundation FRs | {n} |
| Est. Domain FRs | {n} |
| Est. Total FRs | {n} |
```

### _domain-mapping.md

```markdown
# Domain Mapping: {spec-id}

## Context
- **Spec ID**: {spec-id}
- **Project**: {project}
- **Status**: Auto-proposed (developer review required for PROPOSED and NEW items)

## Legend
- **TRUSTED**: From domain dictionary — skip review unless wrong
- **PROPOSED [?]**: From codebase scan — developer must verify
- **[NEW]**: No match — developer must fill in or mark N/A

## Mapping
| # | Spec Term | Description | DB Table | ORM Model | GraphQL Type | Source | Notes |
```

## Human Checkpoint (H1)

After generating both files, present summary and ask:
- "Approved — proceed to decomposition"
- "I've made edits — re-read and proceed"
- "Stop — I need more time"

## Post-H1: Domain Dictionary Update

After approval, for each PROPOSED→confirmed or [NEW]→filled term:
- Append to `{project-root}/.claude/skills/domain-dictionary.md`
- This makes future analyses faster (Tier 1 hits increase)

## Quality Criteria

- [ ] All checklist items classified (included/excluded/uncertain)
- [ ] Feature groups have 2-6 items each
- [ ] All domain terms extracted with exact Korean wording
- [ ] 3-tier mapping applied to every term
- [ ] _spec-analysis.md written with all sections
- [ ] _domain-mapping.md written with auto-proposed entries
