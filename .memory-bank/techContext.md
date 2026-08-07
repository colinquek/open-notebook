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
docker compose up -d

# View logs
docker compose logs -f

# Access UI
http://localhost:8502

# Access API
http://localhost:5055/docs
```

### Local Development
```bash
# Backend
cd code/open-notebook
uv run uvicorn api.main:app --port 5055

# Frontend
cd frontend
npm run dev

# Worker
make worker-start
```

## Technical Constraints
1. **Async-first**: All DB queries and AI calls must be async
2. **No connection pooling**: Each DB call opens/closes connection
3. **Single-threaded graph nodes**: Sync nodes use ThreadPool for async calls
4. **Upload limit**: 100MB default (configurable via `OPEN_NOTEBOOK_MAX_UPLOAD_SIZE_MB`)
5. **Chunk size**: 400 tokens default (configurable)

## Dependencies
- `esperanto>=2.25.1,<3` - AI provider SDK
- `surrealdb>=1.0.4` - Database client
- `surreal-commands>=1.3.1,<2` - Background job system
- `podcast-creator>=0.12.0,<1` - Podcast generation
- `langchain-*` - AI orchestration providers
- `httpx[socks]>=0.27.0` - HTTP client

## SSL Fix Implementation (2026-08-07)

### Problem
When behind CloudFlare or other SSL inspection proxies, connection tests to AI providers (Groq, etc.) fail with "Connection error" even when `ESPERANTO_SSL_VERIFY=false` is set.

### Root Cause
The `ESPERANTO_SSL_VERIFY` environment variable is only read by the Esperanto library's internal HTTP client. Direct `httpx.AsyncClient` calls in the application code were not respecting this setting.

### Solution
Added `_get_ssl_verify_setting()` helper function to read the environment variable and applied it to all 19 `httpx.AsyncClient` instantiations across 4 files:

1. `open_notebook/ai/connection_tester.py` (4 locations)
2. `api/credentials_service.py` (7 locations)
3. `open_notebook/ai/model_discovery.py` (7 locations)
4. `open_notebook/utils/version_utils.py` (1 location)

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

### Usage in httpx Calls
```python
async with httpx.AsyncClient(
    timeout=10.0,
    verify=_get_ssl_verify_setting(),
) as client:
    response = await client.get(url, headers=headers)
```

## Docker Build Issue
Frontend build consistently fails on `npm ci` with "Exit handler never called!" error. This appears to be an npm bug that persists through all 5 retry attempts. Backend-only builds complete successfully in ~5-10 minutes.

## Environment Variables
- `OPEN_NOTEBOOK_ENCRYPTION_KEY` - Required for credential encryption
- `ESPERANTO_SSL_VERIFY` - SSL verification (default: true)
- `SURREAL_URL` - Database connection string
- `CORS_ORIGINS` - Allowed CORS origins
- `OPEN_NOTEBOOK_MAX_UPLOAD_SIZE_MB` - Upload size limit (default: 100)
