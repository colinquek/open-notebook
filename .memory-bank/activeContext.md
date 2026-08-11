# Active Context

## Current Session (2026-08-11) - Cerebras Integration Complete

### Focus
Completed SSL verification fix and successfully configured both Groq and Cerebras providers.

### What Was Completed

#### Phase 1: Python SSL Fix (Already in sslfix branch)
- Added _get_ssl_verify_setting() to 4 files
- Updated 19 httpx.AsyncClient calls
- All connection tests now respect ESPERANTO_SSL_VERIFY=false

#### Phase 2: Docker Build Fixes
- Fixed npm "Exit handler never called!" bug
- Added npm cache clean and strict-ssl false
- Build completes in 37 steps

#### Phase 3: Provider Testing - COMPLETED
1. **Groq** - Working
   - Provider: groq (native)
   - API Key: gsk_******************************
   - Status: Connection successful

2. **Cerebras** - Working
   - Provider: openai_compatible
   - API Key: cs_******************************
   - Base URL: https://api.cerebras.ai/v1
   - Status: Connected, 3 models available
   - Models: gpt-oss-120b, zai-glm-4.7, gemma-4-31b

### Current Status
- SSL fix: Applied and tested
- Docker build: Successful
- Services: Running (SurrealDB + Open Notebook)
- Groq: Working
- Cerebras: Working
- Credential: "Cerebras by Sunshine" configured
- Memory Bank: Updated

### Key Discovery
Cerebras requires base_url with /v1 path:
- Correct: https://api.cerebras.ai/v1
- Incorrect: https://api.cerebras.ai (returns 404)

### Next Steps
- No immediate action items
- System ready for production use
- Consider centralizing _get_ssl_verify_setting() in utils module

## Recent Changes

### Files Modified
1. open_notebook/ai/connection_tester.py - SSL fix
2. api/credentials_service.py - SSL fix
3. open_notebook/ai/model_discovery.py - SSL fix
4. open_notebook/utils/version_utils.py - SSL fix
5. Dockerfile - npm fixes
6. docker-compose.sslfix.yml - Local build config

### Credentials
- Deleted: 3 non-working Cerebras test credentials
- Created: "Cerebras by Sunshine" with correct configuration
- Provider changed from groq to openai_compatible for Cerebras

## Important Patterns

### SSL Fix Pattern
```python
def _get_ssl_verify_setting() -> bool:
    setting = os.environ.get("ESPERANTO_SSL_VERIFY", "true").lower()
    return setting not in ("false", "0", "no", "off")

# Usage:
async with httpx.AsyncClient(
    timeout=10.0,
    verify=_get_ssl_verify_setting(),
) as client:
```

### Provider Configuration
```json
{
  "provider": "openai_compatible",
  "base_url": "https://api.cerebras.ai/v1",
  "api_key": "cs_******************************"
}
```
