---
description: "Execute a feature implementation from an approved design plan. Runs implementation, build/test verify-fix loops, parallel code review, and creates a PR."
allowed-tools: ["Task", "Read", "Write", "Edit", "Glob", "Grep", "Bash", "AskUserQuestion"]
argument-hint: "req-id"
---

# /feature:execute {req-id}

Execute feature implementation from an approved design plan. Handles implementation, build verification, test writing, parallel review, and PR creation.

**Usage:** `/feature:execute <req-id>`

## Arguments

Parse from `$ARGUMENTS`:
- `req-id`: First argument — kebab-case identifier matching an existing design plan

## Gate Validation

Run all gates sequentially. Stop at the first failure.

### Gate 1: req-id present

- If `$ARGUMENTS` is empty: use AskUserQuestion to ask "What is the feature request ID?"
- Validate req-id is kebab-case

### Gate 2: design-plan.md exists

1. Detect the project root (nearest directory from cwd upward containing `.claude/`)
2. Search for design-plan.md in this order:
   a. `{project-root}/.claude/workspace/{req-id}/design-plan.md` (standard, ad hoc path)
   b. `{project-root}/.claude/workspace/specs/*/features/{req-id}/design-plan.md` (spec-originated path)
3. If found in standard location (2a): use it directly
4. If found in spec location (2b) but NOT in standard location:
   - Create standard workspace: `mkdir -p {project-root}/.claude/workspace/{req-id}`
   - Copy design-plan to standard location: `cp {spec-path}/design-plan.md {project-root}/.claude/workspace/{req-id}/design-plan.md`
   - Also copy `_feature-request.md` if it exists in the spec features directory
   - Display: "Using spec-originated design plan from `{spec-path}`. Copied to standard workspace."
5. If not found in either location: "Error: design-plan.md not found. Run `/feature:plan {req-id}` or `/spec:plan` first."

### Gate 3: design-plan-approved.md exists

1. Check if `{project-root}/.claude/workspace/{req-id}/design-plan-approved.md` exists
2. If missing:
   - Use AskUserQuestion: "design-plan-approved.md not found. The design plan must be approved before execution."
   - Options: "Approve now (create marker)" / "Stop — I'll review first"
   - If approve now: create `design-plan-approved.md` with content `Approved: {ISO timestamp}`
   - If stop: halt execution

### Gate 4: coding-guide check

1. Check if `{project-root}/.claude/skills/coding-guide/skill.md` exists
2. If exists:
   - Read the coding-guide manifest
   - Read all skill files listed under "Skills to Load" section
   - Store combined content as `$CODING_GUIDE`
3. If missing:
   - Display warning: "No coding-guide found. Implementation will proceed without project-specific conventions."
   - Set `$CODING_GUIDE` to empty string

### Gate 5: git working directory clean

1. Run `git status --porcelain` in project root
2. If output is non-empty:
   - Use AskUserQuestion: "Git working directory has uncommitted changes. Proceeding may mix feature changes with existing modifications."
   - Options: "Proceed anyway" / "Stop — I'll clean up first"
   - If stop: halt execution

## Execution

### Phase 0: Branch Setup

1. Get current branch: `git branch --show-current`
2. If not already on `feature/{req-id}`:
   - Create and checkout: `git checkout -b feature/{req-id}`
3. If already on `feature/{req-id}`: continue
4. Read `design-plan.md` into `$DESIGN_PLAN`
5. Read `_feature-request.md` into `$FEATURE_REQUEST` (if exists)
6. Display to user:
   ```
   Starting feature execution...
   - Req ID: {req-id}
   - Branch: feature/{req-id}
   - Design Plan: {path to design-plan.md}
   - Coding Guide: {found / generic mode}
   ```

### Phase 1: Implementation (executor or deep-executor)

First, count the estimated files from the design plan's "Implementation Files" section.

**If <= 5 files**: use `oh-my-claudecode:executor` (sonnet)
**If > 5 files**: use `oh-my-claudecode:deep-executor` (opus)

Delegate to the selected agent:

```
Implement the feature according to this design plan.

Design Plan:
{$DESIGN_PLAN}

{if $FEATURE_REQUEST is not empty:}
Original Feature Request:
{$FEATURE_REQUEST}
{end if}

{if $CODING_GUIDE is not empty:}
Project Coding Guide:
{$CODING_GUIDE}
{end if}

Implementation Rules:
1. Follow the design plan precisely — implement all items listed
2. Follow the project coding guide conventions if provided
3. Implementation order (if applicable):
   a. DB Migration / Prisma Schema
   b. DTOs (GraphQL types)
   c. Services (Business logic)
   d. Resolvers (API layer)
   e. Module registration
4. Do NOT write tests — that is handled in a separate phase
5. After implementation, create a report listing all changed files

Output Requirements:
- Save implementation report to: {project-root}/.claude/workspace/{req-id}/_implementation-report.md
- Report format:
  ## Changed Files
  | File | Action | Description |
  |------|--------|-------------|
  | path/to/file | Created/Modified | what was done |

  ## Key Decisions
  [Any deviations from plan or notable decisions]

  ## Known Issues
  [Any issues encountered]
- Return a 3-line summary of what was implemented
```

