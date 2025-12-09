# MUXI SDK Design Philosophy

**Version:** 1.1 
**Last Updated:** 2025-12-09 
**Status:** Design Document

---

## Overview

MUXI SDKs provide programmatic access to both **server management** (Server API) and **formation usage** (Formation API). Unlike the CLI, which is optimized for human operators, SDKs are designed as building blocks for automation, CI/CD, testing, and application development.

**Key Principle:** Support both management and usage, but with explicit, composable APIs — no CLI-specific magic.

---

## Design Philosophy: CLI vs SDK

| Aspect | CLI | SDK |
|--------|-----|-----|
| **Audience** | Human operators/developers | Programmatic automation |
| **Profile management** | Yes (wizard, auto-detect) | No (pass creds directly) |
| **Context detection** | Yes (formation-as-directory) | No (explicit everything) |
| **Wizards/prompts** | Yes (interactive) | No (just functions) |
| **Output** | Pretty, colored, human | Structured data, exceptions |
| **Auto-detection** | Yes (local server) | No (explicit URLs) |
| **Config files** | Yes (~/.muxi/cli/profiles.yaml) | No (env vars or app config) |
| **Error handling** | User-friendly messages | Structured exceptions |

**Summary:**
- **CLI = Human-friendly tool** (interactive, context-aware, pretty)
- **SDK = Programmatic building blocks** (explicit, composable, testable)

---

## Architecture: Two Separate Clients

SDKs should provide **two distinct clients** for clarity and separation of concerns:

### 1. ServerClient - Server Management (Server API)

For operations on the MUXI Server itself — deploying, listing, stopping formations, and server administration.

**Responsibilities:**
- Formation lifecycle (deploy, update, stop, restart, delete)
- Formation listing and details
- Server status and logs
- Rollback operations

**API Endpoints:** `/rpc/*` (HMAC authenticated)

### 2. FormationClient - Formation Usage (Formation API)

For interacting with a specific formation's runtime API — chat, health checks, logs, etc.

**Responsibilities:**
- Chat/query operations (including streaming)
- Health checks
- Formation-specific logs
- Agent operations
- Secret management
- MCP operations

**API Endpoints:** Direct to formation port OR via server proxy `/api/{id}/*`

---

## Example Implementations

### Python SDK

```python
# 1. ServerClient - for management
from muxi import ServerClient

server = ServerClient(
    url="https://muxi.company.com:7890",
    key_id="MUXI_PROD_KEY",
    secret_key="sk_..."
)

# Formation lifecycle
result = server.deploy_formation(
    bundle_path="my-bot.tar.gz",
    metadata={"id": "my-bot", "name": "My Bot"}
)
print(f"Deployed: {result.formation_id} v{result.version}")

formations = server.list_formations()
for f in formations:
    print(f"{f.id}: {f.status} (v{f.version})")

server.stop_formation("my-bot")
server.restart_formation("my-bot")
server.delete_formation("my-bot")

# Server management
status = server.status()
print(f"Server uptime: {status.uptime}s, formations: {status.formations.total}")

logs = server.logs(lines=100)
for log in logs:
    print(f"[{log.timestamp}] {log.method} {log.path}")


# 2. FormationClient - for using formations
from muxi import FormationClient

# Developer mode - direct connection
client = FormationClient(
    formation_id="my-bot",
    url="http://localhost:8001"  # Direct to formation
)

# Operator mode - via server proxy
client = FormationClient(
    formation_id="my-bot",
    server_url="https://muxi.company.com:7890",
    key_id="MUXI_PROD_KEY",
    secret_key="sk_..."
)

# Use the formation (sync)
response = client.chat("Hello, how are you?")
print(response.message)

health = client.health()
assert health.status == "healthy"

logs = client.logs(lines=50, follow=False)
for log in logs:
    print(log)
```

#### Python Async Support

```python
from muxi import AsyncServerClient, AsyncFormationClient

# Async server client
server = AsyncServerClient(
    url="https://muxi.company.com:7890",
    key_id="MUXI_PROD_KEY",
    secret_key="sk_..."
)

formations = await server.list_formations()
await server.deploy_formation(bundle_path="my-bot.tar.gz", metadata={...})

# Async formation client
client = AsyncFormationClient(
    formation_id="my-bot",
    server_url="https://muxi.company.com:7890",
    key_id="MUXI_PROD_KEY",
    secret_key="sk_..."
)

# Non-streaming async
response = await client.chat("Hello!")
print(response.message)

# Streaming async
async for chunk in client.chat_stream("Tell me a story"):
    print(chunk.text, end="", flush=True)
```

### TypeScript/JavaScript SDK

