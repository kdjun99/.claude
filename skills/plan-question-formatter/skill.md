---
description: "Planner interview question execution rules: document analysis procedure (summary card with Domain Relevance -> project inventory with planning scope -> checkpoint), question design rules (follow-up flow adapted for planners, KPI Probing, Prioritization Probing, stakeholder conflict probing, planning ownership probing), quality checklist enforcement with KPI/Impact metric criterion, and output format standards. Apply when generating or verifying planner interview question documents."
---

# Planner Question Formatter

Tactical execution rules for producing planner (기획자) interview question documents. This skill defines the PROCESS — what to do before, during, and after question generation. All agents in the plan-interview pipeline MUST follow these rules.

---

## 1. Document Analysis Procedure (MUST execute before question generation)

### 1.1 Candidate Summary Card (Step 1 — Required)

Create a 6-line summary BEFORE generating any questions:

```markdown
## Candidate Profile Summary
- **Career Level:** [Junior/Senior] ([education], [program/company])
- **Keywords:** [top 5-7 product/domain keywords]
- **Main Projects:** [project names, max 3]
- **Analysis Focus:** [problem-sensing/user-empathy/structured-thinking/execution-speed OR strategic-thinking/cross-functional-leadership/data-driven-decisions/stakeholder-management]
- **Domain Relevance:** [relevance to 대외활동/채용/커뮤니티 플랫폼 domain — high/moderate/low with brief reason]
- **Learning Signal:** [blog posts count, certifications, side projects, awards, community activity]
```

**Rules:**
- Career level determines ALL subsequent question strategy
- Keywords extracted from actual project work, not just listed methodologies (e.g., "B2B SaaS growth", "A/B testing", "user retention optimization")
- Domain Relevance MUST be assessed — flags whether domain-fit questions are appropriate
- Learning signal MUST be included (blog, community, certifications, awards)

### 1.2 Project Inventory Table (Step 2 — Required)

Structure each project as a table/list with these 6 fields:

| Field | Description | Required |
|-------|-------------|----------|
| Project Name / Duration / Team Composition | Basic context including cross-functional team structure | Yes |
| Project Type | User-facing feature / Backoffice-admin tool / Hybrid | Yes |
| Product Goal & Hypothesis | What problem was being solved, for whom, what was the hypothesis | Yes |
| Methodologies | User research, A/B testing, prototyping, PRD, wireframe, data analysis, etc. | Yes |
| Personal Planning Scope | Specific scope of planning responsibility ("요구사항 정의 100%, 이해관계자 조율 70%") | Yes |
| Key Challenges | Stakeholder conflicts, scope changes, unclear requirements, data gaps, competing priorities | Yes |
| Business/User Impact Metrics | Conversion rate, retention, NPS, revenue, user adoption, engagement | If available |

**Rules:**
- If business/user impact metrics are missing, flag for clarification question
- If 5+ projects exist, select top 2-3 by business impact for 30-minute interview
- Record planning scope as specific responsibilities, not just percentages
- Note which methodologies were personally conducted vs team-led

### 1.3 Pre-Generation Checkpoint (Step 3 — Required)

Before generating questions, verify:

| Check | Action if Failed |
|-------|-----------------|
| Ambiguous product goals or impact claims | Ask clarification question to user |
| Too many projects (> 3) | Ask user which 2-3 to prioritize |
| No business/user impact metrics anywhere | Design at least 1 question to elicit impact |
| No KPI or success criteria mentioned | Design at least 1 KPI definition question |
| Career level unclear | Ask user to confirm junior vs senior focus |
| Planning scope vague across all projects | Design questions to probe decision ownership |

**Rule:** Never generate questions with incomplete or ambiguous input data. Stop and ask.

---

## 2. Question Design Rules

### 2.1 Follow-Up Question Flow

All deep-dive questions MUST follow this 4-step flow:

```
Problem Definition → Alternatives → Stakeholder Alignment → Impact Measurement
```

| Step | Pattern | Example |
|------|---------|---------|
| 1. Problem Definition | "What problem were you solving?" | "이 기획의 배경과 해결하려는 문제는 무엇이었나요?" |
| 2. Alternatives | "What else did you consider?" | "다른 접근 방식도 고려하셨나요? 왜 이 방향을 선택했나요?" |
| 3. Stakeholder Alignment | "How did you align stakeholders?" | "관련 이해관계자들과 어떻게 합의를 이루셨나요?" |
| 4. Impact Measurement | "How did you measure success?" | "기획의 성과를 어떻게 측정했고, 결과는 어땠나요?" |

### 2.2 Product Decision Probing

For EVERY significant product decision mentioned in portfolio:

