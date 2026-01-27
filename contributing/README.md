# Contributing to MUXI SDKs

## Repo Structure

This is a mono-repo of language SDK submodules. Each SDK lives in its own GitHub repo.

```
sdks/
├── go/           → github.com/muxi-ai/muxi-go
├── python/       → github.com/muxi-ai/muxi-python
├── typescript/   → github.com/muxi-ai/muxi-typescript
├── contributing/
│   ├── conventions.md       # Cross-language API conventions
│   └── designs/             # Per-language architecture docs
│       ├── go.md
│       ├── python.md
│       └── typescript.md
└── (placeholder dirs for future SDKs)
```

## Working with Submodules

```bash
# Clone with submodules
git clone --recurse-submodules git@github.com:muxi-ai/sdks.git

# Make changes inside a submodule
cd go/src
# ... edit files ...

# Commit inside the submodule
cd ..
git add . && git commit -m "Fix something"
git push origin develop

# Update root pointer
cd ..
git add go && git commit -m "Update go submodule"
git push
```

## Branch Strategy

All three SDKs follow the same flow:

```
develop  →  rc  →  main
  (dev)    (test)  (release + publish)
```

- **develop**: Day-to-day work. CI runs tests.
- **rc**: Release candidate. CI runs full test suite.
- **main**: Triggers package publishing (PyPI / npm / Go tags).

## CI/CD

| SDK | Test | Publish Trigger | Registry |
|-----|------|-----------------|----------|
| Go | `go test ./...` | Tag on main | Go modules |
| Python | `pytest` | Push to main | PyPI (OIDC) |
| TypeScript | `npm test` | Push to main | npm (OIDC trusted publishing) |

## Key Conventions

See [conventions.md](conventions.md) for the full cross-language spec. The essentials:

- Two clients: `ServerClient` (HMAC auth) and `FormationClient` (key auth)
- Idempotency header on every request (`X-Muxi-Idempotency-Key`)
- Exponential backoff retries on 429/5xx
- Streaming uses infinite timeouts
- Typed errors with `code`, `message`, `status_code`

## Design Docs

Per-language architecture decisions live in [designs/](designs/):

- [Go](designs/go.md) — struct-based clients, channel streaming, goroutine patterns
- [Python](designs/python.md) — httpx transport, sync/async duality, context managers
- [TypeScript](designs/typescript.md) — fetch-based, ESM-only, AsyncGenerator streaming
