---
name: self-verification
description: Self-check skill for verifying that work results match the user's intent. Apply after completing any non-trivial task — code changes, file generation, subagent delegation, or multi-step work. Catches misalignment, unintended changes, and missing requirements before reporting to the user.
---

# Self-Verification

## Core Principle

> **Verify before reporting. Never assume your output is correct.**

After completing work, check that the result matches what was asked — not just that something was produced.

## When to Verify

| Task Complexity | Verification Level |
|----------------|-------------------|
| **Trivial** (typo fix, rename) | Skip — just confirm the change |
| **Simple** (single file edit, add function) | Quick check — re-read changed lines |
| **Medium** (multi-file change, new feature) | Standard — run full checklist |
| **Complex** (architecture change, pipeline work) | Thorough — checklist + test if possible |

## Verification Checklist

After completing work, check each applicable item:

### 1. Goal Alignment
- [ ] Re-read the user's original request
- [ ] Does the result achieve what was asked?
- [ ] Are there any requirements that were mentioned but not addressed?

### 2. Scope Check
- [ ] List all files modified/created
- [ ] Were any files changed that shouldn't have been?
- [ ] Were any unrelated changes introduced?

### 3. Content Verification
- [ ] Re-read the key sections of output (don't trust memory)
- [ ] Is the content complete, not truncated?
- [ ] For code: are imports, types, and exports correct?
- [ ] For documents: does the structure match the expected format?

### 4. Constraint Compliance
- [ ] Were any stated constraints violated?
- [ ] Were any existing patterns/conventions broken?
- [ ] For code: does it maintain backward compatibility?

### 5. Side Effect Check
- [ ] Were any existing files overwritten unintentionally?
- [ ] Were any dependencies or related files broken?
- [ ] For code: run tests/lint if available and relevant

## Failure Handling

| Severity | Indicator | Action |
|----------|-----------|--------|
| **Critical** | Result contradicts the goal | Fix before reporting. If unfixable, explain what went wrong |
| **Warning** | Minor misalignment or potential issue | Report to user with the finding and ask for guidance |
| **Info** | Cosmetic or style observation | Mention briefly, don't block completion |

## Reporting Format

When reporting completed work to the user, include a brief verification summary:

```
[Work summary — what was done]

Verified:
- {N} files modified: {file list}
- Goal alignment: {confirmed / partial — explain}
- No unintended changes detected
```

If issues were found:

```
[Work summary]

Verified with findings:
- {what was checked}
- ⚠️ {issue found and what was done about it}
```

Keep it concise. The verification summary should be 2-4 lines, not a full report.

## For Subagent Results

When receiving results from a subagent, verify:
- [ ] Output file exists at the expected path
- [ ] File is not empty
- [ ] Key content matches what was delegated (spot-check, not full review)
- [ ] Status indicates success

See `references/verification-by-type.md` for type-specific checklists.
