---
name: spec-decomposition-patterns
description: "Decomposition rules for splitting PDF specs into right-sized, dependency-aware feature-requests. Covers foundation+domain separation pattern, sizing criteria, dependency graph generation, execution wave calculation, and Phase 1 pipeline compatibility."
---

# Spec Decomposition Patterns

Rules and patterns for decomposing PDF design specs into feature-request.md files.

## Foundation + Domain Separation Pattern

Every feature group extracted from the spec MUST be split into exactly two layers:

**Foundation Feature-Request (1 per group per repo):**
- New DB tables and columns (Prisma schema changes + migration)
- GraphQL type/input/enum definitions (or REST DTO for non-GraphQL repos)
- Empty Resolver stubs with `// TODO: implement in domain FR` comments
- Minimal Service scaffolding (constructor, DI setup, empty method signatures)
- NO business logic, NO validation rules, NO complex queries

**Domain Feature-Request (1 per user-facing function):**
- Business logic implementation on top of foundation's schema/API
- 1 user-facing function = 1 domain feature-request
- References foundation's types, tables, and resolver stubs
- Includes: validation, business rules, complex queries, error handling
- Must declare dependency on its group's foundation

**Rules:**

| Rule | Description |
|------|-------------|
| One foundation per group per repo | If a feature group touches 2 repos, each repo gets its own foundation |
| Foundation always first | Domain requests MUST depend on their foundation |
| No logic in foundation | Foundation is purely structural — schema + interface definitions |
| Single-function groups | If a group has only 1 function, still split into foundation + 1 domain (keeps pattern consistent, foundation stays reusable) |
| Read-only features | No foundation needed if reusing existing tables/types — create domain-only with `foundation: none` annotation |
| Schema-only features | If a feature is purely DB/API structure with no logic (e.g., adding a new entity type), foundation-only is acceptable |

**Example: "Application Status Management" group**

```
Group: Application Status Management
Repo: {primary-repo}

Foundation: 01-application-status-foundation.md
  - Add `ApplicationStatusHistory` table
  - Add `ApplicationStatus` enum (PENDING, REVIEWED, ACCEPTED, REJECTED)
  - Add `applicationStatusHistories` relation to Application model
  - Define `ApplicationStatusHistory` GraphQL type
  - Define `updateApplicationStatus` mutation signature
  - Empty resolver + service stubs

Domain 1: 02-application-status-update.md
  - Implement updateApplicationStatus mutation
  - Validation: only employer can update, valid status transition
  - Create history record on status change
  - depends_on: [01-application-status-foundation]

Domain 2: 03-application-status-notification.md
  - Send notification on status change (SQS → serverless)
  - depends_on: [01-application-status-foundation, 02-application-status-update]

Cross-repo: (in {secondary-repo})
Domain 3: 01-application-status-push.md
  - Lambda handler for application status push notification
  - depends_on: [{primary-repo}/02-application-status-update]
```

## Sizing Criteria

**The 1-Day Rule:** Each feature-request must be completable in 1 working day.

| Metric | Limit | What Counts |
|--------|-------|-------------|
| GraphQL Resolvers | ≤ 3 | Query, Mutation, or Subscription entry points |
| Service Methods | ≤ 5 | Public methods in the service class |
| DB Table Changes | ≤ 2 | New tables OR significant alterations (concentrated in foundation) |
| Changed Files | ≤ 10 | .ts, .prisma, .graphql files (excluding auto-generated) |

**When Limits Are Exceeded — Split Strategies:**

| Situation | Strategy |
|-----------|----------|
| Too many resolvers (> 3) | Split by query vs mutation, or by entity |
| Too many service methods (> 5) | Split by CRUD groups or by business operation |
| Too many tables (> 2) | Split foundation into core-tables + extension-tables |
| Too many files (> 10) | Usually indicates scope creep — re-examine if multiple functions are merged |

**Edge Cases:**

| Case | Handling |
|------|----------|
| Single complex resolver | If 1 resolver has complex aggregation/joins, it counts as 1 but may need its own feature-request |
| Read-only feature | No table changes — sizing focuses on resolver + service count |
| REST-only repo (point-api, cms-api) | Replace "Resolver" with "Controller endpoint" — same limits apply |
| Serverless Lambda | 1 Lambda = 1 feature-request regardless of complexity |
| Prisma migration only | Foundation with 0 resolvers is valid — size by table count |

**Per-Repo Adjustment:** Refer to the project's repo-structure skill for per-repo sizing adjustments.

