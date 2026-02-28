---
description: "Analyze a PDF design spec, auto-propose domain mappings, decompose into right-sized feature-requests with dependency graph and execution waves, and generate design-plans for each feature. Replaces spec-analyze, spec-translate, spec-decompose, and spec-confirm."
allowed-tools: ["Task", "Read", "Write", "Edit", "Glob", "Grep", "Bash", "AskUserQuestion"]
argument-hint: "spec-id --pdf {path} --project {project}"
---

# /spec:plan {spec-id} --pdf {path} --project {project}

Unified spec-to-design-plan pipeline. Analyzes a PDF spec, auto-proposes domain mappings using a 3-tier lookup, decomposes into feature-requests, and generates design-plan.md for each FR.

**Usage:** `/spec:plan <spec-id> --pdf <path> --project <project>`

## Arguments

Parse from `$ARGUMENTS`:
- `spec-id`: First argument — kebab-case identifier for this spec
- `--pdf <path>`: Absolute path to the PDF design spec file
- `--project <project>`: Project name (e.g., `linkareer`)

Optional:
- `--force`: Overwrite existing workspace if `_spec-analysis.md` already exists

## Gate Validation

Run all gates sequentially. Stop at the first failure.

### Gate 1: Arguments present

- If `spec-id` is missing: use AskUserQuestion "What is the spec ID? (kebab-case, e.g., `biz-job-performance-v7`)"
- If `--pdf` is missing: use AskUserQuestion "What is the path to the PDF spec file?"
- If `--project` is missing: use AskUserQuestion "What is the project name? (e.g., `linkareer`)"

### Gate 2: PDF exists and readable

1. Check if the PDF file exists at the given path
2. If not found: "Error: PDF not found at `{path}`. Check the path and try again."

### Gate 3: Project root detection

1. Detect the project root by searching for the project directory. Strategy:
   - Check if cwd is within a directory containing `.claude/skills/coding-guide/`
   - Check common paths: `~/dev/{project}/*`, `~/dev/linkareer/linkareer-main/` etc.
   - Use the `--project` value to guide discovery
2. Set `$PROJECT_ROOT` to the detected project root
3. If not found: use AskUserQuestion "Could not detect project root. What is the absolute path?"

### Gate 4: Coding-guide check

1. Check if `{$PROJECT_ROOT}/.claude/skills/coding-guide/skill.md` exists
2. If exists:
   - Read the coding-guide manifest
   - Read all skill files listed under "Skills to Load" section
   - Store combined content as `$CODING_GUIDE`
3. If missing:
   - Display warning: "No coding-guide found. Proceeding in generic mode."
   - Set `$CODING_GUIDE` to empty string

### Gate 5: Duplicate check

1. Check if `{$PROJECT_ROOT}/.claude/workspace/specs/{spec-id}/_spec-analysis.md` exists
2. If exists AND `--force` not provided:
   - Use AskUserQuestion: "Workspace for `{spec-id}` already exists. What would you like to do?"
   - Options: "Overwrite and re-run" / "Stop — use existing workspace"
   - If stop: halt execution

## Execution

### Phase 1: Parallel Analysis (analyst + explore)

Create workspace:
```
mkdir -p {$PROJECT_ROOT}/.claude/workspace/specs/{spec-id}/features
```

Display to user:
```
Starting spec:plan pipeline...
- Spec ID: {spec-id}
- PDF: {path}
- Project: {project}
- Coding Guide: {found / generic mode}
- Workspace: {$PROJECT_ROOT}/.claude/workspace/specs/{spec-id}/
```

Run Phase 1A and 1B **in parallel** (two Task calls in a single response):

#### Phase 1A — analyst (opus): PDF Analysis

Delegate to `oh-my-claudecode:analyst` (opus):

