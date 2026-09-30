# NAMING CONVENTIONS RULES
**Alcance:** All code files (Frontend TypeScript, Backend Python)

---

## RULE: Kebab-Case for React Component Files

**Alcance:** Frontend React files

**Justificación:**
Kebab-case ensures consistency, works on Linux/Mac/Windows (case-sensitive filesystems), and matches industry convention for frontend assets. Prevents import confusion (`from ./kpi-card` vs `from ./KPICard`).

**Evidencia:**
- `frontend/src/components/dashboard/dashboard-header.tsx` (not DashboardHeader.tsx)
- `frontend/src/components/dashboard/income-outcome-chart.tsx` (not IncomeOutcomeChart.tsx)
- `frontend/src/components/ui/card.tsx` (not Card.tsx)
- All 10+ files in `frontend/src/components/` follow this pattern

**Guía Accionable:**
1. Component files: ALWAYS use `kebab-case.tsx` (not PascalCase)
2. Utility files: ALWAYS use `kebab-case.ts` (not camelCase)
3. Test files: ALWAYS use `kebab-case.test.ts`
4. Examples: `dashboard-header.tsx`, `financial-utils.ts`, `kpi-row.test.ts`
5. Exception: NONE. Even single-word components use kebab: `card.tsx` not `Card.tsx`

**Ejemplo Correcto:**
```typescript
// ✅ Correct
export function DashboardHeader() { ... }
export default function IcomeOutcomeChart() { ... }

// File: frontend/src/components/dashboard/dashboard-header.tsx
// Import: import { DashboardHeader } from '@/components/dashboard/dashboard-header'
```

**Ejemplo Incorrecto:**
```typescript
// ❌ WRONG - PascalCase filename
// File: frontend/src/components/dashboard/DashboardHeader.tsx

// ❌ WRONG - camelCase filename
// File: frontend/src/components/dashboard/dashboardHeader.tsx

// ❌ WRONG - snake_case filename
// File: frontend/src/components/dashboard/dashboard_header.tsx
```

---

## RULE: Snake_Case for Python Functions

**Alcance:** Backend Python files

**Justificación:**
PEP 8 standard for Python. Linters (pylint, flake8) enforce it. Ensures consistency across the Python ecosystem.

**Evidencia:**
- `backend/app/routes.py:71` - `def _build_movement(...)`
- `backend/app/routes.py:94` - `def generate_mock_movements(...)`
- `backend/app/routes.py:125` - `def filter_movements(...)`
- `backend/app/routes.py:150` - `def build_metrics_facets(...)`

**Guía Accionable:**
1. Function names: ALWAYS use `snake_case` (PEP 8)
2. Private helpers: Prefix with `_` → `def _helper_function()`
3. Class names: ALWAYS use `PascalCase` (Python convention for classes)
4. Variable names: ALWAYS use `snake_case`
5. Constants: Use `UPPER_SNAKE_CASE` if needed

**Ejemplo Correcto:**
```python
# ✅ Correct
def generate_mock_movements(seed: int = None):
    pass

def _build_movement(month: int, income_probability: float):
    pass

class FinancialMovement(BaseModel):
    pass
```

**Ejemplo Incorrecto:**
```python
# ❌ WRONG - camelCase
def generateMockMovements(seed: int = None):
    pass

# ❌ WRONG - PascalCase
def GenerateMockMovements(seed: int = None):
    pass

# ❌ WRONG - No underscore prefix for private
def build_movement():
    pass
```

---

## RULE: PascalCase for TypeScript Types and Interfaces

**Alcance:** Frontend TypeScript type definitions

**Justificación:**
TypeScript convention. Swagger documentation uses class/interface names, so PascalCase ensures clean API docs. Differentiates types from variable names.

**Evidencia:**
- `frontend/src/lib/financial-types.ts:1` - `export type OperationType = ...`
- `frontend/src/lib/financial-types.ts:5` - `export interface FinancialMovement { ... }`
- `frontend/src/lib/financial-types.ts:13` - `export interface KPIMetrics { ... }`

**Guía Accionable:**
1. Type names: ALWAYS use `PascalCase` → `type OperationType = ...`
2. Interface names: ALWAYS use `PascalCase` → `interface FinancialMovement { ... }`
3. Variable names (values): Use `camelCase` → `const myVar: FinancialMovement = ...`
4. Never mix: Type is `FinancialMovement`, var is `financialMovement`

**Ejemplo Correcto:**
```typescript
// ✅ Correct
export type OperationType = 'income' | 'outcome'

export interface FinancialMovement {
  create_date: string
  amount: number
}

const movement: FinancialMovement = { ... }
```

**Ejemplo Incorrecto:**
```typescript
// ❌ WRONG - camelCase type
export type operationType = 'income' | 'outcome'

// ❌ WRONG - snake_case interface
export interface financial_movement { ... }

// ❌ WRONG - Using type name as variable
const FinancialMovement: FinancialMovement = { ... }
```

---

## RULE: PascalCase for Pydantic Models

**Alcance:** Backend Pydantic data models

**Justificación:**
Python convention for classes. Pydantic models ARE classes. Field names use snake_case for JSON API compatibility.

**Evidencia:**
- `backend/app/routes.py:22` - `class FinancialMovement(BaseModel):`
- `backend/app/routes.py:30` - `class MetricsFacets(BaseModel):`
- `backend/app/routes.py:38` - `class MetricsSummaryItem(BaseModel):`

**Guía Accionable:**
1. Model class names: ALWAYS `PascalCase` → `class MetricsComparison(BaseModel):`
2. Field names inside: ALWAYS `snake_case` → `delta_abs: float`
3. Reason: JSON serialization converts snake_case fields (API respects snake_case)
4. Never mix: Class is `MetricsComparison`, field is `delta_abs`

**Ejemplo Correcto:**
```python
# ✅ Correct
class MetricsComparison(BaseModel):
    current_period: float
    previous_period: float
    delta_abs: float
    delta_pct: float | None
```

**Ejemplo Incorrecto:**
```python
# ❌ WRONG - camelCase class name
class metricsComparison(BaseModel):
    pass

# ❌ WRONG - PascalCase fields
class MetricsComparison(BaseModel):
    CurrentPeriod: float
    DeltaAbs: float
```

