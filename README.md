# MUXI SDKs

Build intelligent agents that actually work. These official SDKs give you everything you need to deploy, manage, and interact with MUXI formations across any language.

**What you get out of the box:**
- HMAC & key-based auth (zero config headaches)
- Automatic retries with exponential backoff
- Idempotency headers on every request
- First-class streaming support (SSE)
- Typed errors and response envelopes

> Looking for the CLI? See [muxi-ai/cli](https://github.com/muxi-ai/cli).


> [!IMPORTANT]
> ## MUXI Ecosystem
>
> This repository is part of the larger MUXI ecosystem.
>
> **📋 Complete architectural overview:** See [muxi/ARCHITECTURE.md](https://github.com/muxi-ai/muxi/blob/main/ARCHITECTURE.md) - explains how all 9 repositories fit together, dependencies, status, and roadmap.

---

## Available SDKs

| Language | Package | Status |
|----------|---------|--------|
| **Go** | [`github.com/muxi-ai/muxi-go`](https://github.com/muxi-ai/muxi-go) | ✅ Stable |
| **Python** | [`pip install muxi`](https://pypi.org/project/muxi/) | ✅ Stable |
| **TypeScript** | [`npm install @muxi-ai/muxi-typescript`](https://www.npmjs.com/package/@muxi-ai/muxi-typescript) | ✅ Stable |

---

## Quick Examples

### Go
```go
client := muxi.NewFormationClient(&muxi.FormationConfig{
    FormationID: "my-agent",
    ServerURL:   os.Getenv("MUXI_SERVER_URL"),
    ClientKey:   os.Getenv("MUXI_CLIENT_KEY"),
})

resp, _ := client.Chat(ctx, &muxi.ChatRequest{Message: "Hello!", UserID: "u1"})
fmt.Println(resp.Response)
```

### Python
```python
from muxi import FormationClient

client = FormationClient(
    server_url="https://server.example.com",
    formation_id="my-agent",
    client_key="<your-key>",
)

for chunk in client.chat_stream({"message": "Tell me a story"}):
    print(chunk.get("text", ""), end="")
```

### TypeScript
```typescript
import { FormationClient } from "@muxi-ai/muxi-typescript";

const client = new FormationClient({
  serverUrl: "https://server.example.com",
  formationId: "my-agent",
  clientKey: "<your-key>",
});

for await (const chunk of await client.chatStream({ message: "Hello!" })) {
  if (chunk.type === "text") process.stdout.write(chunk.text);
}
```

---

## More Languages

Planning SDKs for Ruby, Java, C#, PHP, Swift, Kotlin, Dart, Rust, and C++. Want to contribute or request a language? [Open an issue](https://github.com/muxi-ai/sdks/issues).

---

## Documentation

For full guides, examples, and API reference, visit [muxi.org/docs](https://muxi.org/docs).

Each SDK repo also includes a `README.md` with quickstart and a `USER_GUIDE.md` with detailed patterns.

---

## License

Apache 2.0 — See [LICENSE](LICENSE) for details.
