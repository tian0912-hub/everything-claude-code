---
paths:
  - "**/*.cj"
  - "**/cjpm.toml"
---
# Cangjie Testing

> This file extends [common/testing.md](../common/testing.md) with Cangjie-specific content.

## Test Framework

Cangjie uses annotation-based testing:

- **@Test** — mark test class or function
- **@TestCase** — mark individual test method
- **@Expect** — soft assertion (continue on failure)
- **@Assert** — hard assertion (stop on failure)
- **@AssertThrows[ExType]** — verify exception thrown
- **@PowerAssert** — show intermediate values on failure

## Test Organization

```text
src/
├── utils/
│   ├── shell.cj
│   └── shell_test.cj    # Unit test alongside source
├── auth/
│   ├── token.cj
│   └── token_test.cj
tests/                   # Integration tests (optional)
├── api_test.cj
└── e2e_test.cj
bench/                   # Benchmark tests (optional)
├── performance_test.cj
```

Unit test files named `xxx_test.cj`, placed alongside `xxx.cj`.

## Unit Test Pattern

```cangjie
@Test
class ShellTest {
    @TestCase
    func basicCommand(): Unit {
        let result = Shell.run("echo hello")
        @Assert(result == "hello")
    }

    @TestCase
    func handlesEmptyInput(): Unit {
        let result = Shell.run("")
        @AssertThrows[ShellException](result)
    }
}
```

## Assertion Types

| Annotation | Behavior | Use Case |
|------------|----------|----------|
| @Assert | Stop immediately on failure | Critical assertions |
| @Expect | Continue, collect failures | Multiple validations |
| @AssertThrows[ExType] | Verify exception type | Error handling tests |
| @PowerAssert | Show intermediate values | Complex expression debugging |

```cangjie
@Test
class ValidationTest {
    @TestCase
    func multipleChecks(): Unit {
        let user = User.parse(input)
        @Expect(user.name.size > 0)      // Soft assertion
        @Expect(user.age >= 18)
        @Assert(user.id > 0)             // Hard assertion stops here if fails
    }
}
```

## Parameterized Tests

```cangjie
@Test
class MathTest {
    @Test[input in ["a", "b", "c"]]
    func testStringLength(input: String): Unit {
        @Assert(input.size == 1)
    }

    // Range-based parameterization
    @Test[i in 0..10]
    func testPositive(i: Int64): Unit {
        @Assert(i >= 0)
    }
}
```

## Mocking

Use `mock<T>()` and `spy<T>()` for isolation:

```cangjie
@Test
class OrderServiceTest {
    @TestCase
    func returnsUserWhenFound(): Unit {
        let mockRepo = mock<UserRepository>()

        @On(mockRepo.find(42))
        .returns(User { id: 42, name: "Alice" })

        let service = OrderService(mockRepo)
        let user = service.getUser(42)

        @Assert(user.name == "Alice")
        @Called(mockRepo.find(42)).times(1)
    }
}
```

## Coverage

- Use **cjcov** for coverage reporting
- Target 80%+ line coverage, focus on branch coverage

```bash
cjpm test                         # Run tests
cjcov --html-details -o output    # HTML coverage report
cjcov --branches                  # Branch coverage
cjcov --fail-under-lines 80       # Fail threshold
```

## Test Commands

```bash
cjpm test                         # Run all tests
cjpm test --filter "ShellTest"    # Run specific test class
cjpm test --filter "testString*"  # Pattern matching
cjpm bench                        # Run benchmark tests
```

## References

See skill: `cangjie-regulations` Section 7 for complete testing standards.

---

## HarmonyOS Testing Extensions

### On-Device Test Organization

```
entry/
├── src/main/cangjie/           # Source code
│   └── services/
│       ├── user_service.cj
│       └── user_service_test.cj  # Unit test alongside source
├── src/ohosTest/cangjie/       # On-device tests
│   └── cjpm.toml
└── src/test/cangjie/           # Local unit tests
    └── cjpm.toml
```

### UI Testing with UIInspector

Use `harmonyos-ui-inspect` skill for automated UI verification:

```powershell
# Capture UI state
python .agents/skills/harmonyos-ui-inspect/ui_capture.py --emulator 5555 --out ./ui_output

# Run interaction scenario
python .agents/skills/harmonyos-ui-inspect/ui_capture.py --scenario ./test_scenario.json --out ./ui_output
```

### Test Scenario JSON

```json
{
  "name": "Login Flow Test",
  "steps": [
    {"action": "input", "target": {"hint": "Username"}, "text": "user@example.com"},
    {"action": "input", "target": {"hint": "Password"}, "text": "password123"},
    {"action": "click", "target": {"text": "Login"}},
    {"action": "wait", "seconds": 2}
  ],
  "assertions": [
    {"type": "page_changed", "message": "Should navigate to home"},
    {"type": "exists", "target": {"text": "Welcome"}, "message": "Welcome message should appear"}
  ]
}
```

### Build Verification

Always verify build after code changes:

```bash
# Using harmonyos-build skill
python .agents/skills/harmonyos-build/build.py --project-root <project>

# Or manually
ohpm install && cjpm build
```

### References (HarmonyOS)

See skill: `harmonyos-ui-inspect` for UI testing automation.
See skill: `harmonyos-build` for build verification.
See skill: `harmonyos-evolution` for recording test failures.