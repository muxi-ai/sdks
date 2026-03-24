# Changelog

All notable changes to the MUXI SDKs will be documented in this file.
Changes apply to all 12 SDKs (Go, Python, TypeScript, Ruby, PHP, C#, Java, Kotlin, Swift, Dart, Rust, C++) unless noted otherwise.

## [0.20260324.0] - 2026-03-24

### Added
- `updateSchedulerJob` — update a scheduled job's message, schedule, or title (PUT `/scheduler/jobs/{job_id}`)
- `pauseSchedulerJob` — pause an active scheduled job (POST `/scheduler/jobs/{job_id}/pause`)
- `resumeSchedulerJob` — resume a paused scheduled job (POST `/scheduler/jobs/{job_id}/resume`)
