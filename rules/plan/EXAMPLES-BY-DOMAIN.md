# 📚 Примеры планов для разных доменов и типов проектов

Готовые примеры как применять 10 универсальных правил для конкретных типов систем.

---

## 1. Backend API (REST / GraphQL)

### Структура плана

```
Этап 1: Инфраструктура
├─ Docker setup
├─ Config management
├─ Logger configuration
└─ Health check base

Этап 2: Database
├─ Schema design
├─ Migrations framework
└─ Seed data scripts

Этап 3: Domain layer
├─ Models/Entities
├─ Business logic
└─ Value objects

Этап 4: Infrastructure layer
├─ Repository implementations
├─ External API clients
└─ Cache layer

Этап 5: Application layer
├─ Business services
└─ Use cases

Этап 6: API layer
├─ REST endpoints
├─ Request validation
└─ Response formatting

Этап 7: Authentication & Authorization
├─ JWT setup
├─ Permission checks
└─ RBAC

Этап 8: Testing
├─ Unit tests
├─ Integration tests
└─ E2E tests

Этап 9: Documentation & Monitoring
├─ OpenAPI spec
├─ Monitoring setup
└─ Logging

Этап 10: Deployment
├─ CI/CD pipeline
├─ Production setup
└─ Rollback procedures
```

### Таблица зависимостей

| Этап | Зависит от | Тесты | Разработчики |
|------|-----------|-------|--------------|
| 1 Infrastructure | - | Smoke tests | DevOps |
| 2 Database | 1 | Migration tests | DBA / Backend |
| 3 Domain | 1 | Unit tests | Backend (senior) |
| 4 Infrastructure impl | 2,3 | Integration tests | Backend |
| 5 Application | 3,4 | Unit tests | Backend |
| 6 API | 5 | Integration tests | Backend / QA |
| 7 Auth | 1,6 | Security tests | Backend / Security |
| 8 Testing | 1-7 | - | QA |
| 9 Documentation | 1-7 | - | Tech writer / Backend |
| 10 Deployment | 9 | Smoke tests prod | DevOps |

### Параллельная разработка

```
Время: [████][████][████][████][████][████]

Команда A (Backend 1): Этап 1 → Этап 2 → Этап 3 → Этап 5 → Этап 6
                       (может начать после этапа 3)
                       
Команда B (Backend 2): ─────────────────→ Этап 4 → Этап 6
                       (может начать после этапа 3)
                       
Команда C (QA):        ────────────────────────────→ Этап 8
                       (может начать после этапа 6)
```

---

## 2. Frontend App (React / Vue / Angular)

### Структура плана

```
Этап 1: Project setup
├─ Build tool configuration (Vite/Webpack)
├─ ESLint + Prettier
├─ Testing framework setup
└─ CI/CD basics

Этап 2: Core infrastructure
├─ HTTP client setup
├─ Error handling
├─ Logger
├─ Environment config
└─ Auth interceptor

Этап 3: Shared components library
├─ Button, Input, Modal, etc.
├─ Typography system
├─ Color system
└─ Component tests

Этап 4: State management setup
├─ Redux/Zustand/Pinia initialization
├─ Store structure
└─ DevTools configuration

Этап 5: Routing
├─ Route configuration
├─ Protected routes
├─ Lazy loading
└─ Breadcrumbs

Этап 6: Feature 1 - Pages & Screens
├─ Feature 1 page structure
├─ Feature 1 state (uses stage 4)
├─ Feature 1 components (uses stage 3)
└─ Feature 1 integration (uses stage 2)

Этап 7: Feature 2 (repeat stage 6)

Этап 8: Advanced features
├─ Offline support
├─ Caching strategy
├─ Service worker
└─ PWA setup

Этап 9: Optimization
├─ Code splitting
├─ Image optimization
├─ Performance monitoring
└─ Bundle analysis

Этап 10: Documentation & Deployment
├─ Storybook
├─ README
├─ Build optimization
└─ Production deployment
```

### Таблица зависимостей

