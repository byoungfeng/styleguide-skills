# Google C# Style Guide — Deep Reference

## 1. Formatting Guidelines

### 1.1 Naming Conventions

| Element | Convention | Example | Notes |
|---|---|---|---|
| Class | PascalCase | `class OrderProcessor` | Nouns or noun phrases |
| Interface | I + PascalCase | `interface IRepository` | Prefix mandatory `I` |
| Method | PascalCase | `void ProcessOrder()` | Verbs or verb phrases |
| Property | PascalCase | `public int TotalCount` | Nouns or adjectives |
| Public field | PascalCase | `public readonly int MaxRetries` | Public fields rare; prefer property |
| Private field | `_camelCase` | `_connectionString` | Leading underscore required |
| Protected field | `_camelCase` | `_logger` | Same as private |
| Internal field | `_camelCase` | `_cache` | Same as private |
| Private property | `_camelCase` | `_itemCount` | Yes, private properties also use leading underscore |
| Local variable | camelCase | `int rowCount` | No prefix |
| Parameter | camelCase | `string userName` | No prefix |
| Constant | PascalCase | `const int DefaultPageSize` | Both public and private |
| Static readonly | PascalCase | `static readonly TimeSpan Timeout` | Same as const |
| Enum type | PascalCase | `enum ErrorCode` | Singular unless flags |
| Enum member | PascalCase | `NotFound`, `BadRequest` | Flags can be plural |
| Namespace | PascalCase | `Company.Product.Module` | Max 2 meaningful levels |
| Type parameter | PascalCase `T` | `T`, `TKey` | Single letter or descriptive |
| Event | PascalCase | `event EventHandler OrderProcessed` | Verb past tense |
| Delegate | PascalCase | `delegate void Callback()` | Same as method naming |
| Local function | camelCase | `void processItem()` | Same as local variable convention |
| Record | PascalCase | `record CustomerDto` | Same as class |
| Record positional params | camelCase | `record Person(string firstName)` | These become properties in PascalCase externally |

#### Acronyms

Acronyms are treated as whole words, Pascal-cased:

- `MyRpc` not `MyRPC`
- `XmlWriter` not `XMLWriter`
- `IoStream` not `IOStream`
- `HtmlParser` not `HTMLParser`
- `ApiClient` not `APIClient`

Two-letter acronyms are Pascal-cased:

- `IoStream` (not `IOStream`)
- `DbConnection` (not `DBConnection`)

#### Word breaks

- `OrderProcessor` — each word capitalized
- `OrderProcessingService` — compound words each capitalized
- `CheckoutSession` not `CheckOutSession`
- `LogOnMethod` — preserve readability; follow framework conventions

### 1.2 File Organization

- One file per class (partial classes exempted)
- Filename matches class name in PascalCase: `CustomerService.cs`
- For nested types/types in sub-namespaces, prefix: `CustomerService.EventArgs.cs`
- Partial classes: suffix with part identifier: `OrderProcessor.Validation.cs`
- File encoding: UTF-8 with BOM
- Line endings: CRLF on Windows, LF on Linux/macOS
- No BOM on non-Windows platforms

### 1.3 Modifier Ordering

All modifiers follow this exact sequence:

```
public protected internal private new abstract virtual override sealed static readonly extern unsafe volatile async
```

Examples:

```csharp
public static readonly int MaxConnections = 10;
internal static extern void NativeMethod();
protected internal override void OnConfigChanged() { }
private static readonly object _lock = new object();
public override sealed string ToString() => Id;
protected internal virtual Task ExecuteAsync();
```

### 1.4 Class Member Ordering

Members ordered by category, then by visibility within each category:

1. Nested types (enums, classes, structs, interfaces, delegates)
2. Constants (`const`)
3. Static readonly fields
4. Static fields
5. Instance readonly fields
6. Instance fields (private fields first)
7. Private properties (use `_camelCase`)
8. Internal properties
9. Protected properties
10. Public properties
11. Constructors (static first, then instance; from least to most parameters)
12. Finalizers (`~ClassName()`)
13. Methods