```typescript
// 1. ServerClient - for management
import { ServerClient } from 'muxi';

const server = new ServerClient({
  url: 'https://muxi.company.com:7890',
  keyId: process.env.MUXI_KEY_ID,
  secretKey: process.env.MUXI_SECRET_KEY,
});

// Formation lifecycle
const result = await server.deployFormation({
  bundlePath: 'my-bot.tar.gz',
  metadata: { id: 'my-bot', name: 'My Bot' },
});
console.log(`Deployed: ${result.formation_id} v${result.version}`);

const formations = await server.listFormations();
formations.forEach(f => {
  console.log(`${f.id}: ${f.status} (v${f.version})`);
});

await server.stopFormation('my-bot');
await server.restartFormation('my-bot');
await server.deleteFormation('my-bot');

// Server management
const status = await server.status();
console.log(`Server uptime: ${status.uptime}s`);


// 2. FormationClient - for using formations
import { FormationClient } from 'muxi';

// Developer mode - direct connection
const client = new FormationClient({
  formationId: 'my-bot',
  url: 'http://localhost:8001',
});

// Operator mode - via server proxy
const client = new FormationClient({
  formationId: 'my-bot',
  serverUrl: 'https://muxi.company.com:7890',
  keyId: process.env.MUXI_KEY_ID,
  secretKey: process.env.MUXI_SECRET_KEY,
});

// Non-streaming
const response = await client.chat('Hello, how are you?');
console.log(response.message);

// Streaming
for await (const chunk of client.chatStream('Tell me a story')) {
  process.stdout.write(chunk.text);
}

const health = await client.health();
console.log(health.status);
```

### Go SDK

```go
package main

import (
    "context"
    "fmt"
    "os"

    "github.com/muxi-ai/muxi-go"
)

func main() {
    ctx := context.Background()

    // 1. ServerClient - for management
    server := muxi.NewServerClient(&muxi.ServerConfig{
        URL:       "https://muxi.company.com:7890",
        KeyID:     "MUXI_PROD_KEY",
        SecretKey: "sk_...",
    })

    // Formation lifecycle
    result, err := server.DeployFormation(ctx, &muxi.DeployRequest{
        BundlePath: "my-bot.tar.gz",
        Metadata: map[string]string{
            "id":   "my-bot",
            "name": "My Bot",
        },
    })
    if err != nil {
        panic(err)
    }
    fmt.Printf("Deployed: %s v%s\n", result.FormationID, result.Version)

    formations, err := server.ListFormations(ctx)
    if err != nil {
        panic(err)
    }
    for _, f := range formations {
        fmt.Printf("%s: %s (v%s)\n", f.ID, f.Status, f.Version)
    }

    // 2. FormationClient - for using formations
    client := muxi.NewFormationClient(&muxi.FormationConfig{
        FormationID: "my-bot",
        ServerURL:   "https://muxi.company.com:7890",
        KeyID:       "MUXI_PROD_KEY",
        SecretKey:   "sk_...",
    })

    // Non-streaming
    response, err := client.Chat(ctx, "Hello, how are you?")
    if err != nil {
        panic(err)
    }
    fmt.Println(response.Message)

    // Streaming
    stream, err := client.ChatStream(ctx, "Tell me a story")
    if err != nil {
        panic(err)
    }
    for chunk := range stream.Events() {
        fmt.Print(chunk.Text)
    }
}
```

---

## Streaming Support

All SDKs must support streaming responses for chat operations. This is critical for real-time UX.

### Streaming Patterns by Language

| Language | Pattern | Example |
|----------|---------|---------|
| Python (async) | `async for` | `async for chunk in client.chat_stream(...)` |
| Python (sync) | Iterator | `for chunk in client.chat_stream(...)` |
| TypeScript | `for await` | `for await (const chunk of client.chatStream(...))` |
| Go | Channel | `for chunk := range stream.Events()` |
| Swift | `AsyncSequence` | `for await chunk in client.chatStream(...)` |
| Kotlin | `Flow` | `client.chatStream(...).collect { chunk -> }` |
| Rust | `Stream` | `while let Some(chunk) = stream.next().await` |

### Chunk Structure

```typescript
interface ChatChunk {
  type: 'text' | 'tool_call' | 'tool_result' | 'done' | 'error';
  text?: string;           // For type='text'
  toolCall?: ToolCall;     // For type='tool_call'
  toolResult?: ToolResult; // For type='tool_result'
  error?: string;          // For type='error'
}
```

### Python Streaming Examples

