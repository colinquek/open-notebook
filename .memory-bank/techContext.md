# Technical Context

## Technologies Used

### Backend
- **Python 3.12** - Primary language
- **FastAPI** - REST API framework
- **SurrealDB** - Multi-model database (graph + document)
- **Esperanto** - AI provider abstraction layer
- **LangChain** - AI/LLM orchestration
- **HTTPX** - Async HTTP client
- **Pydantic** - Data validation
- **Loguru** - Logging

### Frontend
- **Next.js** - React framework
- **Node.js 22** - Runtime
- **npm** - Package manager

### Infrastructure
- **Docker** - Containerization
- **Docker Compose** - Multi-container orchestration
- **Supervisor** - Process management
- **uv** - Python package manager (faster alternative to pip)

## Development Setup

### Prerequisites
- Docker and Docker Compose
- Python 3.12 (for local development)
- Node.js 22 (for frontend development)
- uv (Python package manager)

### Quick Start
```bash
# Start all services
docker-compose -f docker-compose.sslfix.yml up -d

# View logs
docker-compose -f docker-compose.sslfix.yml logs -f

# Access UI
http://localhost:8502

# Access API
http://localhost:5055/docs
```

## SSL Fix Implementation (2026-08-07)

### Problem
When behind CloudFlare or other SSL inspection proxies, connection tests to AI providers fail even with `ESPERANTO_SSL_VERIFY=false` set.

### Root Cause
The `ESPERANTO_SSL_VERIFY` environment variable is only read by Esperanto library's internal HTTP client. Direct `httpx.AsyncClient` calls in application code were not respecting this setting.

### Solution
Added `_get_ssl_verify_setting()` helper function and applied to all 19 `httpx.AsyncClient` instantiations across 4 files.

### Function Implementation
```python
def _get_ssl_verify_setting() -> bool:
    """Read ESPERANTO_SSL_VERIFY environment variable.
    
    Returns False if set to 'false' (case-insensitive), True otherwise.
    Defaults to True for secure production behavior.
    """
    setting = os.environ.get("ESPERANTO_SSL_VERIFY", "true").lower()
    return setting not in ("false", "0", "no", "off")
```

### Usage Pattern
```python
async with httpx.AsyncClient(
    timeout=10.0,
    verify=_get_ssl_verify_setting(),
) as client:
    response = await client.get(url, headers=headers)
```

## Docker Build Resolution

### Problem
Frontend build failed on `npm ci` with "Exit handler never called!" error (known npm bug).

### Solution
Added to Dockerfile in `frontend-builder` stage:
```dockerfile
RUN npm cache clean --force || true
RUN npm config set strict-ssl false
```

### Result
✅ Docker build completes successfully (37 steps)
✅ Image tagged as `open-notebook:sslfix`

## Working Provider Configurations

### Groq (Tested & Working)
**Environment Variable**: `GROQ_API_KEY=gsk_******************************hR`

**Credential Configuration**:
```json
{
  "name": "Cerebras by Sunshine",
  "provider": "groq",
  "modalities": ["language"],
  "api_key": "gsk_******************************hR",
  "base_url": null,
  "endpoint": null
}
```

**Test Command**:
```bash
curl -X POST http://localhost:5055/api/credentials/{id}/test
```

**Expected Response**:
```json
{
  "provider": "groq",
  "success": true,
  "message": "Connected. X models available: ..."
}
```

### Cerebras (Under Investigation)
**Environment Variable**: `CEREBRAS_API_KEY=cs_******************************`

**Attempted Configuration**:
```json
{
  "name": "Cerebras",
  "provider": "openai_compatible",
  "modalities": ["language"],
  "api_key": "cs_******************************",
  "base_url": "https://api.cerebras.ai"
}
```

**Issue**: `/models` endpoint returns 404
**Direct API Test**: `curl https://api.cerebras.ai/v1/models` works
**Possible Fix**: Try base_url `https://api.cerebras.ai/v1`

## Environment Variables

### Required
- `OPEN_NOTEBOOK_ENCRYPTION_KEY` - Credential encryption secret
- `SURREAL_URL` - Database connection (ws://surrealdb:8000/rpc)
- `SURREAL_USER` - Database username (default: root)
- `SURREAL_PASSWORD` - Database password (default: root)

### Optional
- `ESPERANTO_SSL_VERIFY` - SSL verification (default: true, set false for CloudFlare)
- `GROQ_API_KEY` - Groq API key
- `CEREBRAS_API_KEY` - Cerebras API key
- `CORS_ORIGINS` - Allowed CORS origins (default: *)
- `OPEN_NOTEBOOK_MAX_UPLOAD_SIZE_MB` - Upload limit (default: 100)

## Technical Constraints
1. **Async-first**: All DB queries and AI calls must be async
2. **No connection pooling**: Each DB call opens/closes connection
3. **Single-threaded graph nodes**: Sync nodes use ThreadPool for async calls
4. **Upload limit**: 100MB default (configurable)
5. **Chunk size**: 400 tokens default (configurable)
