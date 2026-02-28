---
description: "Analyze a feature request and produce a design plan. Uses OMC explore + analyst agents to generate a 5-section design-plan.md for developer review."
allowed-tools: ["Task", "Read", "Write", "Glob", "Grep", "Bash", "AskUserQuestion"]
argument-hint: "req-id"
---

# /feature:plan {req-id}

Analyze a feature request and produce a structured design plan for developer review.

**Usage:** `/feature:plan <req-id>`

The `req-id` must be kebab-case (e.g., `user-profile-edit`, `simple-apply-v1`).

## Arguments

Parse from `$ARGUMENTS`:
- `req-id`: First argument — kebab-case identifier for the feature request

## Gate Validation

Run all gates sequentially. Stop at the first failure.

### Gate 1: req-id present and valid format

- If `$ARGUMENTS` is empty: use AskUserQuestion to ask "What is the feature request ID? (kebab-case, e.g., `user-profile-edit`)"
- If req-id is not kebab-case (lowercase letters, numbers, hyphens only): "Error: req-id must be kebab-case. Got: `{req-id}`"

### Gate 2: coding-guide check

1. Detect the project root: find the nearest directory (from cwd upward) containing `.claude/` with skill files
2. Check if `{project-root}/.claude/skills/coding-guide/skill.md` exists
3. If exists:
   - Read the coding-guide manifest
   - Read all skill files listed under "Skills to Load" section
   - Store the combined content as `$CODING_GUIDE` for later use
4. If missing:
   - Use AskUserQuestion: "No coding-guide found at `.claude/skills/coding-guide/skill.md`. Proceed in generic mode (no project-specific conventions)?"
   - Options: "Proceed in generic mode" / "Stop — I'll create coding-guide first"
   - If stop: halt execution
   - If proceed: set `$CODING_GUIDE` to empty string, display warning: "Running in generic mode — design plan will not include project-specific conventions."

### Gate 3: duplicate check

1. Check if `{project-root}/.claude/workspace/{req-id}/design-plan.md` already exists
2. If exists:
   - Use AskUserQuestion: "design-plan.md already exists for `{req-id}`. What would you like to do?"
   - Options: "Overwrite and re-run" / "Stop — use existing plan"
   - If stop: halt execution
   - If overwrite: continue (existing file will be replaced)

### Gate 4: feature request input

1. Check if `{project-root}/.claude/workspace/{req-id}/_feature-request.md` already exists
   - If exists: read and use as feature request input
2. If not exists:
   - Use AskUserQuestion: "How would you like to provide the feature request?"
   - Options: "I'll describe it now" / "Read from a file"
   - If describe: collect the description from user, save to `_feature-request.md`
   - If file: ask for file path, read the file, save content to `_feature-request.md`

## Execution

### Phase 0: Setup Workspace

1. Create workspace directory:
   ```
   mkdir -p {project-root}/.claude/workspace/{req-id}
   ```
2. Save feature request to `{project-root}/.claude/workspace/{req-id}/_feature-request.md` (if not already saved in Gate 4)
3. Display to user:
   ```
   Starting feature plan...
   - Req ID: {req-id}
   - Project: {project-name}
   - Coding Guide: {found / generic mode}
   - Workspace: {project-root}/.claude/workspace/{req-id}/
   ```

### Phase 1: Codebase Exploration (OMC explore agent)

Delegate to `oh-my-claudecode:explore` agent (haiku):

```
Explore the codebase to understand the current architecture relevant to this feature request.

Feature Request:
{content of _feature-request.md}

{if $CODING_GUIDE is not empty:}
Project Coding Guide:
{$CODING_GUIDE}
{end if}

Tasks:
1. Identify existing modules, services, resolvers, and DTOs related to this feature
2. Find existing DB models/tables that will be affected or referenced
3. Find existing GraphQL types/queries/mutations in the related domain
4. Identify integration points — what existing code will this feature interact with?
5. Note any existing patterns or conventions observed in related modules

Output Requirements:
- Save your findings to: {project-root}/.claude/workspace/{req-id}/_explore-report.md
- Structure the report with these sections:
  ## Related Modules
  ## Related DB Models
  ## Related GraphQL Schema
  ## Integration Points
  ## Observed Patterns
- Return a 3-line summary of what you found
```

Use Task tool with:
- `subagent_type`: `oh-my-claudecode:explore`
- `model`: haiku
- Wait for completion (do NOT run in background)

After completion, verify `_explore-report.md` was created. If missing, report error and stop.

### Phase 2: Requirements Analysis (OMC analyst agent)

Delegate to `oh-my-claudecode:analyst` agent (opus):

