---
paths:
  - "**/*.cj"
---
# Cangjie Patterns

> This file extends [common/patterns.md](../common/patterns.md) with Cangjie-specific content.

## Repository Pattern with Interface

Encapsulate data access behind an interface:

```cangjie
interface OrderRepository {
    func find(id: Int64): Option<Order>
    func findAll(): Array<Order>
    func save(order: Order): Order
    func delete(id: Int64): Unit
}
```

Concrete implementations handle storage details.

## Service Layer Pattern

Business logic in service structs/classes; inject dependencies:

```cangjie
class OrderService {
    private let repo: OrderRepository
    private let payment: PaymentGateway

    init(repo: OrderRepository, payment: PaymentGateway) {
        this.repo = repo
        this.payment = payment
    }

    public func place(request: CreateOrderRequest): Option<OrderSummary> {
        let order = Order.from(request)
        payment.charge(order.total())
        repo.save(order)
    }
}
```

## Option Type Pattern

Use `Option<T>` for nullable returns; use `?T` shorthand:

```cangjie
// Full form
func find(id: Int64): Option<User> { ... }

// Shorthand with ? prefix
func find(id: Int64): ?User { ... }

// Safe access chain
let email = user?.profile?.email

// Default value
let name = user?.name ?? "Unknown"

// Conditional extraction
if (let Some(u) = findUser(42)) {
    println(u.name)
}
```

## Enum State Machine

Model states as enums with associated data:

```cangjie
enum ConnectionState {
    | Disconnected
    | Connecting(attempt: Int32)
    | Connected(sessionId: String)
    | Failed(reason: String, retries: Int32)
}

func handle(state: ConnectionState): String {
    match (state) {
        case Disconnected => connect()
        case Connecting(attempt) if attempt > 3 => abort()
        case Connecting(_) => wait()
        case Connected(sessionId) => useSession(sessionId)
        case Failed(reason, retries) if retries < 5 => retry()
        case Failed(reason, _) => logFailure(reason)
    }
}
```

Match exhaustively — avoid wildcard `_` for critical business enums.

## Builder Pattern

Use for structs with many optional parameters:

```cangjie
struct ServerConfig {
    let host: String
    let port: Int32
    let maxConnections: Int32
}

class ServerConfigBuilder {
    private var host: String = "localhost"
    private var port: Int32 = 8080
    private var maxConnections: Int32 = 100

    func host(h: String): ServerConfigBuilder {
        host = h
        this
    }

    func port(p: Int32): ServerConfigBuilder {
        port = p
        this
    }

    func maxConn(n: Int32): ServerConfigBuilder {
        maxConnections = n
        this
    }

    func build(): ServerConfig {
        ServerConfig { host: host, port: port, maxConnections: maxConnections }
    }
}

// Usage
let config = ServerConfigBuilder()
    .host("api.example.com")
    .port(443)
    .maxConn(500)
    .build()
```

## Sealed Class/Interface

Restrict inheritance to same package:

```cangjie
sealed class Shape {
    public func area(): Float64 { ... }
}

class Rectangle <: Shape {
    let width: Float64
    let height: Float64
    public override func area(): Float64 { width * height }
}

class Circle <: Shape {
    let radius: Float64
    public override func area(): Float64 { 3.14159 * radius * radius }
}
// Only Rectangle, Circle can extend Shape (must be in same package)
```

## Extend Pattern

Add functionality to existing types:

```cangjie
// Extend String with new method
extend String {
    public func isBlank(): Bool {
        this.trim().size == 0
    }
}

// Extend to implement interface
extend String <: Equatable {
    public override func equals(other: String): Bool {
        this == other
    }
}
```

## Pipeline Pattern

Use `|>` for data transformation chains:

```cangjie
let result = rawData
    |> parseJson
    |> validateSchema
    |> transformToModel
    |> saveToDatabase
```

## Derive Annotation

Auto-generate common implementations:

```cangjie
@Derive[Equatable, Hashable]
struct User {
    let id: Int64
    let name: String
    let email: String
}
// Equatable and Hashable implementations auto-generated
```

## References

See skill: `cangjie-lang-features` for comprehensive patterns including generics, extensions, and concurrency.

---

## HarmonyOS UI Patterns

### State Management Pattern

**Parent-Child Communication**:

```cangjie
// One-way (Parent → Child)
@Component
struct ChildComponent {
    @Prop readonly title: String  // One-way from parent
    func build(): Unit { Text(this.title) }
}

// Two-way binding
@Component
struct CounterComponent {
    @Link var value: Int64  // Two-way binding
    func build(): Unit {
        Row {
            Button("-").onClick { _ => this.value -= 1 }
            Text("${this.value}")
            Button("+").onClick { _ => this.value += 1 }
        }
    }
}

// Parent usage
@State private score: Int64 = 0
CounterComponent(value: $score)  // $ prefix for two-way
```

**Cross-Level (@Provide/@Consume)**:

```cangjie
@Entry
@Component
struct RootPage {
    @Provide private theme: String = "dark"
    func build(): Unit { Column { HeaderComponent() } }
}

@Component
struct DeepChild {
    @Consume private theme: String  // No passing through intermediates
    func build(): Unit { Text("Theme: ${this.theme}") }
}
```

### Repository with Ability Context

```cangjie
class UserRepository {
    private let context: UIAbilityContext
    init(context: UIAbilityContext) { this.context = context }
    public func findById(id: Int64): Option<User> { ... }
}

// Usage in Ability
class MainAbility <: UIAbility {
    public override func onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): Unit {
        let repo = UserRepository(this.context)
    }
}
```

### Navigation Pattern

```cangjie
class NavigationService {
    public static func toDetail(id: Int64): Unit {
        Router.pushUrl(Url { url: "pages/DetailPage", params: { "id": id } })
    }
    public static func back(): Unit { Router.back() }
}
```

### ArkTS Interop Pattern

Use `@Interop` macro for Cangjie → ArkTS export:

```cangjie
import ohos.ark_interop_macro.*

// Export function to ArkTS
@Interop[ArkTS]
public func calculateTotal(items: JSArrayEx<Item>): Float64 {
    var total: Float64 = 0.0
    for (item in items) { total += item.price }
    total
}

// Async export for long operations
@Interop[ArkTS, Async]
public func fetchData(url: String): String {
    Http.get(url).body
}
```

### Error Handling in UI

```cangjie
@Entry
@Component
struct DataPage {
    @State private data: Option<Data> = None
    @State private error: Option<String> = None
    @State private isLoading: Bool = false

    func build(): Unit {
        Column {
            if (this.isLoading) {
                LoadingSpinner()
            } else if (let Some(err) = this.error) {
                ErrorView(message: err, onRetry: { => this.loadData() })
            } else if (let Some(d) = this.data) {
                DataView(data: d)
            }
        }
        .aboutToAppear { this.loadData() }
    }
}
```

### References (HarmonyOS)

See skill: `cangjie_arkts_interop` for ArkTS interop patterns.
See skill: `harmonyos-requirements` for design workflow.