Use Task tool with:
- `subagent_type`: selected agent type
- `model`: sonnet (executor) or opus (deep-executor)
- Wait for completion

After completion, verify `_implementation-report.md` was created. If missing, report error and stop.

### Phase 2: Build Verify/Fix Loop (max 5 iterations)

Run the build verification loop. This loop runs in the main context — no agent delegation for the check, only for fixes.

```
loop_count = 0
WHILE loop_count < 5:
  1. Run build check:
     - `npm run type-check` (in project root)
     - If design plan includes DTO changes: `npm run build:gqlgen` first
  2. IF all checks PASS → display "Build verification passed." → break
  3. IF any check FAILS:
     - Display: "Build check failed (attempt {loop_count + 1}/5). Running build-fixer..."
     - Delegate to `oh-my-claudecode:build-fixer` (sonnet):
       ```
       Fix the following build errors.

       Error Output:
       {build error output}

       Changed Files (from implementation):
       {list from _implementation-report.md}

       {if $CODING_GUIDE is not empty:}
       Project Coding Guide:
       {$CODING_GUIDE}
       {end if}

       Fix the errors. Do NOT change test files. Focus only on build/type errors.
       Return a summary of what you fixed.
       ```
     - loop_count++

IF loop_count == 5:
  - Display: "Build verification failed after 5 attempts. Escalating to developer."
  - Use AskUserQuestion: "Build is still failing. How would you like to proceed?"
  - Options: "I'll fix manually, then continue" / "Stop execution"
  - If manual fix: wait for user, then re-run build check once. If still fails, stop.
  - If stop: halt execution
```

### Phase 3: Test Writing (test-engineer agent)

Delegate to `oh-my-claudecode:test-engineer` (sonnet):

```
Write tests for the feature implementation.

Design Plan (Test Plan section):
{Test Plan section from $DESIGN_PLAN}

Implementation Report:
{content of _implementation-report.md}

{if $CODING_GUIDE is not empty:}
Project Coding Guide:
{$CODING_GUIDE}
{end if}

Rules:
1. Follow the test plan from the design plan
2. Follow project testing conventions if coding guide is provided
3. Cover all test cases listed in the design plan
4. Include edge cases
5. Use the project's test file naming convention

After writing tests, append test results to _implementation-report.md:

  ## Test Files Created
  | File | Test Count | Description |
  |------|-----------|-------------|
  | path/to/test | N | what is tested |

Return a summary of tests written.
```

Use Task tool with:
- `subagent_type`: `oh-my-claudecode:test-engineer`
- `model`: sonnet
- Wait for completion

### Phase 4: Test Verify/Fix Loop (max 5 iterations)

Same pattern as Phase 2, but for tests:

```
loop_count = 0
WHILE loop_count < 5:
  1. Run test check:
     - `npm run test -- --testPathPattern="{test file paths from Phase 3}"`
  2. IF all tests PASS → display "Test verification passed." → break
  3. IF any test FAILS:
     - Display: "Tests failed (attempt {loop_count + 1}/5). Running build-fixer..."
     - Delegate to `oh-my-claudecode:build-fixer` (sonnet):
       ```
       Fix the following test failures.

       Test Output:
       {test error output}

       Implementation Report:
       {content of _implementation-report.md}

       {if $CODING_GUIDE is not empty:}
       Project Coding Guide:
       {$CODING_GUIDE}
       {end if}

       Fix the failing tests. You may fix both test code and implementation code if the test expectations are correct but the implementation has a bug.
       Return a summary of what you fixed.
       ```
     - loop_count++

IF loop_count == 5:
  - Display: "Test verification failed after 5 attempts. Escalating to developer."
  - Use AskUserQuestion: "Tests are still failing. How would you like to proceed?"
  - Options: "I'll fix manually, then continue" / "Stop execution"
  - If manual fix: wait for user, then re-run tests once. If still fails, stop.
  - If stop: halt execution
```

### Phase 5: Parallel Code Review (security-reviewer + code-reviewer)

Run both review agents in **parallel** using two Task calls in a single response:

**Security Review:**
```
Review the implementation for security vulnerabilities.

Changed Files:
{list from _implementation-report.md}

Design Plan:
{$DESIGN_PLAN}

{if $CODING_GUIDE is not empty:}
Project Coding Guide:
{$CODING_GUIDE}
{end if}

Focus on:
1. Input validation and sanitization
2. Authentication and authorization checks
3. SQL injection / query injection risks
4. Data exposure risks
5. OWASP Top 10 relevant issues

Return your findings in this format:
## Security Review
### Critical Issues
### Warnings
### Passed Checks
```

Use Task tool with:
- `subagent_type`: `oh-my-claudecode:security-reviewer`
- `model`: sonnet