```
Analyze this feature request against the codebase exploration results. Identify requirements, constraints, risks, and acceptance criteria.

Feature Request:
{content of _feature-request.md}

Codebase Exploration:
{content of _explore-report.md}

{if $CODING_GUIDE is not empty:}
Project Coding Guide:
{$CODING_GUIDE}
{end if}

Analysis Tasks:
1. Break down the feature into concrete implementation requirements
2. Identify DB schema changes needed (new tables, columns, relations, indexes)
3. Identify API schema changes needed (new queries, mutations, types, inputs)
4. Identify service logic changes (business rules, validations, side effects)
5. Flag risks: breaking changes, migration risks, performance concerns, security implications
6. Define acceptance criteria for each requirement
7. Estimate implementation scope: list all files that will need to be created or modified

Return your complete analysis. Do NOT save to a file — return directly.
```

Use Task tool with:
- `subagent_type`: `oh-my-claudecode:analyst`
- `model`: opus
- Wait for completion

### Phase 3: Generate design-plan.md

Using the explore report and analyst results, write `{project-root}/.claude/workspace/{req-id}/design-plan.md`.

The design plan MUST follow this exact format with all 5 sections:

```markdown
# Design Plan: {req-id}

## Summary
[1-2 paragraph overview of the feature and its implementation approach]

---

## 1. DB Schema Changes

### New Tables
| Table | Purpose |
|-------|---------|
| ... | ... |

### Column Changes
| Table | Column | Type | Constraint | Note |
|-------|--------|------|------------|------|
| ... | ... | ... | ... | ... |

### Migration Notes
- [Migration safety considerations]
- [Data backfill requirements]
- [Index additions]

---

## 2. API Schema Changes

### New Queries
| Query | Return Type | Auth | Description |
|-------|------------|------|-------------|
| ... | ... | ... | ... |

### New Mutations
| Mutation | Input | Return Type | Auth | Description |
|----------|-------|------------|------|-------------|
| ... | ... | ... | ... | ... |

### Type Changes
| Type | Change | Fields |
|------|--------|--------|
| ... | ... | ... |

---

## 3. Service Logic Changes

### New Services/Methods
| Service | Method | Purpose |
|---------|--------|---------|
| ... | ... | ... |

### Business Rules
1. [Rule 1]
2. [Rule 2]

### Side Effects
| Trigger | Effect | Mitigation |
|---------|--------|------------|
| ... | ... | ... |

---

## 4. Test Plan

### Unit Tests
| Test Case | Target | Expected |
|-----------|--------|----------|
| ... | ... | ... |

### Edge Cases
1. [Edge case 1]
2. [Edge case 2]

---

## 5. Verification Strategy

### Build Verification
- [ ] `npm run type-check` passes
- [ ] `npm run build:gqlgen` succeeds (if DTO changes)
- [ ] `npm run lint` passes

### Test Verification
- [ ] All new tests pass: `npm run test -- --testPathPattern="..."`
- [ ] No existing tests broken

### Manual Verification
- [ ] [Specific manual check 1]
- [ ] [Specific manual check 2]

---

## Implementation Files (Estimated)

| # | File | Action | Description |
|---|------|--------|-------------|
| 1 | ... | Create/Modify | ... |

## Risks & Open Questions
1. [Risk or question 1]
2. [Risk or question 2]
```

### Phase 4: Human Checkpoint

Display to user:

```
## Design Plan Ready

design-plan.md has been generated at:
  {project-root}/.claude/workspace/{req-id}/design-plan.md

### Quick Summary
[2-3 line summary from the plan]

### Sections
1. DB Schema Changes — {n} tables affected
2. API Schema Changes — {n} queries, {n} mutations
3. Service Logic — {n} new methods
4. Test Plan — {n} test cases
5. Verification Strategy — build + test + manual checks

### Estimated Files: {n}

Please review the design plan. When approved, create a marker file:
  {project-root}/.claude/workspace/{req-id}/design-plan-approved.md

Then run:
  /feature:execute {req-id}
```

## Output

- `{project-root}/.claude/workspace/{req-id}/_feature-request.md` — Feature request input
- `{project-root}/.claude/workspace/{req-id}/_explore-report.md` — Codebase exploration results
- `{project-root}/.claude/workspace/{req-id}/design-plan.md` — Design plan (human-facing, only checkpoint)

## Error Handling

| Error | Action |
|-------|--------|
| req-id not kebab-case | Stop with format guidance |
| coding-guide missing | Warn, ask to proceed in generic mode |
| design-plan.md already exists | Ask to overwrite or stop |
| Feature request input missing | Ask user to provide |
| explore agent failure | Report error: "Codebase exploration failed. Check agent output." Stop. |
| explore report not created | Report error and stop |
| analyst agent failure | Report error: "Requirements analysis failed. Check agent output." Stop. |
| Workspace directory not writable | Report error and stop |

---
Arguments: $ARGUMENTS
