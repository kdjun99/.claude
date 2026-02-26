---
description: "Progressive pressure probing techniques for interviews: 5-layer depth model (Surface→Specifics→Cross-check→Depth→Boundary), conditional branch design, trap point patterns, knowledge boundary mapping, and career-level pressure calibration. Shared across dev and planner interview pipelines. Apply when generating or reviewing follow-up question chains."
---

# Progressive Pressure Probing (점진적 압박 패턴)

Shared interview methodology for designing follow-up questions that progressively corner candidates. Every deep-dive question MUST be designed as a **pressure funnel** — each follow-up narrows the candidate's escape routes and demands increasingly specific, verifiable answers.

This skill is **pipeline-agnostic** — used by both dev:interview and plan:interview pipelines.

---

## 1. Five-Layer Depth Model

Every deep-dive question operates as a **5-layer funnel**. Generate at least 3 layers per question (layers 4-5 are interviewer discretion based on candidate response).

| Layer | Name | Purpose | Pattern |
|-------|------|---------|---------|
| L1 | **표면 (Surface)** | Let them talk freely, build false comfort | "~~ 기능을 구현하실 때 어떤 접근을 하셨나요?" |
| L2 | **구체화 (Specifics)** | Pin down vague claims to concrete details | "구체적으로 어떤 코드/설정/구조로 구현했나요?" |
| L3 | **교차검증 (Cross-check)** | Cross-reference with other claims or logical implications | "앞서 말씀하신 A와 지금 설명하신 B가 같이 동작하려면 C 이슈가 있을 텐데, 어떻게 처리하셨나요?" |
| L4 | **깊이 탐색 (Depth Probe)** | Push to the limit of their actual implementation knowledge | "그 방식의 내부 동작 원리를 설명해 주시겠어요? 예를 들어 [specific edge case]에서는 어떻게 되나요?" |
| L5 | **경계 탐색 (Boundary)** | Find where their knowledge or contribution ends | "그 부분에서 직접 디버깅하거나 코드를 수정하셨나요, 아니면 다른 팀원이 처리했나요?" |

**Key Principle:** L1 is **deliberately easy** — it gives the candidate confidence and space to make claims. Each subsequent layer **tests those claims** with increasing specificity. The candidate who truly did the work will answer naturally; the candidate who inflated will feel the pressure mount.

---

## 2. Conditional Branch Design

Follow-up questions MUST include conditional branches based on candidate response quality. This creates the "cornering" effect — the interviewer always has a next move.

### 2.1 Output Format

```markdown
- **꼬리 질문 체인 (Follow-up Chain):**
  - **L1 (표면):** [easy opening question]
  - **L2 (구체화):** [specifics probe]
    - → 답변이 구체적일 경우: [go deeper into implementation detail]
    - → 답변이 모호할 경우: [redirect to a more concrete, verifiable aspect]
  - **L3 (교차검증):** [cross-check with another claim from portfolio/resume]
  - **L4 (깊이):** [edge case or internal mechanism question] (interviewer discretion)
```

### 2.2 Branch Rules

| Response Type | Next Move |
|---------------|-----------|
| **Concrete answer** | Go deeper into HOW (implementation mechanics, edge cases) |
| **Vague answer** | Redirect to something verifiable (specific numbers, specific code, specific incident) |
| **"I don't remember"** | Probe adjacent area: "그렇다면 그 시스템의 전반적인 아키텍처는 어떤 구조였나요?" |
| **Deflection to team** | Boundary probe: "그 부분에서 본인이 직접 작성한 코드나 설계 문서가 있나요?" |
| **Contradicts prior answer** | Surface the contradiction gently: "앞서 A라고 하셨는데, 지금은 B인 것 같은데 어떤 맥락인가요?" |

---

## 3. Trap Point Design (함정 포인트)

Trap points are pre-designed inconsistency detectors. Analyze the candidate's portfolio/resume for potential inconsistencies, then design questions that honest candidates answer easily but inflated candidates struggle with.

### 3.1 Inconsistency Types

| Type | Detection Method | Trap Question Pattern |
|------|-----------------|----------------------|
| **Scale Mismatch** | Student project claiming production-scale metrics | "해당 트래픽을 처리하려면 서버 몇 대가 필요한데, 인프라 구성은 어떠셨나요?" |
| **Timeline Conflict** | Short duration + large scope | "3개월 동안 이 범위를 다 하시려면 하루 작업량이 상당했을 텐데, 가장 시간이 오래 걸린 부분은?" |
| **Depth Gap** | Claims deep expertise but portfolio shows surface-level usage | "내부적으로 [기술]이 [specific mechanism]을 처리하는 방식을 설명해 주시겠어요?" |
| **Contribution Inflation** | "전담" but team size > 1 | "팀 내에서 코드 리뷰를 하셨나요? 누가 리뷰해 주셨나요?" |
| **Metric Fabrication** | Precise numbers without measurement methodology | "그 수치를 측정하신 도구와 시점은? 측정 기간은 얼마나 되나요?" |

