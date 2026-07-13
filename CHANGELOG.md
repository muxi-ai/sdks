# Changelog

All notable changes to the MUXI SDKs will be documented in this file.
Changes apply to all 12 SDKs (Go, Python, TypeScript, Ruby, PHP, C#, Java, Kotlin, Swift, Dart, Rust, C++) unless noted otherwise.

## [Unreleased]

### Added
- UI widget support (Response Envelope UI): chat streams now decode the `event: ui` frame delivered before `event: done`. Go exposes typed `UIWidget`/`UIOption` on `ChatChunk.UI`, TypeScript exports `UIWidget`/`UIOption` interfaces (chunks yield `{ type: "ui", ui: [...] }`), and Python adds a `parse_ui_widgets` helper for its dict-based stream. Unknown widget types pass through untouched (progressive enhancement). Lead SDKs (Go, Python, TypeScript) only so far; remaining SDKs to follow.
- Envelope `request.idempotency_key` echo is now surfaced on unwrapped responses as `idempotency_key` (Python, TypeScript; Go already exposed it via `RequestInfo.IdempotencyKey`).

## [0.20260408.0] - 2026-04-08

## [0.20260514.0] - 2026-05-14

### Fixed
- Go SDK only: moved the Go module root from `go/src` to `go/` so `go get github.com/muxi-ai/muxi-go@latest` and `import "github.com/muxi-ai/muxi-go"` work with standard Go module layout.
- Go SDK only: updated Go CI, RC, and release workflows to use the repository root for `go.sum`, `.version`, unit tests, and integration tests.

### Fixed
- Made chat and audio-chat SSE parsing heartbeat-aware across all SDKs so `: keepalive` comments no longer cause false idle failures.
- Updated SSE handling to parse full event blocks, including multi-line `data:` payloads and event-only frames such as `event: done`.
- Surfaced route-level `event: error` frames as SDK errors instead of dropping them silently.
- Improved runtime chunk compatibility by preserving non-text chat stream event types such as `progress`, `thinking`, `planning`, and `tool_call`.
- Added SDK-specific regression tests covering keepalives, completion handling, route-level errors, and tolerant parsing of new chunk shapes.

## [0.20260324.0] - 2026-03-24

### Added
- `updateSchedulerJob` — update a scheduled job's message, schedule, or title (PUT `/scheduler/jobs/{job_id}`)
- `pauseSchedulerJob` — pause an active scheduled job (POST `/scheduler/jobs/{job_id}/pause`)
- `resumeSchedulerJob` — resume a paused scheduled job (POST `/scheduler/jobs/{job_id}/resume`)
