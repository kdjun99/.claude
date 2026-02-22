---
description: "Interview question generation guidelines: 6-perspective deep-dive framework, career level analysis strategies, portfolio signal detection, structured question format (Buildup/Deep-Dive/Evaluation Criteria), and ice breaking rules. Apply when generating, reviewing, or refining interview questions for candidate assessment."
---

# Interview Guidelines

Core domain knowledge for generating structured interview questions that systematically assess candidate competency across multiple dimensions. All agents in the interview-question-generator pipeline MUST apply these rules.

---

## 1. Ice Breaking Rules

**Purpose:** Rapport formation + communication style assessment. NOT technical evaluation.

**Count:** 2-4 questions per interview.

**Selection Criteria:**

| Criterion | Rule |
|-----------|------|
| Low Cognitive Load | No right answers, no deep thinking required |
| Interest Based | Use resume hobbies, certifications, recent interests |
| Contextual | Arrival, weather, simple greetings — universally easy |

**Example Patterns (adapt to each candidate):**
- "오늘 오시는 길은 찾기 어렵지 않으셨나요?"
- "이력서에 [취미/특기]가 눈에 띄던데, 최근에도 즐기실 시간이 있으셨나요?"
- "긴장을 푸는 본인만의 방법이 있으신가요?"
- "최근에 개발 외적으로 가장 몰입했던 즐거움은 무엇이었나요?"

---

## 2. Six Perspectives Framework

The core analytical framework for deep-dive questions. Each project section MUST cover **minimum 4 of 6** perspectives.

### 2.1 Why — 문제 정의 및 근본 원인 파악

**Assesses:** Problem awareness, root cause analysis, prioritization judgment

**Probe patterns:**
- "그 문제가 발생했다는 것을 어떻게 인지하셨나요? (로그, 사용자 리포트, 모니터링 등)"
- "단순한 현상(Symptom)이 아니라, 근본 원인(Root Cause)은 무엇이었나요?"
- "왜 그 시점에 그 문제를 해결하는 것이 우선순위가 높았나요?"

### 2.2 Why Not / Trade-off — 대안 검토 및 의사결정

**Assesses:** Alternative evaluation, decision rationale, side effect awareness

**Probe patterns:**
- "해결책으로 선택한 기술/방법 외에 고려했던 다른 대안은 없었나요?"
- "다른 대안들과 비교했을 때, 해당 방식을 선택한 결정적인 근거는 무엇인가요?"
- "새로운 기술이나 방식을 도입함으로써 발생한 단점(Trade-off)이나 사이드 이펙트는 없었나요?"

### 2.3 Business Value — 비즈니스 가치와 설득

**Assesses:** Business communication, stakeholder alignment, data-driven proof

**Probe patterns:**
- "기술적 개선 작업이 사업부나 비개발 조직에는 어떤 이득을 주는지 어떻게 설명하고 설득하셨나요?"
- "개발팀 내부의 엔지니어링 니즈와 사업부의 비즈니스 일정 간의 충돌을 어떻게 조율하셨나요?"
- "사업부에서도 공감할 만한 성과(예: 배포 속도 향상, 장애 대응 비용 절감 등)를 어떻게 데이터로 증명하셨나요?"

### 2.4 How / Who — 구체적인 실행 및 기여도

**Assesses:** Technical depth, personal contribution vs team boundary

**Probe patterns:**
- "구현 과정에서 가장 기술적으로 까다로웠던 부분은 무엇이었고, 구체적으로 어떻게 극복하셨나요?"
- "전체 해결 과정에서 본인이 주도한 부분과 팀원의 도움을 받은 부분의 경계는 어디인가요?"

### 2.5 Verification / Risk — 안정성 검증 및 리스크 관리

**Assesses:** Testing strategy, deployment risk mitigation, quality assurance ownership

**Probe patterns:**
- "QA 리소스가 제한적인 상황에서 기술적 변경의 정합성을 어떻게 스스로 보장하셨나요?"
- "테스트 코드는 어떤 전략(Unit, Integration, E2E 등)으로 작성했으며, 리팩토링 과정에서 어떤 역할을 했나요?"
- "장애 리스크를 최소화하기 위해 도입한 배포 전략(Canary, Blue-Green, Feature Flag 등)이 있었나요?"

### 2.6 So What — 결과 및 회고

**Assesses:** Quantitative measurement ability, retrospective insight

**Probe patterns:**
- "문제가 해결되었다는 것을 정량적인 수치(예: Latency 감소, 처리량 증가, 리소스 절감 등)로 어떻게 증명하셨나요?"
- "지금 다시 그 시점으로 돌아간다면, 아쉬웠던 점이나 다르게 시도해보고 싶은 부분이 있나요?"

### Coverage Rule

| Constraint | Value |
|-----------|-------|
| Minimum perspectives per project | 4 of 6 |
| Must-include for performance/cost issues | Why + Verification + So What |
| Must-include for tech adoption | Why Not/Trade-off |
| Must-include for team projects | How/Who |

---

## 3. Career Level Strategy

**Rule:** Determine career level FIRST from resume/portfolio, then apply corresponding strategy to ALL questions.

### 3.1 Junior (Entry Level)

**Keywords:** Potential, Attitude, Grit, Fundamentals

**Analysis Points:**
- **Growth curve:** Self-learning speed and depth
- **Problem-solving attitude:** "Gave up" vs "Dug in all night to find the root cause"
- **CS fundamentals:** Understanding beyond framework usage (data structures, networking, OS)

