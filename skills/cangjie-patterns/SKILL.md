---
name: cangjie-patterns
description: Cangjie (仓颉) language idioms, naming conventions, error handling patterns, and HarmonyOS/OpenHarmony development best practices.
origin: ECC
---

# Cangjie Development Patterns

Idiomatic Cangjie patterns and best practices for building robust, efficient, and maintainable applications, including HarmonyOS/OpenHarmony extensions.

## When to Activate

- Writing new Cangjie code
- Reviewing Cangjie code
- Refactoring existing Cangjie code
- Building HarmonyOS/OpenHarmony applications

## Core Principles

### 1. Immutability First

Cangjie uses `let` for immutable bindings and `var` for mutable. Prefer immutability.

```cangjie
// GOOD — immutable by default
let name: String = "Alice"
let config = Config.load()

// Only use var when mutation is required
var counter = 0
counter += 1
```

### 2. Expression-Oriented

Cangjie if/match/try are expressions. Use them for concise code.

```cangjie
// GOOD — use expression form
let status = if (count > 0) { "active" } else { "empty" }

let result = match (state) {
    case Connected => "online"
    case Disconnected => "offline"
    case _ => "unknown"
}
```

### 3. Safe by Default

Use `Option<T>` for expected absence, exceptions for unexpected failures.

```cangjie
// GOOD — Option for expected "no value" cases
func findUser(id: Int64): Option<User> {
    // ...
}

// Use ?? for default value
let user = findUser(42) ?? User.default()

// Use ?. for safe chain access
let email = user?.email
```

## Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Package | lowercase, underscores allowed | `network`, `http_client` |
| Source file | lowercase with underscores | `http_client.cj` |
| Class/Interface/Struct/Enum | PascalCase | `HttpClient`, `UserService` |
| Function | camelCase | `getData`, `parseResponse` |
| Constants (global let/static let) | SCREAMING_SNAKE_CASE | `MAX_RETRY_COUNT` |
| Variables | camelCase | `itemCount`, `userName` |
| Boolean variables/functions | is/has/can/should prefix | `isValid`, `hasPermission` |

## Type Declarations

### struct vs class

- **struct**: Lightweight, value semantics, immutable data objects
- **class**: Reference type for inheritance, polymorphism, shared state

```cangjie
// GOOD — struct for immutable data
struct Point {
    let x: Float64
    let y: Float64

    func distance(other: Point): Float64 {
        ((this.x - other.x) ** 2 + (this.y - other.y) ** 2).sqrt()
    }
}

// GOOD — class for shared state
class UserService {
    private let repo: UserRepository

    public func find(id: Int64): Option<User> {
        this.repo.findById(id)
    }
}
```

### Enums with Associated Data

```cangjie
enum ConnectionState {
    | Disconnected
    | Connecting(attempt: Int32)
    | Connected(sessionId: String)
    | Failed(reason: String, retries: Int32)
}

func handleConnection(state: ConnectionState): String {
    match (state) {
        case Disconnected => "Reconnecting..."
        case Connecting(attempt) => "Attempt ${attempt}"
        case Connected(sessionId) => "Session: ${sessionId}"
        case Failed(reason, retries) => "Failed: ${reason} (${retries} retries)"
    }
}
```

### Interfaces

```cangjie
interface Repository<T> {
    func findById(id: Int64): Option<T>
    func save(entity: T): Unit
    func delete(id: Int64): Unit
}

class UserRepository <: Repository<User> {
    public func findById(id: Int64): Option<User> {
        // Implementation
    }

    public func save(entity: User): Unit {
        // Implementation
    }

    public func delete(id: Int64): Unit {
        // Implementation
    }
}
```

## Error Handling Patterns

### Option<T> for Expected Absence

```cangjie
func findUser(id: Int64): Option<User> {
    let result = db.query("SELECT * FROM users WHERE id = ?", [id])
    if (result.isEmpty()) {
        return None
    }
    return Some(User.fromRow(result.first()))
}

// Usage with ?? operator
let user = findUser(42) ?? User.guest()

// Usage with ?. safe navigation
let email = findUser(42)?.email ?? "unknown@example.com"
```

### Exceptions for Unexpected Failures

```cangjie
class ConfigException <: Exception {
    init(message: String) {
        super.init(message)
    }
}

func loadConfig(path: String): Config {
    if (!File.exists(path)) {
        throw ConfigException("Config file not found: ${path}")
    }

    let content = File.read(path)
    return Config.parse(content)
}

// Usage with try/catch
try {
    let config = loadConfig("config.json")
    startService(config)
} catch (e: ConfigException) {
    log.error("Configuration error: ${e.message}")
    exit(1)
} catch (e: IOException) {
    log.error("IO error: ${e.message}")
    exit(1)
}
```

### Result Type Pattern

```cangjie
enum Result<T, E> {
    | Ok(value: T)
    | Err(error: E)
}

func validateInput(data: String): Result<Input, ValidationError> {
    if (data.isEmpty()) {
        return Err(ValidationError.EmptyInput)
    }
    if (data.length > MAX_LENGTH) {
        return Err(ValidationError.TooLong(data.length))
    }
    return Ok(Input.parse(data))
}

// Usage
match (validateInput(input)) {
    case Ok(value) => process(value)
    case Err(error) => log.error("Validation failed: ${error}")
}
```

