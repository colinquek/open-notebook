# Open Notebook - Project Brief

## Project Overview
Open Notebook is an open-source, privacy-focused alternative to Google's Notebook LM: an AI-powered research assistant with multi-provider AI support, fully self-hostable.

## Core Features
- Multi-provider AI support (OpenAI, Anthropic, Groq, Cerebras, Ollama, etc.)
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

## Working Provider Configurations

### Groq (Native Provider)
```json
{
  "name": "Groq",
  "provider": "groq",
  "modalities": ["language"],
  "api_key": "gsk_******************************"
}
```
Status: Working

### Cerebras (OpenAI-Compatible)
```json
{
  "name": "Cerebras by Sunshine",
  "provider": "openai_compatible",
  "modalities": ["language"],
  "api_key": "cs_******************************",
  "base_url": "https://api.cerebras.ai/v1"
}
```
Status: Working - 3 models available (gpt-oss-120b, zai-glm-4.7, gemma-4-31b)

## SSL Fix Implementation (2026-08-07)

### Phase 1: Python Code Changes
Modified 4 files to support ESPERANTO_SSL_VERIFY environment variable:
1. open_notebook/ai/connection_tester.py
2. api/credentials_service.py
3. open_notebook/ai/model_discovery.py
4. open_notebook/utils/version_utils.py

All 19 httpx.AsyncClient calls now use verify=_get_ssl_verify_setting()

### Phase 2: Docker Build Fixes
1. Dockerfile - Added npm cache clean and strict-ssl false
2. docker-compose.sslfix.yml - Local build + ESPERANTO_SSL_VERIFY=false

## Test Results
- Groq: Connection successful
- Cerebras: Connection successful (3 models)
- Docker build: 37/37 steps passing
- Health check: healthy
