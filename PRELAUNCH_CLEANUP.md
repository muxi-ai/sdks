# SDKs Pre-Launch Cleanup Plan

4 repos: root `sdks/`, `muxi-go`, `muxi-python`, `muxi-typescript`.

---

## 1. Root Repo (`sdks/`)

### Remove

| File | Reason |
|------|--------|
| `SDK-DESIGN.md` (1,360 lines) | Internal design doc. Move to `../architecture/sdk-design.md` |
| `SDK-CONVENTIONS.md` (478 lines) | Contributor doc. Move to `contributing/conventions.md` |
| `RELEASE_PLAN.md` (137 lines) | Internal planning. Move to `../architecture/sdk-release-plan.md` |

### Create `contributing/`

| File | Content |
|------|---------|
| `contributing/README.md` | How to work with submodules, CI flow, branch strategy (develop->rc->main) |
| `contributing/conventions.md` | Moved from `SDK-CONVENTIONS.md` - cross-language naming, auth, header conventions |
| `contributing/designs/go.md` | Moved from `go/DESIGN.md` - Go SDK architecture decisions |
| `contributing/designs/python.md` | New - Python SDK architecture (httpx transport, sync/async, PyPI publishing) |
| `contributing/designs/typescript.md` | New - TypeScript SDK architecture (fetch-based, ESM-only, npm publishing) |

### Update

| File | Change |
|------|--------|
| `README.md` | Minor: review "Coming Soon" section, ensure no broken language claims |
| `AGENTS.md` | Update layout section to reflect `contributing/` dir, remove `SDK-DESIGN.md` / `SDK-CONVENTIONS.md` / `RELEASE_PLAN.md` references |

### Keep as-is

`LICENSE`, `.gitmodules` (all 12 submodules), all 9 placeholder dirs.

---

## 2. `muxi-go`

### Remove

| File | Reason |
|------|--------|
| `DESIGN.md` (906 lines) | Move to root `contributing/designs/go.md` |
| `src/.DS_Store` | macOS artifact (also add to `.gitignore`) |

### Update

| File | Change |
|------|--------|
| `.gitignore` | Add `.DS_Store`, `.factory/`, `.claude/`, `.cursor/` |

### OpenSSF Hardening

| Item | Current | Action |
|------|---------|--------|
| `.github/dependabot.yml` | Missing | Add (gomod + github-actions) |
| Action pinning | `@v4`, `@v5` tags | Pin to SHA hashes |
| Token permissions | Missing | Add `permissions: {}` at workflow top |
| `SECURITY.md` | Missing | Add (link to muxi org policy, inherited from `.github` repo) |

### Clean

Everything else looks good - no stale docs, examples are real, tests exist.

---

## 3. `muxi-python`

### Remove

| File | Reason |
|------|--------|
| `dist/` dir (2 files) | Build artifacts checked in. Should be gitignored |

### Update

| File | Change |
|------|--------|
| `.gitignore` | Already has `dist/` listed but files are tracked. Need `git rm --cached dist/`. Also add `.factory/`, `.claude/`, `.cursor/` |
| `setup.py` | Review if needed - modern Python uses `pyproject.toml` alone. Low priority. |

### OpenSSF Hardening

| Item | Current | Action |
|------|---------|--------|
| `.github/dependabot.yml` | Missing | Add (pip + github-actions) |
| Action pinning | `@v4`, `@v5` tags | Pin to SHA hashes |
| Token permissions | Missing | Add `permissions: {}` at workflow top |
| `SECURITY.md` | Missing | Add (link to muxi org policy) |

### Gitignore Bloat

The `.gitignore` is ~130 lines with sections for Node.js, Playwright, Jekyll, FAISS, etc. that don't apply to a Python SDK. Consider trimming to Python-relevant entries only.

### Clean

Source code, tests, examples all look good. No internal planning docs.

---

## 4. `muxi-typescript`

### Remove

Nothing to remove. Repo is already clean.

### Update

| File | Change |
|------|--------|
| `.gitignore` | Add `.DS_Store`, `.factory/`, `.claude/`, `.cursor/` |

### OpenSSF Hardening

| Item | Current | Action |
|------|---------|--------|
| `.github/dependabot.yml` | Missing | Add (npm + github-actions) |
| Action pinning | `@v4` tags | Pin to SHA hashes |
| Token permissions | Missing | Add `permissions: {}` at workflow top |
| `SECURITY.md` | Missing | Add (link to muxi org policy) |
| Dependabot vulnerability | 1 low severity | Address |

### Clean

Already in good shape - no stale docs, source + tests + config only.

---

## 5. Execution Order

### Phase 1: Root repo cleanup
1. Move `SDK-DESIGN.md` to `../architecture/sdk-design.md`
2. Move `RELEASE_PLAN.md` to `../architecture/sdk-release-plan.md`
3. Create `contributing/` structure:
   - `contributing/README.md` (new)
   - `contributing/conventions.md` (moved from `SDK-CONVENTIONS.md`)
   - `contributing/designs/go.md` (moved from `go/DESIGN.md`)
   - `contributing/designs/python.md` (new)
   - `contributing/designs/typescript.md` (new)
4. Update root `AGENTS.md` and `README.md`

### Phase 2: Submodule cleanups
5. **Go**: Delete `DESIGN.md` (now in root contributing), remove `.DS_Store`, update `.gitignore`
6. **Python**: `git rm --cached dist/`, update `.gitignore` (trim + add AI dirs)
7. **TypeScript**: Update `.gitignore`

### Phase 3: OpenSSF hardening (all 3 submodules)
8. Add `.github/dependabot.yml` to each repo
9. Pin all workflow actions to SHA hashes
10. Add `permissions: {}` to all workflows
11. Add `SECURITY.md` to each repo
12. Address TypeScript Dependabot vulnerability

### Phase 4: Post-public
13. Add CodeQL workflows (requires public repos)
14. Add fuzz tests (Go: `FuzzXxx`, Python: hypothesis)
15. Run OpenSSF scorecard, iterate to 10/10

---

## Summary

| Repo | Files to Remove | Files to Add | Files to Update |
|------|----------------|--------------|-----------------|
| `sdks/` (root) | 3 (move to architecture/contributing) | 5 (`contributing/` with README, conventions, 3 design docs) | 2 (README, AGENTS) |
| `muxi-go` | 2 (DESIGN.md, .DS_Store) | 2 (dependabot, SECURITY) | 2 (.gitignore, workflows) |
| `muxi-python` | 1 (dist/) | 2 (dependabot, SECURITY) | 2 (.gitignore, workflows) |
| `muxi-typescript` | 0 | 2 (dependabot, SECURITY) | 2 (.gitignore, workflows) |
