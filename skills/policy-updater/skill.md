---
name: policy-updater
description: "Merge newly extracted or manually provided policy items into an existing policy document. Handles new/modified/conflict classification, change history tracking, and sequential numbering. Use when: (1) updating policy docs with new planning spec content, (2) merging code-extracted policies into existing docs, (3) reconciling policy changes after feature updates. Triggers: 'policy update', 'update policy', 'merge policy', 'policy sync', 'add to policy doc'."
---

# Policy Updater

Merge new policy items into existing Linkareer policy documents while maintaining consistency, tracking changes, and handling conflicts.

## Input Requirements

1. **Existing policy document** — path to current `.md` file (e.g., `business-docs/biz-signup.md`)
2. **New policy items** — from one of:
   - `policy-extractor` output file (`_policy-extract-raw.md`)
   - Planning doc PDF (delegates to `policy-extractor` Mode B first)
   - Manual user input

## Merge Workflow

### Step 1: Load and Parse

Read existing policy document. Parse each section's table into structured items:
- Section name (### heading)
- Items with: NO, 항목, 내용, 최근 수정일, 최근 수정자, 비고

### Step 2: Classify Changes

Compare new items against existing items. Match by **항목** name (fuzzy match for similar names).

Classification:

| Tag | Meaning | Action |
|-----|---------|--------|
| `[NEW]` | No matching 항목 in existing doc | Append to appropriate section |
| `[MODIFIED]` | Same 항목, different 내용 | Show diff, ask user which to keep |
| `[UNCHANGED]` | Same 항목, same 내용 | Skip |
| `[CONFLICT]` | Contradicts existing policy | Flag for human review with both versions |
| `[MOVED]` | Same policy, different section | Ask user which section |

### Step 3: Present Change Summary

```markdown
## Policy Update Summary

### New Items ([NEW]) — {n} items
| Section | 항목 | 내용 (preview) |
| ... |

### Modified Items ([MODIFIED]) — {n} items
| Section | 항목 | Before | After |
| ... |

### Conflicts ([CONFLICT]) — {n} items
| Section | 항목 | Existing | New | Source |
| ... |

### Unchanged — {n} items (skipped)
```

### Step 4: Human Review (Checkpoint)

Ask user to resolve each category:

For `[NEW]` items:
- "Add all" / "Select which to add" / "Skip all"

For `[MODIFIED]` items:
- "Keep existing" / "Use new" / "Merge manually" (per item)

For `[CONFLICT]` items:
- Must resolve each individually — no bulk action

### Step 5: Apply Changes

After user approval:

1. **Insert `[NEW]` items** into appropriate section
   - Auto-increment NO within each section
   - Set 최근 수정일 to today (YY.MM.DD)
   - Set 최근 수정자 from user input or "AI 추출"
   - Set 비고 to source reference

2. **Update `[MODIFIED]` items** in place
   - Update 내용 with new content
   - Update 최근 수정일 to today
   - Update 최근 수정자
   - Append previous version note to 비고 if significant change

3. **Renumber** all NO values sequentially within each section

4. **Update table of contents** (top-level links) if new sections added

### Step 6: Save and Report

Save updated document and output change report:

```markdown
## Update Complete

- File: {path}
- Added: {n} new items
- Modified: {n} items
- Sections affected: {list}
- Date: {today}
```

## Document Structure Template

When creating a new policy document (no existing file), use this template:

```markdown
# {서비스명} 정책서

<aside>
아래 텍스트 클릭시 관련 정책으로 이동합니다.
찾고자 하는 특정 키워드가 있는 경우 **Ctrl + F 기능**을 통해 키워드 입력
</aside>

[{섹션1 이름}](#{anchor})

[{섹션2 이름}](#{anchor})

---

### {섹션1 이름}

| NO | **항목** | **내용** | **최근 수정일** | **최근 수정자** | **비고** |
| --- | --- | --- | --- | --- | --- |

### {섹션2 이름}

| NO | **항목** | **내용** | **최근 수정일** | **최근 수정자** | **비고** |
| --- | --- | --- | --- | --- | --- |
```

## Automated Pipeline (Future)

For automated updates from new planning specs:

```
Planning Doc PDF
    ↓ (policy-extractor Mode B)
_policy-extract-raw.md
    ↓ (policy-updater)
Updated policy document
    ↓ (human review checkpoint)
Approved policy document
```

Invoke via: `policy-extractor` → `policy-updater` sequentially.

## Quality Criteria

- [ ] NO numbering is sequential (no gaps) within each section
- [ ] 최근 수정일 updated for all changed items
- [ ] No duplicate 항목 within the same section
- [ ] Table of contents matches actual sections
- [ ] Change summary presented before applying changes
- [ ] Conflicts explicitly resolved (never auto-merged)