```
You are analyzing a PDF design spec (기획서) for the spec-decomposition pipeline. Your job is to extract structured information from the PDF.

PDF Path: {pdf_path}

Read the PDF and extract:

1. **Cover Page**: Spec title, version, date, author
2. **Checklist (체크리스트)**: Feature items with include/exclude status
   - Look for columns: No, 기능/기능명 (Feature), 반영여부 (Status: O=included, X=excluded), 비고 (Notes)
   - Mark ambiguous items as `uncertain`
3. **Detailed Specs**: For each included checklist item, extract:
   - Screen/UI descriptions
   - Field definitions with types and validation rules
   - Business rules and state transitions
   - Access control requirements
4. **Feature Grouping**: Group included items by domain:
   - Same entity/model → same group
   - Same user flow → same group
   - Same domain → same group
   - Target 2-6 items per group
5. **Domain Terms**: Extract business-specific terms (Korean, exact wording):
   - Business entities: 공고, 지원자, 스크랩, etc.
   - Business metrics: 조회수, 관심, etc.
   - Business states: 진행, 마감, 대기, etc.
   - Business actions: 공고 마감, 숨기기, etc.
   - Business concepts: 채용 현황, 공고 성과, etc.
   - Include brief description for each term
   - Deduplicate across groups

Return your analysis in this EXACT structure (do NOT write to file):

## Metadata
- Title: ...
- Version: ...
- Date: ...

## Checklist
### Included Items
| # | Feature | Page Ref | Notes |
### Excluded Items
| # | Feature | Reason |
### Uncertain Items
| # | Feature | Issue |

## Feature Groups
### Group: {name}
- Items: #{n1}, #{n2}
- Domain: {description}
- Description: {1 sentence}
- Key Entities: {entity names}

## Domain Terms
| Term | Description | Used In Groups |
```

Use Task tool with:
- `subagent_type`: `oh-my-claudecode:analyst`
- `model`: opus

#### Phase 1B — explore (haiku): Codebase Scan

Delegate to `oh-my-claudecode:explore` (haiku):

```
Scan the project codebase to build an entity inventory for domain mapping.

Project Root: {$PROJECT_ROOT}

Tasks:
1. Scan Prisma schema files (*.prisma) — extract ALL model names + @@map table names
2. Scan for @ObjectType() / @InputType() decorators — extract GraphQL type names
3. Scan for @Injectable() service classes — extract service names and module paths
4. Build entity inventory

{if $CODING_GUIDE is not empty:}
Project Coding Guide:
{$CODING_GUIDE}
{end if}

Save your findings to: {$PROJECT_ROOT}/.claude/workspace/specs/{spec-id}/_codebase-scan.md

Format:
## Prisma Models
| Model | Table (@@map) | File |
|-------|--------------|------|

## GraphQL Types
| Type | Decorator | File |
|------|-----------|------|

## Services
| Service | Module Path | File |
|---------|------------|------|

Return a 3-line summary of what you found.
```

Use Task tool with:
- `subagent_type`: `oh-my-claudecode:explore`
- `model`: haiku

#### Phase 1C — Main Context: 3-Tier Domain Mapping Auto-Proposal

After both 1A and 1B complete:

1. **Load domain dictionary**: Read `{$PROJECT_ROOT}/.claude/skills/domain-dictionary.md`
2. **Load codebase scan**: Read `{$PROJECT_ROOT}/.claude/workspace/specs/{spec-id}/_codebase-scan.md`
3. **Load analyst results**: Use the analyst's returned domain terms

4. **For each domain term from analyst**:

   **Tier 1 — Dictionary Lookup (TRUSTED)**:
   - Search domain dictionary for exact match on `기획서 용어` or `동의어`
   - If found: mark as `TRUSTED` — use the dictionary's Prisma Model, DB Table, GraphQL Type
   - TRUSTED items can be skipped during review (already verified)

   **Tier 2 — Codebase Scan Fuzzy Match (PROPOSED [?])**:
   - If not in dictionary: fuzzy-match term against codebase scan results
   - Match strategies: Korean-English translation similarity, entity name similarity, module path keyword matching
   - If plausible match found: mark as `PROPOSED [?]` — fill in code entities from scan
   - Developer must review and confirm

   **Tier 3 — No Match (NEW)**:
   - If no match anywhere: mark as `[NEW]`
   - Developer must fill in code entities or confirm as N/A