## Dependency Graph Rules

**Dependency Types:**

| Type | Notation | Description |
|------|----------|-------------|
| Intra-group | `depends_on: [nn-name]` | Foundation → domain within same group and repo |
| Inter-group | `depends_on: [nn-name]` | Domain in group A depends on foundation of group B (same repo) |
| Cross-repo | `depends_on: [{repo}/nn-name]` | Feature in repo X depends on feature in repo Y |

**Dependency Declaration Rules:**

1. **Foundation has no intra-group dependencies** — it is always the root of its group
2. **All domain requests depend on their foundation** — explicit `depends_on` required
3. **Cross-repo dependencies use repo prefix** — `depends_on: [{repo-name}/01-feature-foundation]`
4. **Transitive dependencies are NOT declared** — only direct dependencies (if A→B→C, A declares B only)
5. **No circular dependencies allowed** — if detected, restructure by extracting shared foundation

**Breaking Complex Dependencies:**

```
Problem: Feature A (main) needs data from Feature B (xen),
         and Feature B needs data from Feature A.

Solution: Extract shared schema into a foundation in the repo
          that owns the canonical data, then both depend on that foundation.

main/01-shared-foundation.md  (no dependencies)
main/02-feature-a.md          (depends_on: [01-shared-foundation])
xen/01-feature-b.md           (depends_on: [{primary-repo}/01-shared-foundation])
```

## Execution Wave Calculation

**Algorithm:**

```
1. Build dependency graph (directed acyclic graph)
2. Topological sort all feature-requests
3. Assign wave = max(wave of all dependencies) + 1
   - If no dependencies: wave = 1
4. Group by wave number
5. Within each wave: all items are parallelizable
```

**Standard Wave Pattern:**

| Wave | Contents | Parallelism |
|------|----------|-------------|
| Wave 1 | All independent foundations | Fully parallel across groups and repos |
| Wave 2 | Domain features depending only on Wave 1 foundations | Parallel within wave |
| Wave 3 | Domain features depending on Wave 2 items | Parallel within wave |
| Wave N (final) | Cross-repo integrations, notification handlers | Parallel within wave |

**Example: 3 groups across 2 repos**

```
Group A ({primary-repo}): Application Status
Group B ({primary-repo}): Resume Scoring
Group C ({secondary-repo}): Status Notifications

Wave 1 (parallel):
  - main/01-app-status-foundation
  - main/04-resume-scoring-foundation

Wave 2 (parallel):
  - main/02-app-status-update        (depends: 01)
  - main/03-app-status-history-view  (depends: 01)
  - main/05-resume-auto-score        (depends: 04)

Wave 3 (parallel):
  - main/06-resume-score-display     (depends: 05)
  - serverless/01-status-push-lambda (depends: main/02)

Total: 3 waves, 7 feature-requests
```

**Wave Assignment in Feature-Request:**

Each feature-request file includes an `execution_wave` field in its Execution Context section.

## Domain Mapping Reference Rule

When generating feature-requests, the spec-decomposer MUST reference `_domain-mapping.md` to use accurate code entities.

**Rules:**

| Rule | Description |
|------|-------------|
| Always reference mapping | Technical Details MUST use DB Table, ORM Model, and GraphQL Type from the mapping |
| No guessing | If a term is NOT in the mapping, use the spec wording as-is with a `(unmapped)` annotation |
| Foundation uses DB/ORM | Foundation FRs reference DB Table and ORM Model for schema definitions |
| Domain uses all three | Domain FRs reference DB Table, ORM Model, and GraphQL Type for resolver/service code |
| Include Domain Terms section | Each FR includes a `## Domain Terms` section listing the mapping entries it references |

**Example — Foundation FR Technical Details with mapping:**

```markdown
## Technical Details

**Database Schema (Prisma)**:
- New table: `rec_posting_daily_metrics` (ORM: `RecPostingDailyMetric`)
  - Fields: id, activityInstanceId (FK → `rec_activityinstances`), date, viewCount, scrapCount, qaCount, applicantCount
  - Unique constraint on (activityInstanceId, date)
- New columns on `rec_activityinstances` (ORM: `RecActivityInstance`):
  - totalViewCount, totalScrapCount, isHidden

## Domain Terms
| Spec Term | DB Table | ORM Model | GraphQL Type |
|-----------|----------|-----------|--------------|
| 공고 | rec_activityinstances | RecActivityInstance | ActivityDto |
| 조회수 | rec_activity_view_logs | RecActivityViewLog | - |
```