```python
# Sync streaming
from muxi import FormationClient

client = FormationClient(formation_id="my-bot", url="http://localhost:8001")

for chunk in client.chat_stream("Tell me a story"):
    if chunk.type == "text":
        print(chunk.text, end="", flush=True)
    elif chunk.type == "tool_call":
        print(f"\n[Calling tool: {chunk.tool_call.name}]")
    elif chunk.type == "error":
        print(f"\nError: {chunk.error}")

# Async streaming
from muxi import AsyncFormationClient

client = AsyncFormationClient(formation_id="my-bot", url="http://localhost:8001")

async for chunk in client.chat_stream("Tell me a story"):
    match chunk.type:
        case "text":
            print(chunk.text, end="", flush=True)
        case "tool_call":
            print(f"\n[Calling tool: {chunk.tool_call.name}]")
        case "done":
            print("\n[Complete]")
```

---

## Retry and Backoff

SDKs should handle transient failures gracefully with configurable retry behavior.

### Default Retry Configuration

```python
# Python
client = FormationClient(
    formation_id="my-bot",
    url="http://localhost:8001",
    retry=RetryConfig(
        max_retries=3,              # Max retry attempts
        initial_delay=0.5,          # Initial delay in seconds
        max_delay=30.0,             # Max delay between retries
        exponential_base=2,         # Exponential backoff multiplier
        retryable_status_codes=[    # Status codes to retry
            408,  # Request Timeout
            429,  # Too Many Requests
            500,  # Internal Server Error
            502,  # Bad Gateway
            503,  # Service Unavailable
            504,  # Gateway Timeout
        ],
    )
)

# Disable retries
client = FormationClient(
    formation_id="my-bot",
    url="http://localhost:8001",
    retry=None  # No retries
)
```

```typescript
// TypeScript
const client = new FormationClient({
  formationId: 'my-bot',
  url: 'http://localhost:8001',
  retry: {
    maxRetries: 3,
    initialDelay: 500,  // ms
    maxDelay: 30000,    // ms
    exponentialBase: 2,
  },
});
```

```go
// Go
client := muxi.NewFormationClient(&muxi.FormationConfig{
    FormationID: "my-bot",
    URL:         "http://localhost:8001",
    Retry: &muxi.RetryConfig{
        MaxRetries:     3,
        InitialDelay:   500 * time.Millisecond,
        MaxDelay:       30 * time.Second,
        ExponentialBase: 2,
    },
})
```

### Retry Behavior

1. **Exponential backoff:** Each retry waits longer: `delay = min(initial_delay * (base ^ attempt), max_delay)`
2. **Jitter:** Add random jitter (±10%) to prevent thundering herd
3. **Idempotency:** Only retry idempotent operations automatically (GET, PUT, DELETE)
4. **POST handling:** POST requests only retried if explicitly marked idempotent or if no response received

---

## Rate Limiting

SDKs should handle rate limits (429 responses) gracefully.

### Rate Limit Handling

```python
from muxi.exceptions import RateLimitError

try:
    response = client.chat("Hello")
except RateLimitError as e:
    print(f"Rate limited. Retry after: {e.retry_after} seconds")
    print(f"Limit: {e.limit}, Remaining: {e.remaining}")
```

### Automatic Rate Limit Handling

When `retry` is enabled, SDKs automatically:

1. Detect 429 responses
2. Read `Retry-After` header (or `X-RateLimit-Reset`)
3. Wait the specified duration
4. Retry the request

```python
# Rate limits handled automatically with retry enabled
client = FormationClient(
    formation_id="my-bot",
    url="http://localhost:8001",
    retry=RetryConfig(max_retries=3)  # Will auto-handle 429s
)

# This will automatically wait and retry on rate limits
response = client.chat("Hello")
```

### Rate Limit Headers

SDKs should expose rate limit info when available:

```python
response = client.chat("Hello")
print(f"Rate limit: {response.rate_limit.limit}")
print(f"Remaining: {response.rate_limit.remaining}")
print(f"Resets at: {response.rate_limit.reset}")
```

---

## Logging and Debug Mode

SDKs should support debug logging for troubleshooting.

### Enabling Debug Mode

```python
# Python - via constructor
client = FormationClient(
    formation_id="my-bot",
    url="http://localhost:8001",
    debug=True  # Logs all requests/responses
)

# Python - via environment variable
# Set MUXI_DEBUG=1 or MUXI_LOG_LEVEL=debug

# Python - via logging configuration
import logging
logging.getLogger("muxi").setLevel(logging.DEBUG)
```

```typescript
// TypeScript
const client = new FormationClient({
  formationId: 'my-bot',
  url: 'http://localhost:8001',
  debug: true,
});

// Or via environment variable
// Set MUXI_DEBUG=1
```

```go
// Go
client := muxi.NewFormationClient(&muxi.FormationConfig{
    FormationID: "my-bot",
    URL:         "http://localhost:8001",
    Debug:       true,
})

// Or use a custom logger
client := muxi.NewFormationClient(&muxi.FormationConfig{
    FormationID: "my-bot",
    URL:         "http://localhost:8001",
    Logger:      myCustomLogger,
})
```

