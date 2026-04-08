# Changelog

All notable changes to the MUXI SDKs will be documented in this file.
Changes apply to all 12 SDKs (Go, Python, TypeScript, Ruby, PHP, C#, Java, Kotlin, Swift, Dart, Rust, C++) unless noted otherwise.

## [0.20260408.0] - 2026-04-08

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
