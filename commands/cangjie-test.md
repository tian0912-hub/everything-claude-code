---
description: Enforce TDD workflow for Cangjie. Write tests first, then implement. Verify 80%+ coverage.
---

# Cangjie TDD Command

This command enforces test-driven development methodology for Cangjie code.

## What This Command Does

1. **Define Interface**: Scaffold function signatures with `throw NotImplementedError`
2. **Write Tests**: Create comprehensive test module (RED)
3. **Run Tests**: Verify tests fail for the right reason
4. **Implement Code**: Write minimal code to pass (GREEN)
5. **Refactor**: Improve while keeping tests green
6. **Check Coverage**: Ensure 80%+ coverage

## When to Use

Use `/cangjie-test` when:
- Implementing new Cangjie functions, methods, or classes
- Adding test coverage to existing Cangjie code
- Fixing bugs (write failing test first)
- Building critical business logic
- Learning TDD workflow in Cangjie

## TDD Cycle

```
RED     -> Write failing test first
GREEN   -> Implement minimal code to pass
REFACTOR -> Improve code, tests stay green
REPEAT  -> Next test case
```

## Example Session

````text
User: /cangjie-test I need a function to validate user registration

Agent:
# TDD Session: Registration Validator

## Step 1: Define Interface

```cangjie
// src/validator.cj
package validator

public struct RegistrationRequest {
    let name: String
    let email: String
    let password: String
}

public enum ValidationResult {
    | Valid
    | Invalid(errors: Array<String>)
}

public func validateRegistration(request: RegistrationRequest): ValidationResult {
    throw NotImplementedError("not implemented")
}
```

## Step 2: Write Tests (RED)

```cangjie
// src/validator_test.cj
package validator

@test
func testValidRegistration(): Unit {
    let request = RegistrationRequest(
        name: "Alice",
        email: "alice@example.com",
        password: "SecureP@ss1"
    )
    let result = validateRegistration(request)
    match (result) {
        case Valid => {} // pass
        case Invalid(_) => throw AssertionError("Expected Valid")
    }
}

@test
func testBlankName(): Unit {
    let request = RegistrationRequest(
        name: "",
        email: "alice@example.com",
        password: "SecureP@ss1"
    )
    let result = validateRegistration(request)
    match (result) {
        case Invalid(errors) => {
            assert(errors.contains("Name is required"), "Expected name error")
        }
        case Valid => throw AssertionError("Expected Invalid")
    }
}

@test
func testInvalidEmail(): Unit {
    let request = RegistrationRequest(
        name: "Alice",
        email: "not-an-email",
        password: "SecureP@ss1"
    )
    let result = validateRegistration(request)
    match (result) {
        case Invalid(_) => {} // pass
        case Valid => throw AssertionError("Expected Invalid")
    }
}

@test
func testShortPassword(): Unit {
    let request = RegistrationRequest(
        name: "Alice",
        email: "alice@example.com",
        password: "short"
    )
    let result = validateRegistration(request)
    match (result) {
        case Invalid(_) => {} // pass
        case Valid => throw AssertionError("Expected Invalid")
    }
}
```

## Step 3: Run Tests — Verify FAIL

```bash
$ cjpm test
running 4 tests
test validator_test::testValidRegistration ... FAILED (NotImplementedError)
test validator_test::testBlankName ... FAILED
test validator_test::testInvalidEmail ... FAILED
test validator_test::testShortPassword ... FAILED
```

Tests fail as expected (NotImplementedError).

## Step 4: Implement Minimal Code (GREEN)

```cangjie
public func validateRegistration(request: RegistrationRequest): ValidationResult {
    let errors = ArrayList<String>()

    if (request.name.trim().isEmpty()) {
        errors.add("Name is required")
    }
    if (!request.email.contains('@')) {
        errors.add("Invalid email format")
    }
    if (request.password.length < 8) {
        errors.add("Password must be at least 8 characters")
    }

    if (errors.isEmpty()) {
        ValidationResult.Valid
    } else {
        ValidationResult.Invalid(errors.toArray())
    }
}
```

## Step 5: Run Tests — Verify PASS

```bash
$ cjpm test
running 4 tests
test validator_test::testValidRegistration ... ok
test validator_test::testBlankName ... ok
test validator_test::testInvalidEmail ... ok
test validator_test::testShortPassword ... ok

test result: ok. 4 passed; 0 failed
```

All tests passing!

## Step 6: Check Coverage

```bash
$ cjpm test --coverage
Coverage: 100.0% of lines
```

Coverage: 100%

## TDD Complete!
````

## Test Patterns

### Unit Tests

```cangjie
@test
func testBasicOperation(): Unit {
    let result = calculate(10, 5)
    assert(result == 50, "Expected 50, got ${result}")
}

@test
func testWithErrorHandling(): Unit {
    match (parseConfig("invalid")) {
        case Ok(_) => throw AssertionError("Expected error")
        case Err(e) => assert(e.message.contains("invalid"))
    }
}
```

### Parameterized Tests

```cangjie
@test
func testStringLength(): Unit {
    let cases = [
        ("hello", 5),
        ("", 0),
        ("cangjie", 7)
    ]

    for ((input, expected) in cases) {
        assert(input.length == expected, "${input}.length should be ${expected}")
    }
}
```

### Option Tests

```cangjie
@test
func testOptionHandling(): Unit {
    let result = findUser(42)

    // Test is defined
    assert(result.isDefined(), "User should exist")

    // Test value
    match (result) {
        case Some(user) => {
            assertEqual(user.name, "Alice")
        }
        case None => throw AssertionError("Expected Some")
    }

    // Test ?? operator
    let name = result?.name ?? "Unknown"
    assertEqual(name, "Alice")
}
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

## Coverage Commands

```bash
# Set up Cangjie environment first
if [ -f ".cangjie-env" ]; then source .cangjie-env; fi
. "${CANGJIE_ENV_SCRIPT:-/Users/tianbin/root/bin/i-cangjie}"

# Summary report
cjpm test --coverage

# HTML report
cjpm test --coverage --report html

# Fail if below threshold
cjpm test --coverage --fail-under 80

# Run specific test
cjpm test --filter testValidateRegistration

# Run with verbose output
cjpm test -v
```

## Coverage Targets

| Code Type | Target |
|-----------|--------|
| Critical business logic | 100% |
| Public API | 90%+ |
| General code | 80%+ |
| UI components | 60%+ |
| Generated code | Exclude |

## HarmonyOS UI Testing

For ArkUI components:

```cangjie
@test
func testCounterPage(): Unit {
    let page = CounterPage()
    page.aboutToAppear()

    // Initial state
    assertEqual(page.count, 0)

    // Simulate interaction
    page.increment()
    assertEqual(page.count, 1)
}
```

## TDD Best Practices

**DO:**
- Write test FIRST, before any implementation
- Run tests after each change
- Use descriptive test names
- Test behavior, not implementation
- Include edge cases (empty, boundary, error paths)

**DON'T:**
- Write implementation before tests
- Skip the RED phase
- Use sleeps in tests — use callbacks
- Mock everything — prefer integration tests when feasible

## Related Commands

- `/cangjie-env` - Configure Cangjie environment and version
- `/cangjie-build` - Fix build errors
- `/cangjie-review` - Review code after implementation
- `/verify` - Run full verification loop

## Related

- Skill: `skills/cangjie-testing/`
- Skill: `skills/cangjie-patterns/`
- Rules: `rules/cangjie/testing.md`