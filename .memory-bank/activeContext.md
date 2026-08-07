# Active Context

## Current Session (2026-08-07)

### Focus
SSL verification fix for CloudFlare SSL inspection scenarios. User is behind CloudFlare and Groq connection tests fail with "Connection error" even with `ESPERANTO_SSL_VERIFY=false` set.

### What Was Done
1. **Identified Root Cause**: `httpx.AsyncClient` calls in application code were not reading `ESPERANTO_SSL_VERIFY` environment variable (only Esperanto library reads it)

2. **Implemented Fix**: Added `_get_ssl_verify_setting()` helper function and applied to all 19 `httpx.AsyncClient` instantiations across 4 files:
   - `open_notebook/ai/connection_tester.py` - 4 locations
   - `api/credentials_service.py` - 7 locations
   - `open_notebook/ai/model_discovery.py` - 7 locations
   - `open_notebook/utils/version_utils.py` - 1 location

3. **Updated docker-compose.yml**: Added `ESPERANTO_SSL_VERIFY=false` environment variable

4. **Created Memory Bank**: Documented entire project structure, SSL fix implementation, and current issues

### Current Status
- ✅ Code changes applied and syntax-validated
- ✅ Memory Bank created with full documentation
- ❌ Docker build failing on frontend npm dependencies
- ⏳ User will reclone from remote and reapply changes

### Docker Build Issue
Frontend build fails consistently on `npm ci` with "Exit handler never called!" error. This is a known npm bug that persists through all 5 retry attempts (45+ minutes). Backend-only builds work fine.

**Workaround**: Use backend-only Dockerfile for testing SSL fix.

### Next Steps for User
1. Reclone repository from remote
2. Reapply SSL fix changes to the 4 files (see `techContext.md` for exact changes)
3. Build backend-only image or wait for npm fix
4. Test Groq connectivity with `ESPERANTO_SSL_VERIFY=false`

## Recent Changes

### Files Modified (SSL Fix)
1. `open_notebook/ai/connection_tester.py`
   - Added `_get_ssl_verify_setting()` function
   - Updated 4 `httpx.AsyncClient` calls

2. `api/credentials_service.py`
   - Added `_get_ssl_verify_setting()` function
   - Updated 7 `httpx.AsyncClient` calls

3. `open_notebook/ai/model_discovery.py`
   - Added `_get_ssl_verify_setting()` function
   - Updated 7 `httpx.AsyncClient` calls

4. `open_notebook/utils/version_utils.py`
   - Added `_get_ssl_verify_setting()` function
   - Updated 1 `httpx.AsyncClient` call

5. `docker-compose.yml`
   - Added `ESPERANTO_SSL_VERIFY=false` environment variable

### Memory Bank Created
All 6 core files created in `.memory-bank/`:
- `projectbrief.md` - Project overview and SSL fix summary
- `productContext.md` - Product goals and user experience
- `techContext.md` - Technologies and SSL fix details
- `systemPatterns.md` - Architecture and design patterns
- `activeContext.md` - Current work (this file)
- `progress.md` - What works and what's left

## Important Patterns and Preferences

### Code Style
- No emojis in code or comments
- No flattering language
- Direct, concise communication
- Type hints required
- Async-first for all I/O

### SSL Fix Pattern
```python
def _get_ssl_verify_setting() -> bool:
    """Read ESPERANTO_SSL_VERIFY environment variable."""
    setting = os.environ.get("ESPERANTO_SSL_VERIFY", "true").lower()
    return setting not in ("false", "0", "no", "off")
```

Always use with httpx:
```python
async with httpx.AsyncClient(
    timeout=10.0,
    verify=_get_ssl_verify_setting(),
) as client:
    # ...
```

## Learnings and Insights

### Key Learnings
1. **Environment Variable Scope**: `ESPERANTO_SSL_VERIFY` only affects Esperanto library's internal HTTP client, not application's direct httpx calls

2. **Docker Build Issues**: npm on certain architectures has persistent "Exit handler never called!" bug that retry logic cannot overcome

3. **Backend-First Testing**: Can test backend fixes without frontend by building backend-only Docker image

4. **Memory Bank Value**: Structured documentation enables effective session-to-session continuity

### Considerations
- SSL verification should default to `True` for security
- Only disable in trusted environments or with custom CA bundles
- Production deployments should use `ESPERANTO_SSL_CA_BUNDLE` instead of disabling verification

## Current Decisions

### Pending Decisions
1. How to resolve Docker frontend build issue?
   - Option A: Fix npm configuration
   - Option B: Use backend-only builds for now
   - Option C: Pre-build frontend and cache

2. Should SSL fix be upstreamed to Esperanto library?
   - Currently each file has its own `_get_ssl_verify_setting()` function
   - Could centralize in a utils module

### Active Decisions
- Using `ESPERANTO_SSL_VERIFY=false` for CloudFlare scenario (user's environment)
- Backend-only Docker builds acceptable for testing
- Memory Bank created for session continuity
