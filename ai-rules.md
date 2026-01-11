---
title: AI Implementation Planning Rules
version: 1.1.0
purpose: Quick reference for AI agents (Cursor, Gemini CLI, Codex)
scope: All implementation planning tasks (Backend, Frontend, AI/ML, Infrastructure)
last_updated: 2026-01-11
---

# AI Implementation Planning Rules

**Purpose:** This file provides all rules AI agents need to create valid implementation plans without reading the entire documentation repository.

**Scope:** Backend, Frontend, AI/ML Systems, Infrastructure, Mobile applications

---

## 1. Linear Independence of Stages

**Rule:** Stage N must be completable and testable independently from stages N+1, N+2, ...

**Check:**
- Can another team start Stage N+1 after Stage N?
- Can Stage N be tested without knowledge of Stage N+1?

**If NO → Plan is invalid, restructure.**

**Example (Bad):**
```
Stage 1: API endpoints
Stage 2: Database (too late!)
Stage 3: Services (waiting for database)
```

**Example (Good):**
```
Stage 1: Database setup
Stage 2: Domain models
Stage 3: API endpoints
```

---

## 2. Layer Separation (Architecture)

**Rule:** Dependencies point only DOWN or HORIZONTALLY. Never UP.

```
API / Controllers      ← HTTP, WebSocket, CLI
     ↓
Application / Services ← Use cases, business orchestration
     ↓
Domain / Models        ← Pure business logic
     ↓
Infrastructure         ← Database, APIs, file system
```

**Check:** Does Domain know about the framework? If YES → Error.

**No framework dependencies in Domain:**
```typescript
// BAD: Domain depends on framework
import { Request, Response } from 'express';
export class User {
  static create(req: Request) { /* ... */ }
}

// GOOD: Domain is pure
export interface CreateUserInput { name: string; email: string; }
export class User {
  static create(input: CreateUserInput): User { /* ... */ }
}
```

---

## 3. Infrastructure on Separate Stages

**Rule:** ALL shared infrastructure (DB, Auth, Logging, Config) on SEPARATE stages BEFORE use.

**Pattern:**
```
Stage 1: Infrastructure (DB, Config, Logging)
Stage 2: Domain (pure logic, no infra dependency)
Stage 3: Infrastructure implementation (repositories)
Stage 4: Services (use domain + infra)
Stage 5: API (use services)
```

**Why:** Stage 2 can be developed and tested WITHOUT database.

---

## 4. Clear Completion Criteria

**Rule:** Each stage has SPECIFIC, VERIFIABLE conditions.

**Bad:**
```
❌ "Works"
❌ "Tests pass"
❌ "No errors"
```

**Good:**
```
✅ "All unit tests passing (coverage > 80%)"
✅ "Integration tests for happy path passing"
✅ "Code review approved"
✅ "Performance: response time < 200ms"
```

---

## 5. No Cyclic Dependencies (DAG)

**Rule:** Dependencies must form a Directed Acyclic Graph.

```
❌ BAD (cycle):
  A → B
  B → C
  C → A ← CYCLE!

✅ GOOD (DAG):
  A → B
  B → C
  C (end)
```

**Check:** Can you draw only downward arrows? If yes → Correct.

---

## 6. Single Responsibility

**Rule:** Each component/service/module has ONE reason to change.

**Bad:**
```typescript
UserService {
  createUser() { /* ... */ }      // User management
  sendWelcomeEmail() { /* ... */ } // Email - WRONG!
  logUserAction() { /* ... */ }    // Logging - WRONG!
}
```

**Good:**
```typescript
UserService { createUser() { /* ... */ } }
EmailService { sendWelcomeEmail() { /* ... */ } }
AuditLogger { logUserAction() { /* ... */ } }
```

---

## 7. Dependency Table (REQUIRED)

**Rule:** Every plan has an EXPLICIT dependency table.

