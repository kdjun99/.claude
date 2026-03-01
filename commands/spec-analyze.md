---
description: "[DEPRECATED] Use /spec:plan instead. Analyze a PDF design spec."
allowed-tools: Read, Write, Glob, Grep, Task
---

> **DEPRECATED**: This command has been replaced by `/spec:plan` which combines spec-analyze, spec-translate, spec-decompose, and spec-confirm into a single unified pipeline.

# /spec-analyze {spec-id} --pdf {path} --project {project}

Analyze a PDF design spec and produce a structured analysis artifact.

## Input

- `spec-id`: Identifier for this spec (kebab-case, e.g., `simple-apply-v1`)
- `--pdf {path}`: Absolute path to the PDF spec file
- `--project {project}`: Project name (e.g., `linkareer`). Determines which repo-structure skill to load (`~/.claude/skills/{project}-repo-structure/skill.md`)

Parse from: $ARGUMENTS

## Gate

1. **Arguments present**: `spec-id`, `--pdf`, and `--project` must all be provided
   - If missing: "Usage: /spec-analyze {spec-id} --pdf {path} --project {project}"
2. **PDF exists**: Verify file exists at the given path
   - If missing: "PDF not found at: {path}"
3. **Project skill exists**: Verify `~/.claude/skills/{project}-repo-structure/skill.md` exists
   - If missing: "Repo structure skill not found for project '{project}'. Expected: ~/.claude/skills/{project}-repo-structure/skill.md"
4. **No duplicate run**: Check if `~/.claude/workspace/specs/{spec-id}/_spec-analysis.md` already exists
   - If exists: "Analysis already exists for '{spec-id}'. To re-analyze, delete `~/.claude/workspace/specs/{spec-id}/_spec-analysis.md` first, or use a different spec-id."

## Execution

### Phase 1: Setup Workspace

1. Parse arguments:
   - Extract `spec-id` as the first argument
   - Extract PDF path from `--pdf` flag
   - Extract project name from `--project` flag
2. Validate PDF file exists using Read tool (attempt to read first page)
3. Resolve repo structure skill path: `~/.claude/skills/{project}-repo-structure/skill.md`
4. Create workspace directory:
   ```
   mkdir -p ~/.claude/workspace/specs/{spec-id}
   ```
5. Display to user:
   ```
   Starting spec analysis...
   - Spec ID: {spec-id}
   - Project: {project}
   - PDF: {path}
   - Repo Structure: ~/.claude/skills/{project}-repo-structure/skill.md
   - Workspace: ~/.claude/workspace/specs/{spec-id}/
   ```

### Phase 2: Delegate to spec-analyzer Agent

Delegate to the `spec-analyzer` agent (subagent_type from agents/spec-analyzer.md):

```
Analyze the following PDF design spec.

pdf_path: {absolute path to PDF}
spec_id: {spec-id}
project: {project}
repo_structure_skill: ~/.claude/skills/{project}-repo-structure/skill.md
output_path: ~/.claude/workspace/specs/{spec-id}/

Read the PDF, extract the checklist, group features by domain, map to repos using the repo-structure skill, and write _spec-analysis.md.
```

Use Task tool with:
- `subagent_type`: The spec-analyzer agent
- `model`: sonnet (as specified in agent frontmatter)
- Wait for completion (do NOT run in background)

### Phase 3: Verify Output

1. Verify `~/.claude/workspace/specs/{spec-id}/_spec-analysis.md` was created
   - If missing: "Error: spec-analyzer agent did not produce _spec-analysis.md. Check the agent output for errors."
2. Verify `~/.claude/workspace/specs/{spec-id}/_domain-mapping.md` was created
   - If missing: "Error: spec-analyzer agent did not produce _domain-mapping.md. Check the agent output for errors."
3. Read _spec-analysis.md to extract summary data:
   - Checklist counts (included/excluded/uncertain)
   - Number of feature groups
   - Impacted repos list
   - Estimated feature-request count
   - Domain terms count (from ## Domain Terms section)

### Phase 4: Present Results

Display to user:

```
## Spec Analysis Complete: {spec-title}

| Metric | Count |
|--------|-------|
| Included items | {n} |
| Excluded items | {n} |
| Uncertain items | {n} |
| Feature groups | {n} |
| Impacted repos | {list} |
| Est. feature-requests | {n} ({f} foundation + {d} domain) |
| Domain terms extracted | {n} |

Artifacts:
- Analysis: ~/.claude/workspace/specs/{spec-id}/_spec-analysis.md
- Domain mapping template: ~/.claude/workspace/specs/{spec-id}/_domain-mapping.md
```

If there are uncertain items:
```
⚠ {n} items have uncertain include/exclude status — review these in the analysis.
```

### Phase 5: Human Checkpoint (H1)

Request user review:

```
## Review Checklist

Please review `_spec-analysis.md` for:
1. **Checklist accuracy** — Are all included/excluded items correct?
2. **Feature grouping** — Do the groups make logical sense? Any items in the wrong group?
3. **Repo mapping** — Are repos assigned correctly per group?
4. **Missing requirements** — Any checklist items that were missed or misinterpreted?
5. **Uncertain items** — Decide include/exclude for any flagged items
6. **Domain terms** — Are all business domain terms captured? Any missing terms?

Then fill in the domain mapping:
1. Open `~/.claude/workspace/specs/{spec-id}/_domain-mapping.md`
2. For each spec term, fill in: DB Table, ORM Model, GraphQL Type
3. Mark N/A for columns that don't apply

After completing the mapping, validate it:
/spec-translate {spec-id}
```

## Output

- `~/.claude/workspace/specs/{spec-id}/_spec-analysis.md` — Full analysis artifact with domain terms
- `~/.claude/workspace/specs/{spec-id}/_domain-mapping.md` — Domain mapping template for developer to fill
- Summary displayed to user with review checklist

## Error Handling

| Error | Action |
|-------|--------|
| PDF not found | Stop with clear error message |
| PDF unreadable | Stop: "Could not read PDF. Verify the file is a valid PDF." |
| Agent timeout | Stop: "spec-analyzer agent timed out. The PDF may be too large. Try splitting it." |
| No checklist found | Agent writes _spec-analysis.md with warning; command flags to user: "No checklist found in PDF — verify this is the correct spec." |
| _spec-analysis.md not created | Stop with error, suggest re-running |