5. **Write `_spec-analysis.md`**: Combine analyst output with metadata into the standard analysis artifact format:

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
{from analyst output}

## Feature Groups
{from analyst output}

## Domain Terms
{from analyst output}

## Estimates
| Metric | Count |
|--------|-------|
| Feature Groups | {n} |
| Est. Foundation FRs | {n} (1 per group) |
| Est. Domain FRs | {n} |
| Est. Total FRs | {n} |
```

6. **Write `_domain-mapping.md`**: Pre-filled with auto-proposed mappings:

```markdown
# Domain Mapping: {spec-id}

## Context
- **Spec ID**: {spec-id}
- **Project**: {project}
- **Created**: {current date}
- **Status**: Auto-proposed (developer review required for PROPOSED and NEW items)

## Legend
- **TRUSTED**: From domain dictionary — verified in previous specs, skip review unless wrong
- **PROPOSED [?]**: From codebase scan — plausible match, developer must verify
- **[NEW]**: No match found — developer must fill in or mark N/A

## Mapping

| # | Spec Term | Description | DB Table | ORM Model | GraphQL Type | Source | Notes |
|---|-----------|-------------|----------|-----------|--------------|--------|-------|
{for each domain term:}
| {n} | {term} | {description} | {table or empty} | {model or empty} | {type or empty} | {TRUSTED/PROPOSED [?]/[NEW]} | {notes} |

## Review Checklist
- [ ] All PROPOSED [?] items verified or corrected
- [ ] All [NEW] items filled in or marked N/A
- [ ] TRUSTED items spot-checked (optional — only if something looks wrong)
```

### H1: Developer Reviews Analysis + Domain Mapping

Display to user:

```
## Phase 1 Complete — Analysis + Domain Mapping Ready

### Checklist Summary
- Included: {n} items
- Excluded: {n} items
- Uncertain: {n} items

### Feature Groups: {n}
{list group names with item counts}

### Domain Mapping Summary
- TRUSTED (from dictionary): {n} terms
- PROPOSED [?] (from codebase scan): {n} terms — needs your review
- [NEW] (no match): {n} terms — needs your input

### Files to Review
1. `_spec-analysis.md` — Checklist + feature groups
2. `_domain-mapping.md` — Domain term → code entity mapping
```

Use AskUserQuestion:
- Question: "Review the analysis and domain mapping. Are the feature groups and domain mappings correct?"
- Options:
  - "Approved — proceed to decomposition"
  - "I've made edits — re-read and proceed"
  - "Stop — I need more time to review"

If "I've made edits": re-read `_spec-analysis.md` and `_domain-mapping.md` from disk before proceeding.
If "Stop": halt execution.

#### Post-H1: Auto-Update Domain Dictionary

After H1 approval:
1. Re-read `_domain-mapping.md` (may have been edited)
2. For each term that was PROPOSED [?] and now has confirmed values, or [NEW] with filled values:
   - Check if term already exists in domain dictionary (same `기획서 용어` + same domain section)
   - If not exists: append to appropriate domain section in `{$PROJECT_ROOT}/.claude/skills/domain-dictionary.md`
   - If exists but mapping changed: update the row
3. Write updated dictionary file (produces git-trackable diff)
4. Display: "Updated domain dictionary with {n} new/updated terms."

### Phase 2: Architecture Decomposition (architect)

Delegate to `oh-my-claudecode:architect` (opus):

```
You are decomposing a spec analysis into right-sized, dependency-aware feature-requests.

Spec Analysis:
{content of _spec-analysis.md}

Domain Mapping (confirmed by developer):
{content of _domain-mapping.md}

{if $CODING_GUIDE is not empty:}
Project Coding Guide:
{$CODING_GUIDE}
{end if}

Decomposition Rules (follow EXACTLY):

1. FOUNDATION + DOMAIN SEPARATION:
   - Every feature group → 1 foundation FR + N domain FRs
   - Foundation: DB tables, GraphQL types, empty stubs. NO business logic.
   - Domain: 1 user-facing function each. Business logic, validation, queries.
   - Read-only features: domain-only with `foundation: none`

