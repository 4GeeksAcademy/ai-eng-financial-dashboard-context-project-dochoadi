# Product Overview

**Last Updated:** 2026-09-30  
**Source:** VERIFICATION.md + Code Inspection

---

## What Does This Application Do?

Financial Metrics Dashboard is a **full-stack web application** that visualizes and analyzes business financial metrics in real-time.

**Evidence:** `README.md:1,18` — "Financial metrics dashboard with a React + TypeScript frontend and a FastAPI backend."

### Core Purpose

The app aggregates financial transactions (income and expenses) and displays:
- Key Performance Indicators (KPIs)
- Trend analysis (monthly income/expense comparison)
- Anomaly detection in spending patterns
- Business segment breakdown (B2B vs B2C)

**Evidence:** `backend/app/routes.py:243-391` — 9 endpoints handle metrics aggregation, summary, comparisons, and alerts.

---

## Core Features

### 1. **KPI Dashboard**
- Displays 4 metrics: Total Income, Total Outcome, Profit, Profit Margin %
- Evidence: `frontend/src/components/dashboard/kpi-row.tsx:1-50` — Renders 4 KPICard components
- Evidence: `frontend/src/App.tsx:57-59` — KPIRow component in main dashboard

### 2. **Trend Analysis Charts**
- Income vs Outcome bar chart (monthly)
- Profit % line chart (monthly trend)
- Evidence: `frontend/src/components/dashboard/income-outcome-chart.tsx` — BarChart component
- Evidence: `frontend/src/components/dashboard/profit-percent-chart.tsx` — LineChart component

### 3. **Financial Filtering**
- Filter by date range (start_date, end_date)
- Filter by operation type (income vs outcome)
- Filter by business segment (B2B vs B2C)
- Filter by category (suppliers, sales, operational, administrative, others)
- Evidence: `backend/app/routes.py:125-143` — filter_movements() function

### 4. **Comparison & Analysis**
- Period-to-period comparison (current vs previous)
- Top-spending category analysis
- Evidence: `backend/app/routes.py:305-340` — get_metrics_comparison() endpoint
- Evidence: `backend/app/routes.py:287-303` — get_top_categories() endpoint

### 5. **Anomaly Detection**
- Detect unusual spending spikes
- Threshold-based alerts (default: 30% above baseline)
- Evidence: `backend/app/routes.py:342-360` — get_metrics_alerts() endpoint

---

## Data Model

### Core Entity: FinancialMovement

Represents a single financial transaction.

**Evidence:** `frontend/src/lib/financial-types.ts:5-10` & `backend/app/routes.py:22-28`

```typescript
{
  create_date: string        // ISO date (YYYY-MM-DD)
  amount: number             // USD, e.g., 5234.67
  operation_type: 'income' | 'outcome'
  category: 'suppliers' | 'sales' | 'operational' | 'administrative' | 'others'
  business_type: 'B2B' | 'B2C'
}
```

### Data Source

**Current State:** Mock data (not real database)

- Evidence: `backend/app/routes.py:94-104` — `generate_mock_movements(seed=42)` generates 360 synthetic transactions
- Coverage: 12 months (2025-09-02 to 2026-08-28)
- Seeded: `seed=42` ensures reproducibility

**Future State:** Database integration needed (see current-state.md)

---

## User Flows

### 1. View Dashboard (Default)
```
User opens frontend → React App mounts
  ↓
App.tsx useEffect runs → fetch /api/metrics
  ↓
Vite proxy routes to backend:8000
  ↓
Backend generates mock data (360 movements)
  ↓
Frontend computes KPIs (totalIncome, totalOutcome, profit, margin%)
  ↓
Charts render (Income vs Outcome, Profit %)
  ↓
User sees full dashboard
```

Evidence: `frontend/src/App.tsx:29-43` — entire fetch→compute→render flow

### 2. Apply Filters (Future)
```
User selects date range, category, business type
  ↓
Frontend query params sent to backend
  ↓
Backend filters movements by selected criteria
  ↓
Results recomputed and returned
  ↓
Charts update with filtered data
```

Evidence: `backend/app/routes.py:248-260` — all endpoints support optional filters

---

## Key Entry Points

| Component | Path | Purpose |
|-----------|------|---------|
| **Frontend Root** | `frontend/src/main.tsx:1-10` | React DOM mount point |
| **Frontend App** | `frontend/src/App.tsx:1-74` | Main dashboard component, data fetching |
| **Backend App** | `backend/app/main.py:1-14` | FastAPI initialization, middleware, router |
| **Backend Routes** | `backend/app/routes.py:19+` | 9 API endpoints |
| **Data Generators** | `backend/app/routes.py:94-104` | Mock movement generator |

---

## API Endpoints (Backend)

All endpoints return JSON. Base path: `/api` (proxied from frontend).

| Method | Path | Purpose | Evidence |
|--------|------|---------|----------|
| GET | `/health` | Health check | routes.py:243-245 |
| GET | `/api/metrics` | Get filtered movements | routes.py:248-260 |
| GET | `/api/metrics/facets` | Get available filters | routes.py:262-266 |
| GET | `/api/metrics/summary` | Get aggregated data by period | routes.py:268-285 |
| GET | `/api/metrics/categories/top` | Get top-spending categories | routes.py:287-303 |
| GET | `/api/metrics/comparison` | Compare periods | routes.py:305-340 |
| GET | `/api/metrics/alerts` | Detect anomalies | routes.py:342-360 |
| GET | `/api/metrics/b2b` | Filter B2B only | routes.py:362-375 |
| GET | `/api/metrics/b2c` | Filter B2C only | routes.py:378-391 |

---

## Current Limitations

### No Real Database
- Evidence: `backend/app/routes.py:94-96` — All endpoints call `generate_mock_movements(seed=42)`
- Impact: Data is synthetic, not persistent

### No Authentication
- Evidence: `backend/app/main.py:7-12` — CORS allows all origins, no auth middleware
- Impact: Anyone can access the API

### No User Accounts
- Dashboard doesn't track user preferences
- No saved filters or reports

### No Real-Time Updates
- Frontend fetches once on mount
- No WebSocket or polling for live updates

---

## Success Metrics (How Is It Doing?)

✅ **Working:** 
- Mock data generation with deterministic seed
- All 9 API endpoints responding correctly
- Frontend fetches and displays data without errors
- Charts render correctly

❌ **Missing:**
- Persistent data storage
- User authentication
- Real-time data
- Advanced filtering UI

Evidence: See `VERIFICATION.md` (Phase 1 verification results) and `PHASE3_SUMMARY.md` (validation status).

---

## How It Works End-to-End

1. **User opens browser** → `http://localhost:5173`
2. **Browser loads React app** (via Vite dev server)
3. **App mounts** → useEffect triggers
4. **Fetch request** → `GET /api/metrics` (relative path)
5. **Vite proxy intercepts** → Routes to `http://backend:8000/api/metrics`
6. **Backend generates data** → 360 synthetic FinancialMovement objects
7. **Frontend processes** → Computes KPIs, groups data by month
8. **React renders** → 4 KPI cards + 2 charts displayed
9. **User sees dashboard** ✅

Evidence: `frontend/src/App.tsx:13-43` (full client-side), `backend/app/routes.py:248-260` (server-side)

---

**This document is the source of truth for what the application does. Update it when adding features.**

Last verified: 2026-09-30 by Claude Haiku 4.5
