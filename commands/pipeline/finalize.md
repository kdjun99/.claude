---
description: 'Finalize pipeline design by incorporating discussion feedback'
allowed-tools: ['Read', 'Grep', 'Glob', 'Write', 'AskUserQuestion', 'Task']
argument-hint: [pipeline-id]
---

# Pipeline Finalize

Incorporate discussion feedback into the pipeline draft to produce a confirmed design.

## Pipeline Context

```
/draft (user:pipeline) -> > /finalize (user:pipeline) -> /build (user:pipeline) -> /detail (user:pipeline) -> /detail-apply (user:pipeline)
```

## Input

$ARGUMENTS: Pipeline ID (kebab-case)

If no pipeline-id provided, list available pipelines in `~/.claude/workspace/pipelines/` and ask user to select.

## Gate

Verify ALL of the following before proceeding:

1. **pipeline_draft.md exists**
   - Path: `~/.claude/workspace/pipelines/{pipeline-id}/pipeline_draft.md`
   - If missing: "Pipeline draft not found. Run `/draft (user:pipeline)` first."

2. **pipeline_draft.md Todo is complete**
   - Read `pipeline_draft.md`, find `## Todo` section
   - Parse for `- [ ]` patterns (incomplete items)
   - If incomplete items found:
     - Display: "The following items in pipeline_draft.md are incomplete:"
     - List each incomplete item
     - "Complete these items and run again."
     - STOP execution.

3. **pipeline_discussion.md exists and has content**
   - Path: `~/.claude/workspace/pipelines/{pipeline-id}/pipeline_discussion.md`
   - If missing: "Discussion file not found. Write your feedback to pipeline_discussion.md first."
   - If file exists but empty or only whitespace: "Discussion file is empty. Please add your review notes."
   - STOP execution if either condition.

## Execution

### Phase 1: Load and Analyze

1. Read `pipeline_draft.md` (the design)
2. Read `pipeline_discussion.md` (the feedback)
3. Categorize each discussion point as:
   - **Approved**: Draft item accepted as-is
   - **Modified**: Draft item changed per feedback
   - **Deferred**: Item postponed for later
   - **Rejected**: Item removed

### Phase 2: Merge and Confirm

1. Apply modifications from discussion to the draft design
2. Determine implementation order based on:
   - Component dependencies (Skills first, then Agents, then Commands)
   - Intra-layer dependencies from the design
3. Generate topologically sorted implementation order

### Phase 3: Generate pipeline_final.md

```markdown
# Pipeline Final: {pipeline-id}

## Context
- Pipeline ID: {pipeline-id}
- Phase: finalize
- Created: {date}
- Target Project: {from draft}

## Prerequisites
- [x] pipeline_draft.md exists and Todo complete
- [x] pipeline_discussion.md exists with content

## Content

### Decision Summary
| Status | Count |
|--------|-------|
| Approved | {n} |
| Modified | {n} |
| Deferred | {n} |
| Rejected | {n} |

### Confirmed Components
| # | Name | Type | Role | Status | Notes |
|---|------|------|------|--------|-------|
(Status: approved/modified/deferred/rejected)

### Implementation Order
| Order | Component | Type | Depends On |
|-------|-----------|------|------------|

### Changes from Draft
| Component | Change | Reason |
|-----------|--------|--------|

### Deferred Items
| Item | Reason | Revisit When |
|------|--------|-------------|

### Pipeline Flow (confirmed)
(Updated flow diagram reflecting changes)

### Workspace Structure (confirmed)
(Updated workspace structure)

## Todo
- [x] Discussion feedback incorporated
- [x] Components confirmed
- [x] Implementation order determined
- [ ] Human final approval of confirmed design

## Next
- After approval: /build (user:pipeline) {pipeline-id}
```

Save to `~/.claude/workspace/pipelines/{pipeline-id}/pipeline_final.md`

### Phase 4: Summary

Display to user:
- Decision summary (approved/modified/deferred/rejected counts)
- Key changes from draft
- Confirmed implementation order
- Next step: `/build (user:pipeline) {pipeline-id}`

## Output

`~/.claude/workspace/pipelines/{pipeline-id}/pipeline_final.md`

## Notes

- Principle: "Items not mentioned in discussion = approved as-is"
- Deferred items are tracked for future revisit
- The confirmed design is the source of truth for /build

---
Arguments: $ARGUMENTS
