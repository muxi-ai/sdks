# MUXI SDK Launch Plan

## Overview

This document outlines the SDK development strategy for MUXI, prioritizing languages based on target audience reach and leveraging a systematic porting approach to maximize quality while minimizing development time.

## Development Methodology

### Approach: Manual Porting via Claude Code

Rather than using OpenAPI Generator with Mustache templates (which produces functional but non-idiomatic code), we will:

1. **Refactor the existing Go CLI into a standalone Go SDK** - establishing patterns, conventions, and API surface
2. **Use Claude Code to systematically port** the Go SDK to each target language
3. **Ensure idiomatic code** for each language rather than generic generated output

### Why Not OpenAPI Generator?

- Generated code feels "generated" - not idiomatic
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

### Phase 1: Launch (Day 0)

| Language | Effort | Target Audience |
|----------|--------|-----------------|
| **Python** | 3-4 days | AI/ML developers, data scientists, the entire AI ecosystem |
| **TypeScript** | 3-4 days | Web applications, Node.js backends, startup ecosystem |

**Rationale:** These two languages cover 80%+ of early adopters. Python is non-negotiable for any AI platform. TypeScript covers the modern web development world.

---

### Phase 2: Week 1-2

| Language | Effort | Target Audience |
|----------|--------|-----------------|
| **Go** | 3-5 days | Cloud/infrastructure teams, Kubernetes ecosystem, DevOps |
| **PHP** | 3-4 days | Laravel ecosystem, WordPress, established web businesses |

**Rationale:** Go SDK is a refactor of existing CLI code. PHP is underserved by AI platforms—having a polished PHP SDK early is a genuine differentiator.

---

### Phase 3: Week 3-4 (Mobile)

| Language | Effort | Target Audience |
|----------|--------|-----------------|
| **Swift** | 4-5 days | iOS applications, macOS apps |
| **Kotlin** | 4-5 days | Android applications (skip Java for mobile—Kotlin is the standard) |

**Rationale:** Completes the "full platform" story. Mobile agents are a key use case for agentic platforms.

---

### Phase 4: Month 2 (Enterprise)

| Language | Effort | Target Audience |
|----------|--------|-----------------|
| **C#** | 4-5 days | .NET ecosystem, Unity game developers, Windows enterprise |
| **Java** | 4-5 days | Legacy enterprise, banking, insurance, large corporations |

**Rationale:** Opens enterprise sales conversations. C# also enables Unity integration for agentic NPCs in games.

---

### Phase 5: Month 2-3 (Performance & Legacy)

| Language | Effort | Target Audience |
|----------|--------|-----------------|
| **C/C++** | 5-7 days | Legacy enterprise, embedded systems, game engines (Unreal), automotive, robotics, HFT |
| **Rust** | 5-7 days | Performance-critical systems, crypto/blockchain, security-focused applications |

**Rationale:** Enables embedded agents, IoT deployments, and performance-critical use cases. C/C++ also unlocks potential FFI bindings for other languages.

---

## Total Timeline

| Phase | Timeline | Languages | Cumulative Total |
|-------|----------|-----------|------------------|
| Launch | Day 0 | Python, TypeScript | 2 |
| Phase 2 | Week 1-2 | Go, PHP | 4 |
| Phase 3 | Week 3-4 | Swift, Kotlin | 6 |
| Phase 4 | Month 2 | C#, Java | 8 |
| Phase 5 | Month 2-3 | C/C++, Rust | 10 |

**Total: 10 SDK languages within 3 months**

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

- `MuxiError` - Base error type
- `AuthenticationError` - Invalid/missing API key
- `RateLimitError` - Rate limit exceeded (with retry-after info)
- `ValidationError` - Invalid request parameters
- `NotFoundError` - Resource not found
- `ServerError` - MUXI server issues

---

## Development Workflow

### For Each Language After Go:

1. **Feed Claude Code** the complete Go SDK + target language idioms/best practices
2. **Start with models** - generate all type definitions first
3. **Then client** - core HTTP, authentication, configuration
4. **Then resources** - agents, sessions, messages (one at a time)
5. **Then errors** - typed error hierarchy
6. **Add examples** - basic, streaming, real-world use cases
7. **Review and polish** - ensure idiomatic feel
8. **Write tests** - unit tests, integration tests
9. **Documentation** - README, API docs, getting started guide

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
- [ ] Published to language package manager (PyPI, npm, etc.)

---

## Package Naming Convention

| Language | Package Name | Registry |
|----------|--------------|----------|
| Python | `muxi` | PyPI |
| TypeScript | `@muxi/sdk` or `muxi` | npm |
| Go | `github.com/muxi-dev/muxi-go` | Go modules |
| PHP | `muxi/muxi-php` | Packagist |
| Swift | `Muxi` | Swift Package Manager |
| Kotlin | `dev.muxi:muxi-kotlin` | Maven Central |
| C# | `Muxi` | NuGet |
| Java | `dev.muxi:muxi-java` | Maven Central |
| C/C++ | `libmuxi` | TBD (vcpkg, Conan, or direct) |
| Rust | `muxi` | crates.io |

---

## Success Metrics

### Per SDK:

- Clean, readable code that feels native to the language
- < 5 minutes from install to first API call
- Comprehensive examples covering common use cases
- Positive developer feedback on ergonomics

### Overall:

- All Phase 1-3 SDKs (6 languages) shipped within 1 month
- All SDKs (10 languages) shipped within 3 months
- Consistent API surface across all languages
- Documentation parity across all SDKs
