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

### Phase 1: Initial Release

| Language | Repo | Target Audience |
|----------|------|-----------------|
| Python | [muxi-ai/muxi-python](https://github.com/muxi-ai/muxi-python) | AI/ML developers, data scientists, the entire AI ecosystem |
| TypeScript | [muxi-ai/muxi-node](https://github.com/muxi-ai/muxi-node) | Web applications, Node.js backends, startup ecosystem |
| Go | [muxi-ai/muxi-go](https://github.com/muxi-ai/muxi-go) | Cloud/infrastructure teams, Kubernetes ecosystem, DevOps |

**Rationale:** Go SDK is a refactor of existing CLI code. With Python and TypeScript, this cover 80%+ of early adopters. Python is non-negotiable for any AI platform. TypeScript covers the modern web development world.

**Checklist:**
- [ ] [Python](https://github.com/muxi-ai/muxi-python)
- [ ] [TypeScript](https://github.com/muxi-ai/muxi-node)
- [ ] [Go](https://github.com/muxi-ai/muxi-go)

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

| Phase | Timeline | Languages | Cumulative Total |
|-------|----------|-----------|------------------|
| Phase 1 | Initial Release | Python, TypeScript, Go | 3 |
| Phase 2 | Week 2-3 | PHP, C#, Ruby | 6 |
| Phase 3 | Week 3-4 | Swift, Kotlin, Dart | 9 |
| Phase 4 | Month 2 | Java, Rust, C/C++ | 12 |

**Total: 12 SDK languages within 2 months**

---

## SDK Architecture Patterns

### Establish in Go First

The Go SDK sets the patterns that all other SDKs will follow:

```
muxi-{language}/
├── client           # Main client, configuration, authentication
├── agents           # agent.Create(), agent.Get(), agent.List(), agent.Delete()
├── sessions         # Session lifecycle management
├── messages         # Message sending, streaming, async patterns
├── models           # All type definitions
├── errors           # Typed errors (MuxiError, RateLimitError, AuthError, etc.)
├── options          # Configuration options pattern
└── examples/
    ├── basic/
    ├── streaming/
    └── agents/
```

### Key Patterns to Nail

#### 1. Clean Instantiation

```go
// Go
client := muxi.NewClient(
    muxi.WithAPIKey(os.Getenv("MUXI_API_KEY")),
    muxi.WithBaseURL("https://api.muxi.dev"),
)
```

```python
# Python
client = Muxi(api_key=os.environ["MUXI_API_KEY"])
```

```typescript
// TypeScript
const client = new Muxi({ apiKey: process.env.MUXI_API_KEY });
```

#### 2. Intuitive Resource Access

```go
// Go
agent, err := client.Agents.Create(ctx, &muxi.AgentCreateParams{
    Name:         "my-agent",
    Model:        "claude-sonnet-4-20250514",
    Instructions: muxi.String("You are helpful"),
})
```

```python
# Python
agent = await client.agents.create(
    name="my-agent",
    model="claude-sonnet-4-20250514",
    instructions="You are helpful",
)
```

#### 3. Streaming That Doesn't Suck

```go
// Go
stream, err := client.Messages.Stream(ctx, sessionID, "Hello!")
for event := range stream.Events() {
    switch e := event.(type) {
    case *muxi.TextDelta:
        fmt.Print(e.Text)
    case *muxi.ToolCall:
        // handle tool
    }
}
```

```python
# Python
async for event in client.messages.stream(session_id, "Hello!"):
    match event:
        case TextDelta(text=text):
            print(text, end="")
        case ToolCall() as tool:
            # handle tool
```

#### 4. Typed Errors

Every SDK should have consistent, typed error handling:

- `MuxiError` — Base error type
- `AuthenticationError` — Invalid/missing API key
- `RateLimitError` — Rate limit exceeded (with retry-after info)
- `ValidationError` — Invalid request parameters
- `NotFoundError` — Resource not found
- `ServerError` — MUXI server issues

---

## Development Workflow

### For Each Language After Go:

1. **Feed Claude Code** the complete Go SDK + target language idioms/best practices
2. **Start with models** — generate all type definitions first
3. **Then client** — core HTTP, authentication, configuration
4. **Then resources** — agents, sessions, messages (one at a time)
5. **Then errors** — typed error hierarchy
6. **Add examples** — basic, streaming, real-world use cases
7. **Review and polish** — ensure idiomatic feel
8. **Write tests** — unit tests, integration tests
9. **Documentation** — README, API docs, getting started guide

### Quality Checklist Per SDK:

- [ ] Idiomatic code style for the language
- [ ] Full type safety / type hints
- [ ] Async support where applicable
- [ ] Streaming support with clean iteration patterns
- [ ] Comprehensive error handling
- [ ] Retry logic with exponential backoff
- [ ] Request/response logging (debug mode)
- [ ] Examples that actually work
- [ ] README with quick start
- [ ] Published to language package manager

---

## Package Registry

| Language | Package Name | Registry |
|----------|--------------|----------|
| Python | `muxi` | PyPI |
| TypeScript | `muxi` or `@muxi/sdk` | npm |
| Go | `github.com/muxi-ai/muxi-go` | Go modules |
| PHP | `muxi/muxi-php` | Packagist |
| C# | `Muxi` | NuGet |
| Ruby | `muxi` | RubyGems |
| Swift | `Muxi` | Swift Package Manager |
| Kotlin | `dev.muxi:muxi-kotlin` | Maven Central |
| Dart | `muxi` | pub.dev |
| Java | `dev.muxi:muxi-java` | Maven Central |
| Rust | `muxi` | crates.io |
| C/C++ | `libmuxi` | vcpkg / Conan |

---

## Success Metrics

### Per SDK:

- Clean, readable code that feels native to the language
- < 5 minutes from install to first API call
- Comprehensive examples covering common use cases
- Positive developer feedback on ergonomics

### Overall:

- All Phase 1-3 SDKs (9 languages) shipped within 1 month
- All SDKs (12 languages) shipped within 2 months
- Consistent API surface across all languages
- Documentation parity across all SDKs
