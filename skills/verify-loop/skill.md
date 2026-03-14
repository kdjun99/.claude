---
name: verify-loop
description: "Implement with verification plan, then verify-fix-reverify loop until all checks pass. Use when building features, fixing bugs, or any non-trivial code change."
---

# Verify Loop

Plan, implement, verify, fix, repeat — until all verifications pass.

## When to Use

- "/verify-loop" or "검증 루프"
- Any non-trivial implementation task where correctness matters
- "구현하고 검증해줘", "구현 후 검증까지", "모든 검증 통과할 때까지"

## Protocol

### Phase 1: Plan (before writing code)

1. **Implementation Steps** — break the task into ordered steps
2. **Verification Plan** — for each step, define how to verify it:

| Verification Type | Method |
|------------------|--------|
| Build | `npm run build`, `tsc --noEmit`, language-specific build |
| Test | run existing + new tests |
| Lint | `eslint`, `prettier`, project linter |
| Runtime | manual or script-based smoke test |
| Logic | re-read code against requirements |

Output a brief plan before starting:
```
## Implementation Plan
1. {step} → verify: {method}
2. {step} → verify: {method}
...

## Verification Checklist
- [ ] {check 1}
- [ ] {check 2}
...
```

### Phase 2: Implement

Execute the implementation steps in order. No shortcuts.

### Phase 3: Verify-Fix Loop (max 10 iterations)

```
iteration = 0
WHILE iteration < 10:
  1. Run ALL verification checks from the plan
  2. Collect results:
     - PASS items → mark done
     - FAIL items → collect errors
  3. IF all PASS → break, report success
  4. IF any FAIL:
     a. Analyze root cause of each failure
     b. Fix the issues (prefer minimal, targeted fixes)
     c. iteration++
     d. Continue loop (re-verify ALL checks, not just failed ones)

IF iteration == 10 → stop and report remaining failures to user
```

Rules:
- **Re-verify everything** each iteration, not just previously failed checks (fixes can introduce regressions)
- **Minimal fixes only** — don't refactor or improve unrelated code during fix iterations
- **Track iteration count** — report which iteration you're on
- **Root cause first** — understand WHY it failed before fixing

### Phase 4: Report

```
## Verification Complete (iteration {N}/{max})

### Results
- {check 1}: PASS
- {check 2}: PASS
- ...

### Changes Made
- {file}: {what changed}

### Fix History (if any)
- Iteration 1: {what failed} → {what was fixed}
- Iteration 2: {what failed} → {what was fixed}
```

## Verification Type Reference

Pick verification methods appropriate to the task:

| Task Type | Recommended Verifications |
|-----------|--------------------------|
| Feature implementation | build + test + lint + logic review |
| Bug fix | build + test (regression) + reproduce-check |
| Refactor | build + test + no behavior change |
| Config/infra change | build + smoke test |
| Documentation | link check + accuracy review |

## Integration

- Works standalone or combined with other skills (e.g., `feature-implementation`)
- Compatible with `ralph` for persistent execution
- Complements `self-verification` (which runs post-completion; this skill runs during implementation)