**Question Strategy (Growth Focus):**
- **Failure/recovery:** "가장 힘들었던 에러는 무엇이며, 어떻게 끝까지 파고들어 해결했나요?"
- **Learning process:** "튜토리얼이 동작하지 않았을 때 어떤 가설을 세우고 디버깅했나요?"
- **Collaboration attitude:** "의견 충돌 시 내 주장을 굽히고 팀을 따른 경험이 있나요?"

### 3.2 Senior (Experienced)

**Keywords:** Trade-off, Business Impact, Leadership, Crisis Management

**Analysis Points:**
- **Decision quality:** Choosing best option when no correct answer exists
- **Operations experience:** High-traffic, incidents, legacy system management
- **Business sense:** Improving actual business metrics (revenue, cost) through technology

**Question Strategy (Impact Focus):**
- **Trade-offs:** "기술 부채를 감수하고 빠른 배포를 선택한 경험과 그 후처리는 어땠나요?"
- **Persuasion/coordination:** "비즈니스 요구사항과 기술적 제약이 충돌할 때 어떻게 조율했나요?"
- **Incident response:** "운영 중 겪은 가장 큰 장애(Incident)와 재발 방지 대책은 무엇이었나요?"

---

## 4. Portfolio Signal Analysis

Read between the lines of resumes. Detect these 3 signals and design questions to probe them.

### Signal 1: "Introduced" vs "Improved"

| Pattern | Example | Question Strategy |
|---------|---------|-------------------|
| Introduced (shallow) | "Redis 사용" | Ask: Why Redis? What problem did it solve? What alternatives? |
| Improved (deep) | "Cache Stampede로 인해 Jitter 적용" | Ask: How did you discover the stampede? What metrics proved the fix? |

**Rule:** Focus questions on problem-solving PROCESS, not technology adoption.

### Signal 2: Data Realism

| Check | Red Flag | Probe |
|-------|----------|-------|
| Traffic scale | "10M DAU" in student project | "How did you test/simulate this load?" |
| Data volume | Specific row counts without context | "Was this measured in production or estimated?" |
| Performance claims | "100ms -> 10ms" without methodology | "How did you measure before/after? What tool?" |

**Rule:** If quantitative metrics are missing, design at least 1 question to elicit them.

### Signal 3: Team Contribution Boundary

| Claim | Probe Strategy |
|-------|---------------|
| "전담했다" (owned it all) | Ask for specific technical challenge solved; what was the hardest bug? |
| Percentage claims (e.g., "백엔드 30%, 인프라 40%") | Ask what specific tasks fell in each percentage |
| Vague contribution | Ask: "What code would you be most embarrassed to show?" |

**Rule:** Probe contribution boundaries by asking about specific DIFFICULTIES, not general responsibilities.

---

## 5. Question Format Standard

### 5.1 Document Structure

All interview documents MUST follow this structure:

```
1. Candidate Profile Summary (5 lines max)
2. Ice Breaking (2-4 questions)
3. Project A Section (3-6 questions)
4. Project B Section (3-6 questions)
5. [Optional] Common Competency Section
6. Interview Timing Guide Table
7. Quality Checklist
```

### 5.2 Grouping Rule

**CRITICAL:** All questions MUST be grouped by project.
- Verification questions belong INSIDE the project section, NOT in a separate section.
- No "additional questions" or "verification" sections at the end.
- Interviewer should be able to go project-by-project sequentially.

### 5.3 Per-Question Format

```markdown
### Question Title (symbolic/evocative Korean title)
- **연관 기능 (Related Feature):** Specific feature/achievement this question connects to
- **빌드업 질문 (Buildup Question):** Easy entry point — "~~ 기능을 구현하실 때 가장 고민했던 지점은 무엇인가요?"
- **핵심 질문 (Deep Dive Question):** 1-3 probing questions on technical depth, decisions, trade-offs
- **평가 기준 (Evaluation Criteria):**
  - **[Hygiene Check]:** Pass (OK) — minimum requirement / Fail (Red Flag) — disqualifying signal
  - **[Core Competency]:** Plus (+) — practical depth indicator / Minus (-) — shallow understanding
  - **[Advanced Insight]:** Plus (+) — senior-level insight / Minus (-) — limited perspective
```

### 5.4 Buildup → Deep Dive Flow

Follow-up questions should flow: **judgment basis → alternatives → risk → results**

| Step | Pattern |
|------|---------|
| 1. Judgment basis | "Why did you choose X?" |
| 2. Alternatives | "What else did you consider? Why not Y?" |
| 3. Risk | "What could go wrong? How did you mitigate?" |
| 4. Results | "How did you measure success? What would you change?" |

### 5.5 Evaluation Criteria Levels

| Level | Purpose | Format | Applies To |
|-------|---------|--------|-----------|
| Hygiene Check | Junior must-have | Binary: Pass / Fail | Concept understanding, basic awareness |
| Core Competency | Practical depth | Gradient: Plus / Minus | Implementation detail, data-driven decisions |
| Advanced Insight | Senior-level bonus | Gradient: Plus / Minus | Architectural trade-offs, operational foresight |

### 5.6 Timing Guide

| Section | Duration | Notes |
|---------|----------|-------|
| Ice Breaking | 3-5 min | Rapport building |
| Main Project A | 15-20 min | Deepest technical dive |
| Main Project B | 10-15 min | Second priority |
| Common/Wrap-up | 5-10 min | Collaboration, learning, closing |
| **Total** | **~30 min** | |

### 5.7 Quality Checklist (must pass before delivery)

- [ ] Ice breaking: 2-4 questions included
- [ ] Questions per project: 3-6 range maintained
- [ ] Perspective coverage: minimum 4 of 6 per project
- [ ] Quantitative metric question: at least 1 per document
- [ ] Collaboration/contribution question: at least 1 per document
- [ ] All questions within project sections (no separate verification section)
- [ ] 30-minute timing is feasible
