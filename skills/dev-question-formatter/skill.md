---
description: "Interview question execution rules from AGENTS.md: document analysis procedure (summary card -> project inventory -> checkpoint), question design rules (follow-up flow, tech probing, issue investigation), quality checklist enforcement, and output format standards. Apply when generating or verifying interview question documents."
---

# Question Formatter

Tactical execution rules for producing interview question documents. This skill defines the PROCESS — what to do before, during, and after question generation. All agents in the interview-question-generator pipeline MUST follow these rules.

---

## 1. Document Analysis Procedure (MUST execute before question generation)

### 1.1 Candidate Summary Card (Step 1 — Required)

Create a 5-line summary BEFORE generating any questions:

```markdown
## Candidate Profile Summary
- **Career Level:** [Junior/Senior] ([education], [program/company])
- **Keywords:** [top 5-7 technical keywords]
- **Main Projects:** [project names, max 3]
- **Analysis Focus:** [potential/attitude/grit/fundamentals OR trade-off/impact/leadership/crisis]
- **Learning Signal:** [blog posts count, certifications, GitHub activity, awards]
```

**Rules:**
- Career level determines ALL subsequent question strategy
- Learning signal MUST be included (blog, GitHub, certs, awards)
- Keywords extracted from actual project work, not just listed tech stack

### 1.2 Project Inventory Table (Step 2 — Required)

Structure each project as a table/list with these 6 fields:

| Field | Description | Required |
|-------|-------------|----------|
| Project Name / Duration / Team Size | Basic context | Yes |
| Core Features | What the project does | Yes |
| Tech Stack | Technologies actually used | Yes |
| Personal Contribution | Percentage or specific scope | Yes |
| Key Problems | Performance/failure/consistency/cost issues encountered | Yes |
| Quantitative Metrics | Response time, cost, traffic, data scale | If available |

**Rules:**
- If quantitative metrics are missing, flag for clarification question
- If 5+ projects exist, select top 2-3 by impact for 30-minute interview
- Record contribution as specific scope, not just percentages

### 1.3 Pre-Generation Checkpoint (Step 3 — Required)

Before generating questions, verify:

| Check | Action if Failed |
|-------|-----------------|
| Ambiguous feature names or metrics | Ask clarification question to user |
| Too many projects (> 3) | Ask user which 2-3 to prioritize |
| No quantitative metrics anywhere | Design at least 1 question to elicit metrics |
| Career level unclear | Ask user to confirm junior vs senior focus |

**Rule:** Never generate questions with incomplete or ambiguous input data. Stop and ask.

---

## 2. Question Design Rules

### 2.1 Follow-Up Chain Design (꼬리 질문 체인)

All deep-dive questions MUST be designed as **pressure funnels** using the `interview-pressure-probing` skill. Each question needs a multi-layer follow-up chain with conditional branches.

**Key rules (detail in `interview-pressure-probing` skill):**
- Minimum depth: **3 layers** per question (L1→L2→L3). L4-L5 = interviewer discretion.
- Must include **conditional branches** (concrete answer → go deeper / vague answer → redirect)
- Must include **at least 1 trap point** per project section
- The previous 4-step lateral flow (Judgment → Alternatives → Risk → Results) is now embedded as **probe patterns within each layer**, not as the primary structure. The PRIMARY structure is the **vertical depth chain (L1→L2→L3+)**.

### 2.2 Tech Choice Probing

For EVERY technology adoption mentioned in portfolio:

| Probe | Question Pattern |
|-------|-----------------|
| Why this tech? | "왜 [기술]을 선택했나요?" |
| Why not alternatives? | "[대안A]와 비교했을 때 결정적 근거는?" |
| Trade-off awareness | "도입으로 인한 단점이나 사이드 이펙트는?" |

**Rule:** Technology choices MUST be probed with Why / Why Not / Trade-off. Never accept "because it's popular" as sufficient.

### 2.3 Issue Investigation Probing

For projects with performance/failure/cost issues:

| Phase | Question Pattern |
|-------|-----------------|
| Reproduce | "문제를 어떻게 재현했나요?" |
| Measure | "병목 지점을 어떻게 측정/진단했나요?" |
| Mitigate | "해결 방안은 무엇이었고, 어떻게 적용했나요?" |
| Prevent | "재발 방지를 위해 어떤 체계를 구축했나요?" |

### 2.4 Security/Consistency Probing

For claims about security or data consistency:

| Claim Type | Verification Approach |
|-----------|----------------------|
| "보안 처리 완료" | Present threat scenario and ask how they'd handle it |
| "정합성 문제 없음" | Present edge case (concurrent access, network partition) |
| "인증/인가 구현" | Ask about token theft, session hijacking scenarios |