Visibility ordering within each group:

1. `public`
2. `internal`
3. `protected internal`
4. `protected`
5. `private`

```csharp
public class OrderService
{
    // Nested types
    public enum OrderStatus { Pending, Shipped, Delivered }
    private class OrderComparer : IComparer<Order> { }

    // Constants
    public const int DefaultPageSize = 25;
    private const string CacheKey = "orders_cache";

    // Static readonly
    private static readonly TimeSpan _defaultTimeout = TimeSpan.FromSeconds(30);

    // Static fields
    private static int _totalOrders;

    // Instance readonly
    private readonly IRepository<Order> _repository;

    // Instance fields
    private int _currentPage;

    // Private property
    private int _maxRetryCount { get; set; }

    // Public property
    public int TotalCount { get; set; }

    // Constructors
    public OrderService(IRepository<Order> repository)
    {
        _repository = repository;
    }

    // Methods
    public Task<Order> GetByIdAsync(int id) { ... }
    private void ValidateOrder(Order order) { ... }
}
```

### 1.5 Whitespace Rules

#### Indentation

- **2-space** indent. No tabs.
- Tab character in source file is an error.

#### Braces (K&R / Egyptian style)

```csharp
// Correct — brace on same line
if (condition) {
    DoSomething();
}

// Incorrect — brace on next line (Allman style)
if (condition)
{
    DoSomething();
}
```

Applies to: `class`, `interface`, `struct`, `enum`, `if/else`, `for`, `foreach`, `while`, `do/while`, `switch`, `using`, `try/catch/finally`, `lock`, `fixed`, `unsafe`, `checked/unchecked`, `#if/#endif`, `record`, `namespace`, `property`, `indexer`, `method`, `constructor`, `destructor`, `operator`, `event add/remove`, `get/set/init` accessors.

#### Empty blocks

```csharp
// Acceptable — empty block without braces
if (condition) DoSomething();

// Also acceptable — empty block with closing on same line
while (condition) { }
```

#### Space placement

```csharp
// Space after keyword, no space inside parens
if (condition) { }
for (int i = 0; i < count; i++) { }
while (running) { }
catch (Exception ex) { }
using (var conn = new SqlConnection()) { }
lock (lockObj) { }
switch (value) { }
fixed (byte* ptr = buffer) { }

// No space before semicolons
int sum = a + b;   // correct
int sum = a + b ;  // incorrect

// No space before comma
DoSomething(a, b, c);  // correct
DoSomething(a , b , c); // incorrect

// Space after comma
int[] values = { 1, 2, 3 };  // correct
int[] values = { 1,2,3 };    // incorrect

// Binary operators — space on both sides
int sum = a + b;
bool result = (x > 0) && (y < 10);

// Unary operators — no space
int inc = ++count;
bool notFlag = !flag;

// Cast — no space
int value = (int)obj;

// Generic types — no space before <
List<int> items;    // correct
List <int> items;   // incorrect

// Methods — no space before parens
DoWork();           // correct
DoWork ();          // incorrect
```

#### Method declaration spacing

```csharp
// Correct spacing
public void ProcessItem(int id, string name) { }
public async Task<Order> GetOrderAsync(int orderId) { }

// No space before parameters parens
public void ProcessItem (int id) { }   // incorrect

// No space before type parameter
public void Process<T> (T item) { }    // incorrect
```

#### Collection/object initializer spacing

```csharp
// Correct — one space inside braces
int[] numbers = { 1, 2, 3 };

// Correct — object initializer with one space per member
var order = new Order {
    Id = 1,
    Name = "Test"
};

// Attribute — separate line, no indentation change
[Obsolete("Use NewMethod instead")]
public void OldMethod() { }

// Multiple attributes on separate lines
[Serializable]
[Obsolete("Use NewMethod instead")]
public class OldClass { }
```

#### Line continuation indentation (+4 from containing block)

