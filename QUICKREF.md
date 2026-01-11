# Implementation Planning Quick Reference

**One-page reference for implementation planning rules.**

---

## 10 Principles (One-Line Each)

1. **Stage N independent from N+1+** - Each stage testable alone
2. **Dependencies only DOWN** - API → Application → Domain → Infrastructure
3. **Infrastructure FIRST** - Shared infra on separate stages
4. **Concrete completion criteria** - Specific, verifiable conditions
5. **No cyclic dependencies (DAG)** - Directed Acyclic Graph
6. **Single Responsibility** - One reason to change per component
7. **Dependency table REQUIRED** - Explicit table for every plan
8. **Each stage testable** - Can test without later stages
9. **No duplicate definitions** - Define each component once
10. **Document "Why"** - Explain architectural decisions

---

## Stage Order (Always)

```
1. Infrastructure & Config (DB, Auth, Logging, Config)
2. Domain / Models (pure business entities, no framework)
3. Infrastructure Implementation (repositories, external APIs)
4. Application Services (business orchestration)
5. API / Controllers (HTTP/CLI endpoints)
6. Authentication & Authorization (add to ready API)
7. Testing (integrated system tests)
8. Deployment (CI/CD, containers)
```

**For AI/ML systems:** Add "Model Training Pipeline" after Infrastructure

---

## Layer Hierarchy

```
API / Controllers      ← HTTP, WebSocket, CLI
     ↓
Application / Services ← Use cases, business orchestration
     ↓
Domain / Models        ← Pure business logic
     ↓
Infrastructure         ← Database, APIs, file system
```

---

## Dependency Table Template

| Stage | Depends On | Completion Criteria |
|-------|------------|---------------------|
| 1 Infrastructure | - | Config loaded, Docker works |
| 2 Domain | 1 | Unit tests >80% |
| 3 Infrastructure | 1, 2 | Integration tests pass |
| 4 Services | 2, 3 | Services tested |
| 5 API | 4 | Endpoints working |
| 6 Auth | 1, 5 | Auth works, roles enforced |
| 7 Testing | 1-6 | All tests green, coverage >80% |
| 8 Deployment | 7 | Smoke tests pass |

---

## Validation Checklist

- [ ] Dependency table exists
- [ ] Dependencies only point DOWN (DAG)
- [ ] Infrastructure is first
- [ ] Domain is framework-free
- [ ] Each stage has 3+ concrete criteria
- [ ] No cyclic dependencies
- [ ] Each component has single responsibility
- [ ] No duplicate definitions
- [ ] Error handling defined
- [ ] Security review planned
- [ ] Quality gates defined

**All checked?** → Plan is valid. ✅

---

## Error Handling Convention

```
DomainError (business logic)
├── ValidationError
├── NotFoundError
└── BusinessRuleError

InfrastructureError (external systems)
├── DatabaseError
├── NetworkError
└── ExternalServiceError

APIError (HTTP responses)
├── 400 BadRequestError
├── 401 UnauthorizedError
├── 403 ForbiddenError
└── 404 NotFoundError
```

---

## Logging Standards

```json
{
  "timestamp": "ISO8601",
  "level": "DEBUG|INFO|WARN|ERROR|CRITICAL",
  "stage": "Service.method",
  "action": "action_name",
  "correlationId": "req-uuid",
  "message": "Human readable message"
}
```

---

## Security Stage Pattern

| Stage | Completion Criteria |
|-------|---------------------|
| Security Review | SAST passed, DAST passed, penetration test passed |

**Location:** After all code complete, before deployment.

---

## Quality Gates

| Metric | Minimum | Target |
|--------|---------|--------|
| Unit test coverage | 70% | 85% |
| Cyclomatic complexity | < 15 | < 10 |
| Documentation | 100% | 100% |
| Security scan | pass | pass |

---

## Language-Specific Rules

### TypeScript
- No `any` type
- Use interfaces for data types
- Use branded types for IDs
- Strict null checks enabled

### Python
- Use `dataclass(frozen=True)` for value objects
- Use Protocol for interfaces
- Type hints required everywhere
- Result pattern for error handling

---

## Quick Prompt for AI

```
Create an implementation plan for [PROJECT] with:
1. Dependency table (REQUIRED)
2. 5-8 stages maximum
3. Each stage with 3+ completion criteria
4. Infrastructure stage first
5. Domain layer separate from framework
6. No cyclic dependencies
7. Security stage before deployment
8. Quality gates for each stage

Format: Markdown with table
```

---

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Solution |
|--------------|---------|----------|
| Domain imports from express/fastify | Can't test without HTTP | Move to infrastructure |
| Services depend on repos before Domain | Logic coupled to DB | Define Domain first |
| Infrastructure mixed with business logic | Untestable components | Separate stages |
| No dependency table | Hidden dependencies | Create table first |
| Vague completion criteria | Unclear when done | Use specific metrics |
| Security as afterthought | Vulnerabilities | Security-first stage |

---

**Full docs:** [README.md](README.md) | **AI rules:** [ai-rules.md](ai-rules.md) | **Examples:** [examples/](examples/)
