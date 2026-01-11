# Python Implementation Rules

**Purpose:** Rules for creating implementation plans for Python projects.

---

## Domain Layer Rules

### Use dataclasses with frozen=True

**Rule:** Use immutable dataclasses for value objects and domain entities.

```python
# GOOD - Immutable value object
from dataclasses import dataclass

@dataclass(frozen=True)
class UserId:
    value: str

@dataclass(frozen=True)
class Email:
    local_part: str
    domain: str

    @property
    def full(self) -> str:
        return f"{self.local_part}@{self.domain}"

    @classmethod
    def from_string(cls, email_str: str) -> "Email":
        if "@" not in email_str:
            raise InvalidEmailError(f"Invalid email: {email_str}")
        local, domain = email_str.split("@", 1)
        return cls(local, domain)
```

### Use Protocol for Interfaces

**Rule:** Use Protocol for structural typing and repository interfaces.

```python
# GOOD - Protocol for repository interface
from typing import Protocol, runtime_checkable

@runtime_checkable
class UserRepository(Protocol):
    def find_by_id(self, user_id: UserId) -> User | None: ...
    def find_by_email(self, email: Email) -> User | None: ...
    def save(self, user: User) -> None: ...
    def delete(self, user_id: UserId) -> None: ...

# Implementation
class PostgresUserRepository:
    def __init__(self, session: Session):
        self._session = session

    def find_by_id(self, user_id: UserId) -> User | None:
        dto = self._session.query(UserDTO).filter_by(id=user_id.value).first()
        return User.from_dto(dto) if dto else None

    # ... other methods
```

### Type Hints Required

**Rule:** All functions and methods must have type hints.

```python
# GOOD - Full type hints
from typing import Annotated

def create_user(input: CreateUserInput) -> Result[User, ValidationError]:
    """Create a new user with validation."""
    email = Email.from_string(input.email)
    if email.is_err():
        return Err(email.unwrap_err())

    user = User(
        id=UserId.generate(),
        email=email.unwrap(),
        name=input.name,
        role=UserRole.USER,
    )
    return Ok(user)

# BAD - No type hints
def create_user(input):
    # ...
```

---

## Project Structure

```
src/
├── domain/          # Pure business logic (no framework)
│   ├── entities/
│   ├── value_objects/
│   ├── interfaces/
│   └── errors/
├── application/     # Use cases, services
│   └── services/
├── infrastructure/  # DB, external APIs, frameworks
│   ├── repositories/
│   ├── http/
│   └── config/
├── api/             # Controllers, routes, CLI
│   ├── routers/
│   └── controllers/
└── shared/          # Utilities, types used everywhere
    └── types/
```

---

## Python-Specific Principles

### Result Pattern

**Rule:** Use Result pattern for operations that may fail.

```python
# GOOD - Result pattern
from typing import TypeVar, Generic

T = TypeVar("T")
E = TypeVar("E", bound=Exception)


class Result(Generic[T, E]):
    """Result type for operations that can fail."""

    __slots__ = ("_value", "_error")

    def __init__(self, value: T | None, error: E | None):
        if value is None and error is None:
            raise ValueError("Result must have either value or error")
        self._value = value
        self._error = error

    @classmethod
    def ok(cls, value: T) -> "Result[T, E]":
        return cls(value, None)

    @classmethod
    def err(cls, error: E) -> "Result[T, E]":
        return cls(None, error)

    def is_ok(self) -> bool:
        return self._error is None

    def is_err(self) -> bool:
        return self._error is not None

    def unwrap(self) -> T:
        if self._error:
            raise self._error
        return self._value  # type: ignore

    def unwrap_err(self) -> E:
        if self._value is None:
            raise ValueError("No error present")
        return self._error  # type: ignore

    def map(self, f: Callable[[T], U]) -> "Result[U, E]":
        if self.is_ok():
            return Result.ok(f(self._value))  # type: ignore
        return self  # type: ignore
```