```csharp
// Method continuation — +4 indent
CallMethodWithLongParameters(
    parameterOne,
    parameterTwo,
    parameterThree);

// Binary expression continuation — +4 indent
bool result = VeryLongConditionThatExceedsColumnLimit
    && AnotherLongCondition
    && YetAnotherCondition;

// Chained method calls — +4 indent
var orders = repository.Query()
    .Where(o => o.Status == OrderStatus.Active)
    .OrderByDescending(o => o.CreatedAt)
    .Take(pageSize);

// Ternary continuation — +4 indent
var status = condition
    ? ValueWhenTrue
    : ValueWhenFalse;
```

### 1.6 Column Limit

- Maximum line length: **100 characters**
- Exception: lines that would be unreadable if split (e.g., a long URL in a comment)
- Exception: string literals that cannot reasonably be split
- Exception: `#region` directives
- Exception: log message strings (prefer splitting the arguments instead)

### 1.7 One Statement Per Line

```csharp
// Correct
int count = 10;
count++;

// Incorrect
int count = 10; count++;
```

### 1.8 `using` Statements

```csharp
// `using` directive at top of file before namespace
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Company.Common;

namespace MyNamespace {
    // ...
}
```

- System namespaces first, then other namespaces in alphabetical order
- Empty line between System and other groups optional
- `using static` follows the same rules
- Use `using` aliases sparingly (see Coding Guidelines)
- No `using` directives inside namespaces

File-scoped namespace allowed in modern C#:

```csharp
using System;
using System.Linq;

namespace MyNamespace;
```

## 2. Coding Guidelines Deep Dive

### 2.1 Constants and Readonly

- **Prefer `const`** when the value is known at compile time and never changes
- **Use `static readonly`** when value is determined at runtime (e.g., `DateTime` calculations, configuration values)
- **Prefer `readonly` instance fields** over mutable public fields
- **Use `public readonly`** sparingly; prefer property with `{ get; }`

```csharp
// Best: compile-time constant
public const int DefaultPageSize = 25;

// Good: runtime constant
public static readonly TimeSpan RequestTimeout = TimeSpan.FromSeconds(30);

// Good: instance-level constant
private readonly IRepository<Order> _orderRepository;

// Acceptable for DTOs/records
public readonly int Id;
```

**Never use `const` for:**
- Values that may change (version numbers, URLs)
- Reference types other than `string` or `null`
- Values read from configuration

### 2.2 `var` Keyword Usage

**Encouraged when type is obvious:**

```csharp
var name = "John";                          // string is obvious
var order = new Order();                    // Order is obvious  
var repository = new OrderRepository();     // OrderRepository is obvious
var dict = new Dictionary<int, string>();   // Dictionary is obvious
var numbers = new List<int>();              // List is obvious
var tuple = (Id: 1, Name: "test");          // ValueTuple is obvious
```

**Discouraged when type is not obvious:**

```csharp
var result = ProcessOrder();                // What is the return type?
var value = GetConfiguration("key");        // Unclear — might be string, int, bool
var data = repository.Query().First();      // Unclear concrete type
var item = CallMethod();                    // Reader must check method signature
```

**Always explicit for:**

- `var` with `new()` — use explicit type instead in C# 9+ target-typed new:

```csharp
// Prefer explicit for target-typed new
Order order = new();      // OK, matches style
var order = new Order();  // Also acceptable
```

### 2.3 `IEnumerable<T>` vs `IList<T>` vs `IReadOnlyList<T>`

#### Input parameters

```csharp
// Prefer IEnumerable for input when you only iterate
public void ProcessOrders(IEnumerable<Order> orders) { }

// Prefer IReadOnlyList when you need index access but no mutation
public Order GetNthOrder(IReadOnlyList<Order> orders, int index) { }

// Use IList when you need to modify the collection
public void SortOrders(IList<Order> orders) { }

// Use ICollection when you need Count without indexing
public bool CacheOrders(ICollection<Order> orders) { }
```

