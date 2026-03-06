---
name: spec-feature-pipeline
description: "Overall pipeline orchestration: context bootstrap (Layer 1+2) → spec analysis → domain mapping → decomposition → design plans → implementation → PR. Defines artifact flow, human checkpoints, workspace structure, context layer integration, and how to combine with ralph for end-to-end execution. See context-layer-protocol."
---

# Spec-Feature Pipeline

End-to-end pipeline from PDF design spec to implemented PRs.

## When to Use

- "전체 파이프라인으로 이 기획서 처리해줘"
- "spec-feature-pipeline 스킬로 진행해줘"
- PDF spec → 구현된 PR까지 전체 흐름이 필요할 때

## Pipeline Overview

```
PDF Spec
  │
  ├─[Phase 0: Context Bootstrap]──→ $CACHED_CONTEXT + $ARCHITECTURE_CONTEXT
  │   Skills: context-layer-protocol
  │   Tools: project_memory_read (Layer 1), DeepWiki stub (Layer 2 DEFERRED)
  │   No human checkpoint — automatic
  │
  ├─[Phase 1: Analysis]──→ _spec-analysis.md + _domain-mapping.md
  │   Skills: spec-analysis
  │   Agents: analyst(opus) + explore(haiku)
  │   Context: $CACHED_CONTEXT passed to explore agent
  │   H1: Review checklist, groups, domain mapping
  │
  ├─[Phase 2: Decomposition]──→ feature-requests + _requirements.md
  │   Skills: spec-decomposition-patterns
  │   Agents: architect(opus)
  │   Context: project-memory for per-repo sizing adjustment
  │   H2: Review sizing, dependencies, waves
  │
  ├─[Phase 3: Design Plans]──→ design-plan.md per FR
  │   Skills: spec-design
  │   Agents: explore(haiku) + analyst(opus) per FR
  │   Context: Layer 3 (AST grep) for explore, Layer 4 ($IMPLEMENTATION_PATTERNS) for analyst
  │   H3: Review design plans
  │
  └─[Phase 4: Implementation]──→ PRs per FR (wave-ordered)
      Skills: feature-implementation
      Agents: executor(sonnet) + test-engineer + reviewers
      Per FR: implement → verify → review → PR
```

## Usage Patterns with Ralph

### Full Pipeline (rare — large spec)
```
User: "ralph spec-feature-pipeline 스킬로 이 기획서 전체 처리해줘"
Ralph: Phase 1 → H1 → Phase 2 → H2 → Phase 3 → H3 → Phase 4
```

### Phase-by-Phase (recommended)
```
User: "ralph spec-analysis 스킬로 이 기획서 분석해줘"
→ H1 review
User: "ralph spec-decomposition-patterns 스킬로 분해해줘"
→ H2 review
User: "ralph spec-design 스킬로 디자인 플랜 만들어줘"
→ H3 review
User: "ralph feature-implementation 스킬로 01-foundation 구현해줘"
→ PR created
```

### Single Feature (most common)
```
User: "ralph feature-implementation 스킬로 이거 구현해줘" + design-plan.md
→ implement → verify → review → PR
```

## Artifact Flow

```
Phase 0 outputs (ephemeral — not saved to files):
  $CACHED_CONTEXT ─────────────→ Phase 1, Phase 2 (in-memory, max 2000 tokens)
  $ARCHITECTURE_CONTEXT ───────→ Phase 1 (DEFERRED — stub: empty)

Phase 1 outputs:
  _spec-analysis.md ──────────→ Phase 2 input
  _domain-mapping.md ─────────→ Phase 2 input, Phase 3 reference
  _codebase-scan.md ──────────→ Phase 2 reference

Phase 2 outputs:
  features/{fr-id}/_feature-request.md ──→ Phase 3 input
  _requirements.md ───────────────────→ Phase 3 wave ordering

Phase 3 outputs (per FR — uses Layer 3 + Layer 4 context):
  features/{fr-id}/design-plan.md ────→ Phase 4 input
  features/{fr-id}/_explore-report.md → Phase 4 reference
  $IMPLEMENTATION_PATTERNS ───────────→ analyst only (ephemeral, max 1000 tokens)

Phase 4 outputs:
  _implementation-report.md
  _review-report.md
  pr-report.md
  → Git PR
```

## Human Checkpoints

| Checkpoint | When | What to Review | Decision |
|-----------|------|----------------|----------|
| **H1** | After analysis + mapping | Checklist accuracy, group correctness, domain mapping | Approve / Edit / Stop |
| **H2** | After decomposition | FR sizing, dependency graph, wave order | Approve / Edit / Defer FRs / Stop |
| **H3** | After design plans | Implementation approach, risk assessment | Approve / Edit / Stop |
| **H4** | After implementation | Code review results, test coverage | Merge PR / Request changes |

