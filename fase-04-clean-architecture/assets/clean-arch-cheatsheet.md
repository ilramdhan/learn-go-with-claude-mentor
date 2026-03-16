# 📋 Clean Architecture Quick Reference

## Layer Structure

```
┌─────────────────────────────────────────────┐
│              Delivery Layer                  │
│   HTTP Handler | gRPC Handler | CLI Handler  │
│   (dto, request/response mapping)            │
├─────────────────────────────────────────────┤
│              Use Case Layer                  │
│   Application business rules                 │
│   (orchestrates domain objects)              │
├─────────────────────────────────────────────┤
│              Domain Layer                    │
│   Entity | Value Object | Domain Event       │
│   Repository Interface | Domain Service      │
├─────────────────────────────────────────────┤
│           Infrastructure Layer               │
│   GORM Repository | Redis | Kafka | SMTP     │
│   (implements domain interfaces)             │
└─────────────────────────────────────────────┘
```

## Dependency Rule

```
Delivery → UseCase → Domain ← Infrastructure
         ↑                  ↑
         Both depend on Domain, never the reverse
```

## File Naming Convention

```
internal/
├── domain/
│   ├── entity/         user.go, product.go
│   ├── valueobject/    email.go, money.go
│   └── repository/     user_repository.go (interface)
├── usecase/
│   └── auth/           register_usecase.go
├── delivery/
│   └── http/handler/   auth_handler.go
└── infrastructure/
    ├── persistence/     user_repository_impl.go
    └── cache/           redis_cache.go
```

## SOLID Quick Reference

| Principle | Rule | Example |
|-----------|------|---------|
| **S**ingle Responsibility | One reason to change | UserRepository only handles DB ops |
| **O**pen/Closed | Open for extension, closed for modification | New auth method = new use case, not modify existing |
| **L**iskov Substitution | Subtypes must be substitutable | Any Repository impl works with UseCase |
| **I**nterface Segregation | Small specific interfaces | `Reader`, `Writer` not `ReadWriter` |
| **D**ependency Inversion | Depend on abstractions | UseCase depends on Repository *interface* |

## Error Handling Pattern

```go
// Domain errors (always define in domain layer)
var (
    ErrUserNotFound  = errors.New("user not found")
    ErrEmailExists   = errors.New("email already exists")
    ErrUnauthorized  = errors.New("unauthorized")
)

// HTTP mapping
var httpStatusMap = map[error]int{
    ErrUserNotFound:  404,
    ErrEmailExists:   409,
    ErrUnauthorized:  401,
}
```
