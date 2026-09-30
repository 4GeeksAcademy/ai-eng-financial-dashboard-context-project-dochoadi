# Technology Stack

**Last Updated:** 2026-09-30  
**Source:** package.json, Dockerfile, docker-compose.yml, Code Inspection

---

## Languages & Runtimes

| Component | Language | Version | Evidence |
|-----------|----------|---------|----------|
| Frontend | TypeScript | 6.0.2 | `frontend/package.json:39` — `"typescript": "~6.0.2"` |
| Frontend | JavaScript/JSX (via TS) | ES2020 | `frontend/tsconfig.json:19` — `ecmaVersion: 2020` |
| Backend | Python | 3.13 | `backend/Dockerfile:1` — `FROM python:3.13-slim` |
| Runtime (FE) | Node.js | 24-alpine | `frontend/Dockerfile:1` — `FROM node:24-alpine` |

---

## Frontend Stack

### Core Framework
- **React:** 19.2.4
  - Evidence: `frontend/package.json:19` — `"react": "^19.2.4"`
  - Usage: UI components (App.tsx, dashboard components)
  
- **TypeScript:** 6.0.2
  - Evidence: `frontend/package.json:39` — `"typescript": "~6.0.2"`
  - Config: `frontend/tsconfig.json` + `tsconfig.app.json` + `tsconfig.node.json` (project references)

### Build & Dev Server
- **Vite:** 8.0.4
  - Evidence: `frontend/package.json:41` — `"vite": "^8.0.4"`
  - Purpose: Fast development server with HMR (hot reload)
  - Proxy: `frontend/vite.config.ts:11-16` — Routes `/api/*` to `http://backend:8000`

### Styling
- **Tailwind CSS:** 4.2.2
  - Evidence: `frontend/package.json:26` — `"tailwindcss": "^4.2.2"`
  - Integration: `frontend/vite.config.ts:3` — `@tailwindcss/vite` plugin
  - Config: `frontend/index.css` — Global styles

### Component Libraries
- **Lucide React:** 1.8.0 (Icons)
  - Evidence: `frontend/package.json:18` — `"lucide-react": "^1.8.0"`
  - Usage: Icons in KPI cards (TrendingUp, TrendingDown, DollarSign, BarChart2)
  - Ref: `frontend/src/components/dashboard/kpi-row.tsx:4-5`

- **Recharts:** 3.8.1 (Charts)
  - Evidence: `frontend/package.json:21` — `"recharts": "^3.8.1"`
  - Usage: BarChart (Income vs Outcome), LineChart (Profit %)
  - Ref: `frontend/src/components/dashboard/income-outcome-chart.tsx`, `profit-percent-chart.tsx`

### Testing
- **Vitest:** 4.1.4
  - Evidence: `frontend/package.json:42` — `"vitest": "^4.1.4"`
  - Config: Default, uses `frontend/src/lib/financial-utils.test.ts`
  - Run: `npm test` (package.json:11)

### Linting
- **ESLint:** 9.39.4
  - Evidence: `frontend/package.json:25` — `"eslint": "^9.39.4"`
  - Config: `frontend/eslint.config.js:1-23`
  - Extends: @eslint/js, TypeScript, React Hooks, React Refresh plugins
  - Run: `npm run lint` (package.json:9)

### Other
- **Class Variance Authority (CVA):** 0.7.1 (Styling utility)
  - Evidence: `frontend/package.json:16` — `"class-variance-authority": "^0.7.1"`
  
- **clsx:** 2.1.1 (Class concatenation)
  - Evidence: `frontend/package.json:17` — `"clsx": "^2.1.1"`
  
- **Tailwind Merge:** 3.5.0 (Merge Tailwind classes)
  - Evidence: `frontend/package.json:22` — `"tailwind-merge": "^3.5.0"`

---

## Backend Stack

### Web Framework
- **FastAPI:** Latest (from requirements.txt)
  - Evidence: `backend/requirements.txt:1` — `fastapi`
  - Purpose: Async web framework with automatic Swagger docs
  - Usage: 9 endpoints defined in `backend/app/routes.py`

### ASGI Server
- **Uvicorn:** Standard variant
  - Evidence: `backend/requirements.txt:2` — `uvicorn[standard]`
  - Runtime: `backend/Dockerfile:12` — `python -m uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload`
  - Features: Async HTTP, auto-reload in dev

### Data Validation
- **Pydantic:** (Bundled with FastAPI)
  - Usage: 7 data models defined in `backend/app/routes.py:22-62`
  - Models: FinancialMovement, MetricsFacets, MetricsSummaryItem, TopCategoryItem, MetricsComparison, MetricsAlert
  - Evidence: `backend/app/routes.py:22` — `class FinancialMovement(BaseModel):`

### Debugging
- **debugpy:** (Python debugger)
  - Evidence: `backend/requirements.txt:3` — `debugpy`
  - Runtime: `backend/Dockerfile:12` — `python -m debugpy --listen 0.0.0.0:5678` before uvicorn
  - Purpose: Connect VSCode debugger on port 5678

