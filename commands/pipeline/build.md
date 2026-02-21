---
description: 'Scaffold all pipeline component files in target project'
allowed-tools: ['Read', 'Grep', 'Glob', 'Write', 'Bash', 'AskUserQuestion', 'Task']
argument-hint: [pipeline-id]
---

# Pipeline Build

Generate scaffold files for all confirmed pipeline components in the target project.

## Pipeline Context

```
/draft (user:pipeline) -> /finalize (user:pipeline) -> > /build (user:pipeline) -> /detail (user:pipeline) -> /detail-apply (user:pipeline)
```

## Input

$ARGUMENTS: Pipeline ID (kebab-case)

## Gate

Verify ALL of the following before proceeding:

1. **pipeline_final.md exists**
   - Path: `~/.claude/workspace/pipelines/{pipeline-id}/pipeline_final.md`
   - If missing: "Confirmed design not found. Run `/finalize (user:pipeline)` first."

2. **pipeline_final.md Todo is complete**
   - Read `pipeline_final.md`, find `## Todo` section
   - Parse for `- [ ]` patterns (incomplete items)
   - If incomplete items found:
     - Display: "The following items in pipeline_final.md are incomplete:"
     - List each incomplete item
     - "Complete these items and run again."
     - STOP execution.

## Execution

### Phase 1: Scaffold Generation (pipeline-scaffolder agent)

Delegate to `pipeline-scaffolder` agent:
```
Generate scaffold files for all pipeline components.

Confirmed design: ~/.claude/workspace/pipelines/{pipeline-id}/pipeline_final.md
Target project: {target-project-path from pipeline_final.md Context}

Save build report to: ~/.claude/workspace/pipelines/{pipeline-id}/build_report.md
```

Wait for completion.

### Phase 2: Verify

1. Read `build_report.md` to verify it was created
2. Verify scaffolded files exist at reported locations
3. Check for any conflicts reported

### Phase 3: Summary

Display to user:
- Number of files scaffolded (Commands / Agents / Skills)
- Any conflicts encountered
- Component Status Table
- First component to detail
- Next step: `/detail (user:pipeline) {pipeline-id} --component {first-component}`

## Output

`~/.claude/workspace/pipelines/{pipeline-id}/build_report.md` + scaffolded files in target project

## Notes

- Scaffold files contain frontmatter + role summary + TODO placeholders
- Existing files are NEVER overwritten — conflicts are reported
- After scaffolding, detail each component using `/detail` + `/detail-apply`
- Follow dependency order: Skills first, then Agents, then Commands

---
Arguments: $ARGUMENTS
