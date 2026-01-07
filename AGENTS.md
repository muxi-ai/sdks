<coding_guidelines>
## AGENTS GUIDE (sdks root)

Purpose: quick orientation for AI coding agents working in this mono-repo of language SDK submodules.

### Repo layout
```
sdks/
├── go/           # Go SDK submodule (github.com/muxi-ai/muxi-go)
├── python/       # Python SDK submodule (github.com/muxi-ai/muxi-python)
├── typescript/   # TypeScript SDK submodule (github.com/muxi-ai/muxi-typescript)
├── SDK-DESIGN.md
├── SDK-CONVENTIONS.md
└── RELEASE_PLAN.md
```

### Submodule workflow
1. **Check status**: `git status --short` at root shows `m go` etc. for modified submodules
2. **Edit inside submodule**: `cd go/src && <make changes>`
3. **Commit inside submodule**: `cd go && git add . && git commit -m "..."`
4. **Push submodule**: `cd go && git push`
5. **Update root pointer**: `cd .. && git add go && git commit -m "Update go submodule"`

### SDK-specific commands

| SDK | Test | Format | Build |
|-----|------|--------|-------|
| Go | `cd go/src && go test ./...` | `gofmt -w .` | N/A |
| Python | `cd python && pytest` | `black .` | N/A |
| TypeScript | `cd typescript && npm test` | N/A | `npm run build` |

### Conventions (all SDKs)
- Idempotency header (`X-Muxi-Idempotency-Key`) auto-generated on every request — no toggles
- Streaming calls use infinite timeouts — no per-request deadlines
- Retries: exponential backoff on 429/5xx, respect `Retry-After`
- Auth: HMAC for ServerClient, key headers for FormationClient
- Do not add dependencies without user approval
- Do not edit README/docs unless explicitly requested

### CI/CD
All three SDKs have automated release pipelines:
- **Go**: Tags trigger releases via goreleaser
- **Python**: Push to main triggers PyPI publish (OIDC)
- **TypeScript**: Push to main triggers npm publish (OIDC trusted publishing)

### Deliverables
- Keep responses concise; summarize changes and tests run
- Run validators before completing: `go test`, `pytest`, `npm test`
</coding_guidelines>
