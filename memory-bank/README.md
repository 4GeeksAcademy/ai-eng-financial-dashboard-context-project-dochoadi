# Memory Bank Index

**Purpose:** Permanent knowledge base for the Financial Metrics Dashboard project.  
**Audience:** Future developers, agents, and project maintainers.  
**Updates:** When features are added, architecture changes, or new constraints discovered.

---

## Core Documents

### 1. **product-overview.md** — What This Project Does
- **Read this if:** You're new to the project, need to understand its purpose
- **Contains:** 
  - What the app does (dashboard of financial metrics)
  - Core features (KPIs, charts, filtering, alerts)
  - Data model (FinancialMovement entity)
  - User flows (how dashboard works)
  - API endpoints (9 total)
  - Current limitations (mock data, no auth, no real-time)
  - Success metrics (what's working)

### 2. **tech-stack.md** — Technologies in Use
- **Read this if:** You need to set up dev environment, add dependencies, or understand architecture
- **Contains:**
  - Languages & runtimes (TypeScript 6.0.2, Python 3.13, Node 24-alpine)
  - Frontend stack (React 19, Vite 8.0, Tailwind 4.2, Recharts 3.8)
  - Backend stack (FastAPI, Uvicorn, Pydantic)
  - Testing tools (Vitest, pytest)
  - Infrastructure (Docker Compose, Vite proxy)
  - Port mapping (5173, 8000, 5678)
  - External dependencies (currently: none)
  - Missing dependencies for production (DB, auth, caching, etc.)

### 3. **current-state.md** — What Works, What's Missing, What's Next
- **Read this if:** You're planning features, debugging, or assessing project maturity
- **Contains:**
  - What's working ✅ (all 9 endpoints, React app, Docker setup)
  - Known gaps ❌ (no DB, no auth, no real-time)
  - Priorities by impact (critical, high, medium)
  - Testing status (18+ backend tests, minimal frontend tests)
  - Code quality (TypeScript, ESLint, documentation)
  - Performance baseline (no bottlenecks yet, no optimization)
  - Deployment status (dev works, no staging/prod)
  - Next steps (DB → Auth → UI → Real-time)

---

## How to Use This Memory Bank

### For New Developers
1. Read **product-overview.md** (what does this do?)
2. Read **tech-stack.md** (what are the tools?)
3. Read **current-state.md** (what's done, what's missing?)
4. Clone repo, run `docker compose up --build`
5. Check `.agents/rules/` for coding conventions

### For Feature Planning
1. Check **current-state.md** → "Priorities by Impact"
2. Verify current limitations (DB? Auth? Real-time?)
3. Read **product-overview.md** → "API Endpoints" to understand data flow
4. Plan your feature against existing constraints

### For Bug Triage
1. Check **product-overview.md** → "User Flows" (where could it break?)
2. Check **current-state.md** → "Known Gaps" (is it a known limitation?)
3. Check **tech-stack.md** → version numbers (version-specific bug?)
4. Read VERIFICATION.md (Phase 1) for test results

### For Deployment
1. Check **current-state.md** → "Deployment Status"
2. Check **tech-stack.md** → "Containerization" (Docker requirements)
3. Implement missing from **current-state.md** → "Critical" (DB + Auth)
4. Set up CI/CD pipeline

---

## Files This Memory Bank References

| Reference | File | Purpose |
|-----------|------|---------|
| Product details | `frontend/src/App.tsx` | Main React component |
| | `backend/app/main.py` | FastAPI entry point |
| | `README.md` | Project description |
| Technical specs | `package.json` | Frontend dependencies |
| | `requirements.txt` | Backend dependencies |
| | `docker-compose.yml` | Infrastructure |
| | `Dockerfile` (2x) | Container images |
| Conventions | `.agents/rules/` (6 files) | Code rules & patterns |
| Verification | `VERIFICATION.md` | Architecture verification (Phase 1) |
| | `PHASE2_SUMMARY.md` | Rules extraction (Phase 2) |
| | `PHASE3_SUMMARY.md` | Rules implementation (Phase 3) |

---

## Update Schedule

- **Every sprint:** Update **current-state.md** with new gaps/priorities
- **When adding tech:** Update **tech-stack.md** version numbers
- **When changing features:** Update **product-overview.md** APIs
- **Never delete:** These documents are knowledge, not logs

---

## Contact Points for Questions

| Question | Answer Location |
|----------|-----------------|
| "What does this app do?" | product-overview.md |
| "How do I run it locally?" | tech-stack.md → Infrastructure |
| "What's broken or missing?" | current-state.md → Known Gaps |
| "How should I name files/functions?" | .agents/rules/NAMING.md |
| "What's the API?" | product-overview.md → API Endpoints |
| "Is feature X built?" | current-state.md → What's Working / Known Gaps |
| "What's the tech stack?" | tech-stack.md |

---

**This memory bank was created during Phase 4 of project development (2026-09-30).**  
**Scope:** Financial Metrics Dashboard — Full-stack React + FastAPI application.  
**Status:** Fully operational, ready for feature development.  
**Next:** Database integration, authentication, real-time updates.

Last verified: 2026-09-30 by Claude Haiku 4.5