## Workspace Structure

```
{project-root}/.claude/workspace/specs/{spec-id}/
├── _spec-analysis.md           # Phase 1
├── _codebase-scan.md           # Phase 1
├── _domain-mapping.md          # Phase 1
├── _requirements.md            # Phase 2
├── execution-plan.md           # Phase 2 (generated after H2)
└── features/
    ├── {fr-id-1}/
    │   ├── _feature-request.md   # Phase 2
    │   ├── _explore-report.md    # Phase 3
    │   └── design-plan.md        # Phase 3
    └── {fr-id-2}/
        └── ...

{project-root}/.claude/workspace/{req-id}/     # Standalone features
├── _feature-request.md
├── _explore-report.md
├── design-plan.md
├── design-plan-approved.md
├── _implementation-report.md
├── _review-report.md
└── pr-report.md
```

## Execution Wave Strategy

### From Spec (multi-FR)
Execute in wave order from _requirements.md:
- **Wave 1**: All foundations (parallel — no dependencies)
- **Wave 2+**: Domain features (parallel within wave)
- **Final wave**: Cross-repo integrations

Within each wave, FRs can be executed in parallel.

### Standalone Feature
No waves — single FR goes through explore → analyze → design → implement.

## Execution Plan Template

After H2 approval, generate:

```markdown
# Execution Plan: {spec-id}

## Overview
- **Spec**: {title}
- **Total FRs**: {n} ({approved} approved, {deferred} deferred)
- **Execution Waves**: {n}

## Wave Execution Order

### Wave 1 (parallel)
| FR ID | Type | Design Plan | Status |
|-------|------|-------------|--------|
| {fr-id} | foundation | features/{fr-id}/design-plan.md | pending |

### Wave 2 (after Wave 1)
| FR ID | Type | Depends On | Design Plan | Status |
|-------|------|------------|-------------|--------|

## Deferred FRs
| FR ID | Reason |
```

## Skill Dependencies

```
spec-feature-pipeline (orchestration)
├── context-layer-protocol (Phase 0 — shared protocol for all layers)
├── spec-analysis (Phase 1 — uses Layer 1 + Layer 2 stub)
├── spec-decomposition-patterns (Phase 2 — uses Layer 1 for per-repo adjustment)
├── spec-design (Phase 3 — uses Layer 3 AST-primary + Layer 4 precision)
├── feature-implementation (Phase 4)
├── codebase-learn (prerequisite — populates Layer 1 project-memory)
└── linkareer-repo-structure (project-specific, optional — Layer 1 fallback)
```

## Tool Availability Matrix

See `context-layer-protocol` for the full matrix. Summary for pipeline phases:

| Phase | Layer 1 (project-memory) | Layer 2 (DeepWiki) | Layer 3 (AST grep) | Layer 4 (AST + Read) |
|-------|-------------------------|-------------------|-------------------|---------------------|
| Phase 0 | Required | Stub (DEFERRED) | - | - |
| Phase 1 | $CACHED_CONTEXT | - | explore agent | - |
| Phase 2 | Per-repo sizing | - | - | - |
| Phase 3 | - | - | explore + ast_grep | analyst + ast_grep |
| Phase 4 | - | - | - | - |

## Project-Specific Extensions

### Coding Guide Integration
If `{project-root}/.claude/skills/coding-guide/skill.md` exists:
- Read on first use, pass to all agents as `$CODING_GUIDE`
- Affects: implementation patterns, test conventions, naming

### Domain Dictionary
If `{project-root}/.claude/skills/domain-dictionary.md` exists:
- Used in Phase 1C (Tier 1 trusted lookups)
- Updated after H1 with newly confirmed terms

### Repo Structure
If `{project-root}/.claude/skills/{project}-repo-structure/skill.md` exists:
- Used in Phase 2 for feature-to-repo assignment
- Provides sizing guidance per repo

## Quality Gates (per phase)

| Phase | Gate | Evidence |
|-------|------|----------|
| Bootstrap | Layer 1 loaded (or fallback noted) | $CACHED_CONTEXT populated or "full scan" logged |
| Analysis | All items classified, groups formed, 3-tier mapping applied | _spec-analysis.md + _domain-mapping.md complete |
| Decomposition | All FRs pass sizing, no circular deps, domain mapping cross-checked | _requirements.md sizing table |
| Design | All 6 sections present per plan (incl. Reference Implementation) | design-plan.md files |
| Implementation | Build + tests pass, reviews clean | verify commands output |
