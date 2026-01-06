# Шаблон: Таблица зависимостей

Используй этот шаблон для отслеживания зависимостей между этапами.

---

## Как заполнять

| Этап | Зависит от | Условия завершения |
|------|-----------|------------------|
| 1 | - | [List conditions] |
| 2 | 1 | [List conditions] |
| 3 | 1, 2 | [List conditions] |

**Правила:**

1. **Этап N зависит ТОЛЬКО от этапов 1..N-1**
   - Никогда от этапов после N
   - Если не выполняется → переделай структуру

2. **Зависимости перечисляй номерами**
   - `1` = зависит от этапа 1
   - `1, 2, 3` = зависит от этапов 1, 2 и 3
   - `-` = нет зависимостей (только для этапа 1)

3. **Условия завершения конкретные**
   - ❌ "работает"
   - ✅ "unit tests passed (>80% coverage)"
   - ✅ "integration tests passed"
   - ✅ "code review approved"

---

## Пример: Backend API

```
| Этап | Зависит от | Условия завершения |
|------|-----------|------------------|
| 1 Infrastructure | - | Config loaded, logger configured, health check ready |
| 2 Database | 1 | Migrations created, schema valid, seed data loaded |
| 3 Domain | 1 | Unit tests passed (>80%), business logic verified |
| 4 Repositories | 2, 3 | Integration tests with DB passed, CRUD works |
| 5 Services | 3, 4 | Unit tests passed, integration with repos works |
| 6 API | 5 | API endpoints working, OpenAPI spec matches, E2E tests pass |
| 7 Auth | 1, 6 | Auth implemented, security review passed, role-based access works |
| 8 Testing | 1-7 | All tests passing, coverage >80%, no critical issues |
```

---

## Пример: Frontend

```
| Этап | Зависит от | Условия завершения |
|------|-----------|------------------|
| 1 Setup | - | Build works, linter configured, tests run |
| 2 State management | 1 | Store initialized, tests pass, DevTools work |
| 3 Components | 1, 2 | Component library built, stories written |
| 4 Pages | 2, 3 | Pages display, routing works, E2E tests pass |
| 5 API integration | 4 | API calls work, error handling done, caching works |
| 6 Optimization | 1-5 | Bundle size OK, performance metrics good |
```

---

## Как проверять

### Нет циклических зависимостей?

```
✅ Правильно (DAG):
   1 → 2 → 3 → 4
   
❌ Неправильно (цикл):
   1 → 2 → 3 → 1 (STOP!)
```

### Каждый этап независим от следующих?

```
✅ Правильно:
   Этап 2 зависит от: 1
   Этап 2 НЕ зависит от: 3, 4, 5, ...
   
❌ Неправильно:
   Этап 2 зависит от: 1, 5 (почему от 5?!)
```

### Условия конкретные?

```
❌ Плохо:
   "Tests passed"
   "Ready"
   "Done"
   
✅ Хорошо:
   "Unit tests >80% coverage passed"
   "Integration tests for all happy paths passed"
   "Code review from 2 seniors approved"
   "Performance: response time <200ms for 95th percentile"
```

---

## Когда использовать эту таблицу

1. **На планировании** → обсудить зависимости с командой
2. **На спринт-планировании** → какие этапы можно делать параллельно?
3. **На рефакторинге плана** → видна вся структура зависимостей
4. **На ретро** → почему некоторые этапы заняли больше?

---

## Параллельная разработка (из таблицы)

Если у тебя много этапов с малыми зависимостями → можно параллелизм:

```
Из таблицы:

Этап 1: -         (никто не может начать без этого)
Этап 2: 1         (может начать team A)
Этап 3: 1         (может начать team B - параллельно с team A!)
Этап 4: 2, 3      (может начать team C после teams A и B)

Team A: [Этап 1] → [Этап 2]
Team B:   [Этап 1 одновременно] → [Этап 3]
Team C:                        [Этап 4]
```

---

## Финальная проверка таблицы

- [ ] Зависимости только вниз (DAG)
- [ ] Нет этапа зависящего от будущих этапов
- [ ] Условия конкретные, не субъективные
- [ ] Таблица согласуется с текстом плана
- [ ] Нет опечаток в номерах этапов

**Таблица готова!** → используй её для [`plan-template.md`](plan-template.md)

