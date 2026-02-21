# Artifact Patterns — Artifact Structure Standard

## Artifact Structure Template

Every pipeline artifact must include these 5 sections:

```markdown
# [Artifact Title]

## Context
- Pipeline ID: {pipeline-id}
- Phase: [draft|finalize|build|detail|detail-apply]
- Component: [component name if applicable, omit otherwise]
- Created: [YYYY-MM-DD]
- Target Project: [absolute path to target project]

## Prerequisites
- [x] {previous artifact filename} exists
- [x] {previous artifact} Todo all completed
- [ ] (show unmet items here)

## Content
(Phase-specific content — see Phase Content Guide below)

## Todo
- [x] Completed work items
- [ ] Items requiring Human confirmation
- [ ] Not-yet-completed items

## Next
- Next step: /[next command] {args}
- Required: All Todo items above completed + [additional conditions]
```

---

## Todo-based Gate Rules

### Gate Verification Process

When a command executes, it first parses the previous artifact's Todo:

```
1. Verify previous artifact file exists
2. Find ## Todo section
3. Search for `- [ ]` pattern (incomplete items)
4. If incomplete items found:
   -> Block execution
   -> Display: "The following items are incomplete:" + list
   -> Guide: "Complete these items and run again"
5. If all items are `- [x]`:
   -> Gate passes, proceed to next step
```

### Todo Writing Guide

**Good Todo:**
```markdown
## Todo
- [x] Process decomposed into 5 steps
- [x] Automation feasibility assessed for each step
- [ ] Human checkpoint locations (3 places) — needs user confirmation
- [ ] spec-analyzer agent model choice — decide haiku vs sonnet
```

**Bad Todo:**
```markdown
## Todo
- [ ] Needs review     <- Review what? Too vague
- [x] Done             <- What's done? Not specific
```

**Rules:**
- Each item must be specific and verifiable
- Items requiring Human confirmation must explain why
- Distinguish auto-completed items from Human confirmation items

---

## Phase Content Guide

### pipeline_draft.md

```markdown
## Content

### Process Analysis
| Step | Description | Automatable | Human Checkpoint |
|------|-------------|-------------|-----------------|

### Component Design
| Type | Name | Role | Dependencies |
|------|------|------|-------------|

### Artifact Flow
(Diagram showing which artifacts are produced where and passed where)

### Workspace Structure
(Directory structure design)

### Design Principles Check
(How each of the 6 principles is applied)
```

### pipeline_final.md

```markdown
## Content

### Confirmed Components
(With approved/modified/deferred status)

### Implementation Order
| Order | Component | Type | Depends On |
|-------|-----------|------|------------|

### Changes from Draft
(Items changed during discussion)
```

### build_report.md

```markdown
## Content

### Scaffolded Files
| File | Type | Location |
|------|------|----------|

### Component Status
| # | Component | Type | Status | Depends On |
|---|-----------|------|--------|------------|
| 1 | xxx | Skill | scaffolded | - |
| 2 | yyy | Agent | scaffolded | xxx |

(Status: scaffolded -> draft -> detailed)
```

### details/{component}_draft.md

```markdown
## Content

### Component Info
- Type: [Command|Agent|Skill]
- Location: [file path]
- Role: [role description]

### Proposed Content
(Draft of what the actual file should contain)
- Frontmatter structure
- Key sections
- Execution flow (for Command)
- System prompt outline (for Agent)
- Knowledge scope (for Skill)

### Design Decisions
(Rationale for design choices)

### Questions for Human
(Items to confirm with human)
```

### details/{component}.md (final)

```markdown
## Content

### Implementation Summary
- File: [created/modified file path]
- Lines: [approximate line count]

### Changes from Draft
(Items changed based on feedback)

### Key Content
(Summary of written content)
```

---

## Workspace Directory Structure

```
~/.claude/workspace/pipelines/{pipeline-id}/
├── _process-analysis.md            # pipeline-analyzer internal artifact
├── _component-design.md            # pipeline-architect internal artifact
├── pipeline_draft.md               # /draft final artifact
├── pipeline_discussion.md          # Human-written (discussion notes)
├── pipeline_final.md               # /finalize final artifact
├── build_report.md                 # /build final artifact
└── details/                        # /detail + /detail-apply artifacts
    ├── {component}_draft.md        # Design draft (awaiting Human feedback)
    ├── {component}.md              # Final report (implementation complete)
    └── ...
```

**Naming conventions:**
- `_` prefix: Internal intermediate artifacts (not directly referenced by next step)
- No prefix: External artifacts (input for next step)
- `_draft` suffix: Draft awaiting Human feedback
