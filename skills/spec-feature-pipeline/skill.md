---
name: spec-feature-pipeline
description: "Overall pipeline orchestration: spec analysis → domain mapping → decomposition → design plans → implementation → PR. Defines artifact flow, human checkpoints, workspace structure, and how to combine with ralph for end-to-end execution."
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
  ├─[Phase 1: Analysis]──→ _spec-analysis.md + _domain-mapping.md
  │   Skills: spec-analysis
  │   Agents: analyst(opus) + explore(haiku)
  │   H1: Review checklist, groups, domain mapping
  │
  ├─[Phase 2: Decomposition]──→ feature-requests + _requirements.md
  │   Skills: spec-decomposition-patterns
  │   Agents: architect(opus)
  │   H2: Review sizing, dependencies, waves
  │
  ├─[Phase 3: Design Plans]──→ design-plan.md per FR
  │   Skills: spec-design
  │   Agents: explore(haiku) + analyst(opus) per FR
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
Phase 1 outputs:
  _spec-analysis.md ──────────→ Phase 2 input
  _domain-mapping.md ─────────→ Phase 2 input
  _codebase-scan.md ──────────→ Phase 2 reference

Phase 2 outputs:
  features/{fr-id}/_feature-request.md ──→ Phase 3 input
  _requirements.md ───────────────────→ Phase 3 wave ordering

Phase 3 outputs:
  features/{fr-id}/design-plan.md ────→ Phase 4 input
  features/{fr-id}/_explore-report.md → Phase 4 reference

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
├── spec-analysis (Phase 1)
├── spec-decomposition-patterns (Phase 2, existing)
├── spec-design (Phase 3)
├── feature-implementation (Phase 4)
└── linkareer-repo-structure (project-specific, optional)
```

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
| Analysis | All items classified, groups formed | _spec-analysis.md complete |
| Decomposition | All FRs pass sizing, no circular deps | _requirements.md sizing table |
| Design | All 5 sections present per plan | design-plan.md files |
| Implementation | Build + tests pass, reviews clean | verify commands output |