### Debug Output

When debug is enabled, SDKs log:

```
[MUXI] POST https://muxi.company.com:7890/api/my-bot/chat
[MUXI] Headers: Authorization=HMAC ..., Content-Type=application/json
[MUXI] Body: {"message": "Hello"}
[MUXI] Response: 200 OK (234ms)
[MUXI] Response Body: {"message": "Hi there!", "agent": "default"}
```

### Log Levels

| Level | What's Logged |
|-------|---------------|
| `error` | Errors only |
| `warn` | Errors + warnings (rate limits, retries) |
| `info` | + Request summaries (method, URL, status, duration) |
| `debug` | + Full request/response headers and bodies |

---

## Key Use Cases Enabled by SDKs

### 1. CI/CD Deployment

**Automated deployment from GitHub Actions, Jenkins, GitLab CI:**

```python
# deploy.py
import os
from muxi import ServerClient

server = ServerClient(
    url=os.getenv("MUXI_SERVER_URL"),
    key_id=os.getenv("MUXI_KEY_ID"),
    secret_key=os.getenv("MUXI_SECRET_KEY")
)

# Deploy from CI/CD artifact
result = server.deploy_formation(
    bundle_path="dist/my-bot-v1.2.3.tar.gz",
    metadata={
        "id": "my-bot",
        "version": os.getenv("GIT_TAG"),
        "commit": os.getenv("GIT_COMMIT")
    }
)

print(f"✓ Deployed {result.formation_id} v{result.version}")

# Health check after deployment
import asyncio
from muxi import AsyncFormationClient

async def verify_deployment():
    client = AsyncFormationClient(
        formation_id="my-bot",
        server_url=os.getenv("MUXI_SERVER_URL"),
        key_id=os.getenv("MUXI_KEY_ID"),
        secret_key=os.getenv("MUXI_SECRET_KEY")
    )
    
    # Wait for formation to be ready
    for _ in range(30):
        try:
            health = await client.health()
            if health.status == "healthy":
                print("✓ Health check passed")
                return
        except Exception:
            pass
        await asyncio.sleep(1)
    
    raise Exception("Formation did not become healthy")

asyncio.run(verify_deployment())
```

**GitHub Actions workflow:**

```yaml
# .github/workflows/deploy.yml
name: Deploy to MUXI

on:
  push:
    tags:
      - 'v*'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build formation bundle
        run: |
          tar -czf my-bot.tar.gz formation.yaml agents/ secrets.enc
      
      - name: Deploy to MUXI
        env:
          MUXI_SERVER_URL: ${{ secrets.MUXI_SERVER_URL }}
          MUXI_KEY_ID: ${{ secrets.MUXI_KEY_ID }}
          MUXI_SECRET_KEY: ${{ secrets.MUXI_SECRET_KEY }}
          GIT_TAG: ${{ github.ref_name }}
          GIT_COMMIT: ${{ github.sha }}
        run: |
          pip install muxi
          python deploy.py
```

### 2. Integration Testing

**Test formations in CI/CD:**

```python
# tests/test_formation.py
import pytest
from muxi import ServerClient, FormationClient

@pytest.fixture
async def deployed_formation(test_server_url, test_credentials):
    """Deploy a test formation, yield it, then clean up."""
    server = ServerClient(
        url=test_server_url,
        key_id=test_credentials.key_id,
        secret_key=test_credentials.secret_key
    )
    
    # Deploy test formation
    result = server.deploy_formation(
        bundle_path="tests/fixtures/test-bot.tar.gz",
        metadata={"id": "test-bot-ci", "name": "CI Test Bot"}
    )
    
    # Create client for the formation
    client = FormationClient(
        formation_id=result.formation_id,
        server_url=test_server_url,
        key_id=test_credentials.key_id,
        secret_key=test_credentials.secret_key
    )
    
    yield client
    
    # Cleanup
    server.delete_formation(result.formation_id)


async def test_chat_response(deployed_formation):
    """Test that formation responds to chat."""
    response = await deployed_formation.chat("Hello!")
    
    assert response.message is not None
    assert len(response.message) > 0


async def test_streaming_response(deployed_formation):
    """Test streaming chat response."""
    chunks = []
    async for chunk in deployed_formation.chat_stream("Count to 5"):
        if chunk.type == "text":
            chunks.append(chunk.text)
    
    full_response = "".join(chunks)
    assert "1" in full_response
    assert "5" in full_response


async def test_health_check(deployed_formation):
    """Test formation health endpoint."""
    health = await deployed_formation.health()
    
    assert health.status == "healthy"
    assert health.agents is not None
```

### 3. Monitoring Dashboard

**Build custom monitoring:**

