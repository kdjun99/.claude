---
description: 'Generate design draft for a single pipeline component'
allowed-tools: ['Read', 'Grep', 'Glob', 'Write', 'AskUserQuestion', 'Task']
argument-hint: [pipeline-id] --component [component-name]
---

# Pipeline Detail

Generate a design draft for a single pipeline component for human review.

## Pipeline Context

```
/draft (user:pipeline) -> /finalize (user:pipeline) -> /build (user:pipeline) -> > /detail (user:pipeline) -> /detail-apply (user:pipeline)
```

## Input

$ARGUMENTS: `{pipeline-id} --component {component-name}`

Parse:
- `pipeline-id`: First argument
- `component-name`: Value after `--component`

If `--component` not provided, read `build_report.md` and show the next component to detail (first scaffolded component in dependency order), then ask user to confirm.

## Gate

Verify ALL of the following before proceeding:

1. **build_report.md exists**
   - Path: `~/.claude/workspace/pipelines/{pipeline-id}/build_report.md`
   - If missing: "Build report not found. Run `/build (user:pipeline)` first."

2. **Component is scaffolded**
   - Read `build_report.md`, find the component in Component Status table
   - If component not found: "Component '{name}' not found in build report."
   - If status is already "detailed": "Component '{name}' is already detailed."

3. **Predecessor details are complete**
   - Read `build_report.md` to find this component's dependencies (Depends On column)
   - For each dependency:
     - Check if `details/{dep}.md` exists (the final version, NOT `_draft.md`)
     - If exists, read it and verify `## Todo` has no `- [ ]` items
   - If any predecessor is incomplete:
     - Display: "Predecessor '{dep}' is not yet detailed. Complete it first:"
     - Show which predecessors are missing/incomplete
     - "Run `/detail (user:pipeline) {pipeline-id} --component {dep}` first."
     - STOP execution.

## Execution

### Phase 1: Context Loading

1. Read `~/.claude/workspace/pipelines/{pipeline-id}/pipeline_final.md`
2. Read `~/.claude/workspace/pipelines/{pipeline-id}/build_report.md`
3. Read completed predecessor detail files: `details/{dep}.md`
4. Identify component type (Command/Agent/Skill) from build_report.md

### Phase 2: Design Draft (pipeline-detail-drafter agent)

Delegate to `pipeline-detail-drafter` agent:
```
Create a design draft for the following pipeline component.

Component: {component-name}
Component Type: {Command|Agent|Skill}

Pipeline design: ~/.claude/workspace/pipelines/{pipeline-id}/pipeline_final.md
Build report: ~/.claude/workspace/pipelines/{pipeline-id}/build_report.md
Predecessor details: {list of details/{dep}.md paths}
Target project: {target-project-path}

Save draft to: ~/.claude/workspace/pipelines/{pipeline-id}/details/{component-name}_draft.md
```

Wait for completion.

### Phase 3: Update Status

1. Read `build_report.md`
2. Update the component's status from "scaffolded" to "draft" in the Component Status table
3. Save updated `build_report.md`

### Phase 4: Present to User

1. Read `details/{component-name}_draft.md`
2. Display to user:
   - Component type and role
   - Proposed content summary
   - Design decisions
   - Questions for human (if any)
3. Guide: "Review the draft and provide feedback. Then run:"
   - `/detail-apply (user:pipeline) {pipeline-id} --component {component-name}`
   - Or with feedback: `/detail-apply (user:pipeline) {pipeline-id} --component {component-name} {your feedback}`

## Output

`~/.claude/workspace/pipelines/{pipeline-id}/details/{component-name}_draft.md`

## Notes

- This command is READ-ONLY for project files — it only analyzes and proposes
- The actual file creation happens in `/detail-apply` after human review
- Review the draft carefully — component quality determines pipeline quality
- Mark Todo items as complete in the _draft.md before running /detail-apply

---
Arguments: $ARGUMENTS
