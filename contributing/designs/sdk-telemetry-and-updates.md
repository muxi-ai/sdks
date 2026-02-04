# SDK Telemetry and Version Notifications

**Version:** 1.0  
**Last Updated:** 2025-02-04  
**Status:** Implemented

This document describes two features added to all 12 MUXI SDKs:
1. Console telemetry support via the `_app` parameter
2. SDK version update notifications

---

## 1. Console Telemetry (`_app` Parameter)

### Purpose

The `_app` parameter enables the MUXI Console (console.muxi.dev) to identify requests originating from its interface. This is an internal/undocumented parameter used for telemetry and analytics.

### Implementation

Both `ServerConfig` and `FormationConfig` include an internal `app` field:

| Language | Field Name | Visibility |
|----------|------------|------------|
| TypeScript | `_app` | Not exported in types |
| Python | `_app` | Underscore prefix (internal) |
| Go | `App` | Exported but undocumented |
| Ruby | `app` | Internal, undocumented |
| PHP | `app` | Private property |
| C# | `App` | Internal property |
| Swift | `app` | Internal, undocumented |
| Kotlin | `app` | Internal, undocumented |
| Dart | `app` | Internal, undocumented |
| Java | `app` | Package-private constructor |
| Rust | `app` | `pub(crate)` visibility |
| C++ | `app_` | Underscore suffix (internal) |

When set, the SDK sends an `X-Muxi-App` header with requests:

```
X-Muxi-App: console
```

### Usage (Internal Only)

```typescript
// TypeScript (internal usage by Console)
const client = new FormationClient({
  serverUrl: "https://api.muxi.ai",
  formationId: "my-formation",
  clientKey: "key",
  adminKey: "admin",
  _app: "console"  // Internal parameter
});
```

---

## 2. SDK Version Notifications

### Purpose

Notify developers when a newer SDK version is available, encouraging them to stay up-to-date with security patches and new features.

### How It Works

1. **Server Response**: The MUXI server includes an `X-Muxi-SDK-Latest` header in responses containing the latest SDK version
2. **Version Check**: SDK compares the header value against its current version
3. **Notification**: If newer version available, prints to stderr (once per 12 hours)
4. **Caching**: Version info cached in `~/.muxi/sdk-versions.json`

### Notification Format

```
[muxi] SDK update available: 1.2.0 (current: 1.0.0)
[muxi] Run: pip install --upgrade muxi
```

Update commands vary by language:
- TypeScript: `npm update @muxi-ai/muxi-typescript`
- Python: `pip install --upgrade muxi`
- Go: `go get -u github.com/muxi-ai/muxi-go`
- Ruby: `gem update muxi`
- PHP: `composer update muxi/muxi-php`
- C#: `dotnet add package Muxi --version X.Y.Z`
- Swift: Update Package.swift dependency
- Kotlin: Update Gradle/Maven dependency
- Dart: `dart pub upgrade muxi`
- Java: Update Gradle/Maven dependency
- Rust: `cargo update -p muxi`
- C++: Update CMake dependency

### Disabling Notifications

Set environment variable:

```bash
export MUXI_SDK_VERSION_NOTIFICATION=0
```

Notifications are **enabled by default**. Setting `MUXI_SDK_VERSION_NOTIFICATION=0` disables them.

### Cache File Structure

Location: `~/.muxi/sdk-versions.json`

```json
{
  "python": {
    "current": "1.0.0",
    "latest": "1.2.0",
    "last_notified": "2025-02-04T10:30:00Z"
  },
  "typescript": {
    "current": "1.0.0",
    "latest": "1.1.0",
    "last_notified": "2025-02-04T09:00:00Z"
  }
}
```

### Implementation Files

Each SDK has a version check module:

| SDK | File |
|-----|------|
| TypeScript | `src/version-check.ts` |
| Python | `muxi/version_check.py` |
| Go | `src/version_check.go` |
| Ruby | `lib/muxi/version_check.rb` |
| PHP | `src/VersionCheck.php` |
| C# | `src/Muxi/VersionCheck.cs` |
| Swift | `Sources/Muxi/VersionCheck.swift` |
| Kotlin | `src/main/kotlin/dev/muxi/sdk/VersionCheck.kt` |
| Dart | `lib/src/version_check.dart` |
| Java | `src/main/java/dev/muxi/sdk/VersionCheck.java` |
| Rust | `src/version_check.rs` |
| C++ | `src/version_check.cpp`, `include/muxi/version_check.hpp` |

### Behavior Details

- **Check Frequency**: Once per process (uses static/singleton flag)
- **Notification Throttle**: Max once per 12 hours (per SDK)
- **Output**: stderr (not stdout) to avoid interfering with program output
- **Failure Handling**: All cache operations silently fail (no exceptions)
- **Comparison**: Simple string comparison (semver-compatible)

---

## 3. Request Headers

All SDK requests include these headers:

| Header | Description | Example |
|--------|-------------|---------|
| `X-Muxi-SDK` | SDK identifier and version | `python/1.0.0` |
| `X-Muxi-Client` | Client identifier (same as SDK) | `python/1.0.0` |
| `X-Muxi-Idempotency-Key` | UUID for request deduplication | `550e8400-e29b-41d4-a716-446655440000` |
| `X-Muxi-App` | Application identifier (optional) | `console` |

---

## 4. Server-Side Requirements

For version notifications to work, the MUXI server must:

1. Maintain a registry of latest SDK versions (fetched daily from `.version` files in SDK repos)
2. Include `X-Muxi-SDK-Latest` header in all responses
3. Parse the `X-Muxi-SDK` header to determine which SDK version to return

Example server middleware:

```python
def add_sdk_version_header(response, request):
    sdk_header = request.headers.get("X-Muxi-SDK", "")
    if "/" in sdk_header:
        sdk_name = sdk_header.split("/")[0]
        latest = get_latest_sdk_version(sdk_name)
        if latest:
            response.headers["X-Muxi-SDK-Latest"] = latest
    return response
```