```
| Stage | Depends On | Completion Criteria |
|-------|------------|---------------------|
| 1     | -          | Config loaded       |
| 2     | 1          | Migrations applied  |
| 3     | 1, 2       | Unit tests > 80%    |
```

**Why:** This is the ONLY way to see parallelism and conflicts.

---

## 8. Testability of Each Stage

**Rule:** Each stage must be testable BY ITSELF.

```
Stage 1: Domain models → Unit tests
Stage 2: Database → Integration tests with DB
Stage 3: Services → Unit + Integration tests
Stage 4: API → API tests + E2E
```

**If stage cannot be tested → It's too big, split it.**

---

## 9. No Duplicate Definitions

**Rule:** Each component defined EXACTLY once.

```
❌ BAD:
  Stage 3: UserService defined
  Stage 6: UserService redefined with new methods

✅ GOOD:
  Stage 3: UserService defined fully
  Stage 6: UserService used (not redefined)
```

---

## 10. Document Architectural Decisions

**Rule:** Plan includes document "Why this architecture?"

```markdown
## Architectural Decisions

### 1. Why 3-layer architecture?
- Separation of concerns
- Testability
- Reusability

### 2. Why microservices?
- Independent scalability
- Parallel development
- Separate deployment
```

---

## 11. Error Handling Convention

**Rule:** All errors classified and handled at appropriate layer.

### Error Hierarchy

```
DomainError (business logic failures)
├── ValidationError (invalid input)
├── NotFoundError (resource not found)
└── BusinessRuleError (business rules violated)

InfrastructureError (external system failures)
├── DatabaseError (connection, query failures)
├── NetworkError (API, HTTP failures)
└── ExternalServiceError (3rd party failures)

APIError (HTTP response errors)
├── BadRequestError (400)
├── UnauthorizedError (401)
├── ForbiddenError (403)
└── NotFoundError (404)
```

### Error Location by Layer

| Error Type | Where Defined | Where Handled |
|------------|---------------|---------------|
| DomainError | Domain layer | Application layer (converted to APIError) |
| InfrastructureError | Infrastructure layer | Application layer |
| APIError | API layer | API layer (HTTP responses) |

### Example: Error Flow

```typescript
// Domain - defines error
class UserNotFoundError extends DomainError {
  constructor(userId: string) {
    super(`User not found: ${userId}`, { userId });
  }
}

// Application - handles and converts
class UserService {
  async getUser(id: string): Promise<User> {
    const user = await this.repo.find(id);
    if (!user) {
      throw new UserNotFoundError(id);  // Domain error
    }
    return user;
  }
}

// API - converts to HTTP response
class UserController {
  async getUser(req: Request, res: Response) {
    try {
      const user = await userService.getUser(req.params.id);
      res.json(user);
    } catch (error) {
      if (error instanceof UserNotFoundError) {
        res.status(404).json({ error: error.message });
      } else {
        res.status(500).json({ error: "Internal server error" });
      }
    }
  }
}
```

---

## 12. Logging & Monitoring Standards

**Rule:** Logs are structured, contextual, and follow consistent patterns.

### Structured Log Format

```json
{
  "timestamp": "2026-01-11T10:30:00Z",
  "level": "INFO",
  "stage": "UserService.create",
  "action": "user_created",
  "correlationId": "req-abc-123",
  "userId": "user-456",
  "message": "User created successfully",
  "metadata": {
    "email": "user@example.com",
    "role": "admin"
  }
}
```

### Log Levels

| Level | When to Use | Example |
|-------|-------------|---------|
| DEBUG | Detailed debugging info | "Processing item 1 of 100" |
| INFO | Normal operation | "User created", "API request received" |
| WARN | Unexpected but handled | "Rate limit approaching", "Cache miss" |
| ERROR | Handled failures | "Database timeout (retried)", "Validation failed" |
| CRITICAL | Unhandled failures | "Database connection lost", "Memory critical" |

### Correlation ID Pattern

**Rule:** All requests have unique correlation ID for tracing.

