---
description: 'Apply human feedback to finalize a pipeline component'
allowed-tools: ['Read', 'Grep', 'Glob', 'Write', 'Edit', 'Bash', 'AskUserQuestion', 'Task']
argument-hint: [pipeline-id] --component [component-name] [feedback]
---

# Pipeline Detail Apply

Apply human feedback to a design draft and create the finalized component file.

## Pipeline Context

```
/draft (user:pipeline) -> /finalize (user:pipeline) -> /build (user:pipeline) -> /detail (user:pipeline) -> > /detail-apply (user:pipeline)
```

## Input

$ARGUMENTS: `{pipeline-id} --component {component-name} {optional feedback}`

Parse:
- `pipeline-id`: First argument
- `component-name`: Value after `--component`
- `feedback`: Everything after component-name (optional — if empty, draft is approved as-is)

If `--component` not provided, read `build_report.md` and show the next component with "draft" status, then ask user to confirm.

## Gate

Verify ALL of the following before proceeding:

1. **Draft exists**
   - Path: `~/.claude/workspace/pipelines/{pipeline-id}/details/{component-name}_draft.md`
   - If missing: "Design draft not found. Run `/detail (user:pipeline) {pipeline-id} --component {component-name}` first."

2. **Draft Todo is complete**
   - Read `details/{component-name}_draft.md`, find `## Todo` section
   - Parse for `- [ ]` patterns (incomplete items)
   - If incomplete items found:
     - Display: "The following items in the draft are incomplete:"
     - List each incomplete item
     - "Complete these items in the draft before applying."
     - STOP execution.

3. **build_report.md exists**
   - Path: `~/.claude/workspace/pipelines/{pipeline-id}/build_report.md`
   - If missing: "Build report not found. Run `/build (user:pipeline)` first."

4. **Component status is "draft"**
   - Read `build_report.md`, find the component in Component Status table
   - If component not found: "Component '{name}' not found in build report."
   - If status is not "draft": "Component '{name}' status is '{status}', expected 'draft'."

## Execution

### Phase 1: Context Loading

1. Read `~/.claude/workspace/pipelines/{pipeline-id}/details/{component-name}_draft.md`
2. Read `~/.claude/workspace/pipelines/{pipeline-id}/pipeline_final.md`
3. Read `~/.claude/workspace/pipelines/{pipeline-id}/build_report.md`
4. Read completed predecessor detail files: `details/{dep}.md`
5. Identify component type (Command/Agent/Skill) from build_report.md
6. Extract target project path from `pipeline_final.md` Context section
7. Parse feedback from $ARGUMENTS (everything after component-name)

### Phase 2: Apply (pipeline-detail-applier agent)

Delegate to `pipeline-detail-applier` agent:
```
Implement the following pipeline component based on the approved design draft and human feedback.

Component: {component-name}
Component Type: {Command|Agent|Skill}

Design draft: ~/.claude/workspace/pipelines/{pipeline-id}/details/{component-name}_draft.md
Human feedback: {feedback text, or "No feedback — draft approved as-is"}
Pipeline design: ~/.claude/workspace/pipelines/{pipeline-id}/pipeline_final.md
Build report: ~/.claude/workspace/pipelines/{pipeline-id}/build_report.md
Predecessor details: {list of details/{dep}.md paths}
Target project: {target-project-path}

Save detail report to: ~/.claude/workspace/pipelines/{pipeline-id}/details/{component-name}.md
```

Wait for completion.

### Phase 3: Verify

1. Read `details/{component-name}.md` to verify it was created
2. Verify the component file(s) exist at the reported locations in the target project
3. Read the created component file(s) to confirm they are not empty scaffolds

### Phase 4: Update Status

1. Read `build_report.md`
2. Update the component's status from "draft" to "detailed" in the Component Status table
3. Save updated `build_report.md`

### Phase 5: Summary

1. Read `details/{component-name}.md`
2. Display to user:
   - Component type and file location(s)
   - Implementation summary
   - Changes from draft (if feedback was applied)
   - Remaining Todo items (e.g., manual testing)
3. Determine next component:
   - Read `build_report.md` Component Status table
   - Find first component that is "scaffolded" or "draft" in dependency order
   - If all components are "detailed": "All components are complete! Pipeline implementation finished."
   - Otherwise: Guide to next component
4. Display next step:
   - If next component needs drafting: `/detail (user:pipeline) {pipeline-id} --component {next-component}`
   - If next component has draft ready: `/detail-apply (user:pipeline) {pipeline-id} --component {next-component}`
   - If all done: "Pipeline complete. Test each component individually."

## Output

- `~/.claude/workspace/pipelines/{pipeline-id}/details/{component-name}.md` (detail report)
- Finalized component file(s) in target project

## Notes

- This command WRITES to the target project — it creates/modifies actual component files
- The scaffold file is replaced with the finalized version
- If feedback is empty, the draft content is applied as-is
- After applying, test the component manually before moving to the next one
- Component quality determines pipeline quality — review the output carefully

---
Arguments: $ARGUMENTS
