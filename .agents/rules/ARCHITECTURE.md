# ARCHITECTURE RULES
**Alcance:** Full-stack (Frontend + Backend + Docker)

---

## RULE: Proxy-First API Communication

**Alcance:** Frontend-Backend communication

**Justificación:**
Frontend and backend run in separate containers (Vite on 5173, FastAPI on 8000). Direct URLs break in Docker. Vite proxy centralizes routing, making development seamless without CORS config.

**Evidencia:**
- `frontend/vite.config.ts:10-16` - Proxy configuration
- `frontend/src/App.tsx:13-16` - API consumption via relative path
- `docker-compose.yml:11-15` - Containers separation

**Guía Accionable:**
1. ALWAYS use relative paths for API calls: `fetch('/api/metrics')`
2. NEVER hardcode domain/port: `http://localhost:8000`, `http://backend:8000`
3. Vite proxy AUTOMATICALLY rewrites `/api/*` to backend
4. Environment variables: Use `VITE_API_BASE_URL` env var if needed (see App.tsx line 13)
5. In production: Update proxy target in `vite.config.ts` OR use `VITE_API_BASE_URL`

**Ejemplo Correcto:**
```typescript
// frontend/src/App.tsx:16
const response = await fetch(`${API_BASE_URL}/api/metrics`);
// where API_BASE_URL = import.meta.env.VITE_API_BASE_URL ?? ""
// Vite proxy automatically routes /api to backend:8000
```

**Ejemplo Incorrecto:**
```typescript
// ❌ WRONG - Hardcoded URL breaks in Docker
const response = await fetch('http://backend:8000/api/metrics');

// ❌ WRONG - localhost doesn't work in containers
const response = await fetch('http://localhost:8000/api/metrics');

// ❌ WRONG - Full domain breaks in dev
const response = await fetch('https://api.example.com/metrics');
```

---

## RULE: Backend Modular Design (main.py Orchestrates Only)

**Alcance:** Backend architecture

**Justificación:**
Clear separation allows testable, maintainable code. `main.py` handles app setup, middleware, router inclusion. `routes.py` contains all endpoints. This enables tests to import app without side effects.

**Evidencia:**
- `backend/app/main.py:1-14` - Only orchestration
- `backend/app/routes.py:19+` - All endpoints defined here
- `backend/tests/test_routes.py:5-6` - Tests import app cleanly

**Guía Accionable:**
1. `main.py`: ONLY define FastAPI app, add middleware (CORS), include router
2. `routes.py`: ONLY add @router endpoints, helper functions
3. `main.py`: NEVER add business logic, endpoints, or data processing
4. New functionality: Create function in `routes.py`, add @router.get/post decorator
5. Tests: Always import `from app.main import app`, not creating new instance

**Ejemplo Correcto:**
```python
# backend/app/routes.py
@router.get("/api/new-endpoint")
def get_new_endpoint():
    result = process_data()
    return result

# backend/app/main.py
from app.routes import router
app = FastAPI()
app.include_router(router)
```

**Ejemplo Incorrecto:**
```python
# ❌ WRONG - Logic in main.py
from fastapi import FastAPI
app = FastAPI()

@app.get("/api/endpoint")
def my_endpoint():
    pass

# ❌ WRONG - Middleware in routes
from fastapi import APIRouter
router = APIRouter()
router.add_middleware(...)
```

