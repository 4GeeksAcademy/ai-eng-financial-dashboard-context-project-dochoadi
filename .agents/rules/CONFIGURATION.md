# CONFIGURATION & DEPENDENCIES RULES
**Alcance:** Dependencies, build config, Docker, environment

---

## RULE: Vite Proxy Centralizes Frontend-Backend Connection

**Alcance:** Frontend-backend networking

**Justificación:**
Proxy is the ONLY routing mechanism. Hardcoded URLs break in Docker. Centralized proxy = easy to change URL without code changes (config only).

**Evidencia:**
- `frontend/vite.config.ts:10-16` - Proxy configuration
- `frontend/src/App.tsx:16` - fetch uses relative URL `/api/metrics`
- `docker-compose.yml:13` - backend service name is `backend`

**Guía Accionable:**
1. Frontend API calls: ALWAYS use `/api/...` (relative path)
2. Vite config: Proxy target is `http://backend:8000` (see vite.config.ts:13)
3. Never hardcode: No `http://localhost:8000`, `https://api.com`, domain names
4. Environment override: Use `VITE_API_BASE_URL` env var if needed (see App.tsx:13)
5. Docker change: If backend URL changes, update `vite.config.ts:13` target ONLY

**Ejemplo Correcto:**
```typescript
// frontend/vite.config.ts
proxy: {
  "/api": {
    target: "http://backend:8000",  // Update HERE for prod
    changeOrigin: true,
  },
},

// frontend/src/App.tsx
fetch(`${API_BASE_URL}/api/metrics`)  // Uses relative /api/metrics
```

**Ejemplo Incorrecto:**
```typescript
// ❌ WRONG - Hardcoding URL in code
fetch('http://backend:8000/api/metrics')  // Breaks if URL changes

// ❌ WRONG - Creating new proxy per route
proxy: {
  "/api/metrics": { target: "..." },
  "/api/summary": { target: "..." },
}
```

---

## RULE: Package Scripts Must Include Type Checking

**Alcance:** Frontend build process

**Justificación:**
TypeScript is useless if build skips type checking. `tsc -b` catches errors before production. Full pipeline: check types → compile → bundle.

**Evidencia:**
- `frontend/package.json:8` - `"build": "tsc -b && vite build"`
- Type errors stop build (no silent pass-through)

**Guía Accionable:**
1. build script: MUST include `tsc -b && vite build` (type check first)
2. Order: Type check FIRST, then bundle (if types pass, bundle succeeds)
3. dev script: May skip tsc (hot reload doesn't need it), but build MUST include
4. Never: `vite build` alone (TypeErrors go to production)
5. CI/CD: Run `npm run build` to validate TypeScript before deploy

**Ejemplo Correcto:**
```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "lint": "eslint .",
    "test": "vitest run"
  }
}
```

**Ejemplo Incorrecto:**
```json
{
  "scripts": {
    "build": "vite build"  // ❌ Skips TypeScript check
  }
}
```

---

## RULE: Dockerfiles Optimize Build Cache

**Alcance:** Docker build process

**Justificación:**
Layer order matters. If code changes, Docker rebuilds layers. COPY deps first → RUN install → COPY code = cache reuse. Code-only changes don't rebuild deps.

**Evidencia:**
- `frontend/Dockerfile:5-6` - COPY package*.json → RUN npm install
- `frontend/Dockerfile:8` - COPY . . (code copied LAST)
- Same pattern in `backend/Dockerfile`

**Guía Accionable:**
1. Layer 1: FROM [base image]
2. Layer 2: WORKDIR /app
3. Layer 3: COPY [dependencies file] ./ (package.json, requirements.txt)
4. Layer 4: RUN [install command] (npm install, pip install)
5. Layer 5: COPY . . (copy entire code)
6. Layer 6: EXPOSE [ports]
7. Layer 7: CMD [command]

**Rationale:** Changing code (Layer 5) doesn't invalidate cache for Layers 3-4 (deps).

**Ejemplo Correcto:**
```dockerfile
FROM node:24-alpine
WORKDIR /app
COPY package*.json ./       # Deps first
RUN npm install             # Install (layer cached if package.json unchanged)
COPY . .                    # Code last (frequent changes)
EXPOSE 5173
CMD ["npm", "run", "dev"]
```

**Ejemplo Incorrecto:**
```dockerfile
FROM node:24-alpine
WORKDIR /app
COPY . .                    # ❌ Code first = cache invalidated on every change
RUN npm install             # ❌ Rebuilds deps even if package.json unchanged
EXPOSE 5173
CMD ["npm", "run", "dev"]
```

---

## RULE: docker-compose Service Names Are Stable

**Alcance:** Docker Compose orchestration

**Justificación:**
Service names are DNS addresses inside containers (e.g., `http://backend:8000`). Consistent names ensure config stays in docker-compose.yml, not code.

**Evidencia:**
- `docker-compose.yml:1-23` - Services named `frontend`, `backend` (simple, stable)
- Vite proxy target: `http://backend:8000` (uses service name)
- No image: containers, no node-1, web_app_2 naming

**Guía Accionable:**
1. Service names: `frontend`, `backend` (simple, memorable)
2. Never rename: Renaming requires updates in code (vite.config.ts, Dockerfiles)
3. depends_on: backend < frontend (frontend waits for backend to start)
4. Ports: Explicit mapping (5173:5173, 8000:8000, 5678:5678)
5. Network: Default bridge (automatic, no custom network needed)

**Ejemplo Correcto:**
```yaml
services:
  frontend:
    ports: ["5173:5173"]
    depends_on: [backend]
  
  backend:
    ports: ["8000:8000", "5678:5678"]
```

**Ejemplo Incorrecto:**
```yaml
services:
  web_app:           # ❌ Vague name
    ports: ["5173:5173"]
  
  api_server:        # ❌ Must update vite.config.ts target
    ports: ["8000:8000"]
    
  # Missing depends_on: frontend may start before backend
```

