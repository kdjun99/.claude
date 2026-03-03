---
name: codebase-learn
description: "Scans a codebase and auto-updates repo-structure skill and domain-dictionary with discovered patterns. Use with ralph for persistent execution."
---

# Codebase Learn

Scans a codebase directory and updates repo-structure skill + domain-dictionary with discovered patterns.

## When to Use

- "이 레포 학습해줘"
- "codebase-learn 스킬로 ~/dev/linkareer-main 스캔해줘"
- New repo added locally and repo-structure needs updating
- repo-structure skill has empty or outdated sections

## Prerequisites

- Target repo path (must exist locally)
- Target repo-structure skill file path (e.g., `~/.claude/skills/linkareer-repo-structure/skill.md`)
- (Optional) domain-dictionary path for domain term updates

## Input

User provides:
1. **Repo path**: `~/dev/{repo-name}` (required)
2. **Repo-structure skill**: defaults to `~/.claude/skills/linkareer-repo-structure/skill.md`
3. **Domain dictionary**: defaults to `~/.claude/skills/domain-dictionary.md`

## Scan Protocol

### Step 1: Repo Validation

```
1. Verify target path exists
2. Check for package.json (Node.js) or equivalent
3. Identify project type: NestJS, Express, Serverless, etc.
4. Read project name from package.json
```

### Step 2: Module Structure Scan

**Agent:** `oh-my-claudecode:explore` (haiku)

Tasks:
1. List all directories under `src/`
2. For each module directory, classify:
   - **Domain module**: has `application/` + `domain/` + `presentation/` (or subset)
   - **Infrastructure module**: has `guard/`, `strategy/`, or is flat
   - **Shared module**: has `dto/`, `filter/`, `exception/`, `type/`
3. Record the internal structure pattern for each module type
4. Identify NestJS module registration pattern (`.module.ts` contents)

Output format:
```markdown
### Module Structure ({repo-name})

#### Domain Modules
| Module | application/ | domain/ | presentation/ | exception/ | Extra |
|--------|-------------|---------|---------------|------------|-------|
| board  | Yes         | Yes     | Yes (resolver + dto/) | Yes  | dataloader |

#### Infrastructure Modules
| Module | Structure | Notes |
|--------|-----------|-------|
| auth   | application/ + guard/ + strategy/ | JWT-based |
| prisma | flat (module + service) | DB wrapper |

#### Shared Modules
| Module | Structure | Notes |
|--------|-----------|-------|
| common | dto/ + exception/ + filter/ + logger/ + type/ | Cross-cutting |
```

### Step 3: Build/Verify Commands Scan

Read `package.json` scripts section and extract:

```markdown
### Build Commands ({repo-name})
| Command | Script | Verified |
|---------|--------|----------|
| dev     | nest start --watch | Yes |
| build   | nest build | Yes |
| test    | jest | Yes |
| lint    | eslint --fix | Yes |
| type-check | (not found) | N/A |
| gql-codegen | (not found) | N/A |
```

Key scripts to look for:
- `build`, `start`, `dev`
- `test`, `test:e2e`, `test:cov`
- `type-check`, `tsc`, `typecheck`
- `build:gqlgen`, `codegen`, `generate`
- `lint`, `format`
- `prisma:generate`, `prisma:migrate`

### Step 4: Prisma Schema Scan

```
1. Check prisma/ directory exists
2. Read schema.prisma (or *.prisma files)
3. Extract:
   - All model names + table names (@@map)
   - Relations between models
   - Enums
   - Schema type: single file vs multi-file vs dual-schema
4. Check for prisma.config.ts (Prisma 6.x+)
```

Output format:
```markdown
### Prisma Schema ({repo-name})
| Model | Table (@@map) | Key Fields | Relations |
|-------|--------------|------------|-----------|
| User  | users        | id, email, name, userRole | posts[], scraps[] |
| Board | boards       | id, name, slug, parentId | parent?, children[], posts[] |

Schema type: Single file (prisma/schema.prisma)
Prisma version: {from package.json}
```

### Step 5: GraphQL / API Pattern Scan

