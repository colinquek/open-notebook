# Working Provider Credentials

This document contains the exact configurations used for successful provider connections.

## Environment Setup

### Required Environment Variables
```bash
# SSL Fix for CloudFlare/proxy scenarios
ESPERANTO_SSL_VERIFY=false

# API Keys
GROQ_API_KEY=gsk_******************************hR
CEREBRAS_API_KEY=cs_******************************
```

## Working Configuration: Groq

### Credential Details
```json
{
  "name": "Cerebras by Sunshine",
  "provider": "groq",
  "modalities": ["language"],
  "api_key": "gsk_******************************hR",
  "base_url": null,
  "endpoint": null,
  "api_version": null,
  "endpoint_llm": null,
  "endpoint_embedding": null,
  "endpoint_stt": null,
  "endpoint_tts": null,
  "project": null,
  "location": null,
  "credentials_path": null,
  "num_ctx": null
}
```

### Connection Test
```bash
# Create credential
curl -X POST http://localhost:5055/api/credentials \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Cerebras by Sunshine",
    "provider": "groq",
    "modalities": ["language"],
    "api_key": "gsk_******************************hR"
  }'

# Test connection
curl -X POST http://localhost:5055/api/credentials/{credential_id}/test
```

### Expected Response
```json
{
  "provider": "groq",
  "success": true,
  "message": "Connected. X models available: ..."
}
```

### Status
✅ **WORKING** - Connection successful, models discovered

---

## Under Investigation: Cerebras

### Attempted Configuration
```json
{
  "name": "Cerebras",
  "provider": "openai_compatible",
  "modalities": ["language"],
  "api_key": "cs_******************************",
  "base_url": "https://api.cerebras.ai",
  "endpoint": null
}
```

### Connection Test Result
```json
{
  "provider": "openai_compatible",
  "success": false,
  "message": "Server returned status 404"
}
```

### Issue Analysis
- The code attempts to call `{base_url}/models` endpoint
- With `base_url: "https://api.cerebras.ai"`, it tries `https://api.cerebras.ai/models` → **404**
- Direct API call to `https://api.cerebras.ai/v1/models` → **Works**

### Possible Solutions
1. Try `base_url: "https://api.cerebras.ai/v1"` (includes /v1 in base)
2. Custom endpoint configuration for Cerebras
3. Custom provider implementation

### Next Steps
- [ ] Test with base_url `https://api.cerebras.ai/v1`
- [ ] Check Cerebras API documentation for correct endpoint structure
- [ ] Verify if Cerebras supports OpenAI-compatible /models endpoint

---

## SSL Fix Requirements

### Files Modified
All 4 files must have `_get_ssl_verify_setting()` function:
1. `open_notebook/ai/connection_tester.py`
2. `api/credentials_service.py`
3. `open_notebook/ai/model_discovery.py`
4. `open_notebook/utils/version_utils.py`

### Docker Configuration
```yaml
# docker-compose.sslfix.yml
services:
  open_notebook:
    environment:
      - ESPERANTO_SSL_VERIFY=false
```

### Dockerfile Fixes
```dockerfile
RUN npm cache clean --force || true
RUN npm config set strict-ssl false
```

---

## Verification Commands

### Check Container Health
```bash
curl http://localhost:5055/health
# Expected: {"status": "healthy"}
```

### Check Environment Variable
```bash
docker-compose -f docker-compose.sslfix.yml exec open_notebook env | grep ESPERANTO_SSL_VERIFY
# Expected: ESPERANTO_SSL_VERIFY=false
```

### List Credentials
```bash
curl http://localhost:5055/api/credentials
```

### Test Specific Credential
```bash
curl -X POST http://localhost:5055/api/credentials/{credential_id}/test
```

---

## Summary

| Provider | Status | Configuration | Notes |
|----------|--------|---------------|-------|
| Groq | ✅ Working | provider: groq, no base_url | Connection successful |
| Cerebras | ⚠️ Investigation | provider: openai_compatible, base_url: https://api.cerebras.ai | 404 on /models endpoint |

**Key Success Factor**: SSL fix (`ESPERANTO_SSL_VERIFY=false`) is required for CloudFlare/proxy scenarios.
