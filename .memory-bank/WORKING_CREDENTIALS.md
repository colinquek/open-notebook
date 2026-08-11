# Working Provider Credentials

## Environment Setup

```bash
# SSL Fix for CloudFlare/proxy scenarios
ESPERANTO_SSL_VERIFY=false

# API Keys
GROQ_API_KEY=gsk_******************************
CEREBRAS_API_KEY=cs_******************************
CEREBRAS_ENDPOINT=https://api.cerebras.ai/v1
```

## Groq (Native Provider)

### Configuration
```json
{
  "name": "Groq",
  "provider": "groq",
  "modalities": ["language"],
  "api_key": "gsk_******************************"
}
```

### Test Result
```json
{
  "provider": "groq",
  "success": true,
  "message": "Connected. X models available: ..."
}
```

### Status
WORKING

---

## Cerebras (OpenAI-Compatible)

### Configuration
```json
{
  "name": "Cerebras by Sunshine",
  "provider": "openai_compatible",
  "modalities": ["language"],
  "api_key": "cs_******************************",
  "base_url": "https://api.cerebras.ai/v1"
}
```

### Test Result
```json
{
  "provider": "openai_compatible",
  "success": true,
  "message": "Connected. 3 models available: gpt-oss-120b, zai-glm-4.7, gemma-4-31b"
}
```

### Status
WORKING - 3 models discovered

### Key Finding
Base URL must include /v1 path:
- Correct: https://api.cerebras.ai/v1
- Incorrect: https://api.cerebras.ai (returns 404)

---

## Verification Commands

### Health Check
```bash
curl http://localhost:5055/health
# Expected: {"status": "healthy"}
```

### Test Credential
```bash
curl -X POST http://localhost:5055/api/credentials/{id}/test
```

### List Credentials
```bash
curl http://localhost:5055/api/credentials
```

---

## Summary

| Provider | Status | Configuration | Models |
|----------|--------|---------------|--------|
| Groq | Working | provider: groq | Multiple |
| Cerebras | Working | provider: openai_compatible, base_url: https://api.cerebras.ai/v1 | 3 models |

**Key Success Factor**: ESPERANTO_SSL_VERIFY=false required for CloudFlare/proxy scenarios.
