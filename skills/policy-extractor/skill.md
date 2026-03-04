---
name: policy-extractor
description: "Extract business policies from code or planning docs (기획서 PDF) and output in standardized policy table format. Use when: (1) analyzing a repo's business logic to document policies, (2) extracting policy items from planning spec PDFs, (3) creating initial policy documents for a service domain. Triggers: 'policy extract', 'extract policies', 'extract business rules', 'code to policy', 'spec to policy', 'policy from code'."
---

# Policy Extractor

Extract business policies from source code or planning documents into a standardized table format compatible with Linkareer's policy document system.

**IMPORTANT**: Always read the `CLAUDE.md` in the policy document directory first before generating output. That file is the authoritative format guide. Policy documents are **non-developer-facing** — written for PMs, planners, QA, and business stakeholders.

## Output Format Rules

### Document Structure

```markdown
# {서비스명} {도메인} 정책서

<aside>
아래 텍스트 클릭시 관련 정책으로 이동합니다.
찾고자 하는 특정 키워드가 있는 경우 **Ctrl + F 기능**을 통해 키워드 입력
</aside>

[섹션명 1](#섹션명-1)
[섹션명 2](#섹션명-2)

---

### 섹션명 1

| NO | **항목** | **내용** | **최근 수정일** | **최근 수정자** | **비고** |
| --- | --- | --- | --- | --- | --- |
| 1 | {정책명} | {상세 내용} | {YY.MM.DD} | {작성자} | {참고 링크} |
```

### Key Rules

- **Section headers**: Use `###` (h3). No numbered prefixes like `## 1. 섹션명`
- **Navigation**: Always include `<aside>` block with clickable anchor links at the top
- **Sections by page/feature**: Group by user-facing page or feature, NOT by technical module
  - Good: `회원정보 입력`, `약관 동의`, `가입 처리`
  - Bad: `GraphQL API 명세`, `에러 코드 목록`, `DB 스키마`

### Content (내용) Column Writing Rules

- **Write for non-developers**: No code snippets, no function names, no file paths
- **Use plain Korean**: Say "6자리 인증번호" not "6-digit random code via `Math.floor()`"
- **Include exact values**: "최대 10자", "유효시간 3분", "최대 2MB"
- **Quote user-facing messages**: Use backticks for messages: `"이미 가입된 회원입니다."`
- **Source references go in 비고 only**: Code file paths, Slack links belong in 비고 column

### Column Reference

| Column | Description | Example |
|--------|-------------|---------|
| NO | Sequential within section | `1`, `2`, `3` |
| 항목 | Concise policy name in Korean | `닉네임 입력 규칙` |
| 내용 | Non-technical description with exact values and user-facing messages | See rules above |
| 최근 수정일 | `YY.MM.DD` format | `26.03.04` |
| 최근 수정자 | `AI 추출` for auto-extracted | `김민정`, `AI 추출` |
| 비고 | Slack links, code paths, Jira tickets | `signup.service.ts` |

## Extraction Modes

### Mode A: Code-Based Extraction

Spawn `oh-my-claudecode:analyst` (opus) per target repo to extract policies from:

1. **Validation logic** — DTO validators, class-validator decorators, custom validation pipes
   - Look for: `@IsNotEmpty()`, `@Matches()`, `@MaxLength()`, custom validators
   - Extract: field constraints, format rules, required/optional

2. **Guard/Middleware** — access control, auth rules
   - Look for: `@UseGuards()`, `canActivate()`, role checks
   - Extract: who can access what, permission hierarchy

3. **Business service logic** — core domain rules
   - Look for: conditional throws, if/else business decisions, state transitions
   - Extract: business rules, exception cases, flow logic

4. **DB schema constraints** — Prisma/Sequelize models
   - Look for: `@unique`, `@@unique`, `@default`, relations, enums
   - Extract: uniqueness rules, default values, data relationships

5. **Error messages** — user-facing error strings
   - Look for: `throw new`, `HttpException`, error message constants
   - Extract: exception conditions and user-facing messages

6. **Constants/Config** — magic numbers, thresholds, limits
   - Look for: `const`, enum values, config files
   - Extract: limits, timeouts, thresholds, feature flags

#### Code Extraction Prompt Template

```
Task(subagent_type="oh-my-claudecode:analyst", model="opus", prompt="""
Analyze the {repo-name} repo at {repo-path} for business policies related to [{domain}].

Focus on these directories/files:
- src/{module}/**/*.service.ts (business logic)
- src/{module}/**/*.guard.ts (access control)
- src/{module}/**/*.dto.ts (validation rules)
- prisma/schema.prisma (data constraints)
- src/{module}/**/*.resolver.ts (API rules)

For EACH policy found, extract:
1. Policy name (concise Korean description)
2. Detailed content (exact conditions, values, error messages)
3. Source file and line number
4. Whether it's explicitly documented or implicit in code

Save results to {output-path}/_policy-extract-raw.md in table format.
""")
```

### Mode B: Planning Doc (PDF) Extraction

Spawn `oh-my-claudecode:analyst` (opus) — **never read PDF in orchestrator context**.

```
Task(subagent_type="oh-my-claudecode:analyst", model="opus", prompt="""
Read the planning doc PDF at {pdf-path}.
Extract ALL policy-like statements — rules, constraints, conditions, exceptions, flows.

Policy indicators to look for:
- Conditional statements: "~인 경우", "~할 수 없다", "~만 가능"
- Numeric thresholds: counts, limits, durations
- State transitions: "~에서 ~로 변경", "~시 ~처리"
- Access control: "~만 접근 가능", "~권한 필요"
- Exception handling: "예외적으로", "단, ~인 경우"
- Display rules: "노출/미노출", "활성화/비활성화"

For EACH policy found, record:
1. 항목 (policy name)
2. 내용 (detailed description with exact wording from spec)
3. Page reference in PDF
4. Related feature/screen

Save results to {output-path}/_policy-extract-raw.md in table format.
""")
```

### Mode C: Hybrid (Code + Planning Doc)

Run Mode A and Mode B in parallel, then merge:
1. Match policies by domain/feature
2. Flag discrepancies between code and spec
3. Prefer code as source of truth for current behavior, spec for intended behavior

## Workflow

### Step 1: Identify Target

Ask user for:
- **Source type**: Code repo, Planning doc PDF, or Both
- **Domain/Page**: Which feature area to focus on (e.g., "기업회원 가입")
- **Repo path**: If code-based, which repo(s)
- **PDF path**: If spec-based, file location

### Step 2: Extract

Run appropriate extraction mode (A, B, or C).

### Step 3: Structure by Page/Feature

Organize extracted policies into sections by page/feature. Read the policy directory's `CLAUDE.md` for the canonical format reference.

### Step 4: Human Review (Checkpoint)

Present extracted policies and ask:
- "Approved — save to policy document"
- "I've made edits — re-read and save"
- "Need more context — analyze additional files"

### Step 5: Save

Save to the appropriate subdirectory under the policy document directory (e.g., `business-docs/{domain}.md`)

## Quality Criteria

- [ ] Document starts with `<aside>` navigation block
- [ ] Section headers use `###` (h3), no numbered prefixes
- [ ] Content column is non-technical (no code, no file paths, no regex)
- [ ] All policies have concrete values (no vague "적절한", "일정 기간")
- [ ] User-facing messages quoted exactly in backticks
- [ ] Source references in 비고 column only
- [ ] Policies grouped by page/feature, not by technical module
- [ ] NO numbering is sequential within each section
- [ ] Date format is YY.MM.DD
- [ ] Code inconsistencies flagged with ⚠️ in 비고 column
