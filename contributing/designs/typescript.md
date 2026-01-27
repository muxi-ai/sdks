# TypeScript SDK Design

Architecture decisions for `github.com/muxi-ai/muxi-typescript`.

## Transport Layer

**Zero runtime dependencies.** The SDK uses the Web Fetch API (`globalThis.fetch`) which is available in Node 18+, Deno, Bun, browsers, and edge runtimes (Vercel Edge, Cloudflare Workers).

`FormationClient` has an internal `FormationTransport` class; `ServerClient` uses a separate `Transport` class (in `transport.ts`) for HMAC auth.

Both transports handle:
- JSON serialization/deserialization
- Header injection (auth, idempotency, SDK info)
- Retry with exponential backoff
- Timeout via `AbortController`

## ESM-Only

The SDK is ESM-only (`"type": "module"` in package.json). All internal imports use `.js` extensions per Node ESM resolution rules:

```typescript
import { unwrapEnvelope } from "./envelope.js";
```

**Target**: ES2021, **Module**: NodeNext. This ensures compatibility with modern Node.js while keeping the output readable.

## SSE Streaming

Streaming uses `ReadableStream` from the Fetch response body:

```typescript
async *streamSse(method, path, opts): AsyncGenerator<any, void, void> {
    const resp = await fetch(url, { method, headers, body });
    const reader = resp.body!.getReader();
    const decoder = new TextDecoder();
    // Manual line-by-line SSE parsing
    // Yields parsed JSON from "data:" lines
}
```

No external SSE library. The streaming transport creates a separate fetch call with no timeout (no `AbortController`), allowing indefinite streaming.

Consumers use `for await...of`:

```typescript
for await (const chunk of client.chatStream({ message: "hi" }, userId)) {
    if (chunk.type === "text") process.stdout.write(chunk.text);
}
```

## Auth

- **FormationClient**: Key headers (`X-MUXI-CLIENT-KEY` or `X-MUXI-ADMIN-KEY`), plus optional `X-Muxi-User-ID`.
- **ServerClient**: HMAC-SHA256 signature via `buildAuthHeader()` in `auth.ts`. Uses Web Crypto API (`crypto.subtle`) for signing, with Node `crypto` fallback.

## Error Handling

`mapError()` in `errors.ts` maps HTTP status codes to typed error classes:
- `AuthenticationError`, `AuthorizationError`, `NotFoundError`, `ValidationError`, `RateLimitError`, `ServerError`, `ConnectionError`

All extend `MuxiError` with `code`, `message`, `statusCode`, and optional `retryAfter`.

## Retry Logic

Retries on 429 / 500 / 502 / 503 / 504 and connection errors. Exponential backoff starting at 500ms, doubling each attempt, capped at 30s. Controlled by `maxRetries` option.

## Envelope Unwrapping

`unwrapEnvelope()` in `envelope.ts` extracts `data` from the API's `{success, data, request, timestamp}` wrapper, injecting `request_id` and `timestamp` into the returned object.

## Idempotency

Every request gets `X-Muxi-Idempotency-Key: <uuid>` via `generateUUID()` in `platform.ts`. Uses `crypto.randomUUID()` where available, falls back to manual UUID v4.

## Platform Detection

`platform.ts` provides `getClientInfo()` which returns a string like `node/20.11.0/linux-x64` or `browser/chrome`. Used in the `X-Muxi-Client` header.

## Packaging

- **Build**: TypeScript → JavaScript via `tsc`
- **Output**: `dist/` with `.js` + `.d.ts` files
- **Zero runtime dependencies** (only devDependencies: typescript, ts-node, @types/node)
- **Node**: 18+
- **Registry**: npm (`@muxi-ai/muxi-typescript`), published via OIDC trusted publishing
- **Version**: calver `YYYY.MMDD.patch` in `version.ts`
- **Provenance**: npm provenance attestations enabled (`publishConfig.provenance: true`)

## Browser Compatibility

The SDK works directly in browsers since it uses standard Fetch API. The `clientKey` is safe to expose in browser code (limited permissions). The `adminKey` should only be used server-side.

## File Layout

```
src/
├── index.ts        # Public exports
├── auth.ts         # HMAC signing for ServerClient
├── envelope.ts     # Response envelope unwrapping
├── errors.ts       # Typed error hierarchy + mapError()
├── formation.ts    # FormationClient + FormationTransport
├── platform.ts     # UUID generation, client info detection
├── server.ts       # ServerClient + Transport
├── transport.ts    # HMAC transport for ServerClient
├── version.ts      # Calver version
└── webhook.ts      # verifySignature() + parse()
```
