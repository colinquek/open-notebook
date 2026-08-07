# Active Context

## Current Session (2026-08-07) - COMPLETED ✅

### Focus
SSL verification fix for CloudFlare SSL inspection scenarios. User is behind CloudFlare and Groq connection tests fail with "Connection error" even with `ESPERANTO_SSL_VERIFY=false` set.

### Phase 1: Python Code Changes (Already in sslfix branch)
1. **Identified Root Cause**: `httpx.AsyncClient` calls in application code were not reading `ESPERANTO_SSL_VERIFY` environment variable (only Esperanto library reads it)

2. **Implemented Fix**: Added `_get_ssl_verify_setting()` helper function and applied to all 19 `httpx.AsyncClient` instantiations across 4 files:
   - `open_notebook/ai/connection_tester.py` - 4 locations
   - `api/credentials_service.py` - 7 locations
   - `open_notebook/ai/model_discovery.py` - 7 locations
   - `open_notebook/utils/version_utils.py` - 1 location

3. **Created Memory Bank**: Documented entire project structure, SSL fix implementation, and current issues

### Phase 2: Docker Build Fixes (Session 2 - After Reclone)
1. **Fixed Dockerfile**:
   - Added `npm cache clean --force || true` to clear corrupted cache
   - Added `npm config set strict-ssl false` to disable strict SSL for npm

2. **Updated docker-compose.sslfix.yml**:
   - Changed from remote image to local build
   - Added `ESPERANTO_SSL_VERIFY=false` environment variable

### Current Status
- ✅ Code changes applied (already in sslfix branch)
- ✅ Docker build issues resolved
- ✅ Docker image built successfully (`open-notebook:sslfix`)
- ✅ Services running (SurrealDB + Open Notebook)
- ✅ Groq connection test: **SUCCESS**
- ✅ Memory Bank updated with final state

### Test Results
```
Health check: healthy
Groq test: Connection successful
SSL warnings: None (ESPERANTO_SSL_VERIFY=false working)
```

## Recent Changes

### Phase 1: Python Code Changes (Already in sslfix branch)
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

### Phase 2: Docker Build Fixes (Session 2 - 2026-08-07)
5. `Dockerfile`
   - Added `npm cache clean --force || true`
   - Added `npm config set strict-ssl false`

6. `docker-compose.sslfix.yml`
   - Changed to local build: `image: open-notebook:sslfix`
   - Added `build:` section with context and target
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

### Resolved Decisions
1. ✅ **Docker frontend build issue** - Fixed with npm cache clean + strict-ssl false
2. ✅ **SSL fix approach** - Working with distributed `_get_ssl_verify_setting()` functions

### Active Decisions
- Using `ESPERANTO_SSL_VERIFY=false` for CloudFlare scenario (user's environment)
- Memory Bank maintained for session continuity

### Future Considerations
- Could centralize `_get_ssl_verify_setting()` in a shared utils module
- Production deployments should use `ESPERANTO_SSL_CA_BUNDLE` instead of disabling verification
