# Component Patterns — Command/Agent/Skill Design Patterns

## Command Design Pattern

### Structure Template

```markdown
---
description: 'Brief description for /help'
allowed-tools: ['Read', 'Grep', 'Glob', 'Write', 'Bash', 'AskUserQuestion', 'Task']
argument-hint: [expected args]
---

# Command Name

Description.

## Pipeline Context
(Position of this command in the overall pipeline)

## Input
$ARGUMENTS description + required/optional args

## Gate (Pre-verification)
(Previous artifact existence + Todo completion check logic)

## Execution
### Phase 1: ...
### Phase 2: ...
(Agent delegation or direct execution)

## Output
(Artifact file path + structure)

## Notes
(Caveats, next step guidance)
```

### Command Design Principles

1. **Gate First**: Always pre-verify before execution. Check previous artifact existence + Todo completion.
2. **Orchestration Only**: Commands don't perform complex work directly. Delegate to agents.
3. **Phase-based Execution**: Divide into sequential Phases. Complete each Phase before the next.
4. **Result Consolidation**: Read agent artifacts and consolidate into final artifact.
5. **Next Guidance**: Clearly guide the next step on completion.

### When to Delegate to Agent vs Direct Execution

| Situation | Decision |
|-----------|----------|
| Expert analysis/design needed | Delegate to agent |
| Complex multi-tool usage | Delegate to agent |
| Simple file read/merge | Direct execution |
| User interaction (AskUserQuestion) | Direct execution |
| Gate verification | Direct execution |

---

## Agent Design Pattern

### Structure Template

```markdown
---
name: agent-name
description: Role description. When to use.
tools: Tool1, Tool2, ...
model: haiku|sonnet|opus
skills: skill1, skill2
---

You are a [specific role] specialized in [domain].

## When Invoked

You will receive:
- [Input 1]
- [Input 2]

## Process

1. [Step 1]
2. [Step 2]
...

## Output

Save result to: [file path]

Follow the artifact structure standard:
- ## Context
- ## Content
- ## Todo
- ## Next
```

### Agent Design Principles

1. **Clear Role**: Start with "You are a [role]". Specify the expert domain.
2. **Explicit I/O**: Clearly state what is received and what is produced.
3. **Artifact Structure Compliance**: Always follow Context/Content/Todo/Next structure.
4. **Read/Write Separation**:
   - Analysis/design agents: Read, Grep, Glob only (read-only)
   - Implementation agents: Read, Write, Edit, Grep, Glob, Bash (write-capable)
5. **Judge vs Checklist Separation**:
   - Agent: "Is this really a problem?" — judgment
   - Skill: "How do we check?" — procedure

### Model Selection Guide

| Model | Best For |
|-------|----------|
| **haiku** | Analysis, classification, simple pattern matching. Speed priority. |
| **sonnet** | Design judgment, code writing, complex reasoning. Quality-speed balance. |
| **opus** | High-difficulty architecture decisions, complex refactoring. Quality priority. |

---

## Skill Design Pattern

### Structure Template

```
skill-name/
├── SKILL.md              # Core content (< 500 lines)
└── references/            # Detailed content (loaded on demand)
    ├── topic-a.md
    └── topic-b.md
```

### Skill Design Principles

1. **Progressive Disclosure**: metadata(~100 words) -> body(<5k words) -> references(unlimited)
2. **500-line Rule**: Split to references when SKILL.md exceeds 500 lines
3. **No Duplication**: Don't repeat content between SKILL.md and references
4. **Reference Guidance**: SKILL.md must specify when each reference file should be read
5. **Conciseness**: Claude is already smart. Provide only essential information.

---

## Dependency Graph Design

### Skill -> Agent -> Command Dependency Direction

```
Skill (knowledge)
  ^ referenced by
Agent (work)
  ^ delegated by
Command (orchestration)
  ^ invoked by
User
```

### Inter-component Dependencies

Pipeline components follow this dependency pattern:

```
Skills are independent (depend on nothing)
  -> Implement first

Agents depend on Skills
  -> Implement after Skills complete

Commands depend on Agents
  -> Implement after Agents complete
```

### Intra-layer Dependencies

```
/detail --component A
          |
          v (A complete)
/detail --component B (depends on A)
          |
          v (B complete)
/detail --component C (depends on B)
```

These dependencies are specified in the build_report.md Component Status Table.
