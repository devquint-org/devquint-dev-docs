# TypeScript Implementation Rules

**Purpose:** Rules for creating implementation plans for TypeScript/Node.js projects.

---

## Domain Layer Rules

### No `any` Type

**Rule:** Never use `any`. Use specific types or `unknown` with type guards.

```typescript
// BAD
function process(data: any) { return data.value; }

// GOOD
interface ProcessInput {
  value: string;
}
function process(data: ProcessInput): string { return data.value; }
```

### Use Interfaces for Data Types

**Rule:** Use `interface` instead of `type` alias for data structures (DTOs, entities, inputs).

```typescript
// GOOD - Interface for data
interface CreateUserInput {
  name: string;
  email: string;
  role: UserRole;
}

interface User {
  id: UserId;
  name: string;
  email: string;
  role: UserRole;
  createdAt: Date;
}

// Use type for unions, intersections, mapped types
type UserResponse = User & { token: string };
```

### Strict Null Checks

**Rule:** Enable `strictNullChecks` in tsconfig.json. Handle null/undefined explicitly.

```typescript
// BAD - may be undefined
function getName(user: User): string {
  return user.name; // What if user.name is undefined?
}

// GOOD - explicit handling
function getName(user: User): string {
  if (!user.name) {
    throw new UserNameNotFoundError();
  }
  return user.name;
}
```

---

## Project Structure

```
src/
├── domain/          # Pure business logic (no framework)
│   ├── entities/
│   ├── value-objects/
│   ├── interfaces/
│   └── errors/
├── application/     # Use cases, services
│   └── services/
├── infrastructure/  # DB, external APIs, frameworks
│   ├── repositories/
│   ├── http/
│   └── config/
├── api/             # Controllers, routes, CLI
│   ├── controllers/
│   └── routes/
└── shared/          # Utilities, types used everywhere
    └── types/
```

---

## TypeScript-Specific Principles

### Dependency Injection

**Rule:** Use constructor injection for testability.

```typescript
// GOOD - Inject dependencies
interface UserRepository {
  findById(id: UserId): Promise<User | null>;
}

class UserService {
  constructor(private userRepository: UserRepository) {}

  async getUser(id: UserId): Promise<User> {
    const user = await this.userRepository.findById(id);
    if (!user) throw new UserNotFoundError(id);
    return user;
  }
}

// Test
const mockRepo: UserRepository = { findById: async () => null };
const service = new UserService(mockRepo);
```

### Branded Types for IDs

**Rule:** Use branded types to prevent mixing up different IDs.

```typescript
// GOOD - Branded types
interface UserIdBrand {
  readonly UserId: unique symbol;
}

type UserId = string & UserIdBrand;

interface OrderIdBrand {
  readonly OrderId: unique symbol;
}

type OrderId = string & OrderIdBrand;

// Now you cannot accidentally pass UserId where OrderId is expected
function getOrder(orderId: OrderId): Promise<Order> { /* ... */ }
const userId: UserId = "user-123" as UserId;
getOrder(userId); // TypeScript error!
```

---

## Testing Patterns

### Unit Tests

**Rule:** Test domain logic without framework or database.

```typescript
// domain/user.ts
export class User {
  constructor(
    public readonly id: UserId,
    public readonly email: Email
  ) {}

  isAdmin(): boolean {
    return this.email.localPart === "admin";
  }
}

// domain/user.test.ts
describe("User", () => {
  describe("isAdmin", () => {
    it("returns true for admin email", () => {
      const user = new User(
        "user-123" as UserId,
        new Email("admin@company.com")
      );
      expect(user.isAdmin()).toBe(true);
    });

    it("returns false for non-admin email", () => {
      const user = new User(
        "user-123" as UserId,
        new Email("user@company.com")
      );
      expect(user.isAdmin()).toBe(false);
    });
  });
});
```

### Integration Tests

**Rule:** Test infrastructure with real or mocked external services.

```typescript
// infrastructure/user.repository.test.ts
describe("UserRepository", () => {
  it("saves and retrieves user", async () => {
    const repo = new PrismaUserRepository(prisma);
    const user = new User(/* ... */);

    await repo.save(user);
    const found = await repo.findById(user.id);

    expect(found).toEqual(user);
  });
});
```

---

## Common Patterns

### Result Type Pattern

**Rule:** Use Result type for operations that may fail.

```typescript
// domain/result.ts
class Result<T, E = Error> {
  private constructor(
    private readonly _value: T | null,
    private readonly _error: E | null
  ) {
    if (!(_value || _error)) throw new Error("Invalid Result");
  }

  static ok<T>(value: T): Result<T> {
    return new Result(value, null);
  }

  static fail<T, E = Error>(error: E): Result<T, E> {
    return new Result(null, error);
  }

  isOk(): boolean { return this._error === null; }
  isErr(): boolean { return this._error !== null; }

  unwrap(): T {
    if (this._error) throw this._error;
    return this._value!;
  }

  map<U>(fn: (t: T) => U): Result<U, E> {
    return this.isOk() ? Result.ok(fn(this._value!)) : Result.fail(this._error!);
  }
}

// Usage
function createUser(input: CreateUserInput): Result<User, ValidationError> {
  const email = Email.create(input.email);
  if (email.isErr()) return Result.fail(email.error);

  const user = new User(UserId.generate(), email.value);
  return Result.ok(user);
}
```

### Repository Pattern

**Rule:** Abstract data access behind repository interfaces.

```typescript
// domain/repositories/user.repository.interface.ts
interface UserRepository {
  findById(id: UserId): Promise<User | null>;
  findByEmail(email: Email): Promise<User | null>;
  save(user: User): Promise<void>;
  delete(id: UserId): Promise<void>;
}

// infrastructure/repositories/prisma.user.repository.ts
class PrismaUserRepository implements UserRepository {
  async findById(id: UserId): Promise<User | null> {
    const dto = await prisma.user.findUnique({
      where: { id: id.value }
    });
    if (!dto) return null;
    return User.fromDto(dto);
  }

  // ... other methods
}
```

---

## Dependency Table Template

| Stage | Depends On | Completion Criteria |
|-------|------------|---------------------|
| 1 Infrastructure | - | tsconfig.json validated, ESLint passed, Jest configured |
| 2 Domain | 1 | All entities defined, unit tests >80% coverage |
| 3 Infrastructure | 1, 2 | Repositories implemented, integration tests pass |
| 4 Application | 2, 3 | Services tested, use cases verified |
| 5 API | 4 | Endpoints working, OpenAPI spec matches |
| 6 Auth | 1, 5 | JWT implemented, security review passed |
| 7 Testing | 1-6 | E2E tests pass, coverage >80% |
| 8 Deployment | 7 | Docker build succeeds, CI/CD passes |

---

## tsconfig.json Standards

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "**/*.test.ts"]
}
```
