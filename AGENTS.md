## AGENTS GUIDE (sdks root)

Purpose: quick orientation for AI coding agents working in this mono-repo of language SDK submodules.

### Repo layout
- `go/`, `python/`, `typescript/`, etc. are git submodules; most work here targets `go/`.
- Root docs: `SDK-DESIGN.md`, `SDK-CONVENTIONS.md`, `RELEASE_PLAN.md`.

### Workflow
- Inspect status: `git status --short` (run at repo root). Submodule changes appear as `m go` etc.
- To update Go SDK code, edit inside `go/src/` (the Go module root is `go/src`).
- After modifying Go submodule, commit **inside** `go/`, then return to root, `git add go`, and commit the pointer.

### Go submodule quick commands
- Format: `cd go/src && gofmt -w <files>`
- Tests: `cd go/src && go test ./...`
- Status in submodule: `cd go && git status --short`
- Push submodule: `cd go && git push`

### Conventions & cautions
- Idempotency header must remain automatic on every request; don’t add toggles.
- Streaming calls should keep infinite timeouts; don’t add per-request deadlines unless required.
- Avoid editing README/other docs unless explicitly requested.
- Do not add new dependencies without user approval.

### Deliverables
- Keep responses concise; summarize changes and tests run.
- Ensure validators (at least `go test ./...` when Go code changes) are run before final summary.