### 3.2 Rules

- Each project section MUST have at least 1 trap point
- Trap points are NOT trick questions — they're questions that honest candidates answer easily
- Embed trap points naturally within the follow-up chain (usually at L2 or L3)
- Always design trap points BEFORE generating the full question — they inform the chain direction

---

## 4. Knowledge Boundary Mapping

The goal is to find exactly WHERE the candidate's real knowledge ends. Two escalation patterns:

### 4.1 Vertical Escalation (같은 주제, 점점 깊게)

```
"Redis를 캐시로 사용하셨군요" (L1)
→ "Eviction policy는 어떤 걸 쓰셨나요?" (L2)
→ "그 policy에서 hot key 문제가 생길 수 있는데, 경험하셨나요?" (L3)
→ "Redis Cluster 환경에서 해당 policy가 노드별로 독립 동작하는데, 이로 인한 캐시 불일치는 어떻게 처리하셨나요?" (L4)
```

### 4.2 Horizontal Escalation (관련 주제로 확장, 실제 운영 경험 검증)

```
"Redis 도입 후 캐시 히트율은 어떠셨나요?" (L1)
→ "캐시 무효화(Invalidation) 전략은 어떻게 설계하셨나요?" (L2)
→ "DB와 캐시 간 데이터 정합성 문제는 없었나요?" (L3)
→ "장애 시 캐시 서버가 죽으면 DB로 직접 트래픽이 가는데, 이에 대한 대비책은?" (L4)
```

### 4.3 When to Use Which

| Situation | Escalation | Reason |
|-----------|-----------|--------|
| Candidate claims deep expertise on a topic | Vertical | Test actual depth of that claim |
| Candidate seems surface-level on a topic | Horizontal | Find if they have broader operational understanding |
| Candidate deflects or gives vague answers | Horizontal first, then vertical on a new topic | Find something they CAN talk about, then go deep |

---

## 5. Career-Level Pressure Calibration

### 5.1 Depth Expectation

| Level | Pressure Focus | Acceptable "I Don't Know" | Red Flag |
|-------|---------------|--------------------------|----------|
| **Junior** | L1-L3 (Surface → Cross-check). L4+ is bonus. | L3+ is OK. L1-L2 vague = Red Flag. | L1에서 기본 개념 설명 불가 |
| **Senior** | L1-L5 (Full depth). L4-L5 must be reachable. | L5 is OK. L3 이하 vague = Red Flag. | L3에서 Trade-off/운영 경험 부재 |

### 5.2 Junior Pressure Pattern

Focus on **grit and learning process**:

```
"모르는 것을 어떻게 해결했나요?" (L1)
→ "구체적으로 어떤 검색어/자료를 참고했나요?" (L2)
→ "그 자료의 내용 중 핵심은 뭐였나요?" (L3)
```

- Trap: Not "did you know X?" but "when you didn't know X, what did you do?"
- Accept: Honest process description, even if the path was inefficient
- Red Flag: "그냥 구글링했어요" without any specifics

### 5.3 Senior Pressure Pattern

Focus on **decision quality under ambiguity**:

```
"왜 그 방식을 선택했나요?" (L1)
→ "반대 의견은 없었나요?" (L2)
→ "결과적으로 그 결정의 사이드 이펙트는?" (L3)
→ "다시 한다면 같은 결정을 하시겠어요?" (L4)
```

- Trap: Not "do you know X?" but "when X conflicted with Y, how did you choose?"
- Accept: Clear trade-off articulation with hindsight awareness
- Red Flag: "팀에서 결정했어요" without personal judgment contribution

---

## 6. Integration with Question Format

When generating interview questions, the per-question template MUST include:

```markdown
### [Question Title]
- **연관 기능 (Related Feature):** [specific feature]
- **빌드업 질문 (Buildup Question):** [= L1 Surface question]
- **꼬리 질문 체인 (Follow-up Chain):**
  - **L2 (구체화):** [specifics probe]
    - → 구체적 답변 시: [deeper probe]
    - → 모호한 답변 시: [redirect probe]
  - **L3 (교차검증):** [cross-check probe]
  - **L4 (깊이):** [depth probe — interviewer discretion]
- **함정 포인트 (Trap Point):** [what inconsistency to watch for + how to probe it]
- **평가 기준 (Evaluation Criteria):**
  - **[Hygiene Check]:** Pass / Fail
  - **[Core Competency]:** Plus / Minus
  - **[Advanced Insight]:** Plus / Minus
```

**Key changes from flat format:**
- `빌드업 질문` = L1 (no change)
- `핵심 질문` → replaced by `꼬리 질문 체인` (multi-layer with branches)
- `함정 포인트` added (new)
- `평가 기준` unchanged
