# Proven Examples — linkareer-main Pipeline Reference

## Feature Pipeline (Phase 1 — Production Verified)

### Full Flow

```
/feature-start {req-id} --base subgraph
    -> branch: feature/{req-id}, workspace init

/feature-draft {req-id}
    -> Phase 1: feature-analyzer (haiku) -> _analysis.md
    -> Phase 2: feature-schema-designer (sonnet) -> _api-design.md, _db-design.md
    -> Phase 3: feature-logic-designer (sonnet) -> _service-design.md, _test-design.md
    -> Phase 4: Consolidate -> draft.md
    -> Phase 5: Auto-commit

    (Human: team discussion -> write discussion.md)

/feature-finalize {req-id}
    -> Gate: draft.md + discussion.md verified
    -> feature-finalizer -> final_draft.md
    -> Auto-commit

/feature-implement {req-id}
    -> Gate: final_draft.md verified
    -> Phase 1: feature-implementor -> code implementation
    -> Phase 2: Side Effect Analysis (main context)
    -> Phase 3: implementation_report.md
    -> Auto-commit

/feature-test {req-id}
    -> Gate: implementation_report.md verified
    -> feature-tester -> test writing + execution
    -> test_report.md
    -> Auto-commit

/feature-submit {req-id}
    -> Gate: final_draft.md + implementation_report.md + test_report.md verified
    -> PR description auto-generation
    -> Human confirmation -> push + PR creation

/review-start {req-id}
    -> 5 agents parallel (P0~P4)
    -> review_report.md
```

### Component Inventory

| Type | Name | Model | Role |
|------|------|-------|------|
| Command | feature-start | - | Branch + workspace init |
| Command | feature-draft | - | Design orchestration |
| Command | feature-finalize | - | Spec confirmation orchestration |
| Command | feature-implement | - | Implementation orchestration |
| Command | feature-test | - | Test orchestration |
| Command | feature-submit | - | Push + PR creation |
| Command | review-start | - | Review orchestration |
| Agent | feature-analyzer | haiku | AS-IS/TO-BE analysis |
| Agent | feature-schema-designer | sonnet | API + DB design |
| Agent | feature-logic-designer | sonnet | Service + Test design |
| Agent | feature-implementor | sonnet | Code implementation |
| Agent | feature-tester | sonnet | Test writing |
| Agent | feature-finalizer | sonnet | Spec confirmation |
| Skill | nestjs-conventions | - | NestJS patterns |
| Skill | graphql-api-conventions | - | GraphQL patterns |
| Skill | prisma-patterns | - | Prisma patterns |
| Skill | codebase-structure | - | Codebase structure |
| Skill | testing-patterns | - | Test patterns |

### Key Patterns

1. **1 req-id = 1 branch = 1 workspace**: Requirement ID is the key for everything
2. **draft -> discussion -> finalize**: Human-in-the-Loop design confirmation pattern
3. **Agent-Skill role separation**: Agent = judge, Skill = knowledge provider
4. **Side Effect Analysis**: Auto-analyze impact scope after implementation
5. **Auto-commit at checkpoints**: Commit only at meaningful checkpoints

---

## Review Pipeline (5 Parallel Agents)

### Structure

```
/review-start
    |
    |- P0: review-deploy-safety        (always runs)
    |- P1: review-client-compatibility  (runs when resolver/DTO changed)
    |- P2: review-performance           (runs when resolver/service changed)
    |- P3: review-security              (always runs)
    '- P4: review-code-quality          (always, non-blocking)
    |
    '- Result consolidation -> review_report.md
```

### Agent-Skill Role Separation Example

| | Agent Responsibility | Skill Responsibility |
|---|---------------------|---------------------|
| What | "Is this really a problem?" judgment | "How to check?" procedure |
| Output | Severity (BLOCKER/CRITICAL/WARNING) | Checklist items |
| False Positives | Filters them | Lists all patterns |

### Final Verdict Logic

```
BLOCKED     <- P0 blocker present (deploy blocked)
CONDITIONAL <- P1 breaking changes OR critical P2/P3 (conditional resolution needed)
APPROVED    <- No above conditions (deploy ready)
```
