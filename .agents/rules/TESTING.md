# TESTING RULES
**Alcance:** Frontend (Vitest) + Backend (pytest)

---

## RULE: Test File Placement by Language

**Alcance:** Test organization

**Justificación:**
Frontend tests are found easily if colocated with source. Backend tests in dedicated directory keeps build clean. Both ensure CI discovers tests without extra config.

**Evidencia:**
- Frontend: `frontend/src/lib/financial-utils.test.ts` (same dir as financial-utils.ts)
- Backend: `backend/tests/test_routes.py` (dedicated tests/ directory)

**Guía Accionable:**

**Frontend:**
1. Test file location: SAME directory as source file
2. Naming: `filename.test.ts` (e.g., `financial-utils.ts` → `financial-utils.test.ts`)
3. Framework: Vitest (package.json:12)
4. Run: `npm test` or `npm test:watch`

**Backend:**
1. Test file location: `backend/tests/test_<module>.py`
2. Naming: `test_<module>.py` (e.g., `test_routes.py`)
3. Framework: pytest (requirements.txt)
4. Run: `pytest` or `docker compose exec backend pytest`

**Ejemplo Correcto (Frontend):**
```
frontend/src/lib/
├── financial-utils.ts
└── financial-utils.test.ts  ✅ Same directory
```

**Ejemplo Incorrecto (Frontend):**
```
frontend/
├── __tests__/financial-utils.test.ts  ❌ Separate __tests__ directory
frontend/src/lib/
├── financial-utils.ts
├── __tests__/financial-utils.test.ts  ❌ Nested __tests__
```

---

## RULE: Tests Import Real Code, Never Mocks

**Alcance:** Test methodology

**Justificación:**
Tests must reflect production. Mocks hide bugs (test passes, production fails). Real code + deterministic seed ensures reproducible, reliable tests.

**Evidencia:**
- `backend/tests/test_routes.py:5-6` - Imports real app and functions
- No `/mocks` or `/fixtures` directory
- Tests use `TestClient(app)` (real app, not mock)

**Guía Accionable:**
1. Import real code: `from app.main import app`, `from app.routes import function_name`
2. Never mock internal functions: test the real behavior
3. Use TestClient for FastAPI: `TestClient(app)`
4. Use deterministic seed: `generate_mock_movements(seed=42)`
5. If fixture needed: use conftest.py (pytest) or vitest setup

**Ejemplo Correcto:**
```python
# ✅ Correct
from app.main import app
from app.routes import generate_mock_movements

client = TestClient(app)

def test_generate_mock_movements():
    movements = generate_mock_movements(seed=42)
    assert len(movements) == 360
```

**Ejemplo Incorrecto:**
```python
# ❌ WRONG - Mocking function
from unittest.mock import patch

@patch('app.routes.generate_mock_movements')
def test_something(mock_gen):
    mock_gen.return_value = []  # Fake data, not real logic!
    # This test LIES about real behavior
```

---

## RULE: Test Naming Pattern

**Alcance:** Test function naming

**Justificación:**
Clear names reveal intent. When test fails in CI, name tells you exactly what broke. `test_1()` doesn't help; `test_health_endpoint_returns_ok()` does.

**Evidencia:**
- `backend/tests/test_routes.py:12` - `test_generate_mock_movements_returns_full_year_sorted_data()`
- `backend/tests/test_routes.py:19` - `test_filter_movements_by_date_includes_range_edges()`
- `backend/tests/test_routes.py:29` - `test_health_endpoint_returns_ok()`

**Guía Accionable:**
1. Pattern: `test_<function>_<scenario>` or `test_<feature>_<expected_result>`
2. Must be descriptive: `test_kpi_row_loads` ❌ (unclear) vs `test_kpi_row_renders_four_cards` ✅
3. No abbreviations: `test_mgnr` ❌ vs `test_metrics_generator` ✅
4. Use underscores, not camelCase

**Ejemplo Correcto:**
```python
def test_filter_movements_by_date_includes_range_edges():
def test_metrics_summary_groups_by_month():
def test_alerts_detects_outliers_above_threshold():
```

**Ejemplo Incorrecto:**
```python
def test_filter():  # ❌ Too vague
def test_1():  # ❌ No meaning
def testFilterByDate():  # ❌ camelCase, not PEP 8
def test_filter_movements_by_date():  # ❌ Missing expected result
```

