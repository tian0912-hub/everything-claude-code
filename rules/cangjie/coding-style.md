---
paths:
  - "**/*.cj"
  - "**/cjpm.toml"
---
# Cangjie Coding Style

> This file extends [common/coding-style.md](../common/coding-style.md) with Cangjie-specific content.

## Formatting Tools

- **cjfmt** for code formatting — run before every commit
- **cjlint** for static checks — `cjlint -f src/`
- 4-space indent (default)
- Max line width: 120 characters

```bash
cjfmt src/                   # Format all files in src/
cjfmt -d src/                # Check formatting (dry-run)
cjlint -f src/               # Run lint checks
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

## Immutability

Cangjie uses `let` for immutable bindings and `var` for mutable:

```cangjie
// GOOD — immutable by default
let name: String = "Alice"
let config = Config.load()

// Only use var when mutation is required
var counter = 0
counter += 1
```

Prefer immutable data structures:
- Use `let` by default
- Use struct for lightweight immutable data (value semantics)
- Use class only when inheritance or shared reference needed

## Type Declarations

- **struct**: Lightweight, immutable data objects (coordinates, configs)
- **class**: Reference type for inheritance, polymorphism, shared state
- **interface**: Behavior contract, supports multiple implementation
- **enum**: Finite state sets with associated data

```cangjie
// GOOD — struct for immutable data
struct Point {
    let x: Float64
    let y: Float64
}

// GOOD — enum for state machine
enum ConnectionState {
    | Disconnected
    | Connecting(attempt: Int32)
    | Connected(sessionId: String)
    | Failed(reason: String, retries: Int32)
}
```

## Error Handling

Use `Option<T>` for expected absence, exceptions for unexpected failures:

```cangjie
// GOOD — Option for expected "no value" cases
func findUser(id: Int64): Option<User> {
    // ...
}

// Use ?? for default value
let user = findUser(42) ?? User.default()

// Use ?. for safe chain access
let email = user?.email

// GOOD — Exception for unexpected failures
class ConfigException <: Exception {
    init(message: String) {
        super.init(message)
    }
}

// Use try/catch for error recovery
try {
    processFile(path)
} catch (e: IOException) {
    log.error("IO failed: ${e.message}")
}
```

## Expressions

Cangjie if/match/try are expressions:

```cangjie
// GOOD — use expression form
let status = if (count > 0) { "active" } else { "empty" }

let result = match (state) {
    case Connected => "online"
    case Disconnected => "offline"
    case _ => "unknown"
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
    └── shell.cj
    └── shell_test.cj   # Unit tests alongside source
```

## Visibility

Follow minimum visibility principle:
- Default to `private`
- Escalate to `internal` → `protected` → `public` as needed

```cangjie
// GOOD — private by default
class UserService {
    private let repo: UserRepository
    public func find(id: Int64): Option<User> { ... }
    private func validate(u: User): Bool { ... }
}
```

## References

See skill: `cangjie-lang-features` for comprehensive Cangjie idioms and patterns.
See skill: `cangjie-regulations` for complete project standards.

---

## HarmonyOS (OpenHarmony) Extensions

When developing HarmonyOS applications with Cangjie, follow these additional conventions.

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

Pages use `@Entry` and `@Component` annotations:

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

### Component Naming

| Element | Convention | Example |
|---------|------------|---------|
| Page component | PascalCase + Page | `MainPage`, `DetailPage` |
| Custom component | PascalCase | `UserCard`, `LoadingSpinner` |
| Ability class | PascalCase + Ability | `MainAbility`, `EntryAbility` |

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

### References (HarmonyOS)

See skill: `harmonyos-project-init` for project creation.
See skill: `harmonyos-requirements` for development workflow.
See skill: `cangjie-harmonyos-doc-search` for API documentation.