## Module Organization

Organize by domain, not by type:

```text
src/
├── main.cj             # Program entry
├── auth/               # Domain module
│   ├── mod.cj          # Package declaration
│   ├── token.cj
│   └── middleware.cj
├── orders/
│   ├── mod.cj
│   ├── model.cj
│   └── service.cj
└── utils/
    ├── shell.cj
    └── shell_test.cj   # Unit tests alongside source
```

### Package Declaration

```cangjie
// auth/mod.cj
package auth

import auth.token
import auth.middleware

// Re-export public API
public func createToken(user: User): Token {
    token.create(user)
}
```

## Visibility

Follow minimum visibility principle:

```cangjie
// GOOD — private by default
class UserService {
    private let repo: UserRepository
    public func find(id: Int64): Option<User> { ... }
    private func validate(u: User): Bool { ... }
}
```

| Modifier | Scope |
|----------|-------|
| (none) | private (default) |
| `internal` | Same package |
| `protected` | Same package + subclasses |
| `public` | All packages |

## Concurrency Patterns

### spawn for Concurrent Tasks

```cangjie
import std.concurrency.spawn

func fetchAll(urls: Array<String>): Array<Response> {
    let futures = ArrayList<Future<Response>>()

    for (url in urls) {
        futures.add(spawn { fetch(url) })
    }

    let results = ArrayList<Response>()
    for (future in futures) {
        results.add(future.get())
    }
    return results.toArray()
}
```

### sync.Mutex for Shared State

```cangjie
import std.sync.Mutex

class Counter {
    private let mutex = Mutex()
    private var count: Int64 = 0

    public func increment(): Int64 {
        mutex.lock()
        defer { mutex.unlock() }
        this.count += 1
        return this.count
    }
}
```

## Collection Patterns

### ArrayList Operations

```cangjie
let list = ArrayList<Int64>()
list.add(1)
list.add(2)
list.add(3)

// Functional operations
let doubled = list.map { x => x * 2 }
let evens = list.filter { x => x % 2 == 0 }
let sum = list.reduce(0) { acc, x => acc + x }
```

### HashMap Operations

```cangjie
let map = HashMap<String, Int64>()
map["one"] = 1
map["two"] = 2

// Safe access
let value = map.get("three") ?? 0

// Iteration
for ((key, value) in map) {
    println("${key}: ${value}")
}
```

## HarmonyOS/OpenHarmony Extensions

### Project Structure

```
<project>/
├── AppScope/
│   └── app.json5              # Global config (bundleName, version)
├── entry/                     # Main module
│   ├── src/main/
│   │   ├── cangjie/           # Cangjie source
│   │   │   ├── ability_stage.cj
│   │   │   ├── index.cj       # Entry page (@Entry, @Component)
│   │   │   └── main_ability.cj
│   │   ├── module.json5
│   │   └── resources/
│   ├── cjpm.toml
│   └── build-profile.json5
├── oh-package.json5
└── build-profile.json5
```

### ArkUI Components

```cangjie
@Entry
@Component
struct MainPage {
    @State private message: String = "Hello"

    func build(): Unit {
        Column {
            Text(this.message)
                .fontSize(20)
                .onClick { event => this.message = "Clicked" }
        }
    }
}
```

### State Management Decorators

| Decorator | Purpose | Scope |
|-----------|---------|-------|
| `@State` | Local mutable state | Current component |
| `@Prop` | One-way from parent | Parent → Child |
| `@Link` | Two-way binding | Parent ↔ Child |
| `@Provide` | Provide to descendants | Ancestor → Any descendant |
| `@Consume` | Consume from ancestor | Descendant reads ancestor |

### Lifecycle Hooks

```cangjie
@Entry
@Component
struct LifecyclePage {
    func aboutToAppear(): Unit { /* Before build */ }
    func aboutToDisappear(): Unit { /* OnDestroy */ }
    func onPageShow(): Unit { /* Page visible */ }
    func onPageHide(): Unit { /* Page invisible */ }
    func onBackPress(): Bool { /* Back button - return true to intercept */ }
    func build(): Unit { ... }
}
```

### Navigation

```cangjie
import ohos.router.Router

Router.pushUrl(Url { url: "pages/DetailPage" })
Router.back()
```

## Tooling Commands

```bash
# Build project
cjpm build

# Run tests
cjpm test

# Format code
cjfmt src/

# Lint check
cjlint -f src/

# Package for HarmonyOS
cjpm package
```

## Best Practices

**DO:**
- Use `let` by default, only use `var` when mutation is needed
- Use struct for immutable data, class for shared state
- Use `Option<T>` for expected absence
- Organize by domain, not by type
- Keep functions under 50 lines
- Run `cjfmt` before every commit

**DON'T:**
- Use `var` when `let` suffices
- Catch generic exceptions when specific ones are available
- Use class when struct would work
- Make everything public by default
- Skip the `defer` pattern for resource cleanup

## Related

- Rules: `rules/cangjie/`
- Skill: `skills/cangjie-testing/`
- Commands: `/cangjie-build`, `/cangjie-test`, `/cangjie-review`