2. SIZING (must pass ALL):
   - Resolvers ≤ 3
   - Service methods ≤ 5
   - DB tables ≤ 2 (foundation only)
   - Changed files ≤ 10
   - If exceeded: split (by query vs mutation, by CRUD, by core vs extension)

3. DEPENDENCIES:
   - Foundation has no intra-group dependencies (always root)
   - All domain FRs depend on their group's foundation
   - Cross-group: if domain A uses group B's entities, add dependency
   - Only direct dependencies (no transitive)
   - No circular dependencies (extract shared foundation to break)

4. EXECUTION WAVES:
   - Wave = max(wave of dependencies) + 1
   - No dependencies = Wave 1
   - All items within a wave are parallelizable

5. DOMAIN MAPPING REFERENCE:
   - Technical Details MUST use DB Table, ORM Model, GraphQL Type from the domain mapping
   - If term not in mapping: use spec wording with `(unmapped)` annotation
   - Each FR includes a Domain Terms section

For each FR, generate:
- ID: {nn}-{short-name}-{type} (e.g., 01-app-status-foundation)
- Type: foundation | domain
- Group: feature group name
- Dependencies
- Execution wave
- Scope + Out of Scope
- Technical Details (using domain mapping entities)
- Domain Terms table (subset of mapping)
- Acceptance Criteria

Return your complete decomposition in this structure (do NOT write to files):

## Summary
- Total FRs: {n} ({f} foundation + {d} domain)
- Execution Waves: {n}

## Feature-Request List
### {fr-id}: {title}
- Type: foundation | domain
- Group: {group}
- Wave: {n}
- Depends On: [{deps}]
- Scope: {scope description}
- Out of Scope: {out of scope}
- Technical Details: {details using code entities}
- Domain Terms:
  | Spec Term | DB Table | ORM Model | GraphQL Type |
- Acceptance Criteria:
  - [ ] {criterion}

## Dependency Graph
Wave 1: ...
Wave 2: ...

## Sizing Validation
| FR ID | Resolvers | Services | Tables | Files | Status |
```

Use Task tool with:
- `subagent_type`: `oh-my-claudecode:architect`
- `model`: opus

After architect returns, write `_requirements.md`:

```markdown
# Requirements: {spec-id}

## Summary
- **Source Spec**: {spec-id}
- **Feature Groups**: {n}
- **Total Feature-Requests**: {n} ({f} foundation + {d} domain)
- **Execution Waves**: {n}

## Feature-Request Index
| # | ID | Type | Group | Wave | Depends On | Scope Summary |
|---|-----|------|-------|------|------------|---------------|
{from architect output}

## Dependency Graph
{from architect output}

## Sizing Validation
{from architect output}
```

Also write individual `_feature-request.md` files for each FR:
```
{$PROJECT_ROOT}/.claude/workspace/specs/{spec-id}/features/{fr-id}/
└── _feature-request.md
```

Each `_feature-request.md` follows the standard template:
```markdown
# {Title}

## Metadata
- **ID**: {fr-id}
- **Type**: foundation | domain
- **Group**: {group}
- **Depends On**: [{deps}]

## Execution Context
- **Source Spec**: {spec-id}
- **Execution Wave**: {wave}
- **Work Scope**: {description}

## Scope
{scope}

## Out of Scope
{out of scope}

## Technical Details
{details using code entities from domain mapping}

## Domain Terms
| Spec Term | DB Table | ORM Model | GraphQL Type |
|-----------|----------|-----------|--------------|
{terms}

## Acceptance Criteria
- [ ] {criteria}
```

### H2: Developer Reviews Scope, Sizing, Waves

Display to user:

```
## Phase 2 Complete — Decomposition Ready

### Feature-Requests: {n} total ({f} foundation + {d} domain)
### Execution Waves: {n}

| # | ID | Type | Group | Wave | Deps |
|---|-----|------|-------|------|------|
{index table}