```
Stage 1: Generate correlation ID (UUID)
  ↓
Stage 2: Pass through all layers (context, headers)
  ↓
Stage 3: Include in all logs
  ↓
Stage 4: Use for distributed tracing
```

### Key Metrics to Monitor

| Metric | Where | Alert Threshold |
|--------|-------|-----------------|
| Request latency | API | > 500ms p95 |
| Error rate | API | > 1% |
| DB query time | Infrastructure | > 100ms |
| Cache hit rate | Infrastructure | < 80% |
| Business KPIs | Application | Project-specific |

---

## 13. Security-First Stage

**Rule:** Security review is a dedicated stage BEFORE deployment.

### Security Stage Pattern

| Stage | Depends On | Completion Criteria |
|-------|------------|---------------------|
| N-1 Code Complete | N-2 | All features implemented, tests passing |
| N Security Review | N-1 | SAST passed, DAST passed, penetration test passed |
| N Deployment | N | Security gates cleared, monitoring active |

### Security Checklist

| Category | Check | Tool |
|----------|-------|------|
| SAST | Static analysis for vulnerabilities | SonarQube, Semgrep |
| DAST | Dynamic testing against running app | OWASP ZAP |
| Dependencies | Vulnerable dependencies check | Snyk, Dependabot |
| Secrets | No secrets in code | Gitleaks, TruffleHog |
| Auth | Authentication secure | Manual review |
| Authorization | RBAC/permissions correct | Manual review |
| Data | Encryption at rest and transit | Manual review |

### Security-First Dependency Table Example

| Stage | Depends On | Completion Criteria |
|-------|------------|---------------------|
| 1 Infrastructure | - | Config, secrets vault configured |
| 2 Domain | 1 | Entities defined, validated |
| 3 Repositories | 2, 1 | Integration tests pass |
| 4 Services | 3, 2 | Unit tests >80% |
| 5 API | 4 | Endpoints working |
| 6 Auth Integration | 1, 5 | Auth works, roles enforced |
| 7 Security Review | 1-6 | SAST passed, DAST passed, pentest passed |
| 8 Deployment | 7 | Production secure, monitoring active |

---

## 14. Quality Gates for Each Stage

**Rule:** Each stage has defined quality metrics that MUST be met.

### Quality Gate Template

| Metric | Minimum | Target | Stage |
|--------|---------|--------|-------|
| Unit test coverage | 70% | 85% | Domain, Services |
| Cyclomatic complexity | < 15 | < 10 | All |
| Documentation | 100% | 100% | All |
| Linting | pass | pass | All |
| Type checking | pass | pass | TypeScript |
| Security scan | pass | pass | Security Review |
| Performance tests | pass | pass | Testing |

### Per-Stage Quality Gates

#### Stage 1: Infrastructure
- [ ] Config validated
- [ ] Docker build succeeds
- [ ] Linting passes
- [ ] CI/CD pipeline configured

#### Stage 2: Domain
- [ ] Unit test coverage > 80%
- [ ] No framework dependencies
- [ ] All entities defined
- [ ] Type checking passes

#### Stage 3: Infrastructure Implementation
- [ ] Integration tests pass (real DB)
- [ ] Repository patterns implemented
- [ ] External APIs integrated

#### Stage 4: Application Services
- [ ] Unit test coverage > 80%
- [ ] Business orchestration tested
- [ ] No DB dependencies in tests

#### Stage 5: API
- [ ] OpenAPI spec matches implementation
- [ ] E2E tests for happy paths pass
- [ ] Input validation working

#### Stage 6: Authentication
- [ ] Security review passed
- [ ] RBAC tested
- [ ] JWT/sessions secure

#### Stage 7: Testing
- [ ] All tests passing
- [ ] Coverage > 80%
- [ ] Performance tests pass
- [ ] Load tests pass

#### Stage 8: Deployment
- [ ] Security scan passed
- [ ] Smoke tests pass
- [ ] Rollback tested
- [ ] Monitoring active

---

