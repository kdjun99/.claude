# Delegation Checklist — Subagent Task Handoff

## Before Delegating to a Subagent

Every subagent delegation should include these fields. If any are missing, the calling command should fill them before delegating.

### Required Fields

```markdown
## Task Delegation

**Goal**: {What the subagent should achieve — 1 sentence}
**Component**: {What is being worked on — name, type}
**Output path**: {Where to save results — absolute file path}

**Input files** (read these for context):
- {file path 1} — {what it contains}
- {file path 2} — {what it contains}

**Constraints**:
- {What must NOT be changed}
- {Boundaries of the work}

**Additional context**: {feedback, preferences, or decisions from the user}
```

### Examples

#### Good Delegation

```
Create a design draft for the auth-validator agent.

Goal: Design the agent's system prompt, tools, and output format
Component: auth-validator (Agent, sonnet)
Output path: ~/.claude/workspace/pipelines/auth/details/auth-validator_draft.md

Input files:
- ~/.claude/workspace/pipelines/auth/pipeline_final.md — overall pipeline design
- ~/.claude/workspace/pipelines/auth/build_report.md — component list and dependencies
- ~/.claude/workspace/pipelines/auth/details/auth-patterns.md — predecessor detail

Constraints:
- Read-only agent (no file modifications)
- Must use existing JWT patterns from src/lib/auth.ts

Additional context: User prefers session-based auth over pure JWT
```

#### Bad Delegation

```
Design the auth-validator agent. Check the pipeline files for context.
```

Problems:
- No output path specified
- "Check the pipeline files" — which files? Where?
- No constraints mentioned
- No user preferences passed

---

## Checklist Summary

Before calling a subagent, verify:

- [ ] Goal is a clear, single sentence
- [ ] Output file path is specified
- [ ] All input file paths are listed with descriptions
- [ ] Constraints are stated (or explicitly "none")
- [ ] User feedback/preferences are included (if any)
- [ ] Target project path is provided (if subagent writes files)
