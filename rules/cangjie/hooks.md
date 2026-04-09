---
paths:
  - "**/*.cj"
  - "**/cjpm.toml"
---
# Cangjie Hooks

> This file extends [common/hooks.md](../common/hooks.md) with Cangjie-specific content.

## PostToolUse Hooks

Configure in `~/.claude/settings.json`:

- **cjfmt**: Auto-format `.cj` files after edit
- **cjlint**: Run static checks after editing Cangjie files
- **cjpm check**: Verify compilation after changes

## Example Hook Configuration

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": {
          "toolName": "Edit|Write",
          "filePath": "**/*.cj$"
        },
        "hooks": [
          {
            "type": "command",
            "command": "cjfmt ${file_path}"
          }
        ]
      }
    ]
  }
}
```

## Available Tools

| Tool | Purpose | Command |
|------|---------|---------|
| cjfmt | Code formatting | `cjfmt src/` |
| cjlint | Static analysis | `cjlint -f src/` |
| cjpm check | Compilation check | `cjpm check` |
| cjpm build | Full build | `cjpm build` |
| cjpm test | Run tests | `cjpm test` |

## Hook Best Practices

1. **Formatting**: Run cjfmt after every edit to maintain consistency
2. **Lint**: Run cjlint after cjfmt to catch issues early
3. **Build check**: For larger changes, run `cjpm check` to verify compilation

## CI Integration

Recommended CI pipeline:

```bash
cjpm build               # 1. Compile
cjfmt -d src/            # 2. Check formatting
cjlint -f src/           # 3. Static analysis
cjpm test                # 4. Run tests
cjcov --branches         # 5. Coverage report
```

## References

See skill: `cangjie-toolchains` for complete tool documentation.

---

## HarmonyOS Tools Extension

### Additional Tools

| Tool | Purpose | Command |
|------|---------|---------|
| ohpm | Dependency management | `ohpm install` |
| hdc | Device communication | `hdc list targets` |

### Build Verification

Run build after significant changes:

```powershell
# Using harmonyos-build skill
python .agents/skills/harmonyos-build/build.py --project-root .

# Or manually
ohpm install && cjpm build
```

### UI Inspection

After successful build, verify UI:

```powershell
python .agents/skills/harmonyos-ui-inspect/ui_capture.py --emulator 5555 --out ./ui_output
```

### CI Pipeline (HarmonyOS)

```bash
ohpm install                    # Install dependencies
cjfmt -d src/                   # Check formatting
cjlint -f src/                  # Static analysis
cjpm build                      # Build
cjpm test                       # Run tests
cjcov --branches                # Coverage report
```

### References (HarmonyOS)

See skill: `harmonyos-build` for build automation.
See skill: `harmonyos-ui-inspect` for UI verification.
See skill: `harmonyos-project-init` for project setup.