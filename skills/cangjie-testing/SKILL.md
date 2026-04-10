---
name: cangjie-testing
description: Cangjie testing patterns including unit tests, integration tests, mocking, and test coverage. Follows TDD methodology with idiomatic Cangjie practices.
origin: ECC
---

# Cangjie Testing Patterns

Comprehensive Cangjie testing patterns for writing reliable, maintainable tests following TDD methodology.

## When to Activate

- Writing new Cangjie functions or methods
- Adding test coverage to existing code
- Creating test utilities
- Following TDD workflow in Cangjie projects
- Building HarmonyOS/OpenHarmony applications

## TDD Workflow for Cangjie

### The RED-GREEN-REFACTOR Cycle

```
RED     → Write a failing test first
GREEN   → Write minimal code to pass the test
REFACTOR → Improve code while keeping tests green
REPEAT  → Continue with next requirement
```

### Step-by-Step TDD in Cangjie

```cangjie
// Step 1: Define the interface/signature
// calculator.cj
package calculator

public func add(a: Int64, b: Int64): Int64 {
    throw NotImplementedError("not implemented")
}

// Step 2: Write failing test (RED)
// calculator_test.cj
package calculator

@test
func testAdd(): Unit {
    let result = add(2, 3)
    assert(result == 5, "add(2, 3) should equal 5")
}

// Step 3: Run test - verify FAIL
// $ cjpm test
// FAIL: calculator_test::testAdd - NotImplementedError

// Step 4: Implement minimal code (GREEN)
public func add(a: Int64, b: Int64): Int64 {
    a + b
}

// Step 5: Run test - verify PASS
// $ cjpm test
// PASS: calculator_test::testAdd

// Step 6: Refactor if needed, verify tests still pass
```

## Unit Test Structure

### Basic Test Function

```cangjie
package myapp

@test
func testBasicCalculation(): Unit {
    let result = calculate(10, 5)
    assert(result == 50, "Expected 50, got ${result}")
}

@test
func testWithMultipleAssertions(): Unit {
    let list = ArrayList<Int64>()
    list.add(1)
    list.add(2)
    list.add(3)

    assert(list.size() == 3, "List should have 3 elements")
    assert(list[0] == 1, "First element should be 1")
    assert(list[2] == 3, "Third element should be 3")
}
```

### Test with Setup and Teardown

```cangjie
package myapp

class DatabaseTest {
    private var db: TestDatabase

    @before
    func setup(): Unit {
        this.db = TestDatabase.create()
    }

    @after
    func teardown(): Unit {
        this.db.close()
    }

    @test
    func testInsertUser(): Unit {
        let user = User(name: "Alice")
        this.db.insert(user)

        let found = this.db.find("Alice")
        assert(found.isDefined(), "User should be found")
    }
}
```

## Assertion Patterns

### Basic Assertions

```cangjie
@test
func testAssertions(): Unit {
    // Equality
    assert(1 + 1 == 2, "Math should work")

    // Boolean conditions
    assert(isValid("test"), "Input should be valid")
    assert(!isValid(""), "Empty input should be invalid")

    // Option checks
    let maybe = findUser(42)
    assert(maybe.isDefined(), "User should exist")
    assert(maybe.isEmpty() == false, "User should not be empty")

    // Exception testing
    assertThrows<ConfigException>({
        loadConfig("nonexistent.json")
    })
}
```

### Custom Assertion Helpers

```cangjie
func assertEqual<T>(got: T, expected: T, message: String = ""): Unit where T <: Equatable<T> {
    if (got != expected) {
        throw AssertionError("Expected ${expected}, got ${got}. ${message}")
    }
}

func assertTrue(condition: Bool, message: String = ""): Unit {
    assert(condition, message)
}

func assertFalse(condition: Bool, message: String = ""): Unit {
    assert(!condition, message)
}

func assertThrows<E>(block: () -> Unit): Unit where E <: Exception {
    try {
        block()
        throw AssertionError("Expected exception ${E} but none was thrown")
    } catch (e: E) {
        // Expected
    }
}

// Usage
@test
func testWithCustomAssertions(): Unit {
    assertEqual(calculate(5, 3), 15)
    assertTrue(isValid("test"))
    assertFalse(isValid(""))
    assertThrows<ValidationError>({ validate("") })
}
```

## Table-Driven Tests

### Parameterized Tests

```cangjie
@test
func testAdd(): Unit {
    let cases = [
        ("positive numbers", 2, 3, 5),
        ("negative numbers", -1, -2, -3),
        ("zero values", 0, 0, 0),
        ("mixed signs", -1, 1, 0),
        ("large numbers", 1000000, 2000000, 3000000)
    ]

    for ((name, a, b, expected) in cases) {
        let result = add(a, b)
        assert(result == expected, "Case '${name}': add(${a}, ${b}) = ${result}, expected ${expected}")
    }
}
```

### Test Cases with Error Expectations

```cangjie
struct TestCase {
    let name: String
    let input: String
    let expected: Option<Int64>
    let expectError: Bool
}

@test
func testParseInt(): Unit {
    let cases = [
        TestCase(name: "valid number", input: "42", expected: Some(42), expectError: false),
        TestCase(name: "negative number", input: "-10", expected: Some(-10), expectError: false),
        TestCase(name: "zero", input: "0", expected: Some(0), expectError: false),
        TestCase(name: "invalid letters", input: "abc", expected: None, expectError: true),
        TestCase(name: "empty string", input: "", expected: None, expectError: true)
    ]

    for (tc in cases) {
        match (parseInt(tc.input)) {
            case Some(value) => {
                assert(!tc.expectError, "Case '${tc.name}': expected error but got ${value}")
                assert(value == tc.expected.getOrElse(0), "Case '${tc.name}': got ${value}")
            }
            case None => {
                assert(tc.expectError, "Case '${tc.name}': expected ${tc.expected} but got error")
            }
        }
    }
}
```

