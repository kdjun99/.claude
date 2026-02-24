---
description: "Decompose analyzed spec into foundation + domain feature-requests with dependency graph. Second step of the spec-decomposition pipeline."
allowed-tools: Read, Write, Glob, Grep, Task
---

# /spec-decompose {spec-id}

Decompose a spec analysis into individual feature-request.md files organized by repo and execution order.

## Input

- `spec-id`: Identifier for this spec (must match a previous /spec-analyze run)

Parse from: $ARGUMENTS

## Gate

1. **Argument present**: `spec-id` must be provided
   - If missing: "Usage: /spec-decompose {spec-id}"
2. **Analysis exists**: Verify `~/.claude/workspace/specs/{spec-id}/_spec-analysis.md` exists
   - If missing: "Spec analysis not found. Run `/spec-analyze {spec-id} --pdf {path}` first."
3. **Analysis Todo complete**: Read _spec-analysis.md, check that `## Todo` section has no `- [ ]` items
   - If incomplete: "The spec analysis has incomplete items. Review and complete _spec-analysis.md before decomposing."
4. **No duplicate run**: Check if `~/.claude/workspace/specs/{spec-id}/_requirements.md` already exists
   - If exists: "Decomposition already exists for '{spec-id}'. To re-decompose, delete `~/.claude/workspace/specs/{spec-id}/_requirements.md` and `feature-requests/` first."

## Execution

### Phase 1: Load Context

1. Read `~/.claude/workspace/specs/{spec-id}/_spec-analysis.md`
2. Extract from analysis:
   - PDF path (from Metadata section)
   - Project name (from Metadata → `Project` field)
   - Feature group count
   - Impacted repos list
   - Estimated feature-request count
3. Resolve repo structure skill path: `~/.claude/skills/{project}-repo-structure/skill.md`
   - If skill file does not exist: "Repo structure skill not found for project '{project}'. Expected: ~/.claude/skills/{project}-repo-structure/skill.md"
4. Display to user:
   ```
   Starting decomposition...
   - Spec ID: {spec-id}
   - Project: {project}
   - Repo Structure: ~/.claude/skills/{project}-repo-structure/skill.md
   - Feature groups: {n}
   - Impacted repos: {list}
   - Estimated feature-requests: {n}
   ```

### Phase 2: Delegate to spec-decomposer Agent

Delegate to the `spec-decomposer` agent (subagent_type from agents/spec-decomposer.md):

```
Decompose the following spec analysis into feature-requests.

analysis_path: ~/.claude/workspace/specs/{spec-id}/_spec-analysis.md
pdf_path: {pdf path from analysis metadata}
spec_id: {spec-id}
repo_structure_skill: ~/.claude/skills/{project}-repo-structure/skill.md
output_path: ~/.claude/workspace/specs/{spec-id}/

Read the analysis, PDF, and repo-structure skill, then generate foundation + domain feature-requests, build dependency graph, calculate execution waves, and write all output files.
```

Use Task tool with:
- `subagent_type`: The spec-decomposer agent
- `model`: sonnet (as specified in agent frontmatter)
- Wait for completion (do NOT run in background)

### Phase 3: Verify Output

1. Verify `~/.claude/workspace/specs/{spec-id}/_requirements.md` was created
   - If missing: "Error: spec-decomposer agent did not produce _requirements.md. Check the agent output for errors."
2. Verify `~/.claude/workspace/specs/{spec-id}/feature-requests/` directory exists and contains files
   - If empty: "Error: No feature-request files generated. Check the agent output for errors."
3. Read `_requirements.md` to extract summary data:
   - Total feature-request count (foundation + domain)
   - Repo distribution
   - Wave count
   - Sizing validation results

### Phase 4: Present Results

Display to user:

```
## Decomposition Complete: {spec-id}

### Summary
| Metric | Count |
|--------|-------|
| Foundation feature-requests | {f} |
| Domain feature-requests | {d} |
| Total feature-requests | {n} |
| Execution waves | {w} |
| Sizing: all pass | {yes/no} |

### Repo Distribution
| Repo | Foundation | Domain | Total |
|------|-----------|--------|-------|
| {repo} | {n} | {n} | {n} |
| ... | | | |

### Execution Waves
Wave 1: {list of FR IDs} (parallel)
Wave 2: {list of FR IDs} (parallel)
...

### Artifacts
- Feature-requests: ~/.claude/workspace/specs/{spec-id}/feature-requests/
- Requirements: ~/.claude/workspace/specs/{spec-id}/_requirements.md
```

If any sizing validation failed:
```
⚠ {n} feature-requests were split during generation due to sizing limits.
Review _requirements.md Sizing Validation section for details.
```

### Phase 5: Human Checkpoint (H2)

Request user review:

```
## Review Checklist

Please review the generated feature-requests for:
1. **Sizing accuracy** — Is each feature-request truly 1-day of work?
2. **Foundation completeness** — Do foundations cover all needed schema/API definitions?
3. **Domain scope** — Is each domain FR focused on exactly 1 user-facing function?
4. **Dependency correctness** — Are all dependencies declared? Any missing cross-repo links?
5. **Wave sequencing** — Can wave items truly run in parallel?
6. **Acceptance criteria** — Are criteria specific and testable?

## Next Steps

1. Share feature-requests with team for discussion
2. Record discussion decisions in: `~/.claude/workspace/specs/{spec-id}/spec-discussion.md`
   - Format: one decision per feature-request (approved / modified / deferred)
   - Include reasons for any modifications or deferrals
3. After discussion, finalize:
   /spec-confirm {spec-id}
```

## Output

- `~/.claude/workspace/specs/{spec-id}/feature-requests/` — Feature-request .md files organized by repo
- `~/.claude/workspace/specs/{spec-id}/_requirements.md` — Complete index with dependency graph, sizing validation, and repo distribution
- Summary displayed to user with review checklist

## Error Handling

| Error | Action |
|-------|--------|
| Analysis not found | Stop: "Run `/spec-analyze {spec-id} --pdf {path}` first." |
| Analysis incomplete | Stop: "Complete _spec-analysis.md Todo items first." |
| Agent timeout | Stop: "spec-decomposer agent timed out. The analysis may have too many groups. Try reducing scope." |
| No FRs generated | Stop with error, suggest checking analysis quality |
| _requirements.md not created | Stop with error, suggest re-running |
| Sizing failures | Display warning, continue (splits are logged in _requirements.md) |
