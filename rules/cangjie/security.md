---
paths:
  - "**/*.cj"
---
# Cangjie Security

> This file extends [common/security.md](../common/security.md) with Cangjie-specific content.

## Secrets Management

Never hardcode sensitive data in source code:

```cangjie
// BAD — hardcoded secret
let apiKey: String = "sk-abc123..."

// GOOD — environment variable
func loadApiKey(): String {
    Environment.get("API_KEY") ?? throw ConfigException("API_KEY not set")
}
```

Keep `.env` and config files with secrets in `.gitignore`.

## Input Validation

Validate at system boundaries:

```cangjie
struct Email {
    private let value: String

    init(input: String) {
        let trimmed = input.trim()
        if (!trimmed.contains("@") || trimmed.size > 254) {
            throw ValidationException("Invalid email: ${input}")
        }
        value = trimmed
    }

    public func asString(): String { value }
}
```

Use Option for safe parsing:

```cangjie
func parsePort(input: String): Option<Int32> {
    let parsed = Int32.parse(input)
    if (let Some(p) = parsed) {
        if (p > 0 && p <= 65535) { Some(p) } else { None }
    } else {
        None
    }
}
```

## FFI Safety

Cangjie FFI calls require `unsafe` block:

```cangjie
// FFI declaration
foreign func c_memcpy(dest: CPointer<Byte>, src: CPointer<Byte>, n: CSize): CPointer<Byte>

// Safe wrapper
func safeCopy(dest: Array<Byte>, src: Array<Byte>): Unit {
    unsafe {
        c_memcpy(dest.cPtr(), src.cPtr(), src.size)
    }
}
```

Best practices:
- Minimize `unsafe` block scope
- Validate all data passed to C functions
- Use `CPointerResource`/`CStringResource` for RAII memory management
- Only use `@FastNative` for short, non-blocking C functions

## Concurrency Safety

Shared mutable state must use synchronization:

```cangjie
// GOOD — Mutex for shared state
class SafeCounter {
    private let mutex = Mutex()
    private var count: Int64 = 0

    public func increment(): Int64 {
        synchronized(mutex) {
            count += 1
            count
        }
    }
}

// GOOD — ThreadLocal for thread-specific data
let threadData = ThreadLocal<Data>()

// GOOD — Atomic operations
let atomicCounter = Atomic<Int64>(0)
atomicCounter.fetchAdd(1)
```

Do not expose lock objects to external code.

## Error Messages

Never expose internal details in user-facing errors:

```cangjie
func handleRequest(req: Request): Response {
    try {
        processOrder(req)
    } catch (e: DatabaseException) {
        log.error("DB error: ${e.message}")  // Server-side log
        Response.error("Service temporarily unavailable")  // Client message
    } catch (e: Exception) {
        log.error("Unexpected: ${e}")
        Response.error("Internal error")
    }
}
```

## Dependency Security

- Use `cjpm tree` to audit dependencies
- Keep dependencies updated
- Review third-party packages before adding
- Prefer `std.*` (standard library) and `stdx.*` (extended stdlib)

```bash
cjpm tree            # View dependency tree
cjpm update          # Update dependencies
cjpm check           # Check dependency status
```

## References

See skill: `cangjie-regulations` Section 10 for complete security standards.
See skill: `cangjie-lang-features/cffi` for FFI safety patterns.

---

## HarmonyOS Security Extensions

### Permissions

Declare required permissions in `module.json5`:

```json5
{
  "module": {
    "requestPermissions": [
      { "name": "ohos.permission.INTERNET" },
      { "name": "ohos.permission.GET_NETWORK_INFO" },
      { "name": "ohos.permission.LOCATION" }
    ]
  }
}
```

### Common Permissions

| Permission | Purpose |
|------------|---------|
| `ohos.permission.INTERNET` | Network access |
| `ohos.permission.GET_NETWORK_INFO` | Network state |
| `ohos.permission.LOCATION` | GPS location |
| `ohos.permission.CAMERA` | Camera access |
| `ohos.permission.READ_MEDIA` | Read media files |
| `ohos.permission.WRITE_MEDIA` | Write media files |

### ArkTS Interop Security

Validate input at interop boundaries:

```cangjie
import ohos.ark_interop_macro.*

// Validate input from ArkTS side
@Interop[ArkTS]
public func processUserInput(input: String): String {
    let sanitized = input.trim()
    if (sanitized.size > MAX_INPUT_LENGTH) {
        throw ValidationException("Input too long")
    }
    sanitizeHtml(sanitized)
}
```

### Network Security

Always use HTTPS:

```cangjie
import std.net.http.*

let client = HttpClient()
let response = client.get("https://api.example.com/data")
```

### Data Storage Security

Store sensitive files in protected app area:

```cangjie
// Files in filesDir are private to the app
let file = context.filesDir.open("secure_data.bin")
```

### UI Security

Mask sensitive information:

```cangjie
TextInput(placeholder: "Password")
    .type(InputType.Password)  // Masks input

Text(maskCardNumber(cardNumber))  // Show only last 4 digits
```

### Input Validation in UI

```cangjie
TextInput(placeholder: "Enter amount")
    .inputFilter { value =>
        let regex = Regex("^[0-9.]*$")
        regex.matches(value)
    }
```

### References (HarmonyOS)

See skill: `cangjie_arkts_interop` for secure interop patterns.
See skill: `harmonyos-requirements` for security requirements analysis.