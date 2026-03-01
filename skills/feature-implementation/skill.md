---
name: feature-implementation
description: "Rules for implementing features from design plans: implementation order, build/test verify-fix loops, parallel code review, and PR creation. Use with ralph for persistent execution."
---

# Feature Implementation

Rules for executing feature implementation from an approved design plan.

## When to Use

- "이 디자인 플랜대로 구현해줘"
- "feature-implementation 스킬로 구현 시작해줘"
- design-plan.md → working code + PR 변환이 필요할 때

## Prerequisites

- `design-plan.md` exists and approved
- Git working directory clean (recommended)
- (Optional) `_feature-request.md` for additional context
- (Optional) `coding-guide/skill.md` for project conventions

## Implementation Order

Always follow this sequence:
1. **DB Migration / Prisma Schema** — tables, columns, relations
2. **DTOs (GraphQL types)** — types, inputs, enums
3. **Services (Business logic)** — validation, queries, side effects
4. **Resolvers (API layer)** — query/mutation handlers
5. **Module registration** — NestJS module imports, providers

## Agent Selection

| Implementation Size | Agent | Model |
|--------------------|-------|-------|
| ≤ 5 files | `oh-my-claudecode:executor` | sonnet |
| > 5 files | `oh-my-claudecode:deep-executor` | opus |

## Build Verify/Fix Loop (max 5 iterations)

```
WHILE iteration < 5:
  1. Run: npm run type-check (+ npm run build:gqlgen if DTO changes)
  2. IF PASS → break
  3. IF FAIL → delegate to oh-my-claudecode:build-fixer (sonnet)
     - Provide: error output + changed files list
     - Rule: fix build/type errors only, do NOT change tests
  4. iteration++

IF iteration == 5 → escalate to developer
```

## Test Writing (test-engineer, sonnet)

Delegate to `oh-my-claudecode:test-engineer` (sonnet):

Rules:
1. Follow test plan from design-plan.md Section 4
2. Follow project testing conventions from coding-guide
3. Cover all test cases + edge cases
4. Use project's test file naming convention

## Test Verify/Fix Loop (max 5 iterations)

Same pattern as build loop, but:
- Run: `npm run test -- --testPathPattern="{test paths}"`
- build-fixer may fix both test code and implementation code

## Parallel Code Review

Run **simultaneously**:

### Security Review (security-reviewer, sonnet)
Focus:
1. Input validation and sanitization
2. Authentication and authorization checks
3. SQL/query injection risks
4. Data exposure risks
5. OWASP Top 10

### Code Review (code-reviewer, opus)
Focus:
1. Design plan adherence — all items implemented?
2. Code conventions — project patterns followed?
3. Error handling
4. Performance — N+1 queries, missing indexes
5. Maintainability — naming, structure, complexity

### Review Result Handling
- No critical issues → proceed to PR
- Critical issues found → fix and re-review
- Developer can override: "Proceed anyway"

## PR Creation

### Branch
```
feature/{req-id}
```

### Commit Message
```
feat({req-id}): {short description}

Based on design plan: .claude/workspace/{req-id}/design-plan.md
```

### PR Body
```markdown
## Summary
{from design-plan.md}

## Changes
{changed files table}

## Review Report
{security + code review summary}

## Test Coverage
{test files table}
```

## Output Artifacts

```
{project-root}/.claude/workspace/{req-id}/
├── design-plan.md              ← input (read)
├── design-plan-approved.md     ← input (gate)
├── _feature-request.md         ← input (optional)
├── _implementation-report.md   ← output
├── _review-report.md           ← output
└── pr-report.md                ← output
```

## Quality Criteria

- [ ] All design plan items implemented
- [ ] Build passes (type-check + gqlgen)
- [ ] All tests pass
- [ ] Security review: no critical issues
- [ ] Code review: no critical issues
- [ ] PR created with proper description
- [ ] Branch follows naming convention
