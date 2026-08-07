# System Patterns

## Architecture Overview

### Three-Tier Architecture
```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│  Frontend   │────▶│   Backend    │────▶│ SurrealDB   │
│  (Next.js)  │     │  (FastAPI)   │     │             │
│  Port 3000  │     │  Port 5055   │     │  Port 8000  │
└─────────────┘     └──────────────┘     └─────────────┘
                           │
                           ▼
                    ┌──────────────┐
                    │    Worker    │
                    │ (surreal-    │
                    │  commands)   │
                    └──────────────┘
```

### Component Layers

#### API Layer (`api/`)
- **Routes** (`api/routers/`) - HTTP endpoints, thin request handling
- **Services** (`api/*_service.py`) - Business logic
- **Models** (`api/models.py`) - Request/response schemas

#### Domain Layer (`open_notebook/domain/`)
- `ObjectModel` - Base class for all domain objects
- `RecordModel` - Database-backed records
- `Credential`, `Model`, `Note`, `Notebook`, `Source` - Domain entities

#### AI Layer (`open_notebook/ai/`)
- `connection_tester.py` - Provider connectivity testing
- `model_discovery.py` - Automatic model discovery
- `provider_registry.py` - Provider metadata registry
- `key_provider.py` - API key management
- `models.py` - Model manager

#### Graph Layer (`open_notebook/graphs/`)
- Chat graphs for conversations
- Podcast generation graphs
- Source processing graphs
- All use LangGraph with SqliteSaver for checkpointing

#### Commands Layer (`commands/`)
- Background job implementations
- Async processing for heavy tasks
- Retry logic with blocklist (`stop_on: [ValueError]`)

## Key Design Patterns

### Service Layer Pattern
```
Route → Service → Repository
```
- Routes handle HTTP concerns (validation, auth, error mapping)
- Services contain business logic
- Repositories handle database access

### Singleton Pattern
- `RecordModel` subclasses are singletons
- Use `clear_instance()` in tests
- `DefaultModels.get_instance()` bypasses singleton cache

### Polymorphic ID Resolution
- `ObjectModel.get()` resolves type from ID prefix
- Subclass must be imported first or resolution fails
- Prefixes: `credential:`, `model:`, `note:`, `notebook:`, etc.

### Error Handling Pattern
```python
from open_notebook.exceptions import ConfigurationError

try:
    # ... operation
except SpecificError as e:
    raise ConfigurationError("User-friendly message") from e
```

Exception hierarchy maps to HTTP status codes:
- `NotFoundError` → 404
- `InvalidInputError` → 400
- `AuthenticationError` → 401
- `ConfigurationError` → 422
- `NetworkError`/`ExternalServiceError` → 502

### Async-First Pattern
All database and AI calls are async:
```python
async def get_data():
    result = await repo_query("SELECT * FROM table")
    return result
```

### SSRF Protection Pattern
All user-supplied URLs go through `validate_url()`:
```python
from open_notebook.utils.url_validation import validate_url, prepare_pinned_http_target

await validate_url(user_url, provider)
target = await prepare_pinned_http_target(url, provider)
async with httpx.AsyncClient() as client:
    response = await client.get(target.url, headers=target.headers)
```

## SSL Fix Pattern (2026-08-07)

### Before
```python
async with httpx.AsyncClient(timeout=10.0) as client:
    response = await client.get(url)
```

### After
```python
def _get_ssl_verify_setting() -> bool:
    setting = os.environ.get("ESPERANTO_SSL_VERIFY", "true").lower()
    return setting not in ("false", "0", "no", "off")

async with httpx.AsyncClient(
    timeout=10.0,
    verify=_get_ssl_verify_setting(),
) as client:
    response = await client.get(url)
```

### Applied To
- All connection test functions
- All model discovery functions
- Version check endpoint
- Total: 19 locations across 4 files

## Critical Implementation Paths

### Credential Flow
1. User creates credential via API
2. API key encrypted with `OPEN_NOTEBOOK_ENCRYPTION_KEY`
3. Credential saved to SurrealDB
4. Test connection via `connection_tester.py`
5. Discover models via `model_discovery.py`
6. Register models linked to credential

### Model Provisioning
1. `provision_langchain_model()` called from graph nodes
2. Checks for credential-linked model
3. Falls back to env var configuration
4. Auto-upgrades to `large_context_model` above 105k tokens
5. Returns LangChain-compatible model instance

### Background Job Flow
1. `submit_command()` queues job in SurrealDB
2. Worker polls for new commands
3. Command executes with retry logic
4. Status updated in database
5. User can check status via API

## Data Flow

### Note Creation
```
User → Frontend → API → Note.save() → [Auto: embed_note command]
                                           ↓
                                      Worker → Embedding → SurrealDB
```

### Source Processing
```
User uploads → API → Source.save() → [No auto-embed]
                        ↓
              User calls source.vectorize()
                        ↓
              Worker → Process → Chunk → Embed → SurrealDB
```

### Chat Flow
```
User message → Frontend → /api/chat
    ↓
Graph execution (LangGraph)
    ↓
provision_langchain_model() → AI provider
    ↓
Response → Checkpoint save → Frontend
```