```python
# monitoring.py
import asyncio
from datetime import datetime
from muxi import AsyncServerClient, AsyncFormationClient

async def monitor_formations(server_url: str, key_id: str, secret_key: str):
    server = AsyncServerClient(
        url=server_url,
        key_id=key_id,
        secret_key=secret_key
    )
    
    while True:
        formations = await server.list_formations()
        
        print(f"\n{'='*60}")
        print(f"MUXI Monitor - {datetime.now().isoformat()}")
        print(f"{'='*60}")
        
        for f in formations:
            client = AsyncFormationClient(
                formation_id=f.id,
                server_url=server_url,
                key_id=key_id,
                secret_key=secret_key
            )
            
            try:
                health = await client.health()
                status_icon = "✓" if health.status == "healthy" else "✗"
                print(f"{status_icon} {f.id}: {health.status} (v{f.version})")
            except Exception as e:
                print(f"✗ {f.id}: ERROR - {e}")
        
        await asyncio.sleep(30)  # Check every 30 seconds

if __name__ == "__main__":
    import os
    asyncio.run(monitor_formations(
        server_url=os.getenv("MUXI_SERVER_URL"),
        key_id=os.getenv("MUXI_KEY_ID"),
        secret_key=os.getenv("MUXI_SECRET_KEY")
    ))
```

### 4. Application Integration

**Use formations as AI backends:**

```python
# app.py - FastAPI application using MUXI formation
from fastapi import FastAPI, HTTPException
from fastapi.responses import StreamingResponse
from muxi import AsyncFormationClient
import os

app = FastAPI()

client = AsyncFormationClient(
    formation_id="customer-support-bot",
    server_url=os.getenv("MUXI_SERVER_URL"),
    key_id=os.getenv("MUXI_KEY_ID"),
    secret_key=os.getenv("MUXI_SECRET_KEY")
)

@app.post("/chat")
async def chat(message: str):
    """Non-streaming chat endpoint."""
    try:
        response = await client.chat(message)
        return {"reply": response.message}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.post("/chat/stream")
async def chat_stream(message: str):
    """Streaming chat endpoint."""
    async def generate():
        async for chunk in client.chat_stream(message):
            if chunk.type == "text":
                yield f"data: {chunk.text}\n\n"
            elif chunk.type == "done":
                yield "data: [DONE]\n\n"
    
    return StreamingResponse(generate(), media_type="text/event-stream")

@app.get("/health")
async def health():
    """Proxy health check to formation."""
    try:
        health = await client.health()
        return {"status": health.status}
    except Exception:
        return {"status": "unhealthy"}
```

### 5. Batch Operations

**Manage multiple formations:**

```python
# batch_update.py
import asyncio
from muxi import AsyncServerClient

async def rolling_update(
    server_url: str,
    key_id: str, 
    secret_key: str,
    bundle_path: str,
    formation_ids: list[str]
):
    """Rolling update across multiple formations."""
    server = AsyncServerClient(
        url=server_url,
        key_id=key_id,
        secret_key=secret_key
    )
    
    for formation_id in formation_ids:
        print(f"Updating {formation_id}...")
        
        try:
            # Update formation
            result = await server.update_formation(formation_id, bundle_path)
            print(f"  ✓ Updated to v{result.version}")
            
            # Wait for healthy
            await wait_for_healthy(server, formation_id)
            print(f"  ✓ Healthy")
            
        except Exception as e:
            print(f"  ✗ Failed: {e}")
            print("  Rolling back...")
            await server.rollback_formation(formation_id)
            raise
        
        # Brief pause between updates
        await asyncio.sleep(5)
    
    print(f"\n✓ All {len(formation_ids)} formations updated")
```

---

## SDK Package Structure

### Python SDK

```
muxi-python/
├── src/
│   └── muxi/
│       ├── __init__.py           # Exports ServerClient, FormationClient, Async variants
│       ├── server_client.py      # ServerClient implementation
│       ├── formation_client.py   # FormationClient implementation
│       ├── async_client.py       # Async implementations
│       ├── streaming.py          # Streaming response handling
│       ├── auth.py               # HMAC signing
│       ├── models.py             # Pydantic models
│       ├── exceptions.py         # Custom exceptions
│       ├── retry.py              # Retry/backoff logic
│       └── _utils.py             # Internal helpers
│
├── tests/
│   ├── test_server_client.py
│   ├── test_formation_client.py
│   ├── test_streaming.py
│   ├── test_auth.py
│   └── test_retry.py
│
├── examples/
│   ├── deploy.py
│   ├── streaming.py
│   ├── testing.py
│   └── monitoring.py
│
├── pyproject.toml
└── README.md
```

### TypeScript SDK

