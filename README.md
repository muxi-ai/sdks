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
> **📋 Complete architectural overview:** See [muxi/ARCHITECTURE.md](https://github.com/muxi-ai/muxi/blob/main/ARCHITECTURE.md) - explains how all repositories fit together, dependencies, status, and roadmap.

---

## Available SDKs

| Language | Package | Install |
|----------|---------|---------|
| **Go** | [muxi-go](https://github.com/muxi-ai/muxi-go) | `go get github.com/muxi-ai/muxi-go` |
| **Python** | [muxi](https://pypi.org/project/muxi/) | `pip install muxi` |
| **TypeScript** | [@muxi-ai/muxi-typescript](https://www.npmjs.com/package/@muxi-ai/muxi-typescript) | `npm install @muxi-ai/muxi-typescript` |
| **Ruby** | [muxi](https://rubygems.org/gems/muxi) | `gem install muxi` |
| **PHP** | [muxi/muxi-php](https://packagist.org/packages/muxi/muxi-php) | `composer require muxi/muxi-php` |
| **C#** | [Muxi](https://www.nuget.org/packages/Muxi) | `dotnet add package Muxi` |
| **Java** | [org.muxi:muxi-java](https://central.sonatype.com/artifact/org.muxi/muxi-java) | Maven/Gradle (see below) |
| **Kotlin** | [org.muxi:muxi-kotlin](https://central.sonatype.com/artifact/org.muxi/muxi-kotlin) | Maven/Gradle (see below) |
| **Swift** | [muxi-swift](https://github.com/muxi-ai/muxi-swift) | Swift Package Manager |
| **Dart** | [muxi](https://pub.dev/packages/muxi) | `dart pub add muxi` |
| **Rust** | [muxi-rust](https://crates.io/crates/muxi-rust) | `cargo add muxi-rust` |
| **C++** | [muxi-cpp](https://github.com/muxi-ai/muxi-cpp) | CMake (header-only) |

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

### Java
```java
FormationClient client = new FormationClient(
    "https://server.example.com",
    "my-agent",
    "<your-key>"
);

ChatResponse response = client.chat(new ChatRequest("Hello!"));
System.out.println(response.getResponse());
```

```gradle
implementation("org.muxi:muxi-java:0.20260212.0")
```

### Kotlin
```kotlin
val client = FormationClient(
    serverUrl = "https://server.example.com",
    formationId = "my-agent",
    clientKey = "<your-key>"
)

val response = client.chat(ChatRequest(message = "Hello!"))
println(response.response)
```

```kotlin
implementation("org.muxi:muxi-kotlin:0.20260212.0")
```

### Ruby
```ruby
require 'muxi'

client = Muxi::FormationClient.new(
  server_url: 'https://server.example.com',
  formation_id: 'my-agent',
  client_key: '<your-key>'
)

response = client.chat(message: 'Hello!')
puts response['response']
```

### C#
```csharp
var client = new FormationClient(
    serverUrl: "https://server.example.com",
    formationId: "my-agent",
    clientKey: "<your-key>"
);

var response = await client.ChatAsync(new ChatRequest { Message = "Hello!" });
Console.WriteLine(response.Response);
```

### Swift
```swift
let client = FormationClient(
    serverURL: "https://server.example.com",
    formationID: "my-agent",
    clientKey: "<your-key>"
)

let response = try await client.chat(message: "Hello!")
print(response.response)
```

### Dart
```dart
final client = FormationClient(
  serverUrl: 'https://server.example.com',
  formationId: 'my-agent',
  clientKey: '<your-key>',
);

final response = await client.chat(message: 'Hello!');
print(response['response']);
```

### Rust
```rust
let client = FormationClient::new(
    "https://server.example.com",
    "my-agent",
    "<your-key>",
);

let response = client.chat("Hello!").await?;
println!("{}", response.response);
```

### PHP
```php
$client = new FormationClient(
    serverUrl: 'https://server.example.com',
    formationId: 'my-agent',
    clientKey: '<your-key>'
);

$response = $client->chat(['message' => 'Hello!']);
echo $response['response'];
```

### C++
```cpp
auto client = muxi::FormationClient(
    "https://server.example.com",
    "my-agent",
    "<your-key>"
);

auto response = client.chat({{"message", "Hello!"}});
std::cout << response["response"] << std::endl;
```

---

## Documentation

For full guides, examples, and API reference, visit [muxi.org/docs](https://muxi.org/docs).

Each SDK repo also includes a `README.md` with quickstart and a `USER_GUIDE.md` with detailed patterns.

---

## Contributing

See [contributing/conventions.md](contributing/conventions.md) for cross-language API conventions and naming standards.

---

## License

MIT — See [LICENSE](LICENSE) for details.
