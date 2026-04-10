---
description: Fix Cangjie build errors, type mismatches, and dependency problems incrementally. Invokes the cangjie-build-resolver agent for minimal, surgical fixes.
---

# Cangjie Build and Fix

This command invokes the **cangjie-build-resolver** agent to incrementally fix Cangjie build errors with minimal changes.

## What This Command Does

1. **Clean and Update**: Execute `cjpm clean` and `cjpm update` to clear cache and install dependencies
2. **Run Diagnostics**: Execute `cjpm build`, `cjlint`, and format checks
3. **Parse Errors**: Identify error codes and affected files
4. **Fix Incrementally**: One error at a time
5. **Verify Each Fix**: Re-run `cjpm build` after each change
6. **Report Summary**: Show what was fixed and what remains

## Why Clean and Update First?

Running `cjpm clean` and `cjpm update` before build helps avoid:
- **Stale cache issues**: Old compiled artifacts causing unexpected errors
- **Missing dependencies**: New dependencies not yet installed
- **Version mismatches**: Dependency versions changed but not reflected locally
- **Incremental build bugs**: Compiler caching incorrect state

## When to Use

Use `/cangjie-build` when:
- `cjpm build` fails with compilation errors
- `cjlint` reports warnings
- Type mismatches block compilation
- Dependency resolution fails
- After pulling changes that break the build

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

### 优先级

1. 环境变量 `CANGJIE_ENV_SCRIPT`
2. 项目配置文件 `.cangjie-env`
3. 默认路径 `/Users/tianbin/root/bin/i-cangjie`

## Diagnostic Commands Run

```bash
# Set up Cangjie environment (reads from .cangjie-env or CANGJIE_ENV_SCRIPT)
if [ -f ".cangjie-env" ]; then source .cangjie-env; fi
. "${CANGJIE_ENV_SCRIPT:-/Users/tianbin/root/bin/i-cangjie}"

# Clean cache and install dependencies first (avoids cache-related issues)
cjpm clean 2>&1
cjpm update 2>&1

# Primary build check
cjpm build 2>&1

# Lint check
cjlint -f src/ 2>&1

# Format check
cjfmt -d src/ 2>&1

# Dependency issues
cjpm deps --check

# Security audit (if available)
cjpm audit
```

## Example Session

````text
User: /cangjie-build

Agent:
# Cangjie Build Resolution

## Environment Setup

```bash
$ . /Users/tianbin/root/bin/i-cangjie
Cangjie environment configured (version: 0.53.4)
```

## Pre-Build Setup

```bash
$ cjpm clean
Cleaning build artifacts... done

$ cjpm update
Updating dependencies... done
```

## Initial Diagnostics

```bash
$ cjpm build
error: type mismatch
  --> src/service/cache.cj:25:5
error: undefined reference `db`
  --> src/handler/api.cj:58:9
error: missing import
  --> src/utils/config.cj:10:3
```

Errors found: 3

## Fix 1: Type Mismatch

File: src/service/cache.cj:25
Error: type mismatch — expected `Int64`, found `String`

```cangjie
// Changed
let count = params.get("count")
// To
let count: Int64 = params.get("count")
    .map { s => s.parseInt() }
    .getOrElse(0)
```

```bash
$ cjpm build
# 2 errors remaining
```

## Fix 2: Undefined Reference

File: src/handler/api.cj:58
Error: undefined reference `db`

```cangjie
// Added import
import myapp.db.Database
```

```bash
$ cjpm build
# 1 error remaining
```

## Fix 3: Missing Import

File: src/utils/config.cj:10
Error: missing import for `Config`

```cangjie
// Added
import std.config.Config
```

```bash
$ cjpm build
# Build successful!
```

## Final Verification

```bash
$ cjlint -f src/
# No warnings

$ cjpm test
running 6 tests
test service.cache.tests.test_insert ... ok
test service.cache.tests.test_get ... ok
test handler.api.tests.test_parse_count ... ok
test utils.config.tests.test_load ... ok

test result: ok. 6 passed; 0 failed
```

## Summary

| Metric | Count |
|--------|-------|
| Build errors fixed | 3 |
| Lint warnings fixed | 0 |
| Files modified | 3 |
| Remaining issues | 0 |

Build Status: SUCCESS
````

## Common Errors Fixed

| Error | Typical Fix |
|-------|-------------|
| Type mismatch | Add explicit type annotation or conversion |
| Undefined reference | Add import statement |
| Missing field | Add field to struct/class |
| Access modifier | Change visibility or add accessor |
| Option unwrap | Use `??` or `match` for safe handling |
| Missing return | Add return statement or expression |

## Fix Strategy

1. **Clean and update first** - Ensure fresh build environment
2. **Build errors second** - Code must compile
3. **Lint warnings third** - Fix suspicious constructs
4. **Formatting fourth** - `cjfmt` compliance
5. **One fix at a time** - Verify each change
6. **Minimal changes** - Don't refactor, just fix

## Stop Conditions

The agent will stop and report if:
- Same error persists after 3 attempts
- Fix introduces more errors
- Requires architectural changes
- Missing external dependencies

## HarmonyOS/OpenHarmony Specific

For HarmonyOS projects, additional checks:

```bash
# Build for HarmonyOS target
cjpm build --target harmonyos

# Check module.json5 validity
cjpm validate

# Package for deployment
cjpm package
```

## Related Commands

- `/cangjie-env` - Configure Cangjie environment and version
- `/cangjie-test` - Run tests after build succeeds
- `/cangjie-review` - Review code quality
- `/verify` - Full verification loop

## Related

- Agent: `agents/cangjie-build-resolver.md`
- Skill: `skills/cangjie-patterns/`
- Rules: `rules/cangjie/`