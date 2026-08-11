# Open Notebook - Project Brief

## Project Overview
Open Notebook is an open-source, privacy-focused alternative to Google's Notebook LM: an AI-powered research assistant with multi-provider AI support, fully self-hostable.

## Core Features
- Multi-provider AI support (OpenAI, Anthropic, Groq, Ollama, etc.)
- Note-taking with AI-assisted insights
- Podcast generation from notes
- Source processing and embedding
- Graph-based knowledge management
- Self-hosted with SurrealDB backend

## Technical Stack
- **Frontend**: Next.js (port 8502)
- **Backend**: FastAPI/Python (port 5055)
- **Database**: SurrealDB (port 8000)
- **Worker**: surreal-commands for async jobs

## Key Requirements
1. Privacy-first: All data stays local unless explicitly sent to AI providers
2. Multi-provider support: Users can choose their preferred AI provider
3. Self-hostable: Can be deployed via Docker Compose
4. Extensible: Plugin architecture for commands and transformations

## Current Focus
- SSL verification fix for CloudFlare SSL inspection scenarios
- Docker build optimization
- Multi-provider connectivity (Groq working, Cerebras investigation)

## SSL Fix Implementation (2026-08-07)

### Phase 1: Python Code Changes
Modified 4 files to support `ESPERANTO_SSL_VERIFY` environment variable:

1. `open_notebook/ai/connection_tester.py` - Added `_get_ssl_verify_setting()` function
2. `api/credentials_service.py` - Added `_get_ssl_verify_setting()` function
3. `open_notebook/ai/model_discovery.py` - Added `_get_ssl_verify_setting()` function
4. `open_notebook/utils/version_utils.py` - Added `_get_ssl_verify_setting()` function

All `httpx.AsyncClient` instantiations (19 total) now pass `verify=_get_ssl_verify_setting()` parameter.

### Phase 2: Docker Build Fixes
1. **Dockerfile** - Added npm cache clean and SSL disable:
   - `RUN npm cache clean --force || true`
   - `RUN npm config set strict-ssl false`

2. **docker-compose.sslfix.yml** - Updated configuration:
   - Changed from remote image to local build
   - Added `ESPERANTO_SSL_VERIFY=false` environment variable

## Working Provider Configurations

### Groq (Working)
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
**Status**: ✅ Connection successful
**Test**: `POST /api/credentials/{id}/test` returns `{"success": true, "message": "Connected..."}`

### Cerebras (Investigation)
```json
{
  "name": "Cerebras",
  "provider": "openai_compatible",
  "modalities": ["language"],
  "api_key": "cs_******************************",
  "base_url": "https://api.cerebras.ai"
}
```
**Status**: ❌ Returns 404 on `/models` endpoint
**Issue**: Cerebras API doesn't expose `/models` endpoint at expected path
**Workaround**: Direct API calls to `https://api.cerebras.ai/v1/models` work

## Test Results
✅ Groq connection test successful with `ESPERANTO_SSL_VERIFY=false`
✅ No SSL errors in container logs
✅ Docker build completes successfully
✅ Health check: healthy
