---
name: request-clarifier
description: Ensures task requests have sufficient context before execution. When a user request is ambiguous or missing critical information, guides Claude to ask clarifying questions instead of guessing. Apply to any task execution — both direct user requests and subagent delegation.
---

# Request Clarifier

## Core Principle

> **Never guess intent. Clarify before executing.**

A wrong assumption costs more than a 10-second question. When critical information is missing, ask — don't infer.

## When to Clarify

Before starting any non-trivial task, mentally check these 5 fields:

| Field | Question | Required? |
|-------|----------|-----------|
| **Goal** | What is the desired end state? | Always |
| **Scope** | What files/areas are affected? | When ambiguous |
| **Constraints** | What must NOT change? | When risk of side effects |
| **Context** | Why is this needed? | When it affects approach |
| **Output** | What does "done" look like? | When deliverable is unclear |

### Decision Flow

```
Is the Goal clear?
├── No → Ask before anything else
└── Yes → Is the Scope clear?
    ├── No → Ask (prevents unintended changes)
    └── Yes → Can I infer Constraints/Context/Output?
        ├── Yes → Proceed
        └── No → Ask only what I can't infer
```

## When NOT to Clarify

Don't slow down the user with unnecessary questions:

- **Simple tasks**: "fix the typo in README.md" — Goal, Scope, Output all clear
- **Explicit instructions**: User already provided all 5 fields
- **Follow-up tasks**: Context carries over from conversation
- **Exploratory requests**: "what does this code do?" — just answer

## How to Clarify

### Good Clarification

Specific, actionable, shows what you already understood:

```
Goal is clear (add user authentication), but I need to clarify:
- Which auth method? (JWT / session-based / OAuth)
- Should it integrate with the existing User model in src/models/user.ts?
```

### Bad Clarification

Vague, dumps all questions at once, doesn't show understanding:

```
Can you provide more details about what you want?
What are the requirements?
What should I do?
```

### Rules

1. **Show what you understood** before asking what's unclear
2. **Ask max 2-3 questions** per round — prioritize by impact
3. **Suggest defaults** when possible: "JWT with RS256? (or do you prefer session-based?)"
4. **Never re-ask** what the user already provided
5. **Batch related questions** into a single ask

## For Subagent Delegation

When a command delegates to a subagent, apply the same principle:

- Ensure the delegation prompt includes Goal, Scope, and Output path
- If predecessor artifacts exist, pass file paths (not content) as context
- Missing context in delegation = subagent guessing = poor output

See `references/delegation-checklist.md` for the subagent delegation template.
