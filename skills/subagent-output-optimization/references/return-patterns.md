# Return Patterns by Subagent Type

## Pattern 1: Analysis Agent (Read-Only)

Agents that explore, research, or analyze and produce a report.

### Bad Return (bloats main context)

```
I analyzed the project structure. Here are the findings:

The project has 47 files across 12 directories. The main entry point is src/index.ts
which imports from 3 modules: auth, api, and db. The auth module uses JWT tokens with
RS256 algorithm and stores refresh tokens in Redis. The api module has 23 endpoints
organized into 5 routers: users, posts, comments, auth, and admin. Each router follows
the pattern of... [500+ more lines]
```

### Good Return

```
Analysis complete. Saved to: ~/.claude/workspace/pipelines/my-pipeline/_process-analysis.md

Summary:
- 47 files, 12 directories, 3 core modules (auth, api, db)
- 23 API endpoints across 5 routers
- JWT auth with Redis session store
- 2 items need human confirmation (see Todo in analysis file)
```

---

## Pattern 2: Builder Agent (Write-Capable)

Agents that create or modify files in the target project.

### Bad Return

```
I created the following files:

File 1: .claude/commands/review.md
---
description: 'Run code review'
allowed-tools: ['Read', 'Grep', 'Glob']
---
# Code Review
[... full file content repeated ...]

File 2: .claude/agents/reviewer.md
[... full file content repeated ...]
```

### Good Return

```
Scaffolding complete. Report saved to: ~/.claude/workspace/pipelines/my-pipeline/build_report.md

Created 5 files:
- 2 Commands: .claude/commands/review.md, .claude/commands/apply-review.md
- 2 Agents: .claude/agents/reviewer.md, .claude/agents/applier.md
- 1 Skill: .claude/skills/review-patterns/SKILL.md

Conflicts: None
Next: Detail first component via /detail
```

---

## Pattern 3: Design Agent (Draft-Producing)

Agents that propose designs or plans for human review.

### Bad Return

```
Here is my proposed design for the review-agent component:

## Frontmatter
name: review-agent
description: ...
tools: Read, Grep, Glob
model: sonnet

## System Prompt
You are a code review specialist who...
[... entire proposed file content ...]

## Design Decisions
1. I chose sonnet because...
2. I limited tools to read-only because...
[... full rationale ...]
```

### Good Return

```
Design draft saved to: ~/.claude/workspace/pipelines/my-pipeline/details/review-agent_draft.md

Component: review-agent (Agent, sonnet, read-only)
Role: Analyzes code changes and produces structured review findings

Key decisions:
- Read-only tools (safety for automated review)
- sonnet model (needs judgment, not just pattern matching)

2 questions for human review — see draft Todo section.
```

---

## Pattern 4: Validation Agent

Agents that verify, test, or check something.

### Bad Return

```
Validation results:

Check 1: File structure — PASS
  Found .claude/commands/review.md at expected location
  File size: 2.4KB
  Frontmatter valid: yes
  Has description: yes
  Has allowed-tools: yes
  ...

Check 2: Dependencies — PASS
  review-agent.md exists
  review-patterns skill exists
  ...

[... 20 more checks with full details ...]
```

### Good Return

```
Validation complete. 22/22 checks passed.

Details saved to: ~/.claude/workspace/pipelines/my-pipeline/validation_report.md

All component files exist and have valid frontmatter.
No missing dependencies detected.
```

Or on failure:

```
Validation failed. 19/22 checks passed, 3 failed.

Details saved to: ~/.claude/workspace/pipelines/my-pipeline/validation_report.md

Failures:
1. review-agent.md — missing 'skills' in frontmatter
2. apply-review.md — references non-existent agent 'fixer'
3. review-patterns/SKILL.md — empty references/ directory

Fix these issues before proceeding.
```

---

## Return Size Guidelines

| Agent Type | Target Return Size | Detail File Size |
|------------|-------------------|-----------------|
| Analysis | 3-5 lines | Unlimited |
| Builder | File list + counts | Full report |
| Design | Key decisions + question count | Full draft |
| Validation | Pass/fail + failure list only | Full report |

## Universal Template

```
{Status}: {1-sentence summary}

{Detailed results saved to}: {file path}

{Key findings/outputs — max 5 bullet points}

{Next action or blockers — 1 line}
```
