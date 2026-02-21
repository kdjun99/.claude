---
name: subagent-output-optimization
description: Principles and patterns for optimizing subagent outputs to minimize main context consumption. Teaches subagents to save detailed results to files and return only essential summaries. Apply when designing or implementing any subagent that returns results to a parent context.
---

# Subagent Output Optimization

## Why This Matters

Subagents execute in their own context window, but **results return to the main context**. Every byte of the return value consumes the parent's limited context window (~200k tokens shared). Verbose subagent outputs are the fastest way to exhaust the main context.

```
Main Context (200k)
├── System context (CLAUDE.md, skills, etc.)
├── Conversation history
├── Subagent A result  ← consumes main context
├── Subagent B result  ← consumes main context
└── ... accumulates with each call
```

## Core Rule

> **Save details to files. Return only what the parent needs to decide the next action.**

## The Two-Channel Pattern

Every subagent output should flow through two channels:

| Channel | What | Where | Size |
|---------|------|-------|------|
| **File** | Full detailed results | Persistent file path | Unlimited |
| **Return** | Actionable summary | Main context | Minimal |

### Return Value Should Answer

1. **Did it succeed?** — Status (success/failure/partial)
2. **What was produced?** — File path(s) to detailed results
3. **What's the key finding?** — 1-3 sentence summary of conclusions
4. **What's next?** — Blocker or next action needed

### Return Value Should NOT Contain

- Raw data, full analysis, or verbose explanations
- Content that's already saved to a file
- Intermediate reasoning or exploration logs
- Full file contents that were read or generated

## Quick Reference

For detailed patterns and examples, see:
- `references/return-patterns.md` — Templates for different subagent types
- `references/file-handoff-patterns.md` — Inter-subagent communication via files