| Этап | Зависит от | Тесты | Разработчики |
|------|-----------|-------|--------------|
| 1 Setup | - | Build test | Frontend lead |
| 2 Infrastructure | 1 | Unit tests | Frontend (senior) |
| 3 Components | 1,2 | Component tests | Frontend / Designer |
| 4 State mgmt | 1,2 | Unit tests | Frontend |
| 5 Routing | 1,2,4 | Navigation tests | Frontend |
| 6 Feature 1 | 2,3,4,5 | E2E tests | Frontend |
| 7 Feature 2 | 2,3,4,5 | E2E tests | Frontend |
| 8 Advanced | 1-7 | Feature tests | Frontend (senior) |
| 9 Optimization | 1-8 | Performance tests | Frontend / DevOps |
| 10 Deployment | 9 | Smoke tests | DevOps / Frontend |

---

## 3. Mobile App (React Native / Flutter / Native)

### Структура плана

```
Этап 1: Project setup & Build
├─ IDE setup (Xcode / Android Studio)
├─ Build configuration
├─ Device testing setup
└─ CI/CD for mobile

Этап 2: Core infrastructure
├─ HTTP client
├─ Local storage
├─ Logging
├─ Error reporting (Sentry)
└─ Analytics

Этап 3: Authentication
├─ Login screen
├─ Sign up flow
├─ Token storage
├─ Biometric auth
└─ Session management

Этап 4: Navigation
├─ Tab/Stack navigation
├─ Deep linking
├─ Protected routes
└─ Splash screen

Этап 5: Design system
├─ Typography
├─ Colors
├─ Components library
└─ Theming (light/dark)

Этап 6: Feature 1
├─ Screens
├─ Local state
├─ API integration
└─ Offline support

Этап 7: Feature 2 (repeat stage 6)

Этап 8: Push notifications
├─ Setup (Firebase Cloud Messaging)
├─ Local notifications
├─ In-app messaging
└─ Notification center

Этап 9: Testing & Quality
├─ Unit tests
├─ Integration tests
├─ UI tests
└─ Performance profiling

Этап 10: Release
├─ App signing
├─ Store submission
├─ Analytics setup
└─ Crash reporting
```

---

## 4. Data Pipeline / ETL

### Структура плана

```
Этап 1: Infrastructure
├─ Data warehouse setup
├─ Message broker setup
├─ Monitoring setup
└─ Logging infrastructure

Этап 2: Source connectors
├─ Database connections
├─ API connections
├─ File system connections
└─ Connection tests

Этап 3: Data models
├─ Source schema definition
├─ Transformation rules
├─ Data quality rules
└─ Schema validation

Этап 4: Transformation layer
├─ ETL jobs for source 1
├─ ETL jobs for source 2
├─ Data cleaning
└─ Data aggregation

Этап 5: Storage layer
├─ Data warehouse schema
├─ Partitioning strategy
├─ Indexing
└─ Storage optimization

Этап 6: Loading
├─ Batch load jobs
├─ Streaming pipelines
├─ Error handling
└─ Retry mechanisms

Этап 7: Data quality
├─ Validation rules
├─ Anomaly detection
├─ Data profiling
└─ Alerts

Этап 8: Analytics & BI
├─ Aggregate tables
├─ Dashboards
├─ Reports
└─ Query optimization

Этап 9: Monitoring
├─ Pipeline health checks
├─ Data freshness alerts
├─ Performance monitoring
└─ Cost analysis

Этап 10: Documentation & Governance
├─ Data dictionary
├─ Lineage documentation
├─ Access control
└─ Compliance
```

---

## 5. Microservices Architecture

### Структура плана

```
Этап 1: Shared infrastructure
├─ Service discovery
├─ API gateway
├─ Message broker
├─ Logging & monitoring
└─ Config server

Этап 2: Service template & commons
├─ Base service template
├─ Common libraries
├─ Error handling
├─ Logging
└─ Metrics framework

Этап 3: Service A - Domain
├─ Models
├─ Business logic
├─ Interfaces
└─ Unit tests

Этап 4: Service A - Infra
├─ Database
├─ Repository impl
├─ External clients
└─ Integration tests

Этап 5: Service A - API
├─ gRPC definitions
├─ REST endpoints
├─ API validation
└─ E2E tests

Этап 6: Service B (repeat 3-5)
Этап 7: Service C (repeat 3-5)

Этап 8: Service integration
├─ Inter-service communication
├─ Distributed tracing
├─ Circuit breakers
└─ Integration tests

Этап 9: Deployment
├─ Docker setup for each service
├─ Kubernetes deployment
├─ Service scaling
└─ Blue-green deployment

Этап 10: Monitoring & Operations
├─ Distributed logging
├─ Health checks
├─ Alerting
└─ Runbooks
```