```
muxi-node/
├── src/
│   ├── index.ts              # Exports ServerClient, FormationClient
│   ├── server-client.ts      # ServerClient implementation
│   ├── formation-client.ts   # FormationClient implementation
│   ├── streaming.ts          # Streaming response handling
│   ├── auth.ts               # HMAC signing
│   ├── types.ts              # Type definitions
│   ├── errors.ts             # Custom errors
│   ├── retry.ts              # Retry/backoff logic
│   └── utils.ts              # Helpers
│
├── tests/
│   ├── server-client.test.ts
│   ├── formation-client.test.ts
│   ├── streaming.test.ts
│   └── auth.test.ts
│
├── examples/
│   ├── deploy.ts
│   ├── streaming.ts
│   ├── testing.ts
│   └── monitoring.ts
│
├── package.json
└── README.md
```

### Go SDK

```
muxi-go/
├── client.go              # Common client logic
├── server_client.go       # ServerClient implementation
├── formation_client.go    # FormationClient implementation
├── streaming.go           # Streaming response handling
├── auth.go                # HMAC signing
├── models.go              # Type definitions
├── errors.go              # Custom errors
├── retry.go               # Retry/backoff logic
├── options.go             # Functional options
│
├── examples/
│   ├── deploy/
│   │   └── main.go
│   ├── streaming/
│   │   └── main.go
│   └── monitoring/
│       └── main.go
│
├── server_client_test.go
├── formation_client_test.go
├── streaming_test.go
├── auth_test.go
│
├── go.mod
├── go.sum
└── README.md
```

---

## API Surface

### ServerClient Methods

```typescript
class ServerClient {
  constructor(config: ServerConfig)
  
  // Formation lifecycle
  deployFormation(options: DeployOptions): Promise<DeployResult>
  listFormations(): Promise<Formation[]>
  getFormation(id: string): Promise<FormationDetails>
  updateFormation(id: string, bundlePath: string): Promise<UpdateResult>
  stopFormation(id: string): Promise<void>
  restartFormation(id: string): Promise<void>
  rollbackFormation(id: string): Promise<RollbackResult>
  deleteFormation(id: string): Promise<void>
  
  // Formation logs
  getFormationLogs(id: string, options?: LogOptions): Promise<string[]>
  
  // Server management
  status(): Promise<ServerStatus>
  logs(options?: LogOptions): Promise<AuditLog[]>
}
```

### FormationClient Methods

```typescript
class FormationClient {
  constructor(config: FormationConfig)
  
  // Core formation operations
  chat(message: string, options?: ChatOptions): Promise<ChatResponse>
  chatStream(message: string, options?: ChatOptions): AsyncIterable<ChatChunk>
  health(): Promise<HealthStatus>
  logs(options?: LogOptions): Promise<string[]>
  
  // Agent operations
  listAgents(): Promise<Agent[]>
  getAgent(name: string): Promise<AgentDetails>
  addAgent(config: AgentConfig): Promise<void>
  removeAgent(name: string): Promise<void>
  updateAgent(name: string, config: AgentConfig): Promise<void>
  
  // Secret operations
  listSecrets(): Promise<string[]>
  getSecret(key: string): Promise<string>
  setSecret(key: string, value: string): Promise<void>
  deleteSecret(key: string): Promise<void>
  
  // MCP operations
  listMCPs(): Promise<MCP[]>
  getMCP(name: string): Promise<MCPDetails>
  addMCP(config: MCPConfig): Promise<void>
  removeMCP(name: string): Promise<void>
}
```

---

## Error Handling

SDKs should use **structured exceptions/errors**, not user-friendly messages.

### Error Hierarchy

```
MuxiError (base)
├── AuthenticationError     # 401 - Invalid/missing credentials
├── AuthorizationError      # 403 - Valid creds, insufficient permissions
├── NotFoundError           # 404 - Resource not found
├── ConflictError           # 409 - Resource conflict (e.g., already exists)
├── ValidationError         # 422 - Invalid request parameters
├── RateLimitError          # 429 - Rate limit exceeded
│   ├── retry_after: float  # Seconds to wait
│   ├── limit: int          # Request limit
│   └── remaining: int      # Remaining requests
├── ServerError             # 5xx - Server-side error
└── ConnectionError         # Network/connection issues
```

### Python

```python
from muxi.exceptions import (
    MuxiError,              # Base exception
    AuthenticationError,    # 401
    AuthorizationError,     # 403
    NotFoundError,          # 404
    ConflictError,          # 409
    ValidationError,        # 422
    RateLimitError,         # 429
    ServerError,            # 5xx
    ConnectionError,        # Network issues
)

try:
    server.deploy_formation(...)
except AuthenticationError as e:
    print(f"Auth failed: {e.message}")
    print(f"Status: {e.status_code}")
except RateLimitError as e:
    print(f"Rate limited. Retry after: {e.retry_after}s")
except NotFoundError as e:
    print(f"Formation not found: {e.details}")
except ServerError as e:
    print(f"Server error: {e.message}")
except MuxiError as e:
    print(f"MUXI error: {e}")
```