### Wave Visualization
Wave 1 (parallel): {list}
Wave 2 (parallel): {list}
...

### Files
- `_requirements.md` — Full index + dependency graph + sizing
- `features/{fr-id}/_feature-request.md` — Per-FR definition (× {n})
```

Use AskUserQuestion:
- Question: "Review the feature-request decomposition. Approve, defer, or modify FRs?"
- Options:
  - "All approved — generate design plans"
  - "I've made edits to some FRs — re-read and proceed"
  - "Some FRs should be deferred — let me specify"
  - "Stop — I need more time to review"

If "I've made edits": re-read all `_feature-request.md` files from disk.
If "Some FRs should be deferred":
  - Use AskUserQuestion: "Which FR IDs should be deferred? (comma-separated, e.g., `03-notification-push, 05-analytics-export`)"
  - Mark deferred FRs, recalculate waves (remove deferred nodes from DAG, compress wave numbers)
  - Update `_requirements.md` with deferred status
If "Stop": halt execution.

### Phase 3: Batch Design-Plan Generation (parallel, per-wave)

For each approved (non-deferred) FR, generate a design-plan.md. Process in wave order, with FRs within each wave running in parallel.

```
For each wave (Wave 1 first, then Wave 2, ...):
  For each FR in wave (up to 5 concurrent via run_in_background):

    Step A — explore (haiku): Find FR-specific code
    Delegate to `oh-my-claudecode:explore` (haiku):
    ```
    Explore the codebase for code related to this specific feature-request.

    Feature-Request:
    {content of _feature-request.md}

    {if $CODING_GUIDE is not empty:}
    Project Coding Guide:
    {$CODING_GUIDE}
    {end if}

    Tasks:
    1. Find existing modules, services, resolvers related to this FR's entities
    2. Find existing DB models/tables referenced in Technical Details
    3. Find existing GraphQL types referenced
    4. Identify integration points with existing code
    5. Note patterns in related modules

    Save findings to: {$PROJECT_ROOT}/.claude/workspace/specs/{spec-id}/features/{fr-id}/_explore-report.md

    Return a 3-line summary.
    ```

    Wait for all explore tasks in this wave to complete.

    Step B — analyst (opus): Generate design analysis
    Delegate to `oh-my-claudecode:analyst` (opus):
    ```
    Analyze this feature-request and generate a design plan.

    Feature-Request:
    {content of _feature-request.md}

    Codebase Exploration:
    {content of _explore-report.md}

    {if $CODING_GUIDE is not empty:}
    Project Coding Guide:
    {$CODING_GUIDE}
    {end if}

    Analysis Tasks:
    1. Break down into concrete implementation requirements
    2. Identify DB schema changes (tables, columns, relations, indexes)
    3. Identify API schema changes (queries, mutations, types, inputs)
    4. Identify service logic changes (business rules, validations, side effects)
    5. Flag risks: breaking changes, migration risks, performance, security
    6. Define acceptance criteria
    7. Estimate file scope

    Return your complete analysis directly (do NOT save to file).
    ```

    Wait for all analyst tasks in this wave to complete.

    Step C — Main Context: Write design-plan.md for each FR
    Using explore report + analyst results, write:
    `{$PROJECT_ROOT}/.claude/workspace/specs/{spec-id}/features/{fr-id}/design-plan.md`

    The design-plan MUST follow the standard 5-section format:

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
    - [safety considerations]

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
    1. [rule]
    ### Side Effects
    | Trigger | Effect | Mitigation |

    ---

    ## 4. Test Plan
    ### Unit Tests
    | Test Case | Target | Expected |
    ### Edge Cases
    1. [edge case]

    ---

    ## 5. Verification Strategy
    ### Build Verification
    - [ ] `npm run type-check` passes
    ### Test Verification
    - [ ] All new tests pass
    ### Manual Verification
    - [ ] [specific check]

    ---

    ## Implementation Files (Estimated)
    | # | File | Action | Description |

    ## Risks & Open Questions
    1. [risk or question]
    ```

  Proceed to next wave.
```