### Testing
- **pytest:** Latest
  - Evidence: `backend/requirements.txt:4` — `pytest`
  - Config: `backend/tests/conftest.py` (default, minimal)
  - Tests: `backend/tests/test_routes.py` (18+ test cases)
  - Run: `pytest` (or `docker compose exec backend pytest`)

- **pytest-cov:** Latest (Coverage reporting)
  - Evidence: `backend/requirements.txt:5` — `pytest-cov`
  - Run: `pytest --cov` for coverage report

### HTTP Client
- **httpx:** Latest
  - Evidence: `backend/requirements.txt:6` — `httpx`
  - Usage: FastAPI TestClient uses httpx under the hood
  - Ref: `backend/tests/test_routes.py:9` — `from fastapi.testclient import TestClient`

---

## Infrastructure & DevOps

### Containerization
- **Docker:** Latest
  - Frontend image: `frontend/Dockerfile:1` — `FROM node:24-alpine`
  - Backend image: `backend/Dockerfile:1` — `FROM python:3.13-slim`

- **Docker Compose:** Latest
  - Orchestration: `docker-compose.yml:1-23`
  - Services: `frontend`, `backend`
  - Network: Default bridge (automatic)
  - Volumes: Code mounting for hot reload
  - Dependencies: `depends_on: [backend]` ensures startup order

### Environment Management
- **.env.example:** Exists
  - Evidence: `frontend/.env.example:1-3`
  - Variables: `VITE_API_BASE_URL` (optional, frontend API target)
  - Default behavior: Empty → uses Vite proxy

### Build Pipeline
- **Frontend Build:**
  - Type check: `tsc -b` (TypeScript compiler)
  - Bundle: `vite build` (Vite bundler)
  - Evidence: `frontend/package.json:8` — `"build": "tsc -b && vite build"`

- **Backend Build:**
  - No compile step (Python is interpreted)
  - Dependencies installed at image build time
  - Evidence: `backend/Dockerfile:6` — `RUN pip install --no-cache-dir -r requirements.txt`

---

## Development Tools

| Tool | Version | Purpose | Evidence |
|------|---------|---------|----------|
| autoprefixer | 10.4.27 | CSS vendor prefixes | frontend/package.json:32 |
| postcss | 8.5.9 | CSS processing | frontend/package.json:37 |
| @vitejs/plugin-react | 6.0.1 | React fast refresh | frontend/package.json:30 |
| @tailwindcss/vite | 4.2.2 | Tailwind integration | frontend/package.json:26 |
| eslint-plugin-react-hooks | 7.0.1 | React Hooks linting | frontend/package.json:35 |
| eslint-plugin-react-refresh | 0.5.2 | Vite refresh linting | frontend/package.json:34 |
| typescript-eslint | 8.58.0 | TypeScript linting | frontend/package.json:40 |

---

## Port Mapping

| Service | Port | Container | Usage |
|---------|------|-----------|-------|
| Frontend | 5173 | `node:24-alpine` | Vite dev server (HMR) |
| Backend | 8000 | `python:3.13-slim` | FastAPI + Uvicorn |
| Debugger | 5678 | `python:3.13-slim` | Python debugpy (VSCode) |

Evidence: `docker-compose.yml:7,19-20` — Port mappings

---

## Network & Proxy

- **Frontend-Backend Communication:**
  - Method: Vite proxy
  - Config: `frontend/vite.config.ts:11-16`
  - Rule: `/api/*` → `http://backend:8000`
  - Why: Avoids CORS in dev, enables seamless Docker networking

- **CORS Configuration:**
  - Dev: `allow_origins=["*"]` (all origins)
  - Prod: Should be restricted (not yet implemented)
  - Evidence: `backend/app/main.py:7-12`

---

## External Dependencies (None Currently)

- No real database (uses mock data with `seed=42`)
- No external APIs called
- No third-party authentication (OAuth, JWT)
- No background job queue
- No message broker (Kafka, RabbitMQ)

---

## Dependency Security

- **Node.js:** Alpine image (small, secure base)
- **Python:** Slim image (minimal libraries)
- **Lock Files:** 
  - Frontend: `frontend/package-lock.json` (npm lock)
  - Backend: N/A (requirements.txt is fixed)
- **Updates:** No auto-updates; versions pinned in package.json/requirements.txt

---

## Missing/Planned Dependencies

Based on current limitations and best practices:

- **Database:** PostgreSQL, MongoDB, or similar (for persistent data)
- **ORM:** SQLAlchemy (if using SQL DB)
- **Authentication:** JWT, OAuth, or similar
- **Caching:** Redis (for performance)
- **Logging:** Structured logging (Pydantic, Structlog)
- **Monitoring:** Prometheus, Sentry (error tracking)
- **API Documentation:** Auto-Swagger (already there via FastAPI)

---

**This document describes the technology in use. Update when adding new tools or frameworks.**

Last verified: 2026-09-30 by Claude Haiku 4.5