### Параллельная разработка

```
Команда A: Этап 1 → Этап 2 → Этап 3,4,5 (Service A)
                                ↓
                              Этап 8 (integration testing)

Команда B: (ждет этап 2) → Этап 3,4,5 (Service B)
                                ↓
                              Этап 8 (integration testing)

Команда C: (ждет этап 2) → Этап 3,4,5 (Service C)
                                ↓
                              Этап 8 (integration testing)

DevOps:   Этап 1 → ... → Этап 9 (deployment) → Этап 10
```

---

## 6. Machine Learning Pipeline

### Структура плана

```
Этап 1: Infrastructure
├─ GPU/Compute setup
├─ Data storage
├─ Experiment tracking
└─ Model registry

Этап 2: Data pipeline
├─ Data collection
├─ Data cleaning
├─ Feature engineering
└─ Data validation

Этап 3: Model development
├─ Baseline model
├─ Model exploration
├─ Hyperparameter tuning
└─ Model evaluation

Этап 4: Model optimization
├─ Quantization
├─ Pruning
├─ Distillation
└─ ONNX conversion

Этап 5: Model serving
├─ REST API
├─ Batch prediction
├─ Real-time streaming
└─ Model versioning

Этап 6: Monitoring
├─ Prediction monitoring
├─ Data drift detection
├─ Model performance tracking
└─ Retraining triggers

Этап 7: Integration
├─ Integration with app
├─ A/B testing
├─ Canary deployment
└─ Rollback procedures

Этап 8: Documentation
├─ Model card
├─ Training documentation
├─ Performance benchmarks
└─ Usage guide
```

---

## 7. DevOps / Infrastructure

### Структура плана

```
Этап 1: Foundation
├─ Cloud account setup
├─ VPC configuration
├─ Security groups
└─ IAM roles

Этап 2: Compute
├─ VM/Container registry setup
├─ Kubernetes cluster
├─ Node configuration
└─ Autoscaling

Этап 3: Storage
├─ Database setup
├─ Object storage
├─ Backup strategy
└─ Disaster recovery

Этап 4: Networking
├─ Load balancer
├─ DNS setup
├─ CDN
└─ VPN

Этап 5: Monitoring & Logging
├─ Metrics collection
├─ Log aggregation
├─ Alerting rules
└─ Dashboards

Этап 6: CI/CD
├─ Git setup
├─ Build pipeline
├─ Deployment automation
└─ Rollback procedures

Этап 7: Security
├─ TLS/SSL
├─ Secret management
├─ Network policies
└─ Security scanning

Этап 8: Disaster Recovery
├─ Backup automation
├─ Recovery testing
├─ Documentation
└─ Runbooks

Этап 9: Documentation
├─ Architecture documentation
├─ Runbooks
├─ Troubleshooting guides
└─ Capacity planning

Этап 10: Optimization
├─ Cost analysis
├─ Performance tuning
├─ Security hardening
└─ Compliance
```

---

## 8. Saas Feature Development

### Структура плана

```
Этап 1: Discovery & Design
├─ Requirements gathering
├─ Wireframes
├─ Design mockups
├─ Prototyping
└─ Stakeholder approval

Этап 2: Database schema
├─ Schema design
├─ Migrations
├─ Indexes
└─ Performance optimization

Этап 3: Backend API
├─ API endpoints definition
├─ Input validation
├─ Business logic
└─ Error handling

Этап 4: Backend implementation
├─ Implement endpoints
├─ Database queries
├─ Caching
└─ Backend tests

Этап 5: Frontend design system updates
├─ New components (if needed)
├─ Styling
├─ Responsive design
└─ Accessibility

Этап 6: Frontend implementation
├─ UI components
├─ API integration
├─ State management
└─ Frontend tests

Этап 7: Feature flags & rollout
├─ Feature flag setup
├─ Canary deployment
├─ Monitoring
└─ Gradual rollout

Этап 8: Documentation
├─ API documentation
├─ User documentation
├─ Admin documentation
└─ Internal runbooks

Этап 9: Testing & QA
├─ E2E testing
├─ Browser testing
├─ Device testing
└─ Performance testing

Этап 10: Release & Monitoring
├─ Release notes
├─ User onboarding
├─ Monitoring setup
└─ Support documentation
```

