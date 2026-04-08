# SDK SSE Follow-up

**Date:** 2026-04-08  
**Audience:** SDK droid / SDK maintainers

## Why this exists

The runtime now keeps long-running chat streams alive by sending SSE comment frames such as:

```text
: keepalive
```

That fixed the HTTP/SSE layer in runtime `v0.20260408.2`. The CLI then needed a follow-up fix because it was still timing out while waiting for parsed chat events, even though keepalive bytes were arriving on the wire. The CLI fix landed in `muxi-ai/cli` commit `7d9b475` (`fix: handle chat SSE keepalives in cli`).

The SDKs likely need the same treatment. Many SDK implementations still describe or implement SSE parsing as “read `data:` lines and yield parsed JSON.” That is now too narrow for chat streams.

## The failure mode

A client can still fail even when the backend is healthy if it:

1. ignores SSE comment frames,
2. treats “no parsed event” as a timeout,
3. assumes every meaningful frame starts with `data:`,
4. drops route-level `event: error` frames, or
5. assumes a very small set of chat chunk types.

This is exactly what happened in the CLI before the follow-up fix.

## What every SDK should do

For chat and audio-chat streaming, each SDK should:

1. **Parse SSE by event blocks**
   - keep the current `event:` value until the blank-line terminator,
   - support multi-line `data:` payload assembly,
   - ignore unknown fields safely.

2. **Treat comment frames as liveness**
   - `: keepalive` must count as activity,
   - do not fail a stream just because only heartbeat comments arrived for a while.

3. **Handle route-level SSE events**
   - `event: done`
   - `event: error` with JSON `{"error": "...", "type": "..."}`

4. **Be tolerant of runtime chat chunk types**
   - at minimum preserve or surface:
     - `content` / `text` / `response`
     - `progress`
     - `thinking`
     - `planning`
     - `tool_call`
     - `error`
     - completion / finished markers

5. **Keep streaming requests on infinite timeouts**
   - no per-request deadline for active streams,
   - if an SDK has its own idle timeout above the transport layer, reset it on heartbeat activity.

6. **Avoid new SSE dependencies unless approved**
   - manual parsers already exist in several SDKs,
   - prefer fixing and unifying current parsers.

## Recommended public behavior

Preferred behavior:

- **Internally consume heartbeat comments** to keep the stream alive.
- **Do not require end users to handle heartbeat frames** unless the SDK already exposes raw SSE frames by design.
- **Surface `event: error` as an exception/error result**, not as a silently dropped chunk.
- **Preserve new runtime chunk types instead of hard-dropping them**. If the SDK has typed chunk models, add a generic/fallback representation rather than assuming only text chunks matter.

## Priority order

### Phase 1 — Highest priority

These SDKs already have documented or obvious chat SSE implementations:

- **Python**
- **TypeScript**
- **Go**

### Phase 2 — Same parser sweep

These SDKs also expose `chatStream` / `chat_stream` or generic `streamSse` helpers and should receive the same audit once Phase 1 is stable:

- C#
- Dart
- Java
- Kotlin
- PHP
- Ruby
- Rust
- Swift
- C++

## Concrete starting points

### Python

Current SSE hooks:

- `python/muxi/formation.py`
  - `_parse_sse_lines`
  - `_parse_sse_lines_async`
  - `stream_sse`
  - `astream_sse`
  - `chat_stream`
  - `audio_chat_stream`
- tests:
  - `python/tests/test_sse.py`

Likely work:

- support SSE comments and event blocks,
- preserve `event: error`,
- add long-idle heartbeat tests,
- make sure sync and async paths behave the same.

### TypeScript

Current SSE hooks:

- `typescript/src/formation.ts`
  - `streamSse`
  - `chatStream`
  - `audioChatStream`
- tests:
  - `typescript/tests/unit/formation_sse.test.ts`
  - `typescript/tests/e2e/formation.test.ts`

Likely work:

- update the line parser into an event-block parser,
- treat comment frames as liveness,
- preserve `tool_call` and other non-text chunk types,
- surface `event: error` correctly.

### Go

Current SSE hooks:

- `go/src/formation_client.go`
  - `ChatStream`
  - `AudioChatStream`
- example:
  - `go/src/examples/chat_stream/main.go`
- design notes:
  - `contributing/designs/go.md`

Likely work:

- locate or add a shared streaming helper in `go/src`,
- make channel-based chat streaming heartbeat-aware,
- ensure callers do not see false idle failures while the stream is alive,
- add parser tests for comment frames and route-level errors.

### Other SDKs with streaming entry points

Known streaming/chat files found in this repo:

- `dart/lib/src/formation_client.dart`
- `swift/Sources/Muxi/FormationClient.swift`
- `java/src/main/java/org/muxi/sdk/FormationClient.java`
- `kotlin/src/main/kotlin/org/muxi/sdk/FormationClient.kt`
- `php/src/FormationClient.php`
- `ruby/lib/muxi/formation_client.rb`
- `rust/src/formation_client.rs`
- `csharp/src/Muxi/FormationClient.cs`
- `cpp/src/formation_client.cpp`

These should be audited after the first three SDKs establish the expected behavior.

## Acceptance criteria

An SDK is done when all of the following are true:

1. A chat stream stays open for several minutes if only heartbeat comments arrive.
2. The SDK does not throw or terminate just because it received `: keepalive`.
3. `event: error` becomes a real surfaced error.
4. `tool_call` and other non-text progress chunks are preserved or surfaced, not silently discarded.
5. Streaming requests still use infinite transport timeouts.
6. Tests cover:
   - comment frame before any data,
   - long gap between meaningful chunks,
   - route-level `event: error`,
   - completion handling,
   - unknown/new chunk type tolerance.

## Suggested implementation shape

Use one shared parser per SDK, then route chat/audio/event/log streaming through it.

Per SDK:

1. parse raw SSE frames,
2. normalize heartbeat / done / error handling,
3. normalize chat chunk envelopes,
4. expose language-idiomatic stream outputs.

Do not duplicate slightly different parsers between sync/async or between chat and other SSE consumers if one helper can serve both.

## Non-goals

- Do not change runtime behavior here.
- Do not add new network dependencies without approval.
- Do not redesign public SDK streaming APIs unless the existing API cannot represent the required events safely.

## Useful references

- Runtime fix context: `muxi-ai/runtime` `v0.20260408.2`
- CLI fix context: `muxi-ai/cli` commit `7d9b475`
- SDK conventions: `contributing/conventions.md`
- SDK designs:
  - `contributing/designs/python.md`
  - `contributing/designs/typescript.md`
  - `contributing/designs/go.md`