## AI-Specific Guidelines

### For Cursor / AI Code Editors

When creating implementation plans:

1. **Always create dependency table first** - it reveals parallelism
2. **Infrastructure ALWAYS comes first** - non-negotiable
3. **Domain must be framework-free** - pure TypeScript/Python
4. **Each stage needs 3+ concrete completion criteria**
5. **Verify DAG property** before presenting plan

### For Gemini CLI / Codex / LLM Agents

When generating code:

1. **Ask for dependency table before generating code**
2. **Generate Domain layer FIRST** (framework-agnostic)
3. **Never generate framework code in Domain**
4. **Add completion criteria comments in generated code**
5. **Flag when infrastructure dependencies are missing**

### Prompt Template for AI

```
Create an implementation plan for [PROJECT] with:
1. Dependency table (REQUIRED)
2. 5-8 stages maximum
3. Each stage with 3+ completion criteria
4. Infrastructure stage first
5. Domain layer separate from framework
6. No cyclic dependencies

Format: Markdown with table
```

---

## Stage Ordering Pattern

**Always (in order):**

```
1. Infrastructure & Config (DB, Auth, Logging, Config)
2. Domain / Models (pure business entities, no framework)
3. Infrastructure Implementation (repositories, external APIs)
4. Application Services (business orchestration)
5. API / Controllers (HTTP/CLI endpoints)
6. Authentication & Authorization (can be added to ready API)
7. Testing (integrated system tests)
8. Deployment (CI/CD, containers)
```

**Exception:** For AI/ML systems, add "Model Training Pipeline" after Infrastructure and before Services.

---

## Quick Validation Checklist

Before presenting any plan:

- [ ] Dependency table exists
- [ ] Dependencies only point DOWN (DAG)
- [ ] Infrastructure is first
- [ ] Domain is framework-free
- [ ] Each stage has 3+ concrete criteria
- [ ] No cyclic dependencies
- [ ] Each component has single responsibility
- [ ] No duplicate definitions

**All checked?** → Plan is valid. ✅

---

## Examples

### Backend API Plan

| Stage | Depends On | Completion Criteria |
|-------|------------|---------------------|
| 1 Infrastructure | - | Config loaded, Docker works, logger configured |
| 2 Database | 1 | Migrations apply, schema valid |
| 3 Domain | 1 | Unit tests >80%, business logic verified |
| 4 Repositories | 2, 3 | Integration tests pass, CRUD works |
| 5 Services | 3, 4 | Services tested, orchestration works |
| 6 API | 5 | Endpoints working, OpenAPI spec valid |
| 7 Auth | 1, 6 | Auth works, roles enforced |
| 8 Testing | 1-7 | All tests green, coverage >80% |
| 9 Deployment | 8 | Deployment script works, smoke tests pass |

### AI/RAG System Plan

| Stage | Depends On | Completion Criteria |
|-------|------------|---------------------|
| 1 Infrastructure | - | Config, Vector DB, LLM API keys configured |
| 2 Domain | 1 | Entities defined, business rules verified |
| 3 Vector Store | 1 | Index created, search tested |
| 4 Embedding Pipeline | 1, 3 | Embeddings generated, chunking works |
| 5 RAG Service | 2, 3, 4 | Retrieval + generation tested |
| 6 API | 5 | Endpoints working, latency < 2s |
| 7 Evaluation | 1-6 | RAGAS score > 0.7, edge cases handled |
| 8 Deployment | 7 | Container works, scaling configured |

---

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Solution |
|--------------|---------|----------|
| Domain has imports from express/fastify | Can't test without HTTP | Move to infrastructure layer |
| Services depend on repositories before Domain | Logic coupled to DB | Define Domain entities first |
| "Infrastructure" mixed with "Business logic" | Untestable components | Separate into different stages |
| No dependency table | Hidden dependencies | Create table before any code |
| Vague completion criteria | Unclear when done | Use specific metrics and tests |

---

**This file is self-contained. No need to read other files.**
