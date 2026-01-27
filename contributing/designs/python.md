# Python SDK Design

Architecture decisions for `github.com/muxi-ai/muxi-python`.

## Transport Layer

**httpx** is the sole HTTP dependency. It provides:
- Pooled sync (`httpx.Client`) and async (`httpx.AsyncClient`) transports in one library
- HTTP/2 support out of the box
- Streaming response iteration (`iter_lines` / `aiter_lines`) for SSE

Both `FormationClient` and `AsyncFormationClient` share a single `_FormationTransport` class that exposes sync and async methods. The transport owns both an `httpx.Client` and an `httpx.AsyncClient`, created lazily.

The `ServerClient` uses a separate `Transport` class (in `transport.py`) because it uses HMAC auth instead of key headers and has different endpoint patterns (`/rpc` prefix).

## Sync / Async Duality

Every formation endpoint exists twice:

```
FormationClient.chat(...)           → sync, returns dict
AsyncFormationClient.chat(...)      → async, returns dict

FormationClient.chat_stream(...)    → sync, returns Generator
AsyncFormationClient.chat_stream()  → async, returns AsyncGenerator
```

Both classes delegate to the same `_FormationTransport`. Sync methods call `request_json` / `stream_sse`; async methods call `arequest_json` / `astream_sse`.

The `ServerClient` / `AsyncServerClient` follow the same pattern via the shared `Transport`.

## SSE Streaming

SSE parsing is done manually (no external library):
- `_parse_sse_lines(lines: Iterable[str])` — sync generator
- `_parse_sse_lines_async(lines: AsyncIterable[str])` — async generator

Both yield parsed JSON dicts from `data:` lines. The transport sets `timeout=None` for streaming requests to prevent httpx from closing long-lived connections.

## Auth

- **FormationClient**: Key headers (`X-MUXI-CLIENT-KEY` or `X-MUXI-ADMIN-KEY`), plus optional `X-Muxi-User-ID`.
- **ServerClient**: HMAC-SHA256 signature via `build_auth_header()` in `auth.py`. Signs `method + path + timestamp + body_hash`.

## Error Handling

All non-2xx responses are mapped through `map_error()` in `errors.py`, producing typed exceptions:
- `AuthenticationError`, `AuthorizationError`, `NotFoundError`, `ValidationError`, `RateLimitError`, `ServerError`, `ConnectionError`

Each carries `code`, `message`, `status_code`, and optional `retry_after`.

## Retry Logic

Retries on 429 / 5xx / connection errors with exponential backoff (starting 500ms, capped at 30s). Respects `Retry-After` header for rate limits. Controlled by `max_retries` config.

## Envelope Unwrapping

The API wraps responses in `{success, data, request, timestamp}`. The `_unwrap_envelope()` helper extracts `data` and injects `request_id` / `timestamp` into the returned dict so callers get flat, clean responses.

## Idempotency

Every request gets `X-Muxi-Idempotency-Key: <uuid4>` automatically. No opt-out.

## Packaging

- **Build**: `pyproject.toml` + `setup.py` (legacy compat for `tomli` parsing)
- **Single dependency**: `httpx>=0.24.0`
- **Python**: 3.10+
- **Registry**: PyPI, published via OIDC trusted publishing from GitHub Actions
- **Version**: `muxi/version.py` — calver `YYYY.MMDD.patch`

## File Layout

```
muxi/
├── __init__.py          # Public API: ServerClient, FormationClient, AsyncXxx, webhook
├── auth.py              # HMAC signing for ServerClient
├── errors.py            # Typed error hierarchy + map_error()
├── formation.py         # FormationClient, AsyncFormationClient, SSE helpers
├── server.py            # ServerClient, AsyncServerClient
├── transport.py         # HMAC transport for ServerClient
├── version.py           # Calver version
├── webhook.py           # verify_signature() + parse()
└── client/__init__.py   # (empty, reserved)
```
