---
description: "Analyze a PDF design spec (기획서) — extract checklist, group features, map to repos. First step of the spec-decomposition pipeline."
allowed-tools: Read, Write, Glob, Grep, Task
---

# /spec-analyze {spec-id} --pdf {path}

Analyze a PDF design spec and produce a structured analysis artifact.

## Input

- `spec-id`: Identifier for this spec (kebab-case, e.g., `simple-apply-v1`)
- `--pdf {path}`: Absolute path to the PDF spec file

Parse from: $ARGUMENTS

## Gate

1. **Arguments present**: Both `spec-id` and `--pdf` must be provided
   - If missing: "Usage: /spec-analyze {spec-id} --pdf {path}"
2. **PDF exists**: Verify file exists at the given path
   - If missing: "PDF not found at: {path}"
3. **No duplicate run**: Check if `~/.claude/workspace/specs/{spec-id}/_spec-analysis.md` already exists
   - If exists: "Analysis already exists for '{spec-id}'. To re-analyze, delete `~/.claude/workspace/specs/{spec-id}/_spec-analysis.md` first, or use a different spec-id."

## Execution

### Phase 1: Setup Workspace

1. Parse arguments:
   - Extract `spec-id` as the first argument
   - Extract PDF path from `--pdf` flag
2. Validate PDF file exists using Read tool (attempt to read first page)
3. Create workspace directory:
   ```
   mkdir -p ~/.claude/workspace/specs/{spec-id}
   ```
4. Display to user:
   ```
   Starting spec analysis...
   - Spec ID: {spec-id}
   - PDF: {path}
   - Workspace: ~/.claude/workspace/specs/{spec-id}/
   ```

### Phase 2: Delegate to spec-analyzer Agent

Delegate to the `spec-analyzer` agent (subagent_type from agents/spec-analyzer.md):

```
Analyze the following PDF design spec.

pdf_path: {absolute path to PDF}
spec_id: {spec-id}
output_path: ~/.claude/workspace/specs/{spec-id}/

Read the PDF, extract the checklist, group features by domain, map to repos, and write _spec-analysis.md.
```

Use Task tool with:
- `subagent_type`: The spec-analyzer agent
- `model`: sonnet (as specified in agent frontmatter)
- Wait for completion (do NOT run in background)

### Phase 3: Verify Output

1. Verify `~/.claude/workspace/specs/{spec-id}/_spec-analysis.md` was created
   - If missing: "Error: spec-analyzer agent did not produce _spec-analysis.md. Check the agent output for errors."
2. Read _spec-analysis.md to extract summary data:
   - Checklist counts (included/excluded/uncertain)
   - Number of feature groups
   - Impacted repos list
   - Estimated feature-request count

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

Artifact: ~/.claude/workspace/specs/{spec-id}/_spec-analysis.md
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

After review, proceed to decomposition:
/spec-decompose {spec-id}
```

## Output

- `~/.claude/workspace/specs/{spec-id}/_spec-analysis.md` — Full analysis artifact
- Summary displayed to user with review checklist

## Error Handling

| Error | Action |
|-------|--------|
| PDF not found | Stop with clear error message |
| PDF unreadable | Stop: "Could not read PDF. Verify the file is a valid PDF." |
| Agent timeout | Stop: "spec-analyzer agent timed out. The PDF may be too large. Try splitting it." |
| No checklist found | Agent writes _spec-analysis.md with warning; command flags to user: "No checklist found in PDF — verify this is the correct spec." |
| _spec-analysis.md not created | Stop with error, suggest re-running |