**Code Review:**
```
Review the implementation for code quality, correctness, and adherence to conventions.

Changed Files:
{list from _implementation-report.md}

Design Plan:
{$DESIGN_PLAN}

{if $CODING_GUIDE is not empty:}
Project Coding Guide:
{$CODING_GUIDE}
{end if}

Focus on:
1. Design plan adherence — are all items implemented?
2. Code conventions — does the code follow project patterns?
3. Error handling — are errors handled appropriately?
4. Performance — any N+1 queries, missing indexes, unnecessary computations?
5. Maintainability — naming, structure, complexity

Return your findings in this format:
## Code Review
### Critical Issues
### Suggestions
### Passed Checks
```

Use Task tool with:
- `subagent_type`: `oh-my-claudecode:code-reviewer`
- `model`: opus

After both complete, merge results into `{project-root}/.claude/workspace/{req-id}/_review-report.md`:

```markdown
# Review Report: {req-id}

## Security Review
{security reviewer output}

## Code Review
{code reviewer output}

## Summary
- Security: {PASS / WARNINGS / CRITICAL}
- Code Quality: {PASS / SUGGESTIONS / CRITICAL}
- Overall: {APPROVED / NEEDS ATTENTION / BLOCKED}
```

If any **critical issues** are found:
- Display the critical issues to the user
- Use AskUserQuestion: "Review found critical issues. How would you like to proceed?"
- Options: "Fix critical issues and continue" / "Proceed anyway" / "Stop execution"
- If fix: address the critical issues in the main context, then re-run the relevant review
- If proceed: continue to Phase 6 with a note in the PR
- If stop: halt execution

### Phase 6: Git Commit + Push + PR

1. **Stage changes:**
   ```bash
   git add -A
   ```
   Verify staged files match expectations from `_implementation-report.md`.

2. **Commit:**
   ```bash
   git commit -m "feat({req-id}): implement feature

   Based on design plan: .claude/workspace/{req-id}/design-plan.md

   Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>"
   ```

3. **Push:**
   ```bash
   git push -u origin feature/{req-id}
   ```

4. **Create PR:**
   Read the design plan summary and review report to construct the PR body.

   ```bash
   gh pr create --title "feat({req-id}): {short description from design plan}" --body "$(cat <<'EOF'
   ## Summary
   {Summary from design-plan.md}

   ## Changes
   {Changed files table from _implementation-report.md}

   ## Review Report
   {Summary from _review-report.md}

   ## Test Coverage
   {Test files table from _implementation-report.md}

   ## Design Plan
   See: `.claude/workspace/{req-id}/design-plan.md`

   ---
   Generated with [Claude Code](https://claude.com/claude-code)
   EOF
   )"
   ```

5. **Save PR report:**
   Write `{project-root}/.claude/workspace/{req-id}/pr-report.md`:

   ```markdown
   # PR Report: {req-id}

   ## PR
   - URL: {pr-url}
   - Branch: feature/{req-id}
   - Created: {ISO timestamp}

   ## Summary
   {brief description}

   ## Files Changed
   {count} files ({created} created, {modified} modified)

   ## Review Status
   - Security: {status}
   - Code Quality: {status}

   ## Next Steps
   - [ ] Get team review on PR
   - [ ] Address review comments
   - [ ] Merge when approved
   ```

6. **Display to user:**
   ```
   ## Feature Implementation Complete!

   PR created: {pr-url}
   Branch: feature/{req-id}

   ### Workspace Artifacts
   - Design Plan: .claude/workspace/{req-id}/design-plan.md
   - Implementation Report: .claude/workspace/{req-id}/_implementation-report.md
   - Review Report: .claude/workspace/{req-id}/_review-report.md
   - PR Report: .claude/workspace/{req-id}/pr-report.md

   ### Summary
   - Files changed: {n}
   - Tests written: {n}
   - Build: PASS
   - Tests: PASS
   - Security review: {status}
   - Code review: {status}
   ```

## Output

- `{project-root}/.claude/workspace/{req-id}/_implementation-report.md` — Implementation summary + test files
- `{project-root}/.claude/workspace/{req-id}/_review-report.md` — Security + code review combined
- `{project-root}/.claude/workspace/{req-id}/pr-report.md` — PR URL and summary (human-facing)

## Error Handling

| Error | Action |
|-------|--------|
| req-id missing | Ask user via AskUserQuestion |
| design-plan.md not found | Stop: "Run `/feature:plan {req-id}` first." |
| design-plan-approved.md not found | Ask to approve or stop |
| coding-guide missing | Warn, proceed without conventions |
| Git dirty working directory | Ask to proceed or stop |
| Implementation agent failure | Report error, ask user for guidance |
| Build fails after 5 attempts | Escalate to developer |
| Test fails after 5 attempts | Escalate to developer |
| Review finds critical issues | Ask user how to proceed |
| Git push fails | Report error, suggest manual push |
| PR creation fails | Report error, provide manual gh command |
| Branch already exists | Checkout existing branch, ask to continue or reset |

---
Arguments: $ARGUMENTS