### Error Hierarchy

**Rule:** Create specific error classes for different failure modes.

```python
# domain/errors/__init__.py
from abc import ABC


class DomainError(Exception, ABC):
    """Base class for domain errors."""

    def __init__(self, message: str, context: dict | None = None):
        super().__init__(message)
        self.context = context or {}


class UserNotFoundError(DomainError):
    def __init__(self, user_id: UserId):
        super().__init__(f"User not found: {user_id.value}", {"user_id": user_id.value})


class UserAlreadyExistsError(DomainError):
    def __init__(self, email: Email):
        super().__init__(f"User already exists: {email.full}", {"email": email.full})


class ValidationError(DomainError):
    """Raised when input validation fails."""

    def __init__(self, field: str, message: str):
        super().__init__(f"Validation error for {field}: {message}", {"field": field})
        self.field = field
```

---

## Testing Patterns

### Unit Tests

**Rule:** Test domain logic without framework or database.

```python
# tests/domain/test_user.py
import pytest
from domain import User, Email, UserId


class TestUser:
    def test_is_admin_with_admin_email(self):
        user = User(
            id=UserId("user-123"),
            email=Email("admin", "company.com"),
        )
        assert user.is_admin() is True

    def test_is_admin_with_regular_email(self):
        user = User(
            id=UserId("user-123"),
            email=Email("user", "company.com"),
        )
        assert user.is_admin() is False

    def test_create_user_with_invalid_email_raises(self):
        with pytest.raises(InvalidEmailError):
            Email.from_string("not-an-email")
```

### Pytest Configuration

```python
# pytest.ini
[tool:pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts = -v --tb=short
filterwarnings =
    ignore::DeprecationWarning
```

---

## Dependency Injection

**Rule:** Use simple constructor injection for testability.

```python
# application/user_service.py
from typing import Protocol


class UserRepository(Protocol):
    def find_by_id(self, user_id: UserId) -> User | None: ...
    def save(self, user: User) -> None: ...


class UserService:
    def __init__(self, user_repository: UserRepository):
        self._repo = user_repository

    def get_user(self, user_id: UserId) -> User:
        user = self._repo.find_by_id(user_id)
        if user is None:
            raise UserNotFoundError(user_id)
        return user


# tests/application/test_user_service.py
class MockUserRepository(UserRepository):
    def __init__(self):
        self.users: dict[UserId, User] = {}

    def find_by_id(self, user_id: UserId) -> User | None:
        return self.users.get(user_id)

    def save(self, user: User) -> None:
        self.users[user.id] = user


def test_get_user_returns_user_when_found():
    repo = MockUserRepository()
    service = UserService(repo)

    user = User(id=UserId("123"), email=Email("test", "test.com"))
    repo.save(user)

    result = service.get_user(UserId("123"))
    assert result == user
```

---

## pyproject.toml Standards

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
python_functions = ["test_*"]
addopts = "-v --tb=short"

[tool.mypy]
strict = true
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
disallow_incomplete_defs = true

[tool.ruff]
target-version = "py311"
select = ["E", "F", "I", "UP", "W"]
line-length = 100
```

---

## Dependency Table Template

| Stage | Depends On | Completion Criteria |
|-------|------------|---------------------|
| 1 Infrastructure | - | pyproject.toml validated, mypy passed, pytest configured |
| 2 Domain | 1 | All entities defined, unit tests >80% coverage |
| 3 Infrastructure | 1, 2 | Repositories implemented, integration tests pass |
| 4 Application | 2, 3 | Services tested, use cases verified |
| 5 API | 4 | Endpoints working, OpenAPI spec matches |
| 6 Auth | 1, 5 | JWT implemented, security review passed |
| 7 Testing | 1-6 | E2E tests pass, coverage >80% |
| 8 Deployment | 7 | Docker build succeeds, CI/CD passes |
