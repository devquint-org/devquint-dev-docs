# Пример плана: Microservices & Infrastructure

Для Docker, Kubernetes, DevOps, распределённых систем.

---

## Структура этапов

```
Этап 1: Foundation (VPC, cloud setup, networking)

Этап 2: Shared infrastructure (service discovery, API gateway, monitoring)

Этап 3: Service template & commons (base library, errors, logging)

Этап 4-6: Service A (Domain → Infrastructure → API)

Этап 7-9: Service B (Domain → Infrastructure → API)

Этап 10-12: Service C (Domain → Infrastructure → API)

Этап 13: Service integration (inter-service communication, tracing)

Этап 14: Deployment (Docker, K8s, CI/CD)

Этап 15: Monitoring & operations (logging, alerts, runbooks)
```

---

## Таблица зависимостей (упрощённо)

| Этап | Зависит от | Условия |
|------|-----------|---------|
| 1 Foundation | - | VPC created, security groups configured |
| 2 Shared infra | 1 | Service discovery works, API gateway routing works |
| 3 Commons | 2 | Base library published, templates ready |
| 4-6 Service A | 2,3 | Service deployed, APIs working |
| 7-9 Service B | 2,3 | Service deployed, APIs working |
| 10-12 Service C | 2,3 | Service deployed, APIs working |
| 13 Integration | 4-12 | Inter-service calls work, tracing works |
| 14 Deployment | 13 | K8s deployment works, auto-scaling OK |
| 15 Monitoring | 1-14 | Metrics collected, alerts working |

---

## Параллелизм

```
Team Foundation: [1] → [2] → [3]
Team A: (ждет 3) → [4-6]
Team B: (ждет 3) → [7-9]
Team C: (ждет 3) → [10-12]
All: → [13] → [14] → [15]
```

Services A, B, C могут разрабатываться параллельно!

---

## Ключевые моменты

- **Foundation first** - без этого ничего не запустится
- **Shared infrastructure before services** - все сервисы зависят от этого
- **Commons library before services** - нужна для консистентности
- **Services параллельно** - главное преимущество микросервисов
- **Integration в конце** - после готовых сервисов
- **Deployment и monitoring в конце** - для интегрированной системы

---

Остальное используй от Backend примера - логика управления зависимостями и этапами одна и та же.

👉 [`backend.md`](backend.md) для деталей архитектурных решений.

