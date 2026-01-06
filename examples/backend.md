# Пример плана: Backend API (REST/GraphQL)

Скопируй эту структуру и адаптируй под свой проект.

---

## Типичная структура

```
Этап 1: Инфраструктура
  - Config, Docker, logger

Этап 2: Database
  - Migrations, schema

Этап 3: Domain models
  - Business logic entities

Этап 4: Infrastructure layer
  - Repositories, DB impl

Этап 5: Application services
  - Business orchestration

Этап 6: API endpoints
  - REST / GraphQL

Этап 7: Authentication
  - Auth, RBAC

Этап 8: Testing
  - Unit, integration, E2E

Этап 9: Deployment
  - Docker, CI/CD
```

---

## Таблица зависимостей

| Этап | Зависит от | Условия |
|------|-----------|---------|
| 1 Infrastructure | - | Config loaded, Docker works, logger configured |
| 2 Database | 1 | Migrations apply, schema valid, seed works |
| 3 Domain | 1 | Unit tests >80%, business logic verified |
| 4 Infrastructure impl | 2,3 | Integration tests pass, CRUD works |
| 5 Application | 3,4 | Services tested, orchestration works |
| 6 API | 5 | Endpoints working, OpenAPI spec valid, E2E tests pass |
| 7 Auth | 1,6 | Authentication works, roles enforced, security review passed |
| 8 Testing | 1-7 | All tests green, coverage >80% |
| 9 Deployment | 8 | Deployment script works, smoke tests pass |

---

## Условия завершения (детально)

### Этап 1: Инфраструктура
- Config management working
- Docker build succeeds
- Logger initialized and tested
- Health check endpoint created
- CI/CD skeleton ready

### Этап 2: Database
- Migration framework configured
- Schema migrations created and tested
- Seed data scripts ready
- Connection pooling configured
- Backup strategy documented

### Этап 3: Domain models
- Core entities defined and typed
- Business rules implemented
- Unit tests passing (>80% coverage)
- No external dependencies

### Этап 4: Infrastructure implementation
- All repositories implemented
- Database queries tested
- Caching layer ready (if needed)
- Integration tests passing
- External API clients ready

### Этап 5: Application services
- Business orchestration implemented
- Use cases defined
- Services unit tested
- Integration with repositories verified

### Этап 6: API endpoints
- All endpoints implemented
- Input validation working
- Error handling consistent
- API spec (OpenAPI) matches actual
- E2E tests passing

### Этап 7: Authentication & Authorization
- JWT/sessions implemented
- Permission checks enforced
- Roles and scopes defined
- Security review passed

### Этап 8: Testing
- Unit tests: >80% coverage
- Integration tests for all happy paths
- E2E tests for critical flows
- Load tests for performance

### Этап 9: Deployment
- Docker image builds and runs
- CI/CD pipeline works
- Smoke tests on production-like env
- Rollback plan documented

---

## Параллельная разработка

```
Team A: [1] → [2] → [4] → [5] → [6] → [7] → [8] → [9]
Team B:      [1] → [3] → [5] → [6] → [7] → [8]
Team C:                              [7] → [8] (security review)
Team D:                                  [8] → [9] (QA)
```

- **Team A:** Infrastructure + Database + Repositories + Services + API
- **Team B:** Domain models (parallel to infrastructure)
- **Team C:** Authentication (can start after API basics)
- **Team D:** Testing and Deployment (full stack testing)

---

## Архитектурные решения (шаблон)

### 1. Почему 3-слойная архитектура?
- Разделение concerns: бизнес-логика отдельна от фреймворка и БД
- Тестируемость: domain может быть протестирован без реальной БД
- Гибкость: можно менять БД или фреймворк без изменения домена

### 2. Почему такой порядок этапов?
- Infrastructure first: все нужно для работы сначала
- Database before repositories: нужна схема для реализации
- Domain before repositories: нужна логика для правильных запросов
- API after services: API только склеивает готовые сервисы
- Auth last: может быть добавлена к готовой API

### 3. Почему отдельный этап 8 для тестирования?
- Лучше тестировать интегрированную систему
- Баги интеграции видны именно на этом этапе
- Эта этап может быть параллельный предыдущим частично

---

## Риски

| Риск | Mitigation |
|------|-----------|
| Миграции БД ломают deploy | Reversible migrations from start |
| API contract меняется | Spec-first design, automated tests |
| Auth реализована неправильно | Security review on this stage |
| Performance не OK | Load testing on stage 8, optimization early |

---

## Чеклист перед началом

- [ ] Все этапы понимаеш?
- [ ] Таблица зависимостей логична?
- [ ] Условия завершения конкретные?
- [ ] Команда согласна с порядком?
- [ ] Параллелизм ясен?

**Готово!** → Используй [`plan-template.md`](../templates/plan-template.md) для своего проекта.

