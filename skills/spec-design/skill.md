---
name: spec-design
description: "Rules for generating design plans from feature-requests. Covers codebase exploration, requirements analysis, and the 5-section design-plan.md format. Use with ralph for persistent execution."
---

# Spec Design (Design Plan Generation)

Rules for generating design-plan.md files from feature-requests.

## When to Use

- "이 feature-request의 디자인 플랜 만들어줘"
- "spec-design 스킬로 디자인 플랜 생성해줘"
- Feature-request → design-plan.md 변환이 필요할 때

## Prerequisites

- `_feature-request.md` file exists
- (Optional) `_explore-report.md` from previous codebase exploration
- (Optional) `coding-guide/skill.md` for project conventions

## Workflow Per Feature-Request

### Step A: Codebase Exploration (explore, haiku)

Delegate to `oh-my-claudecode:explore` (haiku):

Tasks:
1. Find existing modules, services, resolvers related to FR's entities
2. Find existing DB models/tables referenced in Technical Details
3. Find existing GraphQL types referenced
4. Identify integration points with existing code
5. Note patterns in related modules

Save to: `{workspace}/features/{fr-id}/_explore-report.md`

### Step B: Requirements Analysis (analyst, opus)

Delegate to `oh-my-claudecode:analyst` (opus):

Tasks:
1. Break down into concrete implementation requirements
2. Identify DB schema changes (tables, columns, relations, indexes)
3. Identify API schema changes (queries, mutations, types, inputs)
4. Identify service logic changes (business rules, validations, side effects)
5. Flag risks: breaking changes, migration, performance, security
6. Define acceptance criteria
7. Estimate file scope

Return analysis directly (do NOT save to file).

### Step C: Write design-plan.md

## Design Plan Template (5-Section Format)

```markdown
# Design Plan: {fr-id}

## Summary
[1-2 paragraph overview]

---

## 1. DB Schema Changes

### New Tables
| Table | Purpose |

### Column Changes
| Table | Column | Type | Constraint | Note |

### Migration Notes
- [Safety considerations]

---

## 2. API Schema Changes

### New Queries
| Query | Return Type | Auth | Description |

### New Mutations
| Mutation | Input | Return Type | Auth | Description |

### Type Changes
| Type | Change | Fields |

---

## 3. Service Logic Changes

### New Services/Methods
| Service | Method | Purpose |

### Business Rules
1. [Rule]

### Side Effects
| Trigger | Effect | Mitigation |

---

## 4. Test Plan

### Unit Tests
| Test Case | Target | Expected |

### Edge Cases
1. [Edge case]

---

## 5. Verification Strategy

### Build Verification
- [ ] `npm run type-check` passes
- [ ] `npm run build:gqlgen` succeeds (if DTO changes)

### Test Verification
- [ ] All new tests pass
- [ ] No existing tests broken

### Manual Verification
- [ ] [Specific check]

---

## Implementation Files (Estimated)
| # | File | Action | Description |

## Risks & Open Questions
1. [Risk or question]
```

## Batch Processing (for spec pipeline)

When processing multiple FRs from a spec decomposition:

1. Process in **wave order** (Wave 1 first, then Wave 2, etc.)
2. Within each wave: run explore agents in **parallel** (up to 5 concurrent)
3. Wait for all explore in wave to complete
4. Run analyst agents in **parallel** (up to 5 concurrent)
5. Wait for all analyst in wave to complete
6. Write design-plan.md for each FR
7. Proceed to next wave

## Quality Criteria

- [ ] All 5 sections present in design-plan.md
- [ ] DB Schema Changes reference actual table/model names from domain mapping
- [ ] API Schema Changes include auth requirements
- [ ] Service Logic includes business rules from feature-request
- [ ] Test Plan covers acceptance criteria from feature-request
- [ ] Implementation Files list is realistic (matches sizing criteria)
- [ ] Risks section identifies at least migration and breaking change risks
