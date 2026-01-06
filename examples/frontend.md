# Пример плана: Frontend (React/Vue/Angular)

Типичная структура для фронтенд-приложений.

---

## Структура этапов

```
Этап 1: Setup
  - Build, ESLint, testing framework

Этап 2: Infrastructure (HTTP client, error handling, logger)

Этап 3: State management (Redux/Zustand/Pinia)

Этап 4: Components library (shared UI components)

Этап 5: Pages & screens

Этап 6: Routing & navigation

Этап 7: API integration

Этап 8: Advanced features (offline, PWA, optimization)

Этап 9: Testing & QA
```

---

## Таблица зависимостей

| Этап | Зависит от | Условия |
|------|-----------|---------|
| 1 Setup | - | Build succeeds, linter passes, tests run |
| 2 Infrastructure | 1 | HTTP client works, error handling tested |
| 3 State mgmt | 1,2 | Store initialized, DevTools work, tests pass |
| 4 Components | 1,2 | Component library built, stories written |
| 5 Pages | 2,3,4 | Pages render, state flows work |
| 6 Routing | 1,5 | Navigation works, routes protected |
| 7 API integration | 2,5 | API calls work, caching works |
| 8 Advanced | 1-7 | Offline mode works, PWA manifest ready |
| 9 Testing & QA | 1-8 | E2E tests pass, performance OK |

---

## Ключевые моменты

- **Не создавай компоненты без state management** (хаос будет)
- **API integration после pages** (нужны готовые компоненты)
- **Routing после pages** (нужны готовые страницы)
- **Testing в конце** (тестируешь интегрированное приложение)

Остальное похоже на Backend - используй [`backend.md`](backend.md) как reference.