### TypeScript

```typescript
import {
  MuxiError,
  AuthenticationError,
  AuthorizationError,
  NotFoundError,
  ConflictError,
  ValidationError,
  RateLimitError,
  ServerError,
} from 'muxi';

try {
  await server.deployFormation(...);
} catch (err) {
  if (err instanceof RateLimitError) {
    console.error(`Rate limited. Retry after: ${err.retryAfter}s`);
  } else if (err instanceof AuthenticationError) {
    console.error('Auth failed:', err.message);
  } else if (err instanceof NotFoundError) {
    console.error('Formation not found:', err.details);
  } else if (err instanceof ServerError) {
    console.error('Server error:', err.message);
  } else if (err instanceof MuxiError) {
    console.error('MUXI error:', err.message);
  }
}
```

### Go

```go
import "github.com/muxi-ai/muxi-go"

result, err := server.DeployFormation(ctx, req)
if err != nil {
    switch e := err.(type) {
    case *muxi.AuthenticationError:
        log.Printf("Auth failed: %s", e.Message)
    case *muxi.RateLimitError:
        log.Printf("Rate limited. Retry after: %v", e.RetryAfter)
    case *muxi.NotFoundError:
        log.Printf("Not found: %s", e.Details)
    case *muxi.ServerError:
        log.Printf("Server error: %s", e.Message)
    default:
        log.Printf("Error: %v", err)
    }
}
```

---

## Authentication

SDKs accept credentials **explicitly** — no profile management or auto-detection.

### Environment Variables (Recommended)

```python
import os
from muxi import ServerClient

server = ServerClient(
    url=os.getenv("MUXI_SERVER_URL"),
    key_id=os.getenv("MUXI_KEY_ID"),
    secret_key=os.getenv("MUXI_SECRET_KEY")
)
```

### Direct Config

```python
server = ServerClient(
    url="https://muxi.company.com:7890",
    key_id="MUXI_PROD_KEY",
    secret_key="sk_..."
)
```

### Application Config File

```python
import json

with open("config.json") as f:
    config = json.load(f)

server = ServerClient(**config["muxi_server"])
```

**No ~/.muxi/cli/profiles.yaml support in SDKs!** That's CLI-specific.

---

## Testing Strategy

### Unit Tests

Test HMAC signing, request construction, response parsing, retry logic:

```python
def test_hmac_signature():
    """Test HMAC signature generation."""
    from muxi.auth import generate_signature
    
    sig = generate_signature(
        key_id="test",
        secret="secret",
        method="POST",
        path="/rpc/formations",
        timestamp="2025-10-24T12:00:00Z",
        body='{"id":"test"}'
    )
    
    assert sig is not None
    assert len(sig) > 0


def test_exponential_backoff():
    """Test retry delay calculation."""
    from muxi.retry import calculate_delay
    
    config = RetryConfig(initial_delay=0.5, exponential_base=2, max_delay=30)
    
    assert calculate_delay(config, attempt=0) == 0.5
    assert calculate_delay(config, attempt=1) == 1.0
    assert calculate_delay(config, attempt=2) == 2.0
    assert calculate_delay(config, attempt=10) == 30.0  # Capped at max
```

### Integration Tests

Test against a real MUXI Server (requires server running):

```python
@pytest.mark.integration
def test_deploy_formation(test_server):
    """Test deploying a formation."""
    server = ServerClient(url=test_server.url, ...)
    
    result = server.deploy_formation(
        bundle_path="tests/fixtures/test-bot.tar.gz",
        metadata={"id": "test-bot"}
    )
    
    assert result.formation_id == "test-bot"
    assert result.status == "running"
    
    # Cleanup
    server.delete_formation("test-bot")


@pytest.mark.integration
async def test_streaming_response(test_formation):
    """Test streaming chat response."""
    chunks = []
    async for chunk in test_formation.chat_stream("Say hello"):
        chunks.append(chunk)
    
    assert len(chunks) > 0
    assert any(c.type == "text" for c in chunks)
    assert chunks[-1].type == "done"
```

---

## Documentation Requirements

Each SDK should include:

1. **README.md** - Installation, quick start, examples
2. **API Documentation** - Auto-generated from docstrings/comments
3. **Examples/** - Real-world use cases (CI/CD, testing, monitoring, streaming)
4. **Migration Guide** - If breaking changes occur
5. **Contributing Guide** - How to contribute to the SDK

---

## Implementation Checklist

For each SDK:

- [ ] ServerClient implementation
  - [ ] HMAC authentication
  - [ ] Formation lifecycle methods
  - [ ] Server management methods
  - [ ] Error handling
- [ ] FormationClient implementation
  - [ ] Chat endpoint (non-streaming)
  - [ ] Chat streaming
  - [ ] Health endpoint
  - [ ] Agent operations
  - [ ] Secret operations
  - [ ] MCP operations
- [ ] Async variants (where applicable)
- [ ] Retry/backoff logic
  - [ ] Configurable retry settings
  - [ ] Exponential backoff with jitter
  - [ ] Rate limit handling (429)
- [ ] Logging/debug mode
- [ ] Type definitions (TypeScript) or type hints (Python)
- [ ] Exception/error classes
- [ ] Unit tests (auth, retry, request construction)
- [ ] Integration tests (against real server)
- [ ] Documentation
  - [ ] README with examples
  - [ ] API docs (auto-generated)
  - [ ] Examples directory (including streaming)
- [ ] CI/CD
  - [ ] Automated testing
  - [ ] Publishing to package registry
- [ ] Version management (semantic versioning)

---

## SDK Repositories

| Language | Repository | Package Registry |
|----------|------------|------------------|
| Python | [github.com/muxi-ai/muxi-python](https://github.com/muxi-ai/muxi-python) | PyPI: `muxi` |
| TypeScript | [github.com/muxi-ai/muxi-node](https://github.com/muxi-ai/muxi-node) | npm: `muxi` |
| Go | [github.com/muxi-ai/muxi-go](https://github.com/muxi-ai/muxi-go) | Go modules |
| PHP | [github.com/muxi-ai/muxi-php](https://github.com/muxi-ai/muxi-php) | Packagist: `muxi/muxi-php` |
| Ruby | [github.com/muxi-ai/muxi-ruby](https://github.com/muxi-ai/muxi-ruby) | RubyGems: `muxi` |
| C# | [github.com/muxi-ai/muxi-csharp](https://github.com/muxi-ai/muxi-csharp) | NuGet: `Muxi` |
| Swift | [github.com/muxi-ai/muxi-swift](https://github.com/muxi-ai/muxi-swift) | Swift Package Manager |
| Kotlin | [github.com/muxi-ai/muxi-kotlin](https://github.com/muxi-ai/muxi-kotlin) | Maven Central |
| Dart | [github.com/muxi-ai/muxi-dart](https://github.com/muxi-ai/muxi-dart) | pub.dev: `muxi` |
| Java | [github.com/muxi-ai/muxi-java](https://github.com/muxi-ai/muxi-java) | Maven Central |
| Rust | [github.com/muxi-ai/muxi-rust](https://github.com/muxi-ai/muxi-rust) | crates.io: `muxi` |
| C/C++ | [github.com/muxi-ai/muxi-cpp](https://github.com/muxi-ai/muxi-cpp) | vcpkg/Conan: `libmuxi` |

---

## Related Documents

- **CLI Design:** [../cli/docs/CLI-COMMAND-DESIGN.md](../../cli/docs/CLI-COMMAND-DESIGN.md)
- **Server API:** [../server/docs/api-reference.md](../../server/docs/api-reference.md)
- **Formation API:** [../runtime/docs/formation-api-v1-final.yaml](../../runtime/docs/formation-api-v1-final.yaml)
- **Architecture:** [../MUXI-ARCHITECTURE.md](../../MUXI-ARCHITECTURE.md)
- **SDK Launch Plan:** [./SDK-LAUNCH-PLAN.md](./SDK-LAUNCH-PLAN.md)

---

## Summary

**SDK Design Principles:**

1. **Two clients:** ServerClient (management) + FormationClient (usage)
2. **Explicit everything:** No auto-detection, no profiles, no wizards
3. **Streaming first:** All chat operations support streaming
4. **Async support:** Async variants for languages that support it
5. **Resilient:** Built-in retry/backoff and rate limit handling
6. **Observable:** Debug logging for troubleshooting
7. **Structured errors:** Use exceptions/errors, not friendly messages
8. **Environment-driven:** Config via env vars or app config
9. **Composable:** SDKs are building blocks for automation
10. **Language-idiomatic:** Follow each language's best practices

**Key Use Cases:**
- CI/CD deployment automation
- Integration testing (deploy fixture, test, cleanup)
- Custom monitoring dashboards
- Applications using formations as backends
- DevOps automation (auto-restart, batch updates)

**What SDKs Are NOT:**
- Not interactive tools
- Not context-aware (no directory detection)
- Not CLI replacements (different audience)

---

**Version:** 1.1  
**Last Updated:** 2025-12-09  
**Status:** Design Document  
**Next Steps:** Implement Python and TypeScript SDKs first (Phase 1), then expand per launch plan
