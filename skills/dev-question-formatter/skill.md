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

### 2.1 Follow-Up Question Flow

All deep-dive questions MUST follow this 4-step flow:

```
Judgment Basis → Alternatives → Risk → Results
```

| Step | Pattern | Example |
|------|---------|---------|
| 1. Judgment Basis | "Why did you choose X?" | "왜 그 방식을 선택했나요?" |
| 2. Alternatives | "What else did you consider?" | "다른 대안은 검토했나요?" |
| 3. Risk | "What could go wrong?" | "도입 후 발생한 부작용은 없었나요?" |
| 4. Results | "How did you measure?" | "개선 효과를 어떻게 수치로 증명했나요?" |

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
- **빌드업 질문 (Buildup Question):** [easy entry point question]
- **핵심 질문 (Deep Dive Question):**
  - [probing question 1]
  - [probing question 2]
  - [probing question 3]
- **평가 기준 (Evaluation Criteria):**
  - **[Hygiene Check]:** Pass — [OK signal] / Fail — [Red Flag signal]
  - **[Core Competency]:** Plus — [depth indicator] / Minus — [shallow indicator]
  - **[Advanced Insight]:** Plus — [senior insight] / Minus — [limited perspective]
```

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
| 7 | Timing feasibility | Total ~30 min | Sum timing guide, verify 28-35 min range |

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