## Mocking Patterns

### Interface-Based Mocking

```cangjie
// Define interface for dependency
interface UserRepository {
    func findById(id: Int64): Option<User>
    func save(user: User): Unit
}

// Production implementation
class PostgresUserRepository <: UserRepository {
    public func findById(id: Int64): Option<User> {
        // Real database query
    }

    public func save(user: User): Unit {
        // Real database insert
    }
}

// Mock implementation for tests
class MockUserRepository <: UserRepository {
    private var users = HashMap<Int64, User>()

    public func findById(id: Int64): Option<User> {
        this.users.get(id)
    }

    public func save(user: User): Unit {
        this.users[user.id] = user
    }

    // Test helper methods
    public func givenUser(user: User): Unit {
        this.users[user.id] = user
    }

    public func wasCalled(id: Int64): Bool {
        this.users.containsKey(id)
    }
}

// Test using mock
@test
func testUserService(): Unit {
    let mock = MockUserRepository()
    mock.givenUser(User(id: 123, name: "Alice"))

    let service = UserService(mock)
    let result = service.getUserName(123)

    assertEqual(result, "Alice")
    assertTrue(mock.wasCalled(123))
}
```

### Spy Pattern

```cangjie
class SpyEmailService <: EmailService {
    public var sentEmails = ArrayList<Email>()
    public var sendCount: Int64 = 0

    public func send(email: Email): Unit {
        this.sentEmails.add(email)
        this.sendCount += 1
    }

    public func lastEmail(): Option<Email> {
        if (this.sentEmails.isEmpty()) {
            return None
        }
        return Some(this.sentEmails[this.sentEmails.size() - 1])
    }
}

@test
func testNotificationService(): Unit {
    let spy = SpyEmailService()
    let service = NotificationService(spy)

    service.notifyUser(42, "Welcome!")

    assertEqual(spy.sendCount, 1)
    match (spy.lastEmail()) {
        case Some(email) => {
            assertEqual(email.to, "user_42@example.com")
            assertEqual(email.subject, "Welcome!")
        }
        case None => throw AssertionError("Expected email to be sent")
    }
}
```

## Test Organization

### File Structure

```text
src/
├── calculator/
│   ├── mod.cj
│   ├── basic.cj
│   ├── basic_test.cj      # Tests alongside source
│   ├── advanced.cj
│   └── advanced_test.cj
└── utils/
    ├── string.cj
    └── string_test.cj
```

### Test Package Structure

```cangjie
// calculator/basic_test.cj
package calculator

import std.testing.*

@test
func testBasicOperations(): Unit {
    // Tests for basic operations
}
```

## HarmonyOS/OpenHarmony UI Testing

### Component Testing

```cangjie
@test
func testCounterComponent(): Unit {
    let counter = CounterComponent()
    counter.aboutToAppear()

    // Initial state
    assertEqual(counter.count, 0)

    // Simulate click
    counter.increment()
    assertEqual(counter.count, 1)

    counter.increment()
    assertEqual(counter.count, 2)
}
```

### UI Interaction Testing

```cangjie
@test
func testButtonClick(): Unit {
    var clicked = false
    let button = Button("Click Me")
        .onClick { event =>
            clicked = true
        }

    // Simulate click event
    button.simulateClick()

    assertTrue(clicked, "Button click should set clicked to true")
}
```

## Coverage Commands

```bash
# Run all tests
cjpm test

# Run specific test file
cjpm test calculator/basic_test.cj

# Run with verbose output
cjpm test -v

# Generate coverage report
cjpm test --coverage

# Coverage threshold
cjpm test --coverage --fail-under 80
```

## Coverage Targets

| Code Type | Target |
|-----------|--------|
| Critical business logic | 100% |
| Public APIs | 90%+ |
| General code | 80%+ |
| Generated code | Exclude |
| UI components | 60%+ (focus on logic) |

## Test Commands Summary

```bash
# Run all tests
cjpm test

# Run tests matching pattern
cjpm test --filter "testAdd"

# Run with coverage
cjpm test --coverage

# Run specific package tests
cjpm test mypackage/

# Run in parallel
cjpm test --parallel

# Stop on first failure
cjpm test --fail-fast

# Verbose output
cjpm test -v
```

## Best Practices

**DO:**
- Write tests FIRST (TDD)
- Use table-driven tests for comprehensive coverage
- Test behavior, not implementation
- Keep tests independent and isolated
- Use meaningful test names that describe the scenario
- Clean up resources in teardown

**DON'T:**
- Test private functions directly (test through public API)
- Use `sleep()` in tests (use callbacks or conditions)
- Ignore flaky tests (fix or remove them)
- Mock everything (prefer integration tests when possible)
- Skip error path testing

## Integration with CI/CD

```yaml
# GitHub Actions example
test:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - uses: harmonyos/setup-cangjie@v1
      with:
        version: 'latest'

    - name: Run tests
      run: cjpm test --coverage

    - name: Check coverage
      run: |
        coverage=$(cjpm test --coverage --report json | jq '.coverage')
        if [ $(echo "$coverage < 80" | bc) -eq 1 ]; then
          echo "Coverage $coverage% is below 80%"
          exit 1
        fi
```

## Related

- Rules: `rules/cangjie/testing.md`
- Skill: `skills/cangjie-patterns/`
- Commands: `/cangjie-build`, `/cangjie-test`, `/cangjie-review`