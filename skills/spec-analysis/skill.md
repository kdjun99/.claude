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

## PDF Reading Constraints

Claude API has image size/count limits. PDF pages become images in context — accumulated images across the conversation trigger `invalid_request_error` (2000px limit for many-image requests). Follow these rules strictly:

### Rule 1: ALWAYS delegate PDF reading to subagents

**This is the most important rule.** The orchestrator (ralph/main context) must NEVER read PDFs directly. Delegate to subagents so each gets a fresh context without accumulated images.

```
BAD (orchestrator reads PDF directly):
  Read(spec.pdf)  → images pile up in main context
  Read(spec2.pdf) → API Error 400 (too many images)

GOOD (subagent reads PDF):
  Task(analyst, "Read spec.pdf and extract: checklist, features, domain terms.
       Save results to _spec-analysis-raw.md")
  → Subagent reads PDF in isolated context
  → Returns text summary only (no images in main context)
```

### Rule 2: One PDF per subagent

When multiple PDFs exist (APP/CMS/MO/PC), spawn one subagent PER PDF:

```
GOOD (parallel subagents, each with one PDF):
  Task(analyst, "Read [PC].pdf, extract checklist → save to _pc-analysis.md")
  Task(analyst, "Read [MO].pdf, extract checklist → save to _mo-analysis.md")
  Task(analyst, "Read [APP].pdf, extract checklist → save to _app-analysis.md")
  → All run in parallel, each in isolated context
  → Orchestrator reads the text output files (no images)
```

### Rule 3: Use `pages` parameter for large PDFs

Even within subagents, chunk large PDFs:
```
Read(large.pdf, pages="1-10")
Read(large.pdf, pages="11-20")
```

Max 20 pages per request. For PDFs with unknown page count, start with `pages="1-10"`.

### Rule 4: Multi-version spec strategy

When spec has multiple platform versions (APP, CMS, MO, PC):
1. Identify the **primary version** (usually PC or CMS — most comprehensive)
2. Spawn subagent for primary version → full extraction
3. Spawn subagents for other versions → **diff-only** (platform-specific differences)
4. Orchestrator merges text outputs from all subagents

### Rule 5: Orchestrator reads TEXT files only

After subagents process PDFs:
```
Orchestrator reads: _pc-analysis.md, _mo-analysis.md (text only, safe)
Orchestrator NEVER reads: *.pdf (images accumulate in context)
```

## Workspace Setup

```
{project-root}/.claude/workspace/specs/{spec-id}/
├── _spec-analysis.md       ← output
├── _domain-mapping.md      ← output
└── _codebase-scan.md       ← output (from explore agent)
```

## Phase 1A: PDF Analysis (analyst, opus)

Delegate to `oh-my-claudecode:analyst` (opus) with these extraction rules:

### Cover Page
Extract: title, version, date, author

### Checklist Extraction (체크리스트)
Look for columns: No, 기능/기능명, 반영여부, 비고

| Marker | Status |
|--------|--------|
| O, 반영, ✓ | Included |
| X, 미반영, ✗ | Excluded |
| Ambiguous | Uncertain (flag for human review) |

### Detailed Spec Extraction
For each included item:
- Screen/UI descriptions
- Field definitions with types and validation rules
- Business rules and state transitions
- Access control requirements
- Page references for traceability

### Feature Grouping Rules (priority order)
1. **Same entity/model** — features on same data entity
2. **Same user flow** — sequential steps in user journey
3. **Same domain** — same business domain
4. **Explicit spec grouping** — respect spec's own section dividers

Target: 2-6 items per group. Single-item groups OK if truly independent.

### Domain Term Extraction
Extract business-specific Korean terms EXACTLY as written:
- Business entities: 공고, 지원자, 스크랩, Q&A
- Business metrics: 조회수, 관심, 전일 대비 증감
- Business states: 진행, 마감, 평가진행중, 최종합격, 불합격
- Business actions: 공고 마감, 숨기기, 간편지원
- Business concepts: 채용 현황, 공고 성과, 관심유저

Rules:
- Use EXACT spec wording (do NOT translate or guess code names)
- Include brief description for each
- Deduplicate across groups
- Group by feature group for traceability

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
