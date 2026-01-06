# 🌍 Универсальные правила создания планов реализации

Независимый от архитектуры, технологии и типа проекта набор правил для создания пошаговых планов.

**Применимо для:**
- ✅ Бэкенд систем (Node.js, Python, Java, Go, Rust, C#)
- ✅ Фронтенд приложений (React, Vue, Angular, Svelte, Next.js)
- ✅ Мобильных приложений (iOS, Android, React Native, Flutter)
- ✅ Desktop приложений (Electron, Qt, WPF)
- ✅ Микросервисной архитектуры
- ✅ Монолитных систем
- ✅ Отдельных модулей и библиотек
- ✅ DevOps и Infrastructure
- ✅ Data science и ML pipelines
- ✅ Игр и real-time систем

---

## Правило 1: Линейная независимость этапов

### Описание

Каждый этап (phase/sprint/milestone) должен быть **полностью завершаемым и тестируемым** независимо от последующих этапов.

### Антипаттерн ❌

```
Фаза 1: Создать UI компоненты
  ↓
Фаза 2: Создать API endpoints
  ↓
Фаза 3: Создать бизнес-логику (НУЖНА ДЛЯ ФАЗЫ 1 И 2!)
```

**Проблема:** Фаза 3 требуется для завершения фаз 1 и 2 → невозможно закончить без рефакторинга.

### Правильный паттерн ✅

```
Фаза 1: Создать бизнес-логику и модели данных
  ↓
Фаза 2: Создать API endpoints (используют бизнес-логику из фазы 1)
  ↓
Фаза 3: Создать UI компоненты (используют endpoints из фазы 2)
```

**Преимущество:** Каждая фаза может быть независимо закончена и протестирована.

### Как проверить

```
Для каждой фазы N:
1. Зависит ли фаза N от фаз N+1, N+2, ...?
   → Если да ❌ ПЕРЕСТРОЙ!
2. Может ли фаза N быть протестирована без фаз N+1, N+2?
   → Если нет ❌ ПЕРЕСТРОЙ!
3. Если фаза N выполнена, может ли команда начать фазу N+1?
   → Если нет ❌ ПЕРЕСТРОЙ!
```

### Примеры для разных доменов

#### Backend API

```
❌ Плохо:
- Фаза 1: Создать endpoints
- Фаза 2: Создать БД миграции
- Фаза 3: Создать бизнес-логику

✅ Хорошо:
- Фаза 1: Создать domain модели и бизнес-логику (без endpoints)
- Фаза 2: Создать БД schema и миграции
- Фаза 3: Создать API endpoints (используют фазы 1-2)
- Фаза 4: Integration тесты
```

#### Frontend App

```
❌ Плохо:
- Фаза 1: Создать UI компоненты
- Фаза 2: Создать роутинг
- Фаза 3: Создать state management (используется везде!)

✅ Хорошо:
- Фаза 1: Создать state management и типы данных
- Фаза 2: Создать базовые UI компоненты
- Фаза 3: Создать страницы и роутинг
- Фаза 4: Интеграция с API
```

#### Mobile App

```
❌ Плохо:
- Фаза 1: Создать UI screens
- Фаза 2: Создать навигацию
- Фаза 3: Создать authentication (нужна везде!)

✅ Хорошо:
- Фаза 1: Создать authentication и user store
- Фаза 2: Создать UI компоненты
- Фаза 3: Создать screens и навигацию
- Фаза 4: Интеграция с API
```

---

## Правило 2: Разделение слоев (Layer separation)

### Описание

Всегда разделяй на слои. Нижние слои НЕ должны зависеть от верхних.

### Универсальная иерархия слоев

```
┌─────────────────────────────┐
│ API Layer / Controllers     │ (HTTP, WebSocket, CLI, мобильный UI)
├─────────────────────────────┤
│ Application Layer / Services│ (Бизнес-логика, агрегаторы)
├─────────────────────────────┤
│ Domain Layer / Models       │ (Entities, Value Objects, Interfaces)
├─────────────────────────────┤
│ Infrastructure Layer        │ (БД, API clients, File system)
└─────────────────────────────┘
```

**Правило стрелок:** Стрелки зависимостей указывают ТОЛЬКО ВНИЗ (или горизонтально в пределах слоя).

```
❌ НИКОГДА:
  Domain → Application (стрелка вверх)
  
✅ ВСЕГДА:
  API → Application → Domain → Infrastructure
                  ↓
           Infrastructure (не зависит от никого выше)
```

### Примеры кода

#### Backend (Node.js / Express)

```
❌ Плохо - Domain зависит от Express:
// domain/user.ts
import { Request, Response } from 'express'; // ❌ зависимость вверх!

export class User {
  static create(req: Request, res: Response) {
    const name = req.body.name; // тесно связан с Express
  }
}

✅ Хорошо - Разделены слои:
// domain/user.ts (No dependencies!)
export interface CreateUserInput {
  name: string;
  email: string;
}

export class User {
  constructor(public name: string, public email: string) {}
  
  static create(input: CreateUserInput): User {
    return new User(input.name, input.email);
  }
}

// application/user-service.ts
import { User, CreateUserInput } from '../domain/user';
import { userRepository } from '../infrastructure/db';

export class UserService {
  async create(input: CreateUserInput): Promise<User> {
    const user = User.create(input);
    return userRepository.save(user);
  }
}

// api/user-controller.ts
import { Router } from 'express';
import { UserService } from '../application/user-service';

const router = Router();
router.post('/users', async (req, res) => {
  try {
    const user = await userService.create(req.body);
    res.json(user);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});
```

#### Frontend (React)

```
❌ Плохо - Domain знает о React:
// domain/cart.ts
import { useState } from 'react'; // ❌ зависимость вверх!

export function useCart() {
  const [items, setItems] = useState([]);
  return { items, setItems };
}

✅ Хорошо - Разделены слои:
// domain/cart-item.ts (No React!)
export interface CartItem {
  id: string;
  productId: string;
  quantity: number;
  price: number;
}

export interface Cart {
  items: CartItem[];
  addItem(item: CartItem): void;
  removeItem(itemId: string): void;
  getTotalPrice(): number;
}

export class CartImpl implements Cart {
  items: CartItem[] = [];
  
  addItem(item: CartItem) {
    this.items.push(item);
  }
  
  removeItem(itemId: string) {
    this.items = this.items.filter(i => i.id !== itemId);
  }
  
  getTotalPrice() {
    return this.items.reduce((sum, item) => sum + item.price, 0);
  }
}

// application/cart-service.ts
import { Cart, CartItem } from '../domain/cart-item';

export class CartService {
  constructor(private cart: Cart) {}
  
  addProduct(productId: string, quantity: number, price: number) {
    this.cart.addItem({ id: Date.now().toString(), productId, quantity, price });
  }
  
  getTotal() {
    return this.cart.getTotalPrice();
  }
}

// ui/CartComponent.tsx (React-specific)
import { useState } from 'react';
import { CartService } from '../application/cart-service';
import { CartImpl } from '../domain/cart-item';

export function CartComponent() {
  const [cart] = useState(() => new CartImpl());
  const [cartService] = useState(() => new CartService(cart));
  
  return (
    <div>
      <button onClick={() => cartService.addProduct('1', 1, 10)}>
        Add to cart
      </button>
      <p>Total: ${cartService.getTotal()}</p>
    </div>
  );
}
```

#### Python (Django / FastAPI)

```
❌ Плохо:
# models.py
from django.db import models
from django.http import JsonResponse

class Order(models.Model): # ❌ Database-specific
    status = models.CharField(max_length=20)
    
    def to_response(self): # ❌ HTTP-specific
        return JsonResponse({'status': self.status})

✅ Хорошо:
# domain/order.py
from dataclasses import dataclass
from enum import Enum

class OrderStatus(str, Enum):
    PENDING = "pending"
    COMPLETED = "completed"

@dataclass
class Order:
    id: str
    status: OrderStatus
    items: list
    
    def complete(self):
        self.status = OrderStatus.COMPLETED

# application/order_service.py
from .domain.order import Order, OrderStatus

class OrderService:
    def __init__(self, repository):
        self.repository = repository
    
    async def complete_order(self, order_id: str) -> Order:
        order = await self.repository.get(order_id)
        order.complete()
        await self.repository.save(order)
        return order

# api/order_handler.py (FastAPI)
from fastapi import APIRouter
from ..application.order_service import OrderService

router = APIRouter()

@router.post("/orders/{order_id}/complete")
async def complete_order(order_id: str, service: OrderService):
    order = await service.complete_order(order_id)
    return {"id": order.id, "status": order.status}
```

---

## Правило 3: Разделение инфраструктуры на отдельные этапы

### Описание

Если есть shared infrastructure (auth, logging, error handling, config), выделите для них отдельный этап ПЕРЕД использованием.

### Общие инфраструктурные сервисы

```
// Обычно нужны везде:
- Authentication / Authorization
- Logging и Monitoring
- Error handling и Recovery
- Configuration management
- Database / Data storage
- Caching
- Message queues
- Rate limiting
- Health checks
- Encryption
```

### Структура этапов

```
Этап 1: Инфраструктура и конфигурация
  - Config management
  - Logger setup
  - Error handler
  - Health check base
  
Этап 2: Core domain (без зависимости от инфры!)
  - Domain models
  - Business logic
  - Interfaces для repositories
  
Этап 3: Infrastructure реализация
  - Database layer
  - Repository implementations
  - External API clients
  
Этап 4: Application services
  - Business orchestration
  
Этап 5: API endpoints
  - Controllers / Handlers
```

### Примеры

#### Microservice Architecture

```
Этап 1: Shared libraries и инфра
  - Logging library
  - Error handling utils
  - Config server connection
  - Health check framework

Этап 2: Service A - Domain
  - Models
  - Interfaces

Этап 3: Service A - Infrastructure
  - DB migrations
  - External API clients

Этап 4: Service A - Application
  - Services
  - Use cases

Этап 5: Service A - API
  - gRPC handlers
  - REST endpoints

Этап 6: Service B (repeat 2-5)
```

#### Frontend Feature

```
Этап 1: Shared infrastructure
  - API client setup (Axios/Fetch config)
  - Auth interceptor
  - Error handler
  - Logger
  
Этап 2: Domain
  - Types/Interfaces
  - Models
  - Validation
  
Этап 3: State management
  - Redux/Zustand store (uses Этап 2)
  
Этап 4: Components
  - UI components (stateless)
  - Hooks (uses Этап 3)
  
Этап 5: Pages/Features
  - Page components (uses Этап 4)
  - Feature integration
```

---

## Правило 4: Четкие условия завершения для каждого этапа

### Описание

Каждый этап должен иметь **конкретные, проверяемые** условия завершения.

### Плохие условия ❌

```
- "Работает"
- "Готово"
- "Реализовано"
- "Тесты пройдены"
- "No obvious bugs"
- "Looks good"
```

**Проблема:** Субъективно, не ясно что считается "готовым".

### Хорошие условия ✅

```
- "Все unit тесты пройдены (coverage > 80%)"
- "Integration тесты для всех happy paths пройдены"
- "Code review с минимум 1 reviewer"
- "Нет критичных security issues (OWASP Top 10)"
- "Performance тесты: response time < 200ms для 95th percentile"
- "E2E тесты на Chrome, Firefox, Safari"
- "Соответствует API контракту (OpenAPI spec)"
- "Документация обновлена"
- "Нет breaking changes для existing users"
```

### Шаблон условий

```
Условие для завершения [Этап N]:

**Функциональность:**
- [ ] Основной happy path работает (тест X описывает)
- [ ] Все error cases обработаны (тесты Y-Z описывают)

**Качество кода:**
- [ ] Unit тесты пройдены (> 80% coverage)
- [ ] Нет critical code smells (SonarQube)
- [ ] Type checking пройден (TypeScript strict / mypy)

**Интеграция:**
- [ ] API контракт соблюдается (Swagger/OpenAPI spec)
- [ ] Зависимые компоненты работают вместе
- [ ] Database миграции применены

**Документация:**
- [ ] README обновлен
- [ ] API документирована
- [ ] Edge cases описаны

**Мониторинг:**
- [ ] Metrics добавлены
- [ ] Логирование настроено
- [ ] Alerts настроены
```

---

## Правило 5: Отсутствие циклических зависимостей

### Описание

Зависимости должны быть **ациклическими** (DAG - Directed Acyclic Graph).

### Антипаттерн ❌

```
Этап A → Этап B → Этап C → Этап A (цикл!)
```

### Примеры циклических зависимостей

#### Frontend

```
❌ Плохо:
- Feature A Component → Feature B Store (uses A)
- Feature B Store → Feature A Action
- Feature A Action → Feature B Component (цикл!)

✅ Хорошо:
- Shared Store → Feature A Component → Feature A Action
- Shared Store → Feature B Component → Feature B Action
```

#### Backend

```
❌ Плохо:
- OrderService → UserService (gets user by ID)
- UserService → OrderService (counts user orders)
- OrderService → UserService (цикл!)

✅ Хорошо:
- UserRepository (no service deps)
- OrderRepository (no service deps)
- UserService → UserRepository
- OrderService → OrderRepository (не зависит от UserService!)
```

### Как найти циклы

```
Для каждого компонента/модуля X:
1. Составь список всех зависимостей (A, B, C, ...)
2. Для каждой зависимости, составь ее список зависимостей
3. Если X появился во втором списке → ЦИКЛ!
4. Перестрой архитектуру используя общий слой
```

---

## Правило 6: Разделение Concerns (Single Responsibility)

### Описание

Каждый компонент/сервис/модуль должен иметь **одну ответственность**.

### Антипаттерны ❌

#### Backend

```
// ❌ Всё в одном классе:
class UserController {
  @Post('/users')
  async createUser(req, res) {
    // Валидация
    if (!req.body.email) throw Error('Invalid');
    
    // Бизнес-логика
    const hashedPassword = bcrypt.hash(req.body.password);
    
    // БД операция
    const user = new User(req.body);
    await db.users.insert(user);
    
    // Email отправка
    await sendWelcomeEmail(user.email);
    
    // Response
    res.json(user);
  }
}
```

#### Frontend

```
// ❌ Всё в одном компоненте:
export function UserList() {
  const [users, setUsers] = useState([]);
  const [filter, setFilter] = useState('');
  const [sortBy, setSortBy] = useState('name');
  const [page, setPage] = useState(1);
  const [loading, setLoading] = useState(false);
  
  // Fetching логика
  useEffect(() => {
    fetch(`/api/users?filter=${filter}&sort=${sortBy}&page=${page}`)
      .then(r => r.json())
      .then(data => setUsers(data))
      .catch(err => alert(err));
  }, [filter, sortBy, page]);
  
  // Фильтрация
  const filtered = users.filter(u => u.name.includes(filter));
  
  // Сортировка
  const sorted = filtered.sort((a, b) => a[sortBy].localeCompare(b[sortBy]));
  
  // Рендеринг
  return (
    <div>
      <input onChange={e => setFilter(e.target.value)} />
      <table>
        {sorted.map(u => <tr><td>{u.name}</td></tr>)}
      </table>
    </div>
  );
}
```

### Правильные разделение ✅

#### Backend

```
// ✅ Разделены concerns:

// validation.ts
export function validateUserInput(data: any): void {
  if (!data.email) throw new ValidationError('Email required');
  if (!data.password) throw new ValidationError('Password required');
}

// user-service.ts
export class UserService {
  constructor(private repo: UserRepository, private emailService: EmailService) {}
  
  async createUser(input: CreateUserInput): Promise<User> {
    validateUserInput(input);
    const hashedPassword = await hashPassword(input.password);
    const user = new User(input.email, hashedPassword);
    await this.repo.save(user);
    await this.emailService.sendWelcome(user.email);
    return user;
  }
}

// user-controller.ts
@Controller('/users')
export class UserController {
  @Post()
  async create(@Body() input: CreateUserInput, @Res() res: Response) {
    const user = await this.userService.createUser(input);
    res.json(user);
  }
}
```

#### Frontend

```
// ✅ Разделены concerns:

// hooks/useUsers.ts
export function useUsers(filter: string, sortBy: string, page: number) {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    setLoading(true);
    fetch(`/api/users?filter=${filter}&sort=${sortBy}&page=${page}`)
      .then(r => r.json())
      .then(setUsers)
      .finally(() => setLoading(false));
  }, [filter, sortBy, page]);
  
  return { users, loading };
}

// utils/sortUsers.ts
export function sortUsers(users: User[], sortBy: string): User[] {
  return [...users].sort((a, b) => a[sortBy].localeCompare(b[sortBy]));
}

// components/UserTable.tsx
function UserTable({ users, onSort }: Props) {
  return (
    <table>
      {users.map(u => <tr><td>{u.name}</td></tr>)}
    </table>
  );
}

// components/UserListPage.tsx
export function UserListPage() {
  const [filter, setFilter] = useState('');
  const [sortBy, setSortBy] = useState('name');
  const [page, setPage] = useState(1);
  const { users, loading } = useUsers(filter, sortBy, page);
  const sortedUsers = sortUsers(users, sortBy);
  
  return (
    <div>
      <input onChange={e => setFilter(e.target.value)} />
      <UserTable users={sortedUsers} onSort={setSortBy} />
    </div>
  );
}
```

---

## Правило 7: Таблица зависимостей для каждого плана

### Описание

Явная таблица показывающая как этапы зависят друг от друга.

### Формат

```markdown
| Этап | Зависит от | Условия завершения |
|------|-----------|------------------|
| 1    | -         | Config готова |
| 2    | 1         | Все unit tests |
| 3    | 2         | Integration tests |
| 4    | 3         | API ready |
| 5    | 4         | E2E tests |
```

### Примеры для разных типов систем

#### SaaS Backend

```
| Этап | Зависит от | Условия |
|------|-----------|---------|
| 1 Infrastructure | - | Docker setup, config management |
| 2 Database schema | 1 | Migrations working, seed data |
| 3 Domain models | 1 | Unit tests passing |
| 4 Repositories | 2,3 | Integration tests with DB |
| 5 Business logic | 3,4 | Unit tests for services |
| 6 API endpoints | 5 | API spec verified |
| 7 Authentication | 1,6 | Security review passed |
| 8 Admin panel | 6 | E2E tests passing |
| 9 Documentation | 5,6,8 | README updated |
| 10 Deployment | 9 | Production ready |
```

#### Mobile App

```
| Этап | Зависит от | Условия |
|------|-----------|---------|
| 1 Project setup | - | Build on iOS and Android |
| 2 Navigation | 1 | All screens navigable |
| 3 Auth | 1 | Login/logout working |
| 4 Shared components | 1 | Component library |
| 5 Feature screens | 3,4 | Screens display correctly |
| 6 State management | 5 | Data flows correctly |
| 7 API integration | 6 | Network calls working |
| 8 Offline support | 7 | Cached data displays |
| 9 Push notifications | 3,7 | Notifications received |
| 10 Testing | 5-9 | All E2E tests pass |
```

---

## Правило 8: Каждый этап должен быть тестируемым

### Описание

После завершения этапа, должна существовать возможность написать осмысленные тесты.

### Типы тестов по этапам

```
Этап 1 - Инфраструктура
  └─ Smoke tests (все сервисы запущены)
  
Этап 2 - Domain
  └─ Unit tests (бизнес-логика корректна)
  
Этап 3 - Infrastructure
  └─ Integration tests (DB, API clients работают)
  
Этап 4 - Application
  └─ Unit tests (сервисы корректны)
  └─ Integration tests (работают с infrastructure)
  
Этап 5 - API
  └─ Unit tests (контроллеры корректны)
  └─ Integration tests (API endpoints работают)
  └─ E2E tests (full flow работает)
```

### Примеры

#### Backend

```typescript
// Этап 2: Domain
describe('Order domain', () => {
  it('should create order with valid items', () => {
    const order = new Order([
      { productId: '1', quantity: 2 }
    ]);
    expect(order.items.length).toBe(1);
  });
});

// Этап 3: Infrastructure  
describe('OrderRepository', () => {
  it('should save and retrieve order from DB', async () => {
    const order = new Order([...]);
    await repository.save(order);
    const retrieved = await repository.findById(order.id);
    expect(retrieved.id).toBe(order.id);
  });
});

// Этап 5: API
describe('POST /orders', () => {
  it('should create order via HTTP', async () => {
    const response = await request(app)
      .post('/orders')
      .send({ items: [...] });
    expect(response.status).toBe(201);
  });
});
```

#### Frontend

```typescript
// Этап 2: Domain
describe('Cart domain', () => {
  it('should add item to cart', () => {
    const cart = new Cart();
    cart.addItem({ id: '1', price: 10 });
    expect(cart.items.length).toBe(1);
  });
});

// Этап 4: State management
describe('CartStore', () => {
  it('should persist cart to localStorage', () => {
    const store = new CartStore();
    store.addItem({ id: '1', price: 10 });
    expect(localStorage.getItem('cart')).toBeDefined();
  });
});

// Этап 5: Components
describe('CartList component', () => {
  it('should render items', () => {
    const { getByText } = render(
      <CartList items={[{ id: '1', price: 10 }]} />
    );
    expect(getByText('$10')).toBeInTheDocument();
  });
});
```

---

## Правило 9: Избегай дублирования определений

### Описание

Каждый компонент/функция/константа должна быть определена **ровно один раз**.

### Антипаттерн ❌

```
Документ:

Этап 3: Создать UserService
- UserService определен с методами getUser(), createUser()

Этап 5: Создать Admin API
- UserService переопределен с дополнительными методами
```

**Проблема:** Где правда? Какую версию использовать?

### Правильно ✅

```
Этап 3: Создать UserService
- UserService определен с методами getUser(), createUser(), deleteUser()
- Unit тесты пройдены

Этап 5: Создать Admin API
- Использует UserService из этапа 3
- Admin API добавляет доп. endpoints для администрирования
```

---

## Правило 10: Документируй архитектурные решения

### Описание

Объясни **почему** ты выбрал такую структуру, а не другую.

### Что документировать

```markdown
## Архитектурные решения

### 1. Почему 3-слойная архитектура?
- Разделение concerns: бизнес-логика отдельно от фреймворка
- Тестируемость: можно тестировать domain без UI/DB
- Переиспользуемость: domain layer можно использовать везде

### 2. Почему отдельный этап для infrastructure?
- Dependencies: все компоненты, которые нужны в domain, должны быть готовы
- Тестируемость: domain может быть протестирован без реальной БД
- Flexibility: можно менять DB без изменения domain layer

### 3. Почему микросервисы вместо монолита?
- Масштабируемость: каждый сервис может масштабироваться независимо
- Разработка: разные команды работают параллельно
- Deployment: обновление одного сервиса не требует обновления всех

### 4. Почему Redux вместо Context API?
- DevTools: удобная отладка
- Time-travel: можно просмотреть историю состояния
- Middleware: можно логировать, отслеживать ошибки
```

---

## Шпаргалка: 10 правил на одной странице

```
1. ✅ Линейная независимость: Фаза N не зависит от N+1, N+2, ...
2. ✅ Разделение слоев: Стрелки зависимостей только ВНИЗ
3. ✅ Инфра на отдельных этапах: ДО использования в бизнес-логике
4. ✅ Четкие условия: Конкретные, проверяемые, не субъективные
5. ✅ No cycles: Зависимости должны быть DAG
6. ✅ Single Responsibility: Каждый компонент - одна ответственность
7. ✅ Таблица зависимостей: Явная и согласованная
8. ✅ Тестируемость: Каждый этап должен быть тестируемым
9. ✅ No duplicates: Каждый компонент определен один раз
10. ✅ Документируй решения: Объясни почему ты выбрал этот путь
```

---

## Итого

Эти 10 правил применяются **везде:**
- На бэкенде и фронтенде
- В монолитах и микросервисах
- На любом языке программирования
- Для любого типа приложения

**Суть:** Хороший план - это план, который позволяет разным людям работать параллельно без блокировок, тестировать независимо и легко рефакторить без масштабного ломания.

