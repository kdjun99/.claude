# Core Principles — 6 Pipeline Design Principles

## 1. Human-in-the-Loop

Always require human confirmation before irreversible actions (push, PR creation, file deletion, external system changes).

**When to require confirmation:**

| Action Type | Confirmation |
|-------------|-------------|
| Local file create/modify | Optional (confirm-then-proceed) |
| Git commit | Confirm-then-proceed |
| Git push / PR creation | **Always required** |
| Design direction decisions | **Always required** (team discussion) |
| External API calls | **Always required** |

**Implementation:**
- Use AskUserQuestion tool
- Provide "Proceed / Edit then proceed / Cancel" options
- For design phases, apply draft -> Human review -> finalize pattern

**Violation risk:** Unintended PRs, wrong code deployed, hard-to-reverse changes

---

## 2. Artifact-Driven Handoff

Inter-step information transfer must be file-based (artifacts). Never depend on conversation context.

**Why it matters:**
- Session termination doesn't lose progress — artifact files persist
- Other people/sessions can review intermediate results and participate
- Reproducible: same input files yield same results

**Implementation:**
- Save each step's output as md files in workspace directory
- Next step starts by reading previous step's artifact files
- Internal intermediate artifacts use `_` prefix (e.g., `_analysis.md`)
- Final artifacts have no prefix (e.g., `draft.md`)

**Violation risk:** Context loss, non-reproducible results, unable to split sessions

---

## 3. Idempotent Steps

Re-running any step with the same input must be safe.

**Examples:**
- Branch creation: reuse if already exists (not an error)
- Workspace directory: skip if already exists
- Artifact files: overwrite with latest results (update)
- PR creation: update if already exists

**Implementation:**
- Check state before action (file exists, branch exists, etc.)
- Distinguish "already completed" from "not yet done"
- Operate as update rather than duplicate creation

**Violation risk:** Duplicate branches, duplicate commits, duplicate PRs

---

## 4. Fail-Safe

On failure, stopping is safer than attempting cleanup.

**Guidelines:**
- Agent execution failure -> halt and notify user
- Error during file creation -> preserve created files (don't delete)
- Gate verification failure -> don't proceed, explain which conditions are unmet

**Implementation:**
- Prevent failures through pre-verification (Gate) rather than try-catch
- On failure, preserve partial results and inform user of state
- Prefer manual judgment over automatic rollback

**Violation risk:** Cleanup attempts on half-created state causing additional problems

---

## 5. Progressive Autonomy

Start with many confirmations, expand automation scope as trust grows.

**Automation levels by maturity:**

```
Early: Human confirmation at every step
  | (pipeline stabilization)
Mid: Analysis/design automatic, only irreversible actions confirmed
  | (sufficient production experience)
Late: Mostly automatic, only final submission confirmed
```

**Implementation:**
- Provide `--auto` option to skip confirmation steps
- Default is always "confirmation required"
- User decides the automation level

---

## 6. Composable Pipeline

Each step must be independently executable, and steps must be composable into pipelines.

**Independent execution requirements:**
- Each command runs independently given required input files
- Previous step doesn't need to run in the same session
- Mid-pipeline entry possible (e.g., manually create files, then run next step)

**Composability:**
- Running the full pipeline in order is the default
- Re-running specific steps is possible when needed
- Steps from different pipelines can be combined

**Violation risk:** Full pipeline restart required on single step failure
