---
name: spec-design
description: "Rules for generating design plans from feature-requests. Covers codebase exploration (Layer 3 AST-primary), requirements analysis (Layer 4 precision), and the 6-section design-plan.md format. See context-layer-protocol. Use with ralph for persistent execution."
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

### Step A: Codebase Exploration (explore, haiku) — ENHANCED

See `context-layer-protocol` Layer 3 for full specification.

Delegate to `oh-my-claudecode:explore` (haiku):

Tasks (existing):
1. Find existing modules, services, resolvers related to FR's entities
2. Find existing DB models/tables referenced in Technical Details
3. Find existing GraphQL types referenced
4. Identify integration points with existing code
5. Note patterns in related modules

Tasks (NEW — Layer 3, AST grep PRIMARY):
6. For each found service/resolver, use `ast_grep_search` to extract:
   - Public method signatures and constructor dependencies:
     `ast_grep_search("@Injectable() class $SERVICE { constructor($DEPS) { $$$BODY } }")`
   - Decorator usage patterns:
     `ast_grep_search("@UseGuards($GUARD) $METHOD")`
     `ast_grep_search("@Roles($ROLE) $METHOD")`
7. For resolvers, extract query/mutation signatures:
   `ast_grep_search("@Query(() => $TYPE) async $NAME($ARGS) { $$$BODY }")`
   `ast_grep_search("@Mutation(() => $TYPE) async $NAME($ARGS) { $$$BODY }")`
8. For key types, extract type/interface definitions:
   `ast_grep_search("@ObjectType() class $TYPE { $$$FIELDS }")`
   `ast_grep_search("@InputType() class $TYPE { $$$FIELDS }")`

#### Future Enhancement: LSP Tools (when explore-high agent is available)

When an `explore-high` agent is created with `lsp_find_references`, `lsp_goto_definition`,
and `lsp_hover` capabilities, tasks 6-8 can be upgraded to use LSP for more precise results:
- `lsp_document_symbols`: full type information for method signatures
- `lsp_find_references`: caller graph and dependency direction mapping
- `lsp_hover`: complete type definitions for complex GraphQL types
Until then, AST grep provides sufficient structural pattern matching for Layer 3.

Save to: `{workspace}/features/{fr-id}/_explore-report.md`

### Step B: Requirements Analysis (analyst, opus) — ENHANCED

See `context-layer-protocol` Layer 4 for full specification.

**Pre-analysis scan (Layer 4 — AST grep precision):**

Before delegating to analyst, gather implementation pattern data:

1. Find similar existing implementations to use as reference:
   `ast_grep_search("@Resolver(() => $TYPE) class $NAME { }")`
   → Identifies resolver patterns to follow

2. Find test patterns in the same module:
   `ast_grep_search("describe('$SERVICE', () => { })")`
   → Identifies test conventions to replicate

3. Find error handling patterns:
   `ast_grep_search("throw new $EXCEPTION($MSG)")`
   → Identifies exception types used in similar modules

4. Find validation patterns:
   `ast_grep_search("@IsNotEmpty() $FIELD: $TYPE")`
   → Identifies class-validator decorators used

Apply Context Budget: filter to max 1000 tokens, keep top 3-5 most relevant
patterns from the same module as the target FR's entities.
Pass as `$IMPLEMENTATION_PATTERNS` to analyst.
Note: `$IMPLEMENTATION_PATTERNS` is passed ONLY to analyst (opus), never to explore (haiku).

**Delegate to `oh-my-claudecode:analyst` (opus):**

Tasks (existing):
1. Break down into concrete implementation requirements
2. Identify DB schema changes (tables, columns, relations, indexes)
3. Identify API schema changes (queries, mutations, types, inputs)
4. Identify service logic changes (business rules, validations, side effects)
5. Flag risks: breaking changes, migration, performance, security
6. Define acceptance criteria
7. Estimate file scope

Tasks (NEW — Layer 4 enhanced):
8. Reference `$IMPLEMENTATION_PATTERNS` to ensure design-plan recommendations
   match the project's actual coding patterns (not generic NestJS patterns)
9. Include "Reference Implementation" section in analysis — point to
   1-2 existing similar features as implementation guides

Return analysis directly (do NOT save to file).

### Step C: Write design-plan.md

## Design Plan Template (6-Section Format)

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

## 6. Reference Implementation

### Similar Existing Feature
- **Module**: {module path}
- **Why Similar**: {brief explanation}

### Patterns to Follow
| Pattern | Source | Example |
|---------|--------|---------|
| {pattern name} | {source file path} | {brief code reference} |

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

- [ ] All 6 sections present in design-plan.md (5 core sections + Section 6: Reference Implementation)
- [ ] DB Schema Changes reference actual table/model names from domain mapping
- [ ] API Schema Changes include auth requirements
- [ ] Service Logic includes business rules from feature-request
- [ ] Test Plan covers acceptance criteria from feature-request
- [ ] Reference Implementation points to existing similar feature with concrete file paths
- [ ] Implementation Files list is realistic (matches sizing criteria)
- [ ] Risks section identifies at least migration and breaking change risks
