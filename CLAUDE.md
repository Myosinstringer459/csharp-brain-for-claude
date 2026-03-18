# C# / .NET Code Rules

## Standards
- **C# 14**, **.NET 10**. No legacy patterns.
- File-scoped namespaces: `namespace MyApp.Features.Orders;`
- Primary constructors for services, handlers, repositories
- `sealed` by default — all classes sealed unless designed for inheritance
- Nullable Reference Types always enabled

## Code Style

### Properties & DTOs
- `required` + `init` for all DTOs and records
- `field` keyword in property accessors (C# 14 semi-auto properties)
- Records for Value Objects: `sealed record` with private constructor, static `Create()` returning `Result<T>`

### Performance (hot paths)
- `Span<T>` / `Memory<T>` for zero-alloc parsing
- `ArrayPool<T>.Shared` for temporary buffers
- `FrozenDictionary` / `FrozenSet` for read-heavy lookup
- `ValueTask` when result is often synchronous (caches, lookups)
- `SearchValues<T>` for fast character searching
- `params ReadOnlySpan<T>` for zero-alloc variadic (C# 14)
- `CompositeFormat` for pre-compiled format strings

### Async
- `async Task` everywhere — never `async void` (except event handlers)
- `ConfigureAwait(false)` in all library/infrastructure code
- `CancellationToken` always passed and checked
- `IAsyncEnumerable` for streaming data
- `Parallel.ForEachAsync` with `MaxDegreeOfParallelism` for bounded parallelism

### Pattern Matching
- Pattern matching over if-chains — always
- Use relational, property, and list patterns

## Architecture

### Default: Modular Monolith + Vertical Slices
- Layers: Domain -> Application -> Infrastructure -> Api
- Feature folders: `Features/Orders/CreateOrder/` (Command + Handler + Validator + Endpoint)
- Not layer folders (`Controllers/`, `Services/`, `Repositories/`)

### Error Handling: Result Pattern
- `Result<T>` for all expected errors (not found, validation, conflict)
- Never throw exceptions for flow control
- Exceptions only for infrastructure failures (DB down, network timeout)
- `Match()` for mapping Result to HTTP responses

### CQRS
- Commands return `Result<T>` — write operations
- Queries return `Result<T>` — read operations
- Strict separation: no writes in query handlers

### API: Minimal API
- No controllers. Endpoints in static classes (`*Endpoints.cs`)
- `MapGroup()` for shared filters/tags/rate limits
- `TypedResults` — `Results<Ok<Dto>, BadRequest<ProblemDetails>, NotFound>`
- Problem Details (RFC 9457) for all error responses
- Explicit handlers (scoped, injected into endpoint). No MediatR if simple.

### Domain Layer
- Rich Domain Model — entities contain business logic and invariants
- Aggregate Root pattern: `AggregateRoot<TId>` -> `Entity<TId>` -> domain events
- Value Objects: `sealed record` with `Create()` / `From()` returning `Result<T>`
- Domain events for side effects

## Naming Conventions

| Type | Pattern | Example |
|------|---------|---------|
| Command | `{Action}{Entity}Command` | `CreateOrderCommand` |
| Query | `Get{Entity}Query` | `GetOrderByIdQuery` |
| Handler | `{CommandOrQuery}Handler` | `CreateOrderCommandHandler` |
| Repository | `I{Entity}Repository` | `IOrderRepository` |
| DTO | `{Entity}{Purpose}Dto` | `OrderSummaryDto` |
| Endpoint | `{Entity}Endpoints` | `OrderEndpoints` |

## Forbidden Patterns
- Suffixes: Manager, Helper, Util, Processor — name by responsibility
- Exceptions for flow control — use Result pattern
- String interpolation in log messages — use structured logging templates
- `any` / dynamic types without justification
- Global mutable state
- Business logic in endpoints/controllers
- Anemic Domain Model — entities without behavior
- Repository over Repository — wrapping EF Core without added value

## EF Core
- Fluent configuration in separate `*Configuration.cs` files (no Data Annotations)
- `AsNoTracking()` for all read queries
- `AsSplitQuery()` when including multiple collections
- Keyset pagination for large datasets, offset for small
- Compiled queries for hot paths
- Dapper for complex reporting queries (hybrid with EF Core)

## Infrastructure
- DI via extension methods: `AddInfrastructure(this IServiceCollection, IConfiguration)`
- Keyed Services for multiple implementations of same interface
- Typed HttpClient + `AddResilienceHandler()` (Polly)
- `Channel<T>` + `BackgroundService` for background jobs
- `[LoggerMessage]` source generator for high-performance logging
- `[JsonSerializable]` source generator for System.Text.Json (AOT-ready)

## Security
- `CryptographicOperations.FixedTimeEquals()` for token comparison
- SHA256 hash before storing tokens
- Built-in `AddRateLimiter()` per IP
- Reject `..` and `\` in file paths (path traversal)
- `WebUtility.HtmlEncode()` for user input in Value Objects

## Testing
- xUnit + NSubstitute + FluentAssertions
- Integration tests via `WebApplicationFactory` > unit tests for backend
- `TimeProvider` injected via DI (`FakeTimeProvider` in tests)
- Architecture tests with NetArchTest
- Arrange-Act-Assert pattern always

## Docker
- Multi-stage builds: `sdk` for build -> `aspnet` runtime for final image
- Non-root user in production images
- Health checks for all dependencies

## Workflow
1. Read existing code before suggesting changes
2. Complex task -> short plan first, then code
3. After writing code: `dotnet build` + `dotnet format` — fix immediately if it fails
4. Never guess — ask when ambiguous