| Probe | Question Pattern |
|-------|-----------------|
| Why this approach? | "왜 이 방향으로 기획했나요? 어떤 근거로 결정했나요?" |
| Why not alternatives? | "다른 접근법과 비교했을 때 결정적 근거는?" |
| Trade-off awareness | "이 방향을 선택함으로써 포기한 것은 무엇인가요?" |
| Scope rationale | "MVP 범위를 이렇게 정한 기준은 무엇인가요?" |

**Rule:** Product decisions MUST be probed with Why / Why Not / Trade-off. Never accept "because the stakeholder asked for it" as sufficient — probe the planner's own judgment.

### 2.3 KPI Probing (NEW — Signal: KPI Awareness)

For projects with measurable outcomes or success claims:

| Phase | Question Pattern |
|-------|-----------------|
| KPI Selection | "이 기획의 성공/실패를 어떤 KPI로 판단했나요? 그 KPI를 선택한 근거는?" |
| Measurement Method | "KPI를 어떻게 측정했나요? 어떤 도구/체계를 사용했나요?" |
| Target Achievement | "KPI 목표치는 어떻게 설정했고, 달성 여부는?" |
| Miss Response | "목표를 달성하지 못했을 때 어떻게 대응했나요? 기획 방향을 수정했나요?" |

**Rule:** KPI probing distinguishes planners who DEFINE success criteria from those who merely REPORT results. At minimum 1 KPI definition question per document.

### 2.4 Prioritization Probing (NEW — Signal: Project Prioritization Insight)

For candidates with multiple projects or features:

| Phase | Question Pattern |
|-------|-----------------|
| Backlog Overview | "동시에 진행 중이던 기획 건은 몇 개였나요?" |
| Criteria | "우선순위를 정한 기준이나 프레임워크는 무엇이었나요?" |
| Deferred Items | "디퍼(defer)된 기획 건은 어떻게 관리했나요?" |
| Priority Change | "우선순위가 급변한 경험이 있나요? 어떻게 대응했나요?" |

**Rule:** For Senior candidates, prioritization probing is mandatory. For Junior, assess basic awareness of why their feature was prioritized.

### 2.5 Stakeholder Conflict Probing

For projects involving cross-functional collaboration:

| Phase | Question Pattern |
|-------|-----------------|
| Identify | "기획 과정에서 가장 큰 의견 충돌은 무엇이었나요?" |
| Understand | "각 이해관계자의 입장은 무엇이었나요?" |
| Resolve | "어떻게 합의를 이끌어내셨나요? 본인의 역할은?" |
| Outcome | "합의 결과에 만족하셨나요? 다르게 했다면?" |

### 2.6 User Validation Probing

For claims about user research or user-centered design:

| Claim Type | Verification Approach |
|-----------|----------------------|
| "사용자 조사 진행" | Ask method, sample size, key findings, how findings changed the plan |
| "A/B 테스트 진행" | Ask hypothesis, success criteria, results, decision made from results |
| "프로토타입 테스트" | Ask fidelity level, user feedback, iteration made |
| "사용자 피드백 반영" | Ask specific feedback example, how it changed the feature scope |

### 2.7 Planning Ownership Probing

| Claim Pattern | Probe Strategy |
|--------------|----------------|
| "기획을 담당했다" (vague) | Ask for specific decisions made and decisions that needed approval |
| "전체 기획 리드" | Ask about the hardest stakeholder pushback and how they handled it |
| Percentage claims ("기획 100%") | Ask what specific parts of planning they did vs what was predefined |
| Vague "팀으로 진행" | Ask for exact personal/team boundary in planning decisions |

**Rule:** Always probe planning ownership by asking about specific DECISIONS and DIFFICULTIES, not general responsibilities.

### 2.8 Problem Definition Probing

For every project, probe how the candidate framed and defined the core problem — not just identified it:

| Phase | Question Pattern |
|-------|-----------------|
| Sensing | "이 문제를 처음 인식하게 된 계기는 무엇인가요? (데이터, 사용자 피드백, 관찰 등)" |
| Framing | "그 문제를 구체적으로 어떻게 정의하셨나요? 문제의 범위와 경계는 어떻게 설정했나요?" |
| Validation | "정의한 문제가 실제 핵심 문제가 맞는지 어떻게 확인하셨나요? (사용자 검증, 데이터 분석 등)" |
| Reframing | "기획 진행 중 처음 정의한 문제와 실제 문제가 달랐던 경험이 있나요? 어떻게 재정의하셨나요?" |

**Follow-up chain (꼬리 질문):**
- Sensing → "그 신호를 다른 팀원들도 문제로 인식했나요, 본인이 먼저 제기한 건가요?"
- Framing → "여러 문제 중 이 문제를 우선 해결하기로 한 근거는 무엇인가요?"
- Validation → "문제 정의가 잘못되었다는 것을 뒤늦게 알게 된 경험이 있나요?"
- Reframing → "문제를 재정의한 후 기획 방향이 어떻게 바뀌었나요? 이해관계자들은 어떻게 설득하셨나요?"