### 2.5 Team Contribution Probing

| Claim Pattern | Probe Strategy |
|--------------|----------------|
| "본인이 전담" | Ask for specific technical difficulty solved |
| Percentage claims | Ask what tasks fell in each percentage |
| Vague "팀으로 진행" | Ask for exact personal/team boundary |

**Rule:** Always probe contribution by asking about specific DIFFICULTIES, not general responsibilities.

---

## 3. Output Format Rules

### 3.1 Document Structure (mandatory order)

```
1. Candidate Profile Summary (from Step 1)
2. Ice Breaking (2-4 questions)
3. Project A Section (3-6 questions per project)
4. Project B Section
5. [Optional] Common Competency Section
6. Interview Timing Guide Table
7. Quality Checklist
```

### 3.2 Grouping Rule

- ALL questions grouped by project — no exceptions
- Verification questions belong INSIDE project sections
- No separate "additional" or "verification" section at document end

### 3.3 Per-Question Template

```markdown
### [Symbolic Korean Title]
- **연관 기능 (Related Feature):** [specific feature/achievement]
- **빌드업 질문 (Buildup Question):** [= L1 Surface — easy entry point]
- **꼬리 질문 체인 (Follow-up Chain):**
  - **L2 (구체화):** [specifics probe]
    - → 구체적 답변 시: [deeper follow-up]
    - → 모호한 답변 시: [redirect to verifiable aspect]
  - **L3 (교차검증):** [cross-check with other claims or logical implications]
  - **L4 (깊이):** [edge case / internal mechanism — interviewer discretion]
- **함정 포인트 (Trap Point):** [what inconsistency to watch for + how to probe it]
- **평가 기준 (Evaluation Criteria):**
  - **[Hygiene Check]:** Pass — [OK signal] / Fail — [Red Flag signal]
  - **[Core Competency]:** Plus — [depth indicator] / Minus — [shallow indicator]
  - **[Advanced Insight]:** Plus — [senior insight] / Minus — [limited perspective]
```

**Template Rules:**
- `빌드업 질문` = L1 (deliberately easy, builds false comfort)
- `꼬리 질문 체인` replaces the old `핵심 질문` — must have ≥3 layers with conditional branches
- `함정 포인트` = pre-designed inconsistency detector (at least 1 per project section, can be shared across questions)
- See `interview-pressure-probing` skill Section 6 for full format specification

### 3.4 Timing Guide Template

| Section | Duration | Notes |
|---------|----------|-------|
| Ice Breaking | 3-5 min | Rapport |
| Project A | 15-20 min | Primary deep dive |
| Project B | 10-15 min | Secondary |
| Common/Wrap-up | 5-10 min | Collaboration, learning |
| **Total** | **~30 min** | |

---

## 4. Quality Checklist (MUST pass before delivery)

Every interview document MUST satisfy ALL of the following:

| # | Criterion | Threshold | Check Method |
|---|-----------|-----------|-------------|
| 1 | Ice breaking questions | 2-4 included | Count questions in ice breaking section |
| 2 | Questions per project | 3-6 per project section | Count questions per project heading |
| 3 | Perspective coverage | Min 4 of 6 per project | Tag each question's perspective, verify count |
| 4 | Quantitative metric question | At least 1 per document | Search for measurement/number/performance questions |
| 5 | Collaboration question | At least 1 per document | Search for team/contribution/role boundary questions |
| 6 | Project-based grouping | All questions in project sections | Verify no orphan "verification" or "additional" sections |
| 7 | Timing guide | Included (advisory) | Verify timing guide table exists (not a hard constraint) |
| 8 | Follow-up chain depth | ≥3 layers per question | Each question has L1(빌드업) + L2 + L3 minimum in 꼬리 질문 체인 |
| 9 | Conditional branches | ≥1 branch per chain | Check for "구체적 답변 시" / "모호한 답변 시" patterns |
| 10 | Trap points | ≥1 per project section | Check 함정 포인트 field exists in at least 1 question per project |

### Failure Protocol

If any criterion fails:
1. Identify which criterion failed
2. Report the gap (e.g., "Project B has only 2 perspectives covered")
3. Suggest specific fix (e.g., "Add a Why Not/Trade-off question about X")
4. Do NOT deliver the document until all criteria pass

---

## 5. Response Principles

- When asked to "review/improve/write" interview questions, apply ALL above rules
- Keep questions per project to 3-6 — do not over-generate
- If project info is ambiguous, ask for clarification FIRST before generating
- Compress questions around the most impactful technical decisions
- Every question must serve a purpose — no filler questions
