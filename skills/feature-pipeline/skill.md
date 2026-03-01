---
name: feature-pipeline
description: "Feature pipeline orchestration: design-plan → implementation → build/test verification → code review → PR. Coordinates feature-implementation skill with verify-fix loops. Use with ralph for persistent execution."
---

# Feature Pipeline

design-plan.md를 기반으로 구현, 검증, 리뷰, PR 생성까지의 파이프라인.

## When to Use

- "feature-pipeline 스킬로 이거 구현해줘"
- "이 디자인 플랜 구현해줘"
- design-plan.md → working code + PR 이 필요할 때

## Pipeline Flow

```
design-plan.md (input)
  │
  ├─[Phase 1: Implementation]──→ code changes + _implementation-report.md
  │   Agent: executor(sonnet) or deep-executor(opus)
  │
  ├─[Phase 2: Build Verify/Fix]──→ build passes (max 5 iterations)
  │   Agent: build-fixer(sonnet) on failure
  │
  ├─[Phase 3: Test Writing]──→ test files + updated report
  │   Agent: test-engineer(sonnet)
  │
  ├─[Phase 4: Test Verify/Fix]──→ tests pass (max 5 iterations)
  │   Agent: build-fixer(sonnet) on failure
  │
  ├─[Phase 5: Code Review]──→ _review-report.md
  │   Agents: security-reviewer(sonnet) + code-reviewer(opus) [parallel]
  │
  └─[Phase 6: PR Creation]──→ Git branch + commit + PR
      ⏸ H4: Team reviews PR
```

## Prerequisites

- `design-plan.md` exists (from spec-pipeline Phase 3 or feature:plan)
- `design-plan-approved.md` marker (or approve during execution)
- Git working directory clean (recommended)
- (Optional) `_feature-request.md` for additional context
- (Optional) `coding-guide/skill.md`

## Input Sources

Design plan can come from two paths:

### From Spec Pipeline
```
{project-root}/.claude/workspace/specs/{spec-id}/features/{fr-id}/design-plan.md
```

### From Standalone Feature Plan
```
{project-root}/.claude/workspace/{req-id}/design-plan.md
```

## Phase 1: Implementation

Refer to `feature-implementation` skill for full rules.

**Agent selection:**
| Size | Agent | Model |
|------|-------|-------|
| ≤ 5 files | `oh-my-claudecode:executor` | sonnet |
| > 5 files | `oh-my-claudecode:deep-executor` | opus |

**Implementation order:**
1. DB Migration / Prisma Schema
2. DTOs (GraphQL types)
3. Services (Business logic)
4. Resolvers (API layer)
5. Module registration

**Output:** `_implementation-report.md`

## Phase 2: Build Verify/Fix Loop

```
iteration = 0
WHILE iteration < 5:
  1. npm run type-check (+ npm run build:gqlgen if DTO changes)
  2. PASS → break
  3. FAIL → oh-my-claudecode:build-fixer (sonnet)
  4. iteration++
IF iteration == 5 → escalate to developer
```

## Phase 3: Test Writing

**Agent:** `oh-my-claudecode:test-engineer` (sonnet)

Rules:
- Follow test plan from design-plan.md Section 4
- Follow project testing conventions
- Cover all test cases + edge cases

## Phase 4: Test Verify/Fix Loop

Same pattern as Phase 2 but runs test suite.
build-fixer may fix both test and implementation code.

## Phase 5: Parallel Code Review

Run simultaneously:

| Review | Agent | Model | Focus |
|--------|-------|-------|-------|
| Security | `oh-my-claudecode:security-reviewer` | sonnet | OWASP, auth, injection, data exposure |
| Code | `oh-my-claudecode:code-reviewer` | opus | Design adherence, conventions, performance |

**Output:** `_review-report.md`

**Critical issue handling:**
- Fix and re-review
- Or developer overrides with "Proceed anyway"

## Phase 6: PR Creation

**Branch:** `feature/{req-id}`

**Commit:**
```
feat({req-id}): {short description}

Based on design plan: .claude/workspace/{req-id}/design-plan.md

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
```

**PR body:** Summary + changes table + review report + test coverage

**Output:** `pr-report.md`

## Workspace Structure

```
{project-root}/.claude/workspace/{req-id}/
├── design-plan.md              ← input
├── design-plan-approved.md     ← gate
├── _feature-request.md         ← input (optional)
├── _explore-report.md          ← from spec-pipeline or feature:plan
├── _implementation-report.md   ← Phase 1 output
├── _review-report.md           ← Phase 5 output
└── pr-report.md                ← Phase 6 output
```

## Usage Patterns

### From Spec Pipeline (multi-FR, wave-ordered)
```
"feature-pipeline 스킬로 Wave 1 전체 구현해줘"
→ Wave 1의 모든 foundation FR을 순서대로 또는 병렬로 처리
```

### Standalone Feature
```
"feature-pipeline 스킬로 user-profile-edit 구현해줘"
→ 단일 FR 처리: implement → verify → review → PR
```

### With Ralph (persistent)
```
"ralph feature-pipeline 스킬로 이거 구현해줘"
→ 실패 시 자동 재시도, architect 검증 포함
```

## Quality Gates

| Phase | Gate | Evidence |
|-------|------|----------|
| Implementation | All design plan items implemented | _implementation-report.md |
| Build | type-check + gqlgen pass | Command output |
| Tests | All tests pass | Test runner output |
| Review | No critical issues | _review-report.md |
| PR | Branch pushed, PR created | pr-report.md with URL |
