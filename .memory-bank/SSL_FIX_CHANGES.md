# SSL Fix Changes - Reapplication Guide

## Overview
This document provides the exact changes to reapply the SSL verification fix for CloudFlare SSL inspection scenarios.

## Problem
When behind CloudFlare or other SSL inspection proxies, Groq connection tests fail with "Connection error" even when `ESPERANTO_SSL_VERIFY=false` is set.

## Root Cause
The `ESPERANTO_SSL_VERIFY` environment variable is only read by the Esperanto library's internal HTTP client. Direct `httpx.AsyncClient` calls in the application code were not respecting this setting.

## Solution
Add `_get_ssl_verify_setting()` helper function to read the environment variable and apply it to all `httpx.AsyncClient` instantiations.

---

## Files to Modify (5 files)

### 1. open_notebook/ai/connection_tester.py

**Add after imports (line 25):**
```python
def _get_ssl_verify_setting() -> bool:
    """Read ESPERANTO_SSL_VERIFY environment variable.
    
    Returns False if set to 'false' (case-insensitive), True otherwise.
    Defaults to True for secure production behavior.
    """
    setting = os.environ.get("ESPERANTO_SSL_VERIFY", "true").lower()
    return setting not in ("false", "0", "no", "off")
```

**Update 4 locations where `httpx.AsyncClient` is used:**

Location 1 - Line ~117 (`_test_azure_connection`):
```python
# BEFORE:
async with httpx.AsyncClient(timeout=10.0) as client:

# AFTER:
async with httpx.AsyncClient(
    timeout=10.0,
    verify=_get_ssl_verify_setting(),
) as client:
```

Location 2 - Line ~160 (`_test_ollama_connection`):
```python
# BEFORE:
async with httpx.AsyncClient(timeout=10.0) as client:

# AFTER:
async with httpx.AsyncClient(
    timeout=10.0,
    verify=_get_ssl_verify_setting(),
) as client:
```

Location 3 - Line ~211 (`_test_openai_compatible_connection`):
```python
# BEFORE:
async with httpx.AsyncClient(timeout=10.0) as client:

# AFTER:
async with httpx.AsyncClient(
    timeout=10.0,
    verify=_get_ssl_verify_setting(),
) as client:
```

Location 4 - Line ~264 (`_test_anthropic_compatible_connection`):
```python
# BEFORE:
async with httpx.AsyncClient(timeout=10.0) as client:

# AFTER:
async with httpx.AsyncClient(
    timeout=10.0,
    verify=_get_ssl_verify_setting(),
) as client:
```

---

### 2. api/credentials_service.py

**Add after imports (line 10):**
```python
def _get_ssl_verify_setting() -> bool:
    """Read ESPERANTO_SSL_VERIFY environment variable.
    
    Returns False if set to 'false' (case-insensitive), True otherwise.
    Defaults to True for secure production behavior.
    """
    setting = os.environ.get("ESPERANTO_SSL_VERIFY", "true").lower()
    return setting not in ("false", "0", "no", "off")
```

**Update 7 locations where `httpx.AsyncClient()` is used:**

Pattern to find and replace:
```python
# BEFORE:
async with httpx.AsyncClient() as client:

# AFTER:
async with httpx.AsyncClient(
    verify=_get_ssl_verify_setting(),
) as client:
```

```python
# BEFORE:
async with httpx.AsyncClient(timeout=10.0) as client:

# AFTER:
async with httpx.AsyncClient(
    timeout=10.0,
    verify=_get_ssl_verify_setting(),
) as client:
```

Locations:
- Line ~455 (Ollama discovery)
- Line ~489 (OpenAI-compatible discovery)
- Line ~521 (Anthropic-compatible discovery)
- Line ~548 (OMLX discovery)
- Line ~578 (Azure discovery)
- Line ~610 (Standard provider discovery)
- Line ~674 (Generic discovery)

---

### 3. open_notebook/ai/model_discovery.py

**Add after imports (line 8):**
```python
def _get_ssl_verify_setting() -> bool:
    """Read ESPERANTO_SSL_VERIFY environment variable.
    
    Returns False if set to 'false' (case-insensitive), True otherwise.
    Defaults to True for secure production behavior.
    """
    setting = os.environ.get("ESPERANTO_SSL_VERIFY", "true").lower()
    return setting not in ("false", "0", "no", "off")
```

