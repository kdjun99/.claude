---
description: "Apply team discussion feedback and finalize feature-requests. Third step of the spec-decomposition pipeline."
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# /spec-confirm {spec-id}

Apply team discussion feedback to feature-requests, copy approved items to confirmed/, generate execution plan.

## Input

- `spec-id`: Identifier for this spec (must match a previous /spec-decompose run)

Parse from: $ARGUMENTS

## Gate

1. **Argument present**: `spec-id` must be provided
   - If missing: "Usage: /spec-confirm {spec-id}"
2. **Feature-requests exist**: Verify `~/.claude/workspace/specs/{spec-id}/feature-requests/` directory exists and contains .md files
   - If missing: "Feature-requests not found. Run `/spec-decompose {spec-id}` first."
3. **Discussion file exists**: Verify `~/.claude/workspace/specs/{spec-id}/spec-discussion.md` exists and is not empty
   - If missing: Display the discussion template and guide:
     ```
     Team discussion file not found.

     Create: ~/.claude/workspace/specs/{spec-id}/spec-discussion.md

     Use this format:

     # Spec Discussion: {spec-id}

     ## Decisions

     | FR ID | Decision | Notes |
     |-------|----------|-------|
     | 01-app-status-foundation | approved | — |
     | 02-app-status-update | modified | Change validation: allow admin override |
     | 03-app-status-notification | deferred | Not needed for MVP |

     ## Modifications

     ### 02-app-status-update
     - Add admin override bypass for status validation
     - Update acceptance criteria to include admin flow

     ## General Notes
     - {any overall feedback or context}
     ```
     STOP execution.
4. **No duplicate run**: Check if `~/.claude/workspace/specs/{spec-id}/confirmed/execution-plan.md` already exists
   - If exists: "Confirmation already exists for '{spec-id}'. To re-confirm, delete the `confirmed/` directory first."

## Execution

### Phase 1: Parse Discussion

1. Read `~/.claude/workspace/specs/{spec-id}/spec-discussion.md`
2. Parse the Decisions table — for each row extract:
   - `fr_id`: Feature-request ID
   - `decision`: approved | modified | deferred
   - `notes`: Any inline notes
3. For each `modified` decision, read the Modifications section to extract specific changes
4. Validate all FR IDs in the discussion match actual files in `feature-requests/`:
   - List any FR IDs in discussion that don't match a file → warn user
   - List any feature-request files not mentioned in discussion → warn user (default: approved)
5. Display summary:
   ```
   Parsing discussion...
   - Approved: {n} feature-requests
   - Modified: {n} feature-requests
   - Deferred: {n} feature-requests
   - Unmentioned (default approved): {n} feature-requests
   ```

### Phase 2: Apply Decisions

Create the confirmed directory structure:
```
mkdir -p ~/.claude/workspace/specs/{spec-id}/confirmed
```

For each feature-request, based on its decision:

**Approved:**
1. Read the original feature-request file from `feature-requests/{repo}/{fr-id}.md`
2. Copy as-is to `confirmed/{repo}/{fr-id}.md`

**Modified:**
1. Read the original feature-request file
2. Apply the modifications from the discussion:
   - Update Scope section if scope changed
   - Update Technical Details if implementation approach changed
   - Update Acceptance Criteria if criteria changed
   - Add/modify any other sections per feedback
3. Add a `## Modifications` section at the end:
   ```
   ## Modifications (from team discussion)
   - {change description 1}
   - {change description 2}
   ```
4. Write modified version to `confirmed/{repo}/{fr-id}.md`

**Deferred:**
1. Do NOT copy to confirmed/
2. Record in deferred list with reason (for execution-plan.md)

**Unmentioned (not in discussion):**
1. Treat as approved — copy as-is to confirmed/

### Phase 3: Recalculate Waves

After applying decisions, the dependency graph may have changed (deferred items remove nodes):

1. Read all confirmed feature-requests
2. Check each `Depends On` reference:
   - If a dependency was deferred → warn: "FR '{id}' depends on deferred '{dep}'. This dependency is broken."
   - If a dependency was modified → no issue (still exists)
3. Recalculate execution waves using only confirmed FRs:
   - Remove deferred FRs from graph
   - Recompute: wave = max(wave of remaining dependencies) + 1
   - Re-assign wave numbers (may compress — e.g., if all Wave 2 items were deferred, Wave 3 becomes Wave 2)
4. Update `execution_wave` in each confirmed FR's Execution Context section

### Phase 4: Generate Execution Plan

