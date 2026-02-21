# Verification by Work Type

## Code Changes

### After modifying existing code

```
1. Re-read the diff (changed lines only)
   - Does every change serve the stated goal?
   - Were any lines deleted that shouldn't have been?
   - Are there leftover debug statements or comments?

2. Check structural integrity
   - Imports: are new imports added? Are removed imports still needed elsewhere?
   - Exports: if a function/type was renamed, are all references updated?
   - Types: does the change introduce any type mismatches?

3. Run validation (if available)
   - TypeScript: check for type errors in changed files
   - Linter: run on changed files if configured
   - Tests: run related test files if they exist

4. Boundary check
   - Are there other callers of the modified function/component?
   - Could this change break anything outside the modified file?
```

### After creating new code

```
1. File basics
   - Created at the correct path?
   - Follows project naming conventions?
   - Correct file extension?

2. Content completeness
   - All requested functionality implemented?
   - No TODO/placeholder left without explanation?
   - Proper error handling at system boundaries?

3. Integration
   - Exported from the correct module?
   - Imported where needed?
   - Registered in any config/router/index files if required?
```

---

## Document / Artifact Generation

### After creating markdown files (skills, agents, commands, designs)

```
1. Structure check
   - Required sections present? (e.g., Context/Content/Todo/Next for pipeline artifacts)
   - Frontmatter valid? (YAML syntax, required fields)
   - No broken markdown (unclosed code blocks, missing headers)

2. Content accuracy
   - File paths referenced actually exist?
   - Component names match what's defined elsewhere?
   - No copy-paste artifacts from templates (placeholder text left in)

3. Consistency
   - Terminology matches the rest of the project
   - Naming conventions followed (kebab-case, etc.)
   - Language is correct (English for AI-facing documents)
```

---

## Subagent Delegation Results

### After a subagent returns

```
1. Output file verification
   - File exists at the specified path?
   - File is not empty?
   - File size is reasonable (not suspiciously small or truncated)?

2. Content spot-check
   - Read the first and last sections of the output file
   - Does it match the type of work that was delegated?
   - Are key identifiers correct (pipeline ID, component name, etc.)?

3. Status check
   - Did the subagent report success?
   - Are there any warnings or errors in the summary?
   - If the subagent updated a tracking file (e.g., build_report.md), verify the update
```

---

## Multi-Step / Pipeline Work

### After completing a sequence of steps

```
1. Step completeness
   - Were all planned steps executed?
   - Check the task list / Todo items — any skipped?

2. Artifact chain
   - Each step's output exists and feeds into the next step?
   - No broken references between artifacts?

3. Final state
   - End state matches the originally stated goal?
   - All tracking documents updated (implementation plan, build report, etc.)?
```

---

## Quick Reference Matrix

| Work Type | Must Verify | Nice to Verify | Skip |
|-----------|------------|----------------|------|
| **Single file edit** | Changed lines, imports | Related tests | Full project scan |
| **New file creation** | Path, content, integration | Naming conventions | Performance |
| **Multi-file refactor** | All changed files, exports, callers | Tests, types | Unrelated files |
| **Document generation** | Structure, frontmatter, paths | Terminology | Formatting details |
| **Subagent result** | File exists, not empty, spot-check | Full content review | Line-by-line audit |
| **Pipeline step** | Step completeness, artifact chain | Tracking docs | Re-running prior steps |