#### Return types

```csharp
// Prefer IReadOnlyList/IReadOnlyCollection for public API returns
public IReadOnlyList<Order> GetActiveOrders() { }

// IEnumerable is acceptable for deferred execution
public IEnumerable<Order> GetOrdersSlowly() { }

// Avoid array for public API (unless performance is critical)
public Order[] GetOrders() { }           // avoid
public IReadOnlyList<Order> GetOrders()  // prefer
```

### 2.4 Generators vs Containers Performance

| Approach | When to Use |
|---|---|
| `yield return` (generator) | Streaming large datasets, deferred execution, pipeline composition |
| List/Array (container) | Small bounded sets, multiple enumerations needed, need random access |
| `IEnumerable<T>` return | Default for lazy evaluation unless concrete type is needed |

```csharp
// Generator — good for large streams
public IEnumerable<int> GetEvenNumbers(int max)
{
    for (int i = 0; i < max; i++) {
        if (i % 2 == 0) yield return i;
    }
}

// Container — good when caller needs Count/index
public IReadOnlyList<int> GetFirstTenEvenNumbers()
{
    return Enumerable.Range(0, 20).Where(i => i % 2 == 0).ToList();
}
```

### 2.5 Property Styles

#### Expression-bodied properties

```csharp
// Simple getter
public int TotalCount => _items.Count;

// Computed property
public string FullName => $"{FirstName} {LastName}";

// Simple method
public override string ToString() => $"Order {Id}";
```

#### Auto-properties

```csharp
// Standard auto-property
public int Id { get; set; }

// Read-only auto-property (set in constructor)
public int Id { get; }

// Init-only (C# 9+)
public int Id { get; init; }

// Private set
public int Id { get; private set; }
```

#### Full property with backing field

```csharp
private int _value;
public int Value
{
    get => _value;
    set => _value = value > 0 ? value : 0;
}
```

#### When to use expression body vs `{ get; set; }`

| Scenario | Style |
|---|---|
| Simple computed property | Expression body: `=> expression;` |
| Auto-property with no logic | `{ get; set; }` |
| Property with backing field logic | Full block with expression-body accessors |
| Method that is one expression | Expression body: `=> expression;` |
| Method with multiple statements | Block body `{ ... }` |

### 2.6 Expression Body Syntax Rules

```csharp
// Method returning value
public int Square(int x) => x * x;

// Void method
public void Log(string message) => Console.WriteLine(message);

// Property
public string Name => _name;

// Indexer
public string this[int index] => _items[index];

// Operator
public static Point operator +(Point a, Point b) => new Point(a.X + b.X, a.Y + b.Y);

// Constructor (C# 12+ primary constructors)
public class Point(int x, int y) { }

// Conversion
public static implicit operator int(Point p) => p.X;
```

**Not allowed for:**