---

## 9. Library / Package Development

### Структура плана

```
Этап 1: Foundation
├─ Build system setup (npm/pip/cargo)
├─ Linting & formatting
├─ Testing framework
└─ CI/CD for package

Этап 2: Core module
├─ Main API design
├─ Core logic
├─ Error handling
└─ Documentation

Этап 3: Utilities module (if needed)
├─ Helper functions
├─ Utilities
├─ Constants
└─ Tests

Этап 4: Testing
├─ Unit tests (> 80% coverage)
├─ Integration tests
├─ Edge cases
└─ Performance tests

Этап 5: Documentation
├─ API documentation
├─ Usage examples
├─ Migration guides
└─ Contributing guide

Этап 6: Type definitions
├─ TypeScript types
├─ JSDoc comments
├─ Type exports
└─ Types testing

Этап 7: Optimization
├─ Bundle size optimization
├─ Performance benchmarks
├─ Tree-shaking support
└─ Performance documentation

Этап 8: Publishing
├─ Version management
├─ CHANGELOG
├─ Release process
└─ npm/PyPI publishing

Этап 9: Examples & Demos
├─ Example projects
├─ Demo applications
├─ Storybook/Sandbox
└─ Tutorial content

Этап 10: Community & Support
├─ Issue templates
├─ Contributing guidelines
├─ Code of conduct
└─ Support channels
```

---

## 10. Game Development

### Структура плана

```
Этап 1: Engine & Project setup
├─ Engine installation (Unity/Unreal/Godot)
├─ Project structure
├─ Build configuration
└─ Version control

Этап 2: Core gameplay mechanics
├─ Player controller
├─ Input handling
├─ Physics setup
└─ Collision detection

Этап 3: Game objects & entities
├─ Player entity
├─ Enemy entities
├─ Collectibles
└─ Hazards

Этап 4: Level design
├─ Level structure
├─ Level layout
├─ Spawn points
└─ Exit/checkpoints

Этап 5: Gameplay loops
├─ Combat system
├─ Scoring system
├─ Health system
└─ UI integration

Этап 6: Graphics & Audio
├─ 3D models
├─ Animations
├─ Sound effects
└─ Background music

Этап 7: UI & Menus
├─ Main menu
├─ In-game UI
├─ Pause menu
└─ Settings menu

Этап 8: Level progression
├─ Multiple levels
├─ Difficulty progression
├─ Boss encounters
└─ Story progression

Этап 9: Polish & optimization
├─ Performance optimization
├─ Graphics optimization
├─ Load time optimization
└─ Bug fixes

Этап 10: Release
├─ Testing on target platforms
├─ Platform-specific builds
├─ Store submission
└─ Post-launch support
```

---

## Как использовать эти примеры

### Для своего проекта:

1. **Найди похожий домен** в этом документе
2. **Скопируй структуру** (модификуй под свои нужды)
3. **Создай таблицу зависимостей** для своего проекта
4. **Определи параллельные потоки** работы команды
5. **Применяй 10 универсальных правил** из UNIVERSAL-IMPLEMENTATION-RULES.md

### Шаблон адаптации:

```markdown
# План реализации: [Твой проект]

## Структура плана

[Скопируй структуру из примера]

## Таблица зависимостей

[Адаптируй таблицу для своего проекта]

## Команды и параллелизм

[Определи кто что делает параллельно]

## Условия завершения

[Для каждого этапа - конкретные условия]

## Архитектурные решения

[Объясни почему ты выбрал эту структуру]
```

---

## Итого

- **10 примеров для разных доменов**
- **Каждый адаптирован к специфике домена**
- **Применяются все 10 универсальных правил**
- **Готовы к копипасте и модификации**
- **Включают параллельную разработку где возможно**

Выбери похожий пример и адаптируй под свой проект! 🚀