```
1. Check for GraphQL setup:
   - Code-first: @Resolver, @Query, @Mutation decorators
   - Schema-first: *.graphql files
   - Federation: @apollo/subgraph, @key directives
2. Or REST setup:
   - @Controller, @Get, @Post decorators
   - Swagger/OpenAPI decorators
3. Extract:
   - API type: GraphQL (code-first/schema-first) or REST
   - Auth pattern: JWT, session, API key
   - Validation: class-validator, zod, joi
```

### Step 6: Domain Term Extraction

```
1. From Prisma models: extract model names as domain entities
2. From GraphQL types: extract @ObjectType names
3. From module names: infer domain concepts
4. Map Korean terms where identifiable from comments or variable names
5. Cross-reference with existing domain-dictionary entries
```

Output format:
```markdown
### Domain Terms ({repo-name})
| Code Entity | DB Table | GraphQL Type | Module | Suggested 기획서 용어 |
|-------------|----------|-------------|--------|---------------------|
| Board       | boards   | Board       | board  | 게시판               |
| Post        | posts    | Post        | post   | 게시글               |
```

### Step 7: Test Pattern Scan

```
1. Find test files: *.spec.ts, *.test.ts, *.e2e-spec.ts
2. Identify test location pattern:
   - Co-located (src/**/*.spec.ts)
   - Separate directory (test/)
   - Both
3. Check test framework: jest, vitest, mocha
4. Check test utilities: supertest, @nestjs/testing
```

## Update Protocol

After scanning, update target files:

### Update repo-structure skill

1. Read current repo-structure skill file
2. For each section, merge scan results:
   - **Repo Matrix row**: Add or update the scanned repo's row
   - **Module Convention**: Add patterns discovered (domain vs infrastructure)
   - **Build Commands**: Add verified commands for this repo
   - **Prisma Structure**: Add schema info for this repo
3. Mark all additions as `Verified` with scan date
4. Preserve existing unverified entries (don't overwrite)
5. Add `Local Paths` entry if not present

### Update domain-dictionary

1. Read current domain-dictionary (create if not exists)
2. For each discovered domain term:
   - If already in dictionary → skip (preserve existing)
   - If new → add to `Verified Entries` section with repo source
3. Leave `기획서 용어` column empty if not auto-detectable (human fills later)

### Repo Discovery Section

Add or update the following in repo-structure:

```markdown
## Repo Discovery

Before scanning, verify repo existence dynamically:
1. Check ~/dev/{repo-name}/ exists
2. If yes: use verified data from this skill
3. If no: use Repo Matrix metadata only (architecture-level, no file-level claims)

### Discovered Repos
| Repo | Path | Last Scanned | Status |
|------|------|-------------|--------|
| {repo-name} | ~/dev/{repo-name} | {date} | Verified |
```

## Multi-Repo Batch Scan

When scanning multiple repos at once:

```
"codebase-learn 스킬로 ~/dev/linkareer-* 전부 학습해줘"
```

1. Glob `~/dev/linkareer-*` to find all repos
2. Run Step 1-7 for each repo in **parallel** (explore agents)
3. Merge all results into single repo-structure update
4. Single domain-dictionary update with all terms

## Output

After learning completes:
1. Updated `{repo-structure}/skill.md` with new sections
2. Updated or created `domain-dictionary.md`
3. Learning report summarizing what was discovered and updated

```markdown
# Codebase Learning Report

## Scanned
- Repo: {repo-name}
- Path: {path}
- Date: {date}

## Discovered
- Modules: {n} ({domain} domain, {infra} infrastructure, {shared} shared)
- Prisma Models: {n}
- GraphQL Types: {n}
- Build Scripts: {n}
- Domain Terms: {n} new, {m} existing

## Updated Files
- {repo-structure-path}: {sections updated}
- {domain-dictionary-path}: {terms added}
```

## Quality Criteria

- [ ] All src/ modules classified (domain/infrastructure/shared)
- [ ] All package.json scripts extracted
- [ ] Prisma schema fully parsed (models, relations, enums)
- [ ] API type correctly identified (GraphQL/REST, code-first/schema-first)
- [ ] Domain terms extracted with code entity mappings
- [ ] repo-structure skill updated with Verified markers
- [ ] domain-dictionary updated (new terms only, no overwrites)
- [ ] Learning report generated
