---
description: 'Analyze a process and generate pipeline design draft'
allowed-tools: ['Read', 'Grep', 'Glob', 'Write', 'Bash', 'AskUserQuestion', 'Task']
argument-hint: [process description]
---

# Pipeline Draft

Analyze a target process and generate a pipeline design draft for team review.

## Pipeline Context

```
> /draft (user:pipeline) -> /finalize (user:pipeline) -> /build (user:pipeline) -> /detail (user:pipeline) -> /detail-apply (user:pipeline)
```

## Input

$ARGUMENTS: Process description. Should include:
- What process to automate
- Target project path (optional, will ask if not provided)
- Any reference documents or existing pipeline patterns to consider

## Gate

None — this is the first step in the pipeline.

## Execution

### Phase 0: Input Parsing

1. Extract process description from $ARGUMENTS
2. If target project path not provided, use AskUserQuestion to ask:
   - "What is the target project path?" (provide current working directory as default)
3. Determine Pipeline ID:
   - Generate a kebab-case ID from the process description
   - Use AskUserQuestion to confirm: "Pipeline ID will be: {id}. Proceed?"
4. Create workspace: `~/.claude/workspace/pipelines/{pipeline-id}/`
5. Create details subdirectory: `~/.claude/workspace/pipelines/{pipeline-id}/details/`

### Phase 1: Process Analysis (pipeline-analyzer agent)

Delegate to `pipeline-analyzer` agent:
```
Analyze the following process for pipeline automation.

Process description:
{$ARGUMENTS}

Target project: {target-project-path}

Save analysis to: ~/.claude/workspace/pipelines/{pipeline-id}/_process-analysis.md
```

Wait for completion. Read `_process-analysis.md` to verify it was created.

### Phase 2: Component Design (pipeline-architect agent)

Delegate to `pipeline-architect` agent:
```
Design pipeline components based on the process analysis.

Process analysis: ~/.claude/workspace/pipelines/{pipeline-id}/_process-analysis.md
Target project: {target-project-path}

Save design to: ~/.claude/workspace/pipelines/{pipeline-id}/_component-design.md
```

Wait for completion. Read `_component-design.md` to verify it was created.

### Phase 3: Consolidate

Read both artifacts and merge into `pipeline_draft.md`:

1. Read `_process-analysis.md`
2. Read `_component-design.md`
3. Generate `pipeline_draft.md` following the artifact structure standard:

```markdown
# Pipeline Draft: {pipeline-id}

## Context
- Pipeline ID: {pipeline-id}
- Phase: draft
- Created: {date}
- Target Project: {target-project-path}

## Prerequisites
(None — first step)

## Content

### Process Overview
(From _process-analysis.md)

### Process Analysis
(Step decomposition table from _process-analysis.md)

### Human Checkpoints
(From _process-analysis.md)

### Component Design
(Component inventory from _component-design.md)

### Pipeline Flow
(From _component-design.md)

### Artifact Flow
(From _component-design.md)

### Workspace Structure
(From _component-design.md)

### Implementation Order
(From _component-design.md)

### Design Principles Verification
(From _component-design.md)

## Todo
- [x] Process analyzed and decomposed
- [x] Components designed
- [x] Artifact flow defined
- [x] Design principles verified
- [ ] Human review of overall pipeline design
- [ ] {any questions from analysis/design agents}

## Next
- Write discussion notes in: ~/.claude/workspace/pipelines/{pipeline-id}/pipeline_discussion.md
- Then run: /finalize (user:pipeline) {pipeline-id}
```

4. Save to `~/.claude/workspace/pipelines/{pipeline-id}/pipeline_draft.md`

### Phase 4: Summary

Display to user:
- Pipeline ID
- Number of components designed (Commands / Agents / Skills)
- Key design decisions
- Any open questions requiring human input
- Next step: write `pipeline_discussion.md` and run `/finalize`

## Output

`~/.claude/workspace/pipelines/{pipeline-id}/pipeline_draft.md`

## Notes

- This draft is for discussion. Do not implement anything yet.
- After review, write discussion notes and use `/finalize` to confirm the design.
- Each pipeline step can be run in independent sessions — artifacts persist in workspace.

---
Arguments: $ARGUMENTS