### Phase 4: Generate execution-plan.md

After all design-plans are generated, write the execution plan:

```markdown
# Execution Plan: {spec-id}

## Overview
- **Spec**: {spec-title}
- **Total FRs**: {n} ({approved} approved, {deferred} deferred)
- **Execution Waves**: {n}
- **Generated**: {current date}

## Wave Execution Order

### Wave 1 (parallel — start all simultaneously)
| FR ID | Type | Design Plan | Command |
|-------|------|-------------|---------|
| {fr-id} | foundation | `features/{fr-id}/design-plan.md` | `/feature:execute {fr-id}` |

### Wave 2 (parallel — after Wave 1 completes)
| FR ID | Type | Depends On | Design Plan | Command |
|-------|------|------------|-------------|---------|
| {fr-id} | domain | {deps} | `features/{fr-id}/design-plan.md` | `/feature:execute {fr-id}` |

### Wave N
...

{if deferred FRs exist:}
## Deferred FRs (not included in execution)
| FR ID | Reason |
|-------|--------|
| {fr-id} | {reason} |
{end if}

## Workspace Reference
- Spec analysis: `_spec-analysis.md`
- Domain mapping: `_domain-mapping.md`
- Requirements: `_requirements.md`
- Design plans: `features/{fr-id}/design-plan.md`

## Next Steps
Execute FRs in wave order:
1. Run all Wave 1 commands (can be parallel)
2. After Wave 1 PRs are merged, run Wave 2 commands
3. Continue through all waves
```

### Final Output

Display to user:

```
## spec:plan Complete!

### Spec: {spec-title}
### Feature-Requests: {n} approved, {n} deferred
### Design Plans: {n} generated
### Execution Waves: {n}

### Workspace
{$PROJECT_ROOT}/.claude/workspace/specs/{spec-id}/
├── _spec-analysis.md
├── _codebase-scan.md
├── _domain-mapping.md
├── _requirements.md
├── execution-plan.md
└── features/
    ├── {fr-id-1}/
    │   ├── _feature-request.md
    │   ├── _explore-report.md
    │   └── design-plan.md
    └── {fr-id-2}/
        └── ...

### Domain Dictionary
Updated with {n} new terms.

### Next Step
Execute features in wave order:
  /feature:execute {first-wave-fr-id}

Or see full execution plan:
  execution-plan.md
```

## Output

- `_spec-analysis.md` — PDF analysis with checklist, groups, domain terms
- `_codebase-scan.md` — Entity inventory from codebase
- `_domain-mapping.md` — Auto-proposed domain term → code entity mapping
- `_requirements.md` — FR index + dependency graph + sizing validation
- `execution-plan.md` — Wave-ordered execution guide
- `features/{fr-id}/_feature-request.md` — Per-FR definition (× N)
- `features/{fr-id}/_explore-report.md` — Per-FR codebase exploration (× N)
- `features/{fr-id}/design-plan.md` — Per-FR design plan (× N)
- Updated `domain-dictionary.md` — New confirmed terms appended

## Error Handling

| Error | Action |
|-------|--------|
| Arguments missing | Ask via AskUserQuestion |
| PDF not found | Stop with path guidance |
| PDF unreadable | Stop: "PDF could not be read. Check format and permissions." |
| Project root not detected | Ask user for path |
| coding-guide missing | Warn, proceed in generic mode |
| Workspace already exists | Ask to overwrite or stop (unless `--force`) |
| analyst agent failure | Report error, stop |
| explore agent failure | Report error, stop |
| architect agent failure | Report error, stop |
| Codebase scan empty | Warn: "No entities found in codebase scan. All domain terms will be [NEW]." |
| Domain dictionary not found | Warn: "No domain dictionary. All terms will be PROPOSED or [NEW]." Create empty dictionary after H1. |
| Design-plan generation failure for specific FR | Report which FR failed, continue with remaining FRs |
| All FRs deferred | Stop: "All FRs deferred. Nothing to generate." |

---
Arguments: $ARGUMENTS