Write `~/.claude/workspace/specs/{spec-id}/confirmed/execution-plan.md`:

```markdown
# Execution Plan: {spec-id}

## Summary
- **Source Spec**: {spec-id}
- **Confirmed Feature-Requests**: {n} ({f} foundation + {d} domain)
- **Deferred Feature-Requests**: {n}
- **Execution Waves**: {w}
- **Impacted Repos**: {list}

## Decision Summary

| Decision | Count |
|----------|-------|
| Approved | {n} |
| Modified | {n} |
| Deferred | {n} |
| Total | {n} |

## Confirmed Feature-Request Index

| # | ID | Repo | Type | Wave | Decision | Depends On |
|---|-----|------|------|------|----------|------------|
| 1 | {id} | {repo} | {type} | {wave} | approved | {deps} |
| 2 | {id} | {repo} | {type} | {wave} | modified | {deps} |
| ... | | | | | | |

## Execution Waves (confirmed only)

### Wave 1
| # | FR ID | Repo | Type | Description |
|---|-------|------|------|-------------|
| 1 | {id} | {repo} | foundation | {1-line scope} |
| ... | | | | |

### Wave 2
| # | FR ID | Repo | Type | Description |
|---|-------|------|------|-------------|
| ... | | | | |

### Wave N
...

## Deferred Items

| FR ID | Reason | Impact |
|-------|--------|--------|
| {id} | {reason from discussion} | {which confirmed FRs were affected, if any} |

## Broken Dependencies

| Confirmed FR | Deferred Dependency | Action Needed |
|-------------|---------------------|---------------|
| {id} | {deferred-id} | {manual resolution needed} |

(Empty if no broken dependencies)

## Phase 1 Integration

Execute confirmed feature-requests in wave order using the Phase 1 pipeline:

### Wave 1 (start all in parallel):
/feature-start {first-fr-id}
/feature-start {second-fr-id}
...

### Wave 2 (after Wave 1 completes):
/feature-start {fr-id}
...

### Recommended Workflow:
1. Start all Wave 1 FRs (foundations can run in parallel)
2. For each FR: /feature-start → /feature-draft → review → /feature-submit
3. After all Wave 1 FRs are merged, start Wave 2
4. Continue until all waves complete

## Todo
- [x] Discussion parsed and decisions applied
- [x] Modified FRs updated with feedback
- [x] Waves recalculated for confirmed FRs only
- [x] Execution plan generated
```

### Phase 5: Present Results (H4)

Display to user:

```
## Confirmation Complete: {spec-id}

### Decision Summary
| Decision | Count |
|----------|-------|
| Approved | {n} |
| Modified | {n} |
| Deferred | {n} |

### Confirmed Waves
Wave 1: {FR IDs} (parallel)
Wave 2: {FR IDs} (parallel)
...

### Artifacts
- Confirmed FRs: ~/.claude/workspace/specs/{spec-id}/confirmed/
- Execution plan: ~/.claude/workspace/specs/{spec-id}/confirmed/execution-plan.md
```

If broken dependencies exist:
```
⚠ {n} confirmed feature-requests have broken dependencies due to deferred items.
Review the "Broken Dependencies" section in execution-plan.md.
```

```
## Quality Gate (H4)

Before starting Phase 1, verify:
1. All confirmed FRs are correctly modified (if any were modified)
2. No broken dependencies (or resolution plan documented)
3. Wave sequencing makes sense with deferred items removed
4. Team is aligned on execution order

## Start Phase 1

Begin with Wave 1:
/feature-start {first-wave1-fr-id}
```

## Output

- `~/.claude/workspace/specs/{spec-id}/confirmed/{repo}/` — Finalized feature-request .md files
- `~/.claude/workspace/specs/{spec-id}/confirmed/execution-plan.md` — Full execution plan with waves and Phase 1 guidance
- Summary displayed to user with quality gate

## Error Handling

| Error | Action |
|-------|--------|
| Feature-requests not found | Stop: "Run `/spec-decompose {spec-id}` first." |
| Discussion file empty | Stop: display template, guide user |
| Discussion parse error | Warn: list unparseable rows, continue with parseable ones |
| FR ID mismatch | Warn: list mismatched IDs, continue with matched ones |
| Broken dependencies | Warn in results, document in execution-plan.md |

## Notes

No agent delegation — this command executes directly (mechanical file operations + template generation).
All operations are file reads, writes, and copies. The logic is straightforward:
parse discussion → apply decisions → recalculate waves → generate execution plan.
