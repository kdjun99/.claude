# Skill Candidates

Curated external skills worth installing. Evaluated for Linkareer's stack (NestJS, TypeScript, GraphQL Federation, Next.js, microservices).

> Install: `npx skills add <repo> --skill <name>`
> Install globally: add `-g` flag
> Install to specific agent: add `-a claude-code`

---

## Priority: High

### alirezarezvani/claude-skills

| Skill | What it does | Why useful |
|-------|-------------|------------|
| `engineering-skills` | 23 skills (architecture, backend, QA, DevOps, security, AI/ML, Playwright, AWS) | Broad engineering coverage, directly relevant to our stack |
| `engineering-advanced-skills` | 25 skills (CI/CD, database design, observability, security auditing, release mgmt) | Deep infra/ops skills for microservice architecture |

```bash
# Install
npx skills add alirezarezvani/claude-skills --skill engineering-skills
npx skills add alirezarezvani/claude-skills --skill engineering-advanced-skills
```

### levnikolaevich/claude-code-skills — Codebase Audit

| Skill | What it does | Why useful |
|-------|-------------|------------|
| `ln-620-codebase-auditor` | Coordinates 9 audit workers in parallel | One command to audit entire codebase |
| `ln-621-security-auditor` | Hardcoded secrets, SQL injection, XSS, insecure deps | Security review for all repos |
| `ln-623-code-principles-auditor` | DRY/KISS/YAGNI compliance with quantitative scoring | Code quality enforcement |
| `ln-624-code-quality-auditor` | Cyclomatic complexity, N+1 queries, god classes | Performance & maintainability |
| `ln-625-dependencies-auditor` | Outdated packages, unused deps, CVE scan | Dependency hygiene across 8+ repos |
| `ln-626-dead-code-auditor` | Unreachable code, unused imports/functions | Cleanup for legacy repos (Sequelize->Prisma migration artifacts) |
| `ln-628-concurrency-auditor` | Async races, thread safety, TOCTOU, blocking I/O | Critical for chat-api WebSocket + Lambda concurrency |

```bash
# Install coordinator (includes all 9 workers)
npx skills add levnikolaevich/claude-code-skills --skill ln-620-codebase-auditor
# Or install individually
npx skills add levnikolaevich/claude-code-skills --skill ln-621-security-auditor --skill ln-623-code-principles-auditor --skill ln-624-code-quality-auditor --skill ln-625-dependencies-auditor --skill ln-626-dead-code-auditor --skill ln-628-concurrency-auditor
```

---

## Priority: Medium

### levnikolaevich/claude-code-skills — Test & Performance

| Skill | What it does | Why useful |
|-------|-------------|------------|
| `ln-630-test-auditor` | Coordinates 7 test audit workers | Test quality assessment |
| `ln-640-pattern-evolution-auditor` | Architectural pattern compliance scoring | Track pattern consistency across repos |
| `ln-650-persistence-performance-auditor` | Query efficiency, transaction correctness, N+1 detection | DB performance for Prisma + Sequelize dual ORM |
| `ln-810-performance-optimizer` | Multi-cycle diagnostic pipeline: profile -> research -> optimize | Performance bottleneck resolution |
| `ln-820-dependency-optimization-coordinator` | Coordinates dep upgrades across package managers | Bulk dependency updates |

```bash
npx skills add levnikolaevich/claude-code-skills --skill ln-630-test-auditor --skill ln-640-pattern-evolution-auditor --skill ln-650-persistence-performance-auditor
npx skills add levnikolaevich/claude-code-skills --skill ln-810-performance-optimizer --skill ln-820-dependency-optimization-coordinator
```

### levnikolaevich/claude-code-skills — Layer & API Audit

| Skill | What it does | Why useful |
|-------|-------------|------------|
| `ln-642-layer-boundary-auditor` | Layer boundary violations, transaction ownership | Clean architecture enforcement |
| `ln-643-api-contract-auditor` | DTO leakage, inconsistent error contracts | GraphQL Federation contract consistency |
| `ln-644-dependency-graph-auditor` | Cycle detection, coupling metrics (Ca/Ce/I) | Microservice dependency health |
| `ln-647-env-config-auditor` | Env var sync, naming, startup validation | Config hygiene across 8+ services |

```bash
npx skills add levnikolaevich/claude-code-skills --skill ln-642-layer-boundary-auditor --skill ln-643-api-contract-auditor --skill ln-644-dependency-graph-auditor --skill ln-647-env-config-auditor
```

### vercel-labs/agent-skills

| Skill | What it does | Why useful |
|-------|-------------|------------|
| `vercel-react-best-practices` | React/Next.js performance optimization | Frontend repos (client, biz, cms, community) |
| `web-design-guidelines` | UI accessibility & design audit | Frontend quality checks |

```bash
npx skills add vercel-labs/agent-skills --skill vercel-react-best-practices --skill web-design-guidelines
```

---

## Priority: Low (nice to have)

### alirezarezvani/claude-skills

| Skill | What it does | Why useful |
|-------|-------------|------------|
| `product-skills` | PM toolkit, UX researcher, competitive analysis | Product planning support |
| `pm-skills` | Scrum master, Jira/Confluence expert | Project management automation |

### alirezarezvani/claude-skills — Playwright

| Skill | What it does | Why useful |
|-------|-------------|------------|
| `generate` | Generate Playwright tests | E2E test authoring |
| `review` | Review Playwright test quality | Test quality checks |
| `fix` | Fix flaky Playwright tests | Flaky test debugging |

### levnikolaevich/claude-code-skills — Documentation

| Skill | What it does | Why useful |
|-------|-------------|------------|
| `ln-100-documents-pipeline` | Auto-generate project docs (architecture, API, DB schema) | Documentation bootstrapping |
| `ln-160-docs-skill-extractor` | Extract procedures from docs into skills | Skill creation from existing docs |

---

## Notes

- Already have 25 custom skills in `~/.claude/skills/` — monitor context budget with `/context`
- If skills get excluded, increase budget: `export SLASH_COMMAND_TOOL_CHAR_BUDGET=32000`
- Prefer project-level install over global for repo-specific skills
- `levnikolaevich` skills are designed as a pipeline (ln-620 coordinates ln-621~629) — install the coordinator to get the full workflow