**Rule:** Problem definition probing assesses whether the candidate can correctly FRAME problems before jumping to solutions. At minimum 1 problem definition question per document. Generic "배경이 뭔가요?" is INSUFFICIENT — must probe the framing/validation process.

### 2.9 Backoffice & Ops Probing

For projects classified as Backoffice-admin tool or Hybrid in the Project Inventory:

| Phase | Question Pattern |
|-------|-----------------|
| Bottleneck Discovery | "내부 운영팀의 가장 큰 병목이나 비효율은 무엇이었나요? 어떻게 파악하셨나요?" |
| Systematic Solution | "그 병목을 해결하기 위해 어떤 시스템/도구를 기획하셨나요? 기존 프로세스와 어떻게 달랐나요?" |
| Ops Stakeholder | "운영팀(내부 사용자)의 요구사항은 어떻게 수집하셨나요? 외부 사용자와 다른 점은?" |
| Impact Measurement | "운영 효율화의 성과를 어떻게 측정하셨나요? (처리 시간, 오류율, 인력 절감 등)" |

**Rule:** Treat operational stakeholders (운영팀, CS팀, 관리자) as "users" with needs to discover. Apply the same user-centered planning lens used for external users — backoffice planning is not just "building admin screens."

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
- Validation/stakeholder questions belong INSIDE project sections
- No separate "additional" or "verification" section at document end

### 3.3 Per-Question Template

```markdown
### [Symbolic Korean Title]
- **연관 기능 (Related Feature):** [specific feature/initiative]
- **빌드업 질문 (Buildup Question):** [easy entry point question — "~~ 기능을 기획하게 된 배경이 무엇인가요?"]
- **핵심 질문 (Deep Dive Question):**
  - [probing question 1 — problem definition]
  - [probing question 2 — alternatives/stakeholders]
  - [probing question 3 — impact/KPI]
- **평가 기준 (Evaluation Criteria):**
  - **[Hygiene Check]:** Pass — [OK signal] / Fail — [Red Flag signal]
  - **[Core Competency]:** Plus — [depth indicator] / Minus — [shallow indicator]
  - **[Advanced Insight]:** Plus — [senior insight] / Minus — [limited perspective]
```

### 3.4 Timing Guide Template

| Section | Duration | Notes |
|---------|----------|-------|
| Ice Breaking | 3-5 min | Rapport |
| Project A | 15-20 min | Primary product deep dive |
| Project B | 10-15 min | Secondary |
| Common/Wrap-up | 5-10 min | Collaboration, learning |
| **Total** | **~30 min** | |

---

## 4. Quality Checklist (MUST pass before delivery)

Every planner interview document MUST satisfy ALL of the following:

| # | Criterion | Threshold | Check Method |
|---|-----------|-----------|-------------|
| 1 | Ice breaking questions | 2-4 included | Count questions in ice breaking section |
| 2 | Questions per project | 3-6 per project section | Count questions per project heading |
| 3 | Perspective coverage | Min 4 of 6 planner perspectives per project | Tag each question's perspective (Why / Why Not-Trade-off / Stakeholder / How-Who / Validation-Risk / So What), verify count |
| 4 | KPI/Impact metric question | At least 1 per document probing KPI definition process | Search for KPI selection/measurement/tracking questions — generic metric questions alone are insufficient |
| 5 | Stakeholder/cross-functional question | At least 1 per document | Search for stakeholder alignment/conflict resolution/cross-functional coordination questions |
| 6 | Project-based grouping | All questions in project sections | Verify no orphan "verification" or "additional" sections |
| 7 | Timing feasibility | Total ~30 min | Sum timing guide, verify 28-35 min range |
| 8 | Problem definition question | At least 1 per document probing problem framing process | Search for problem definition/framing/validation questions — generic "배경이 뭔가요?" alone is insufficient |

### Failure Protocol

If any criterion fails:
1. Identify which criterion failed
2. Report the gap (e.g., "Project B has only 2 planner perspectives covered")
3. Suggest specific fix (e.g., "Add a Stakeholder & Communication question about the design team alignment process")
4. Do NOT deliver the document until all criteria pass

---

## 5. Response Principles

- When asked to "review/improve/write" planner interview questions, apply ALL above rules
- Keep questions per project to 3-6 — do not over-generate
- If project info is ambiguous, ask for clarification FIRST before generating
- Compress questions around the most impactful product decisions and planning challenges
- Every question must serve a purpose — no filler questions
- Prioritize questions that reveal planning PROCESS over questions that test domain knowledge