**Example — Domain FR Technical Details with mapping:**

```markdown
## Technical Details

**GraphQL Resolver**:
- `dashboardMetrics(postingId: ID)` → queries `RecActivityInstance` (공고) + aggregates from `RecActivityViewLog` (조회수), `RecScrap` (스크랩)

**Service Methods**:
- `aggregateTodayMetrics(activityInstanceIds: number[])` — queries `rec_activity_view_logs`, `rec_scraps`, `rec_applications` for today's date range

## Domain Terms
| Spec Term | DB Table | ORM Model | GraphQL Type |
|-----------|----------|-----------|--------------|
| 공고 | rec_activityinstances | RecActivityInstance | ActivityDto |
| 조회수 | rec_activity_view_logs | RecActivityViewLog | - |
| 스크랩 | rec_scraps | RecScrap | - |
```

## Feature-Request File Format

Each feature-request generated by spec-decomposer follows this template:

```markdown
# {Title}

## Metadata
- **ID**: {nn}-{short-name}-{type}
- **Repo**: {repo-name}
- **Type**: foundation | domain
- **Group**: {feature-group-name}
- **Depends On**: [{dependency-ids}]

## Execution Context
- **Source Spec**: {spec-id}
- **Execution Wave**: {wave-number}
- **Work Scope**: {foundation-scope | domain-scope description}

## Scope
{What this feature-request covers — specific deliverables}

## Out of Scope
{What is explicitly NOT included — handled by other feature-requests}

## Technical Details
{Implementation guidance using actual code entities from _domain-mapping.md}

## Domain Terms
{Subset of _domain-mapping.md entries referenced in this FR}
| Spec Term | DB Table | ORM Model | GraphQL Type |
|-----------|----------|-----------|--------------|
| {term} | {table} | {model} | {type} |

## Acceptance Criteria
- [ ] {Testable criterion 1}
- [ ] {Testable criterion 2}
```

## Naming Conventions

**Feature-Request IDs:**

| Part | Format | Example |
|------|--------|---------|
| Sequence number | 2-digit, zero-padded | `01`, `02`, `10` |
| Short name | kebab-case, max 3 words | `app-status`, `resume-scoring` |
| Type suffix | `-foundation` or function name | `-foundation`, `-update`, `-notification` |
| Full ID | `{nn}-{name}-{type}` | `01-app-status-foundation` |

**Directory Structure:**

```
feature-requests/
├── 01_{primary-repo}/
│   ├── 01-app-status-foundation.md
│   ├── 02-app-status-update.md
│   ├── 03-app-status-history-view.md
│   ├── 04-resume-scoring-foundation.md
│   └── 05-resume-auto-score.md
├── 02_{secondary-repo}/
│   └── 01-discussion-tags-foundation.md
└── 03_{tertiary-repo}/
    └── 01-status-push-lambda.md
```

**Numbering Rules:**
1. Sequential per repo directory (01, 02, 03...)
2. Foundations always get the lowest number in their group
3. Domain requests follow their foundation in sequence
4. Repo directory prefix: `{nn}_{repo-name}` — ordered by primary dependency (main first)

## Phase 1 Pipeline Compatibility

Feature-requests generated by this pipeline feed into the existing Phase 1 pipeline:
`/feature-start → /feature-draft → ... → /feature-submit`

**Compatibility Requirements:**

| Requirement | How It's Met |
|-------------|-------------|
| Feature-request.md format | Template includes all fields expected by /feature-draft |
| Repo identification | `Repo` field in Metadata maps to project's repo-structure skill |
| Scope clarity | `Scope` + `Out of Scope` prevent scope creep in /feature-draft |
| Technical guidance | `Technical Details` section guides /feature-draft implementation |

**Foundation vs Domain Behavior in /feature-draft:**

| Request Type | /feature-draft Behavior |
|-------------|------------------------|
| Foundation | Design-focused: generates Prisma schema, GraphQL types, empty stubs. Minimal test coverage (schema validation only). |
| Domain | Logic-focused: implements business rules, validation, queries. Full test coverage expected. |

**Execution Context Section** links each feature-request back to the source spec:
- `Source Spec`: which spec-id this came from
- `Execution Wave`: when to execute relative to other feature-requests
- `Work Scope`: foundation-scope (schema/interface) or domain-scope (logic/validation)

This section is informational for the developer — /feature-draft uses `Scope` and `Technical Details` for actual implementation guidance.
