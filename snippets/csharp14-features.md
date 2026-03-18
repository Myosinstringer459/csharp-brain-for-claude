# Snippets -- C# 14 / .NET 10 Features

## Extension Members (headline feature)

```csharp
namespace MyApp.Extensions;

public static class EnumerableExtensions
{
    // Instance extension block
    extension<TSource>(IEnumerable<TSource> source)
    {
        // Extension property (NEW in C# 14)
        public bool IsEmpty => !source.Any();

        // Extension method (same as classic, but new syntax)
        public IEnumerable<TSource> WhereNotNull()
            where TSource : class
            => source.Where(x => x is not null);
    }

    // Static extension block
    extension<TSource>(IEnumerable<TSource>)
    {
        // Static extension property
        public static IEnumerable<TSource> Empty => [];

        // Static extension operator
        public static IEnumerable<TSource> operator +(
            IEnumerable<TSource> left, IEnumerable<TSource> right)
            => left.Concat(right);
    }
}

// Usage:
var items = new List<int> { 1, 2, 3 };
if (items.IsEmpty) { /* ... */ }                      // extension property
var combined = listA + listB;                          // extension operator
var empty = IEnumerable<int>.Empty;                    // static extension property
```

## field Keyword (backed properties)

```csharp
namespace MyApp.Domain;

public sealed class Product
{
    // BEFORE C# 14: explicit backing field required
    // private string _name = string.Empty;
    // public string Name { get => _name; set => _name = value ?? throw ... }

    // C# 14: field keyword replaces backing field
    public string Name
    {
        get;
        set => field = value ?? throw new ArgumentNullException(nameof(value));
    }

    public decimal Price
    {
        get;
        set => field = value >= 0 ? value : throw new ArgumentOutOfRangeException(nameof(value));
    }

    // Lazy initialization pattern
    public IReadOnlyList<string> Tags
    {
        get => field ??= [];
        set;
    }
}
```

## Null-Conditional Assignment

```csharp
// BEFORE C# 14
if (customer is not null)
{
    customer.Order = GetCurrentOrder();
}

// C# 14
customer?.Order = GetCurrentOrder();        // RHS evaluated only if customer is not null
customer?.Balance += 100m;                  // compound assignment also works
// customer?.Count++;                       // increment/decrement NOT allowed
```

## Lambda Parameters with Modifiers (no explicit types)

```csharp
delegate bool TryParse<T>(string text, out T result);

// C# 14: modifiers without types
TryParse<int> parse = (text, out result) => int.TryParse(text, out result);

// Also works with ref, in, scoped, ref readonly
Func<int, int> transform = (ref x) => x * 2;  // hypothetical delegate
```

## nameof with Unbound Generics

```csharp
// BEFORE C# 14
var name1 = nameof(List<int>);    // "List" — had to provide type arg

// C# 14
var name2 = nameof(List<>);       // "List" — no type arg needed
var name3 = nameof(Dictionary<,>); // "Dictionary"
```

## Implicit Span Conversions

```csharp
// C# 14: Span<T> and ReadOnlySpan<T> are first-class citizens
void ProcessData(ReadOnlySpan<int> data) { /* ... */ }

int[] array = [1, 2, 3, 4, 5];
Span<int> span = array;                    // implicit conversion
ProcessData(array);                         // int[] -> ReadOnlySpan<int> implicit
ProcessData(span);                          // Span<T> -> ReadOnlySpan<T> implicit

// params with Span — zero allocation
void Log(params ReadOnlySpan<string> messages) { /* ... */ }
Log("hello", "world");                      // no hidden array allocation
```

## Partial Constructors and Events

```csharp
// Partial constructor — split declaration and implementation
public partial class OrderService
{
    public partial OrderService(ILogger<OrderService> logger);
}

public partial class OrderService
{
    private readonly ILogger<OrderService> _logger;

    public partial OrderService(ILogger<OrderService> logger)
    {
        _logger = logger;
    }
}
```

## EF Core 10 — Named Query Filters

```csharp
// Multiple named filters per entity
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Order>(entity =>
    {
        entity.HasQueryFilter("active", o => o.Status != OrderStatus.Cancelled);
        entity.HasQueryFilter("tenant", o => o.TenantId == _tenantId);
    });
}

// Selective disable
var allOrders = await context.Orders
    .IgnoreQueryFilters("active")    // disable only "active", keep "tenant"
    .ToListAsync(ct);
```

## EF Core 10 — ExecuteUpdateAsync with Lambda

```csharp
// Lambda syntax (not only expression trees)
await context.Orders
    .Where(o => o.Status == OrderStatus.Pending && o.CreatedAt < cutoff)
    .ExecuteUpdateAsync(o => o
        .SetProperty(x => x.Status, OrderStatus.Cancelled)
        .SetProperty(x => x.CancelledAt, DateTime.UtcNow), ct);

// JSON column updates in ExecuteUpdateAsync
await context.Products
    .Where(p => p.Id == productId)
    .ExecuteUpdateAsync(p => p
        .SetProperty(x => x.Metadata.LastUpdated, DateTime.UtcNow), ct);
```

## .NET 10 Runtime Improvements

- JIT: better inlining, method devirtualization, stack allocations
- AVX10.2 support for SIMD operations
- NativeAOT enhancements: smaller binaries, faster startup
- Post-quantum cryptography (ML-DSA via CNG)
- WebSocketStream for simplified WebSocket usage
- Process group support on Windows
- .NET Aspire 13.1 integration
