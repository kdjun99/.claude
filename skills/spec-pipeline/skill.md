---
name: spec-pipeline
description: "Spec pipeline orchestration: PDF analysis → domain mapping → feature-request decomposition → design plans. Coordinates spec-analysis, spec-decomposition-patterns, and spec-design skills. Use with ralph for persistent execution."
---

# Spec Pipeline

PDF design spec을 분석하고 feature-request로 분해한 후 design-plan까지 생성하는 파이프라인.

## When to Use

- "이 기획서를 spec-pipeline 스킬로 분석하고 분해해줘"
- "spec-pipeline으로 전체 스펙 작업 진행해줘"
- PDF spec → design-plan.md 까지 필요할 때

## Pipeline Flow

```
PDF Spec
  │
  ├─[Phase 1: Analysis]──→ _spec-analysis.md + _domain-mapping.md
  │   Skill: spec-analysis
  │   Agents: analyst(opus) + explore(haiku)
  │   ⏸ H1: Review checklist, groups, domain mapping
  │
  ├─[Phase 2: Decomposition]──→ feature-requests/ + _requirements.md
  │   Skill: spec-decomposition-patterns
  │   Agent: architect(opus)
  │   ⏸ H2: Review sizing, dependencies, waves
  │
  └─[Phase 3: Design Plans]──→ design-plan.md per FR
      Skill: spec-design
      Agents: explore(haiku) + analyst(opus) per FR
      ⏸ H3: Review design plans
      │
      └─→ Ready for feature-pipeline
```

## Prerequisites

- PDF spec file
- Project root with `.claude/` directory
- (Optional) `coding-guide/skill.md`
- (Optional) `domain-dictionary.md`
- (Optional) `{project}-repo-structure/skill.md`

## Phase 1: Analysis (spec-analysis skill)

Refer to `spec-analysis` skill for full rules.

**Parallel execution:**
1. `oh-my-claudecode:analyst` (opus) — PDF analysis
2. `oh-my-claudecode:explore` (haiku) — codebase scan

**Then:** 3-tier domain mapping auto-proposal

**Outputs:**
- `_spec-analysis.md` — checklist, feature groups, domain terms
- `_domain-mapping.md` — auto-proposed term → code entity mapping
- `_codebase-scan.md` — entity inventory

**H1 Checkpoint:** Developer reviews analysis + domain mapping.

## Phase 2: Decomposition (spec-decomposition-patterns skill)

Refer to `spec-decomposition-patterns` skill for full rules.

**Delegate to:** `oh-my-claudecode:architect` (opus)

**Input:** _spec-analysis.md + _domain-mapping.md (confirmed by developer)

**Rules:**
- Foundation + domain separation (1 foundation per group, 1 domain per function)
- Sizing: resolvers ≤ 3, services ≤ 5, tables ≤ 2, files ≤ 10
- Dependency graph with wave calculation
- Domain mapping reference (actual code entities)

**Outputs:**
- `features/{fr-id}/_feature-request.md` per FR
- `_requirements.md` — index + dependency graph + sizing validation
- `execution-plan.md` — wave-ordered execution guide

**H2 Checkpoint:** Developer reviews sizing, dependencies, waves. Can defer FRs.

## Phase 3: Design Plans (spec-design skill)

Refer to `spec-design` skill for full rules.

**Process per FR (wave-ordered):**
1. `oh-my-claudecode:explore` (haiku) — FR-specific codebase exploration
2. `oh-my-claudecode:analyst` (opus) — requirements analysis
3. Write `design-plan.md` (5-section format)

**Parallelism:** Within each wave, run multiple FRs in parallel (up to 5).

**Outputs:**
- `features/{fr-id}/_explore-report.md` per FR
- `features/{fr-id}/design-plan.md` per FR

**H3 Checkpoint:** Developer reviews design plans before implementation.

## Workspace Structure

```
{project-root}/.claude/workspace/specs/{spec-id}/
├── _spec-analysis.md           # Phase 1
├── _codebase-scan.md           # Phase 1
├── _domain-mapping.md          # Phase 1
├── _requirements.md            # Phase 2
├── execution-plan.md           # Phase 2
└── features/
    ├── {fr-id-1}/
    │   ├── _feature-request.md   # Phase 2
    │   ├── _explore-report.md    # Phase 3
    │   └── design-plan.md        # Phase 3
    └── {fr-id-2}/
        └── ...
```

## Phase-by-Phase Execution (Recommended)

각 Phase를 개별 실행할 수 있음:

```
"spec-analysis 스킬로 이 기획서 분석해줘"
→ _spec-analysis.md + _domain-mapping.md 생성
→ H1 리뷰

"spec-decomposition-patterns 스킬로 분해해줘"
→ feature-requests/ + _requirements.md 생성
→ H2 리뷰

"spec-design 스킬로 디자인 플랜 만들어줘"
→ design-plan.md per FR 생성
→ H3 리뷰
```

## Handoff to Feature Pipeline

Phase 3 완료 후, 각 FR의 design-plan.md를 feature-pipeline으로 전달:

```
"feature-pipeline 스킬로 {fr-id} 구현해줘"
```

`feature-pipeline`이 design-plan.md를 읽고 구현 → 검증 → PR 생성.

## Quality Gates

| Phase | Gate | Evidence |
|-------|------|----------|
| Analysis | All items classified, groups formed | _spec-analysis.md sections complete |
| Decomposition | All FRs pass sizing, no circular deps | _requirements.md sizing table all ✓ |
| Design Plans | All 5 sections present per plan | design-plan.md per FR |
