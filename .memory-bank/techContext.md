# Technical Context

## Technologies Used

### Backend
- Python 3.12, FastAPI, SurrealDB, Esperanto, LangChain, HTTPX, Pydantic, Loguru

### Frontend
- Next.js, Node.js 22, npm

### Infrastructure
- Docker, Docker Compose, Supervisor, uv

## Development Setup

### Quick Start
```bash
docker-compose -f docker-compose.sslfix.yml up -d
curl http://localhost:5055/health
```

### Access Points
- UI: http://localhost:8502
- API: http://localhost:5055/docs

## SSL Fix Implementation

### Problem
Connection tests failed behind CloudFlare SSL inspection even with ESPERANTO_SSL_VERIFY=false.

### Root Cause
httpx.AsyncClient calls in application code were not reading ESPERANTO_SSL_VERIFY.

### Solution
Added _get_ssl_verify_setting() function to 4 files, applied to 19 httpx calls.

```python
def _get_ssl_verify_setting() -> bool:
    setting = os.environ.get("ESPERANTO_SSL_VERIFY", "true").lower()
    return setting not in ("false", "0", "no", "off")
```

## Docker Build Resolution

### Problem
npm ci failed with "Exit handler never called!"

### Solution
```dockerfile
RUN npm cache clean --force || true
RUN npm config set strict-ssl false
```

## Working Provider Configurations

### Groq (Native)
**Environment:** GROQ_API_KEY=gsk_******************************

**Configuration:**
```json
{
  "provider": "groq",
  "api_key": "gsk_******************************"
}
```

**Test:** Returns success with model list

### Cerebras (OpenAI-Compatible)
**Environment:** CEREBRAS_API_KEY=cs_******************************

**Configuration:**
```json
{
  "provider": "openai_compatible",
  "api_key": "cs_******************************",
  "base_url": "https://api.cerebras.ai/v1"
}
```

**Test:** Connected. 3 models available: gpt-oss-120b, zai-glm-4.7, gemma-4-31b

**Key Finding:** Must include /v1 in base_url

## Environment Variables

### Required
- OPEN_NOTEBOOK_ENCRYPTION_KEY
- SURREAL_URL=ws://surrealdb:8000/rpc
- SURREAL_USER, SURREAL_PASSWORD

### Optional
- ESPERANTO_SSL_VERIFY=false (for CloudFlare/proxy)
- GROQ_API_KEY
- CEREBRAS_API_KEY
- CEREBRAS_ENDPOINT=https://api.cerebras.ai/v1
- CORS_ORIGINS, OPEN_NOTEBOOK_MAX_UPLOAD_SIZE_MB

## Technical Constraints
- Async-first for all I/O
- No connection pooling
- 100MB upload limit (configurable)
- 400 token chunk size (configurable)