- Constructors (except primary constructors in C# 12)
- Finalizers
- Accessors with multiple statements

### 2.7 Structs vs Classes

| Factor | Use Struct | Use Class |
|---|---|---|
| Size | <= 32 bytes | > 32 bytes |
| Mutability | Immutable preferred | Mutable or immutable |
| Identity | Value semantics needed | Reference semantics needed |
| Allocation | Heap allocation overhead critical | Acceptable heap allocation |
| Collections | Lots of short-lived instances | Medium to long-lived |
| Inheritance | Never needs inheritance | May need inheritance |
| Default value | Makes sense as zero/default | Not meaningful as default |

```csharp
// Good struct candidate — small, immutable, value semantics
public readonly struct Money
{
    public decimal Amount { get; }
    public string Currency { get; }

    public Money(decimal amount, string currency)
    {
        Amount = amount;
        Currency = currency;
    }
}

// Poor struct candidate — large, mutable, reference semantics needed
public struct OrderProcessor                     // avoid — too large, mutable
{
    public int Id;
    public string Name;
    public List<OrderItem> Items;
    public void Process() { }
}
```

### 2.8 Lambdas vs Named Methods

```csharp
// Lambda — good for short, inline operations
var evenNumbers = numbers.Where(n => n % 2 == 0);
button.Click += (sender, args) => HandleClick();

// Named method — good when logic is complex or reused
private bool IsEven(int number) => number % 2 == 0;
var evenNumbers = numbers.Where(IsEven);

// Local function — good for helpers within a single method
public void ProcessOrders()
{
    Order ValidateOrder(Order order) { ... }
    var validated = orders.Select(ValidateOrder);
}
```

**Guidelines:**
- Lambdas: 1-3 lines, simple logic
- Named methods: reusable or complex logic (> 3 lines)
- Local functions: method-scoped helpers, especially for iterators/async

### 2.9 Field Initializers

```csharp
// Encouraged — initialize at declaration
private readonly List<Order> _orders = new List<Order>();
private readonly IRepository<Order> _repository;

// Acceptable — constructor-based when value depends on parameters
public OrderService(IRepository<Order> repository)
{
    _repository = repository;
}

// Prefer field initializer over constructor when no dependency
private readonly TimeSpan _timeout = TimeSpan.FromSeconds(30);
```

### 2.10 Extension Methods

```csharp
// Use only when you don't have source access
public static class EnumerableExtensions
{
    public static IEnumerable<T> WhereNotNull<T>(this IEnumerable<T?> source) where T : class
    {
        return source.Where(x => x != null)!;
    }
}

// Avoid when you own the type — add method directly
public class Order
{
    // Add method here instead of extension
    public bool IsValid() => ...;
}
```

- Must be in a `static class`
- Must be in a namespace
- Do not shadow instance methods
- Be careful with LINQ method names to avoid ambiguity

### 2.11 `ref` and `out` Parameters

```csharp
// Use 'out' for multiple return values (prefer tuple instead)
public bool TryParse(string input, out int result) { ... }

// Use 'ref' sparingly — only for interop or performance-sensitive code
public void Swap(ref int a, ref int b) { ... }

// Prefer return values or tuples over ref/out
public (bool Success, int Value) TryParse(string input) { ... }
```

- Avoid `ref`/`out` except for `TryParse` patterns or interop
- Prefer tuples or a result class
- `in` parameters are acceptable for large readonly structs

### 2.12 LINQ Best Practices

```csharp
// Prefer member syntax (method chain) over SQL-style query syntax
// Preferred
var result = orders.Where(o => o.Status == Active)
                   .OrderBy(o => o.CreatedAt);

// Avoid — SQL-style query syntax
var result = from o in orders
             where o.Status == Active
             orderby o.CreatedAt
             select o;

// Use query syntax only for complex group joins
var grouped = from o in orders
              group o by o.Category into g
              select new { Category = g.Key, Count = g.Count() };
```

- Avoid LINQ for trivially simple operations (use `if`, `for`, etc.)
- Materialize enumerables explicitly with `.ToList()` or `.ToArray()` when multiple enumeration needed
- Order LINQ chain for readability (filter first, then transform)
- Prefer `.FirstOrDefault()` over `.First()` or `.SingleOrDefault()` when element may not exist

### 2.13 Array vs List Trade-offs

| | Array | List<T> |
|---|---|---|
| Public API | Avoid | Prefer |
| Fixed size | Good | Overhead |
| Performance critical | Good | Slight overhead |
| Multiple dimension | `[,]` only | Not supported |
| `params` parameter | Required | Cannot use |
| Mutability | Mutable | Mutable |
| Interop | Required | Must convert |

```csharp
// Public API: prefer List<T> or IReadOnlyList<T>
public IReadOnlyList<Order> GetOrders() { ... }

// Internal: array OK for fixed size
private Order[] _cache;

// params: must use array
public void Log(string format, params object[] args) { ... }

// Performance hot path: use array
private readonly int[] _lookupTable;
```

### 2.14 Folder and File Structure

```
Company.Product/
├── src/
│   ├── Project.sln
│   ├── Project.csproj
│   ├── Models/
│   │   ├── Order.cs
│   │   └── Customer.cs
│   ├── Services/
│   │   ├── OrderService.cs
│   │   └── PaymentService.cs
│   └── Controllers/
│       └── OrderController.cs
└── tests/
    └── Project.Tests/
        ├── Project.Tests.csproj
        └── Services/
            └── OrderServiceTests.cs
```

- One project per assembly
- Tests project mirrors source project structure
- Folder names: PascalCase, match namespace components
- Avoid "Helpers" or "Utilities" folders — find specific names

### 2.15 Tuple Return Types

```csharp
// Avoid public API tuples — prefer named class/record
// Discouraged
public (int Id, string Name) GetCustomerInfo(int id) { ... }

// Preferred — named class or record
public record CustomerInfo(int Id, string Name);
public CustomerInfo GetCustomerInfo(int id) { ... }

// Acceptable for internal/private methods with simple, obvious tuples
private (int Count, decimal Total) CalculateSummary() { ... }
```

### 2.16 String Interpolation

```csharp
// Preferred — string interpolation
string message = $"Order {orderId} processed for {customerName}";

// Avoid — concatenation
string message = "Order " + orderId + " processed for " + customerName;

// Avoid — string.Format
string message = string.Format("Order {0} processed for {1}", orderId, customerName);

// Complex expressions in interpolation
string result = $"Status: {(order.IsActive ? "Active" : "Inactive")}";
```

- Always prefer interpolation for readability
- Use `StringBuilder` for large/complex concatenation in loops

### 2.17 `using` Alias Avoidance

```csharp
// Avoid — alias adds indirection
using OrderList = System.Collections.Generic.List<Order>;
using ProjectAlias = Company.Project.SomeClass;

// Prefer — direct usage
using var orders = new List<Order>();
var instance = new Company.Project.SomeClass();
```

- Acceptable aliases: for `Nullable<T>` shorthand in legacy code, or disambiguation in auto-generated code
- Do not alias just to shorten a type name

### 2.18 Object Initializer Syntax

```csharp
// Preferred — object initializer
var order = new Order {
    Id = 1,
    CustomerName = "John",
    Total = 100.00m,
    Items =
    {
        new OrderItem { ProductId = 1, Quantity = 2 }
    }
};

// Avoid — multiple property assignments after construction
var order = new Order();
order.Id = 1;
order.CustomerName = "John";
order.Total = 100.00m;

// Nested initializers
var customer = new Customer {
    Name = "John",
    Address = new Address {
        Street = "123 Main St",
        City = "Springfield"
    }
};
```

- One property per line
- Closing brace on own line aligned with opening
- Nested initializers use same formatting

### 2.19 Namespace Naming

```csharp
// Preferred — max 2 meaningful levels
namespace Company.Product;
namespace Company.Product.Models;
namespace Company.Product.Services;

// Avoid — deep nesting
namespace Company.Product.SubSystem.DataAccess.Repositories.Implementations;

// Follow folder structure
// Folder: src/Company.Product/Models/Order.cs
namespace Company.Product.Models;
```

### 2.20 Default Values and Null Returns for Structs

```csharp
// For reference types, return null when no result
public Order FindOrder(int id) { return null; }

// For value types, use nullable or a Try-pattern
public Order? FindOrderOrNull(int id) { return null; }
public Money? FindPrice(int productId) { return null; }
public bool TryFindPrice(int productId, out Money price) { ... }

// Avoid returning default struct as sentinel
public Money FindPrice(int productId)
{
    return default;  // avoid — ambiguous meaning
}
```

### 2.21 Removing from Containers While Iterating

```csharp
// Preferred — List.RemoveAll
var activeOrders = _orders.RemoveAll(o => o.Status == OrderStatus.Archived);

// Also acceptable — iterate backward
for (int i = _orders.Count - 1; i >= 0; i--) {
    if (_orders[i].Status == OrderStatus.Archived) {
        _orders.RemoveAt(i);
    }
}

// Avoid — foreach with modification (throws)
foreach (var order in _orders) {
    if (order.Status == OrderStatus.Archived) {
        _orders.Remove(order);  // InvalidOperationException!
    }
}
```

### 2.22 Calling Delegates with `?.Invoke()`

```csharp
// Preferred — null-conditional invoke
OnOrderProcessed?.Invoke(this, EventArgs.Empty);

// Avoid — manual null check
if (OnOrderProcessed != null) {
    OnOrderProcessed(this, EventArgs.Empty);  // not thread-safe
}
```

- Always use `?.Invoke()` pattern
- Thread-safe because the null check and invocation happen atomically
- Works with `Action`, `Func<>`, `EventHandler`, and custom delegates

### 2.23 `var` Keyword — Encouraged and Discouraged Cases Summary

**Encouraged:**
- `var order = new Order();`
- `var numbers = new List<int>();`
- `var dict = new Dictionary<string, int>();`
- `var item = new { Name = "test", Id = 1 };` (anonymous types — required)
- `var tuple = (Id: 1, Name: "test");`

**Discouraged:**
- `var result = SomeMethod();` — type not obvious
- `var count = GetCount();` — could be int, long, short
- `var exception = GetException();` — reader expects Exception base type
- `var settings = Configuration.GetSection("x");` — returns IConfigurationSection

### 2.24 Attributes

```csharp
// Each attribute on its own line
[Serializable]
[Obsolete("Use NewVersion instead")]
public class OldVersion { }

// Attribute on same line only for simple, short attributes
[Serializable] public class SimpleClass { }

// Attributes on parameters
public void Process([CallerMemberName] string memberName = "")
{
}

// Attributes on return value
[return: MarshalAs(UnmanagedType.Bool)]
public static extern bool IsWindowVisible(IntPtr hWnd);
```

### 2.25 Named Arguments

```csharp
// Use named arguments for clarity with bool/enum parameters
var order = CreateOrder(
    customerId: 42,
    isExpedited: true,
    shippingMethod: ShippingMethod.Express
);

// Avoid named arguments for well-understood parameters
int.TryParse("42", out int result);   // fine, well known
int.TryParse("42", result: out int result);  // unnecessary

// Required: named arguments for optional parameters when skipping earlier ones
ConfigureConnection(timeout: 30);
```

### 2.26 Region Usage

- Do NOT use `#region` to group members — rely on member ordering instead
- Acceptable: `#region` for auto-generated code sections
- Acceptable: `#region` for large partial class files

### 2.27 Comments and Documentation

```csharp
// Use // for inline comments
// Use /// <summary> for public API documentation

/// <summary>
/// Processes the order identified by the given ID.
/// </summary>
/// <param name="orderId">The unique order identifier.</param>
/// <returns>The processed order, or null if not found.</returns>
public Order? ProcessOrder(int orderId) { ... }
```

- XML doc comments on all public types and public/protected members
- Inline comments explain "why", not "what"
- Avoid obvious comments

### 2.28 Nullable Reference Types

```csharp
// Enable nullable annotations in project file
// <Nullable>enable</Nullable>

// Use ? for nullable reference types
public string? FindName(int id) { ... }

// Suppress only when you know better than the compiler
public string name = null!;

// Constructor ensures non-nullable
public class Order
{
    public string Name { get; set; } = string.Empty;  // never null
    public string? Description { get; set; }           // may be null
}
```

### 2.29 Exception Handling

```csharp
// Catch specific exceptions
try {
    ProcessOrder(order);
} catch (OrderNotFoundException ex) {
    Logger.LogWarning("Order not found: {OrderId}", order.Id);
    return NotFound();
} catch (ValidationException ex) {
    return BadRequest(ex.Message);
}

// Avoid catching System.Exception (except top-level handlers)
try {
    DoSomething();
} catch (Exception ex) {  // avoid — catches everything
    Logger.LogError(ex, "Something failed");
    throw;
}

// Use `using` for IDisposable (or await using for IAsyncDisposable)
using var connection = new SqlConnection(connectionString);
await using var stream = new FileStream(path, FileMode.Open);
```

### 2.30 Async Method Patterns

```csharp
// Name async methods with Async suffix
public Task<Order> GetOrderAsync(int id) { ... }

// Async all the way — avoid .Result or .Wait()
public async Task<Order> GetOrderAsync(int id)
{
    return await _repository.FindByIdAsync(id);
}

// ConfigureAwait(false) in library code
public async Task<Order> GetOrderAsync(int id)
{
    var order = await _repository.FindByIdAsync(id)
        .ConfigureAwait(false);
    return order;
}

// Avoid async void (except for event handlers)
public async void Button_Click(object sender, EventArgs e)  // OK — event handler
{
    await ProcessAsync();
}
```

### 2.31 Pattern Matching

```csharp
// Use pattern matching for type checks
public decimal CalculateDiscount(Order order)
{
    return order switch
    {
        BulkOrder { Quantity: > 100 } => 0.15m,
        BulkOrder { Quantity: > 50 }  => 0.10m,
        SeasonalOrder { Season: "Winter" } => 0.05m,
        _ => 0m
    };
}

// Property patterns
if (order is { Status: OrderStatus.Active, Total: > 1000 })
{
    ApplyDiscount(order);
}

// Declaration pattern
if (order is BulkOrder bulk)
{
    return bulk.Quantity * bulk.UnitPrice;
}

// Not pattern
if (order is not null) { }
```

## 3. Review Checklist

### Naming
- [ ] Classes, methods, enums, public fields/properties: PascalCase
- [ ] Local variables, parameters: camelCase
- [ ] Private/protected/internal fields: `_camelCase`
- [ ] Interfaces: prefix `I` + PascalCase
- [ ] Acronyms as whole words: `MyRpc` not `MyRPC`
- [ ] Filename matches class name, PascalCase
- [ ] Namespace: PascalCase, max 2 meaningful levels

### Organization
- [ ] Modifier order: `public protected internal private new abstract virtual override sealed static readonly extern unsafe volatile async`
- [ ] `using` at top, before namespace; System first, then alphabetical
- [ ] Class member order: nested → static/const/readonly fields → fields/properties → constructors/finalizers → methods
- [ ] Visibility order within groups: public → internal → protected internal → protected → private

### Formatting
- [ ] 2-space indent, no tabs
- [ ] Column limit 100
- [ ] One statement per line
- [ ] K&R braces (Egyptian) always
- [ ] Space after `if`/`for`/`while`/`catch`/`switch`, no space inside parens
- [ ] Braces on same line
- [ ] Line continuation +4 indent
- [ ] No space before `;`, before `,`, before `<` generics, before method `()`
- [ ] Space after `,`, around binary operators

### Coding Practices
- [ ] Prefer `const` then `readonly` over magic values
- [ ] `var` used when type is obvious, avoided when unclear
- [ ] `IEnumerable<T>` for input, `IReadOnlyList<T>` for output
- [ ] LINQ member syntax preferred over SQL-style
- [ ] Expression body for simple properties/methods
- [ ] Named class over `Tuple<T>` for public API
- [ ] String interpolation over concatenation/`string.Format()`
- [ ] Object initializer syntax used
- [ ] `RemoveAll` for removing from `List<T>` during iteration
- [ ] Delegate invocation with `?.Invoke()`
- [ ] `List<T>` over arrays for public API
- [ ] Attributes on separate lines
- [ ] Named arguments for non-obvious parameters (bool, enum)
- [ ] `ConfigureAwait(false)` in library code
- [ ] No `async void` (except event handlers)
- [ ] No `#region` to organize members
- [ ] XML doc comments on all public/protected members
- [ ] Nullable reference types enabled

Full source: `G:\styleguide\csharp-style.md`
