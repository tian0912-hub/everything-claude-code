---
description: Comprehensive Cangjie code review for naming, error handling, Option usage, and idiomatic patterns. Invokes the cangjie-reviewer agent.
---

# Cangjie Code Review

This command invokes the **cangjie-reviewer** agent for comprehensive Cangjie-specific code review.

## What This Command Does

1. **Verify Automated Checks**: Run `cjpm build`, `cjlint`, `cjfmt --check`, and `cjpm test`
2. **Identify Cangjie Changes**: Find modified `.cj` files via `git diff HEAD~1`
3. **Run Security Audit**: Check for hardcoded secrets, unsafe patterns
4. **Review Code Quality**: Analyze naming, structure, Option usage, error handling
5. **Generate Report**: Categorize issues by severity

## When to Use

Use `/cangjie-review` when:
- After writing or modifying Cangjie code
- Before committing Cangjie changes
- Reviewing pull requests with Cangjie code
- Onboarding to a new Cangjie codebase
- Learning idiomatic Cangjie patterns

## Review Categories

### CRITICAL (Must Fix)

- Unchecked `Option<T>` unwrap that can panic
- Hardcoded credentials or secrets
- SQL injection via string interpolation
- Command injection via unvalidated input
- Missing error handling in critical paths
- Thread safety violations (shared mutable state without locks)

### HIGH (Should Fix)

- Using `var` when `let` suffices
- Using class when struct would work
- Large functions (>50 lines)
- Missing visibility modifiers (should be explicit)
- String concatenation in loops
- Wildcard match case `_ =>` on business-critical enums
- Missing `defer` for resource cleanup

### MEDIUM (Consider)

- Unnecessary allocation in hot paths
- Missing `internal` vs `public` distinction
- Public API without documentation comments
- Inconsistent naming within module
- Deep nesting (>3 levels)
- Missing type annotations on public functions

## Automated Checks Run

```bash
# Set up Cangjie environment first
if [ -f ".cangjie-env" ]; then source .cangjie-env; fi
. "${CANGJIE_ENV_SCRIPT:-/Users/tianbin/root/bin/i-cangjie}"

# Build gate (must pass before review)
cjpm build

# Lint checks
cjlint -f src/

# Format check
cjfmt -d src/

# Tests
cjpm test

# Security scan (if available)
cjpm audit
```

## Environment Setup

Cangjie 多版本共存，执行 cjpm 命令前需要先设置环境。

### 配置方法

在项目根目录创建 `.cangjie-env` 文件：

```bash
echo 'CANGJIE_ENV_SCRIPT=/Users/tianbin/root/bin/i-cangjie' > .cangjie-env
```

或设置环境变量：

```bash
export CANGJIE_ENV_SCRIPT=/Users/tianbin/root/bin/i-cangjie
```

详见 `/cangjie-env` 命令。

## Example Session

````text
User: /cangjie-review

Agent:
# Cangjie Code Review Report

## Files Reviewed
- src/service/user.cj (modified)
- src/handler/api.cj (modified)
- src/utils/config.cj (modified)

## Static Analysis Results
- Build: Successful
- Lint: 2 warnings
- Formatting: Passed
- Tests: All passing

## Issues Found

[CRITICAL] Unchecked Option Unwrap
File: src/service/user.cj:28
Issue: Using `get()` on Option without checking isDefined()
```cangjie
let user = findUser(id).get()  // Panics if None
```
Fix: Use safe handling
```cangjie
let user = findUser(id) ?? throw UserNotFound(id)
// or
match (findUser(id)) {
    case Some(u) => process(u)
    case None => handleNotFound()
}
```

[HIGH] Unnecessary var
File: src/handler/api.cj:45
Issue: Variable declared as `var` but never mutated
```cangjie
var response = buildResponse(data)
```
Fix: Use `let` for immutable binding
```cangjie
let response = buildResponse(data)
```

[HIGH] Class should be struct
File: src/utils/config.cj:10
Issue: Class used for immutable data only
```cangjie
class Config {
    let host: String
    let port: Int64
}
```
Fix: Use struct for value semantics
```cangjie
struct Config {
    let host: String
    let port: Int64
}
```

[MEDIUM] Missing visibility modifier
File: src/handler/api.cj:20
Issue: Function has implicit private visibility
```cangjie
func helper(): Unit { ... }
```
Fix: Be explicit about visibility
```cangjie
private func helper(): Unit { ... }
```

## Summary
- CRITICAL: 1
- HIGH: 2
- MEDIUM: 1

Recommendation: Block merge until CRITICAL issue is fixed
````

## Naming Convention Checks

| Element | Convention | Example |
|---------|------------|---------|
| Class/Struct/Enum | PascalCase | `UserService` ✓ |
| Function | camelCase | `getData` ✓ |
| Constants | SCREAMING_SNAKE_CASE | `MAX_COUNT` ✓ |
| Variables | camelCase | `itemCount` ✓ |
| Package | lowercase | `http_client` ✓ |

## Option Handling Patterns

### GOOD Patterns

```cangjie
// Safe unwrap with default
let value = maybeValue ?? defaultValue

// Safe chain
let email = user?.email ?? "unknown@example.com"

// Explicit handling
match (findUser(id)) {
    case Some(user) => process(user)
    case None => handleNotFound()
}
```

### BAD Patterns

```cangjie
// DANGER: Panics on None
let value = maybeValue.get()

// DANGER: Silent failure
if (maybeValue.isDefined()) {
    process(maybeValue.get())  // Redundant check
}
```

## Approval Criteria

| Status | Condition |
|--------|-----------|
| Approve | No CRITICAL or HIGH issues |
| Warning | Only MEDIUM issues (merge with caution) |
| Block | CRITICAL or HIGH issues found |

## HarmonyOS/OpenHarmony Specific

For HarmonyOS projects, additional checks:

### State Management

```cangjie
// GOOD: Proper @State usage
@State private count: Int64 = 0

// BAD: Mutable state without @State
private var count: Int64 = 0  // Won't trigger re-render
```

### Lifecycle Hooks

```cangjie
// Check proper lifecycle hook usage
func aboutToAppear(): Unit { /* Called before build */ }
func aboutToDisappear(): Unit { /* Cleanup */ }
```

### Navigation

```cangjie
// Check proper Router usage
import ohos.router.Router

Router.pushUrl(Url { url: "pages/DetailPage" })
```

## Integration with Other Commands

- Use `/cangjie-env` to configure environment first
- Use `/cangjie-test` first to ensure tests pass
- Use `/cangjie-build` if build errors occur
- Use `/cangjie-review` before committing
- Use `/code-review` for non-Cangjie-specific concerns

## Related

- Agent: `agents/cangjie-reviewer.md`
- Skills: `skills/cangjie-patterns/`, `skills/cangjie-testing/`
- Rules: `rules/cangjie/`