**Update 7 locations where `httpx.AsyncClient()` is used:**

Same pattern as credentials_service.py above.

Locations:
- Line ~287 (OpenAI discovery)
- Line ~359 (Anthropic discovery)
- Line ~415 (Google discovery)
- Line ~462 (Ollama discovery)
- Line ~697 (OpenAI-compatible discovery)
- Line ~760 (Anthropic-compatible discovery)
- Line ~824 (Generic discovery)

---

### 4. open_notebook/utils/version_utils.py

**Add after imports (line 6):**
```python
def _get_ssl_verify_setting() -> bool:
    """Read ESPERANTO_SSL_VERIFY environment variable.
    
    Returns False if set to 'false' (case-insensitive), True otherwise.
    Defaults to True for secure production behavior.
    """
    setting = os.environ.get("ESPERANTO_SSL_VERIFY", "true").lower()
    return setting not in ("false", "0", "no", "off")
```

**Update 1 location:**

Line ~39:
```python
# BEFORE:
async with httpx.AsyncClient(timeout=10.0) as client:

# AFTER:
async with httpx.AsyncClient(
    timeout=10.0,
    verify=_get_ssl_verify_setting(),
) as client:
```

---

### 5. docker-compose.yml

**Update the open_notebook service environment section:**

```yaml
  open_notebook:
    image: lfnovo/open_notebook:v1-latest  # Or your built image
    environment:
      # ... existing env vars ...
      
      # ADD THIS LINE:
      - ESPERANTO_SSL_VERIFY=false
```

---

## Verification Steps

### 1. Syntax Check
After applying changes, verify syntax:
```bash
cd /mnt/c/code/open-notebook/code/open-notebook
python3 -m py_compile open_notebook/ai/connection_tester.py
python3 -m py_compile api/credentials_service.py
python3 -m py_compile open_notebook/ai/model_discovery.py
python3 -m py_compile open_notebook/utils/version_utils.py
```

All should pass without errors.

### 2. Build Docker Image
```bash
# Option A: Full build (may fail on npm)
docker build --target runtime -t open-notebook:sslfix .

# Option B: Backend-only (recommended for testing)
docker build -f Dockerfile.sslfix -t open-notebook:sslfix .
```

### 3. Run Containers
```bash
docker compose down
docker compose up -d

# Wait for services to start
sleep 15

# Check health
curl -s http://localhost:5055/health
```

### 4. Verify Environment Variable
```bash
docker compose exec open_notebook env | grep ESPERANTO_SSL_VERIFY
# Should output: ESPERANTO_SSL_VERIFY=false
```

### 5. Test Groq Connection
```bash
# Create Groq credential
curl -X POST http://localhost:5055/api/credentials \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Groq Test",
    "provider": "groq",
    "modalities": ["language"],
    "api_key": "gsk_YOUR_ACTUAL_KEY_HERE"
  }'

# Test connection (replace {credential_id} with actual ID from response)
curl -X POST http://localhost:5055/api/credentials/{credential_id}/test
```

**Expected Success Response:**
```json
{
  "provider": "groq",
  "success": true,
  "message": "Connected. X models available: ..."
}
```

---

## Quick Reference

### Function to Add (same in all 4 files)
```python
def _get_ssl_verify_setting() -> bool:
    """Read ESPERANTO_SSL_VERIFY environment variable.
    
    Returns False if set to 'false' (case-insensitive), True otherwise.
    Defaults to True for secure production behavior.
    """
    setting = os.environ.get("ESPERANTO_SSL_VERIFY", "true").lower()
    return setting not in ("false", "0", "no", "off")
```

### httpx Pattern to Replace
```python
# Find all instances of:
async with httpx.AsyncClient(...) as client:

# Replace with:
async with httpx.AsyncClient(
    ...,
    verify=_get_ssl_verify_setting(),
) as client:
```

### Environment Variable
```yaml
- ESPERANTO_SSL_VERIFY=false
```

---

## Summary

- **Files to modify**: 5
- **Function to add**: 4 (same function in each Python file)
- **httpx calls to update**: 19 total
- **Docker compose update**: 1 line

Total time to reapply: ~15-20 minutes

---

## Contact/Support

If issues arise during reapplication:
1. Check Memory Bank files in `.memory-bank/` for context
2. Verify syntax with `python3 -m py_compile`
3. Check container logs: `docker compose logs open_notebook`
4. Test individual components before full integration
