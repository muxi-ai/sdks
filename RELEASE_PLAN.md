# MUXI SDK Launch Plan

## Overview

This document outlines the SDK development strategy for MUXI, prioritizing languages based on target audience reach and leveraging a systematic porting approach to maximize quality while minimizing development time.

## Development Methodology

### Approach: Manual Porting via Claude Code

Rather than using OpenAPI Generator with Mustache templates (which produces functional but non-idiomatic code), we will:

1. **Refactor the existing Go CLI into a standalone Go SDK** — establishing patterns, conventions, and API surface
2. **Use Claude Code to systematically port** the Go SDK to each target language
3. **Ensure idiomatic code** for each language rather than generic generated output

### Why Not OpenAPI Generator?

- Generated code feels "generated" — not idiomatic
- Template customization (Mustache) is painful for complex logic
- Quality varies significantly by language generator
- Commercial alternatives (Speakeasy, Stainless) are prohibitively expensive ($720+/mo per language)

### Why This Approach Works

- Go SDK already exists conceptually in the CLI
- Claude Code excels at cross-language porting with pattern preservation
- One "gold standard" establishes conventions for all SDKs
- Results in handcrafted, idiomatic code that developers actually enjoy using

---

## Launch Schedule

### Phase 1: Initial Release ✅ COMPLETE

| Language | Package | Status |
|----------|---------|--------|
| Go | [`github.com/muxi-ai/muxi-go`](https://github.com/muxi-ai/muxi-go) | ✅ Published |
| Python | [`muxi` on PyPI](https://pypi.org/project/muxi/) | ✅ Published |
| TypeScript | [`@muxi-ai/muxi-typescript` on npm](https://www.npmjs.com/package/@muxi-ai/muxi-typescript) | ✅ Published |

**Rationale:** Go SDK is a refactor of existing CLI code. With Python and TypeScript, this covers 80%+ of early adopters. Python is non-negotiable for any AI platform. TypeScript covers the modern web development world.

---

### Phase 2: Week 2-3

| Language | Repo | Target Audience |
|----------|------|-----------------|
| PHP | [muxi-ai/muxi-php](https://github.com/muxi-ai/muxi-php) | Laravel ecosystem, WordPress, established web businesses |
| C# | [muxi-ai/muxi-csharp](https://github.com/muxi-ai/muxi-csharp) | .NET ecosystem, Unity game developers, Windows enterprise |
| Ruby | [muxi-ai/muxi-ruby](https://github.com/muxi-ai/muxi-ruby) | Rails applications, scripting, DevOps tooling |

**Checklist:**
- [ ] [PHP](https://github.com/muxi-ai/muxi-php)
- [ ] [C#](https://github.com/muxi-ai/muxi-csharp)
- [ ] [Ruby](https://github.com/muxi-ai/muxi-ruby)

---

### Phase 3: Week 3-4 (Mobile)

| Language | Repo | Target Audience |
|----------|------|-----------------|
| Swift | [muxi-ai/muxi-swift](https://github.com/muxi-ai/muxi-swift) | iOS applications, macOS apps |
| Kotlin | [muxi-ai/muxi-kotlin](https://github.com/muxi-ai/muxi-kotlin) | Android applications |
| Dart | [muxi-ai/muxi-dart](https://github.com/muxi-ai/muxi-dart) | Flutter apps (iOS + Android + Web from single codebase) |

**Checklist:**
- [ ] [Swift](https://github.com/muxi-ai/muxi-swift)
- [ ] [Kotlin](https://github.com/muxi-ai/muxi-kotlin)
- [ ] [Dart](https://github.com/muxi-ai/muxi-dart)

---

### Phase 4: Month 2 (Enterprise)

| Language | Repo | Target Audience |
|----------|------|-----------------|
| Java | [muxi-ai/muxi-java](https://github.com/muxi-ai/muxi-java) | Legacy enterprise, banking, insurance, large corporations |
| Rust | [muxi-ai/muxi-rust](https://github.com/muxi-ai/muxi-rust) | Performance-critical systems, crypto/blockchain, security-focused applications |
| C/C++ | [muxi-ai/muxi-cpp](https://github.com/muxi-ai/muxi-cpp) | Legacy enterprise, embedded systems, game engines (Unreal), automotive, robotics, HFT |

**Checklist:**
- [ ] [Java](https://github.com/muxi-ai/muxi-java)
- [ ] [Rust](https://github.com/muxi-ai/muxi-rust)
- [ ] [C/C++](https://github.com/muxi-ai/muxi-cpp)

---

## Summary

| Phase | Timeline | Languages | Status |
|-------|----------|-----------|--------|
| Phase 1 | Initial Release | Go, Python, TypeScript | ✅ Complete |
| Phase 2 | Week 2-3 | PHP, C#, Ruby | Planned |
| Phase 3 | Week 3-4 | Swift, Kotlin, Dart | Planned |
| Phase 4 | Month 2 | Java, Rust, C/C++ | Planned |

**Total: 12 SDK languages within 2 months**

---

## Package Registry

| Language | Package Name | Registry | Status |
|----------|--------------|----------|--------|
| Go | `github.com/muxi-ai/muxi-go` | Go modules | ✅ |
| Python | `muxi` | PyPI | ✅ |
| TypeScript | `@muxi-ai/muxi-typescript` | npm | ✅ |
| PHP | `muxi/muxi-php` | Packagist | — |
| C# | `Muxi` | NuGet | — |
| Ruby | `muxi` | RubyGems | — |
| Swift | `Muxi` | Swift Package Manager | — |
| Kotlin | `dev.muxi:muxi-kotlin` | Maven Central | — |
| Dart | `muxi` | pub.dev | — |
| Java | `dev.muxi:muxi-java` | Maven Central | — |
| Rust | `muxi` | crates.io | — |
| C/C++ | `libmuxi` | vcpkg / Conan | — |

---

## Quality Checklist Per SDK

- [x] Idiomatic code style for the language
- [x] Full type safety / type hints
- [x] Async support where applicable
- [x] Streaming support with clean iteration patterns
- [x] Comprehensive error handling
- [x] Retry logic with exponential backoff
- [x] Request/response logging (debug mode)
- [x] Examples that actually work
- [x] README with quick start
- [x] Published to language package manager
- [x] CI/CD with automated releases
