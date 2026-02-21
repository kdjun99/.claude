---
name: pipeline-design-patterns
description: Design patterns and principles for building automation pipelines using the 3-Layer architecture (Command/Agent/Skill). Use when designing, scaffolding, or detailing pipeline components for any project. Covers core principles (Human-in-the-Loop, Artifact-Driven Handoff, Idempotent Steps, Fail-Safe, Progressive Autonomy, Composable Pipeline), artifact structure standards with Todo-based gating, and component design patterns. Essential for pipeline-analyzer, pipeline-architect, pipeline-scaffolder, pipeline-detail-drafter, and pipeline-detail-applier agents.
---

# Pipeline Design Patterns

Battle-tested patterns and principles for designing automation pipelines.
Generalized from production use in the linkareer-main project.

## 3-Layer Architecture

```
User
  |
  v
Command (.claude/commands/)       <- User entry point, orchestrator
  |
  |-> Agent (.claude/agents/)     <- Isolated expert work (separate process)
  |     |
  |     '-> Skill (.claude/skills/) <- Shared domain knowledge injected into Agent
  |
  '-> (direct execution)          <- Simple tasks without Agent delegation
```

| Layer | Responsibility | Context |
|-------|---------------|---------|
| **Command** | Orchestration, input validation, gate verification, agent delegation, result consolidation | Main conversation |
| **Agent** | Independent expert work in specialized domain | Separate process (isolated) |
| **Skill** | Reusable domain knowledge provider | Injected into Agent |

## Core Principles (6)

| Principle | One-liner |
|-----------|-----------|
| Human-in-the-Loop | Always confirm before irreversible actions |
| Artifact-Driven Handoff | Inter-step communication via files (md) only |
| Idempotent Steps | Safe to re-run with same input |
| Fail-Safe | Stop rather than cleanup on failure |
| Progressive Autonomy | Start with more confirmations, automate as trust grows |
| Composable Pipeline | Each step independently executable and composable |

Details: [references/core-principles.md](references/core-principles.md)

## Artifact Structure Standard

Every pipeline artifact follows this structure:

```markdown
# [Title]

## Context
- Pipeline ID / Phase / Component / Created / Target Project

## Prerequisites
- [x] Previous artifact reference status

## Content
(Actual content)

## Todo
- [x] Completed items
- [ ] Incomplete items

## Next
- Next step guidance + required conditions
```

**Todo-based Gate Rule**: Next command parses the previous artifact's `## Todo` section. If any `- [ ]` items remain, execution is blocked.

Details: [references/artifact-patterns.md](references/artifact-patterns.md)

## Component Design Patterns

### Command Design
- Gate verification -> Phase execution (agent delegation) -> Artifact generation -> Next guidance
- Use AskUserQuestion before irreversible actions

### Agent Design
- Separate read-only vs write-capable roles
- Separate judge (Agent) vs checklist (Skill) roles
- Follow artifact structure standard

### Skill Design
- Progressive Disclosure (metadata -> body -> references)
- Split to references when exceeding 500 lines

Details: [references/component-patterns.md](references/component-patterns.md)

## Proven Examples

Battle-tested examples from linkareer-main:
- Feature Pipeline (7 steps): start -> draft -> finalize -> implement -> test -> submit -> review
- Review Pipeline (5 parallel agents): P0~P4 priority-based

Details: [references/examples.md](references/examples.md)
