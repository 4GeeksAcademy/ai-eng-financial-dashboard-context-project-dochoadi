# CODE QUALITY & SECURITY RULES
**Alcance:** Error handling, CORS, data reproducibility

---

## RULE: Frontend Error Handling Must Be Visible to User

**Alcance:** Frontend error handling UX

**Justificación:**
Silent errors confuse users. Loading state stuck forever is bad UX. Try/catch/finally ensures user sees errors, knows what failed, and UI doesn't hang.

**Evidencia:**
- `frontend/src/App.tsx:29-43` - useEffect with try/catch/finally pattern
- Error message in Spanish: "No se pudo cargar la informacion financiera..."
- Loading state always cleared (finally block)

**Guía Accionable:**
1. Pattern: `.then()` → setState(data), `.catch()` → setError(userMessage), `.finally()` → setLoading(false)
2. Error message: User-friendly (no tech jargon), in user's language
3. Error visibility: Render error in UI (show error boundary or error message)
4. Logging: Add console.error() for debugging, but ALSO show UI message
5. Loading state: ALWAYS clear in finally block (even if error)

**Ejemplo Correcto:**
```typescript
useEffect(() => {
  fetchData()
    .then(data => setMetrics(computeKPIs(data)))
    .catch(err => {
      console.error(err);  // Log for debugging
      setError("Descripción clara que entienda el usuario");
    })
    .finally(() => setLoading(false))  // Clear loading always
}, [])
```

**Ejemplo Incorrecto:**
```typescript
// ❌ WRONG - No error handling
useEffect(() => {
  fetchData().then(data => setMetrics(computeKPIs(data)));
}, [])

// ❌ WRONG - Silent error, stuck loading
useEffect(() => {
  fetchData()
    .catch(err => console.error(err))  // Logs but doesn't show UI
    .finally(() => setLoading(false))  // Still runs but user doesn't know what failed
}, [])

// ❌ WRONG - Technical error message
setError("TypeError: Cannot read property 'create_date' of undefined")
```

---

## RULE: Backend Error Handling via HTTPException

**Alcance:** Backend error responses

**Justificación:**
FastAPI + Pydantic automatically validate. Explicit HTTPException provides clear error codes and messages. Avoid hidden errors.

**Evidencia:**
- `backend/app/main.py:7-12` - CORS configured (no try/except hiding errors)
- `backend/app/routes.py` - Relies on Pydantic validation (no silent failures)
- Tests expect specific HTTP status codes

**Guía Accionable:**
1. Validation: Trust Pydantic models (they validate automatically)
2. Bad input: Pydantic returns 422 automatically (invalid request)
3. Logic error: Raise HTTPException(status_code=400, detail="message")
4. Never silently catch: `try/except: pass` hides bugs
5. Error message: Clear, actionable (not tech-heavy)

**Ejemplo Correcto:**
```python
@router.get("/api/metrics")
def get_metrics(start_date: date | None = None):
    # Pydantic validates date format automatically
    # If date is invalid, returns 422
    movements = generate_mock_movements(seed=42)
    if not movements:
        raise HTTPException(status_code=400, detail="No data available")
    return movements
```

**Ejemplo Incorrecto:**
```python
# ❌ WRONG - Silencing errors
try:
    movements = generate_mock_movements(seed=42)
except Exception:
    pass  # Error hidden, user gets empty response

# ❌ WRONG - Custom exception (FastAPI doesn't know how to handle)
def get_metrics():
    raise ValueError("Invalid data")
```

---

## RULE: Deterministic Seed for Reproducible Tests

**Alcance:** Test data generation

**Justificação:**
Tests must be reproducible (same input = same output). Random data = flaky tests in CI. Seed ensures consistency.

**Evidencia:**
- `backend/app/routes.py:96` - `def generate_mock_movements(seed: int | None = None)`
- `backend/tests/test_routes.py:13` - `movements = generate_mock_movements(seed=42)`
- All tests use seed=42 consistently

**Guía Accionable:**
1. Mock data function: Accept optional seed parameter
2. Tests: ALWAYS call with specific seed (e.g., `seed=42`)
3. CI/CD: Same seed ensures identical test data across runs
4. Never: Call mock data without seed in tests
5. Production: API can omit seed (uses random data for demo, that's OK)

**Ejemplo Correcto:**
```python
# Production: Random data OK (demo dashboard)
movements = generate_mock_movements()  # No seed, truly random

# Tests: Reproducible data
movements = generate_mock_movements(seed=42)  # Same seed always
assert len(movements) == 360  # Deterministic result
```

**Ejemplo Incorrecto:**
```python
# ❌ WRONG - Tests use random data
movements = generate_mock_movements()  # Random seed = flaky test
assert len(movements) == 360  # Might fail sometimes

# ❌ WRONG - Different seed per test run
import random
seed = random.randint(1, 1000)
movements = generate_mock_movements(seed=seed)  # Flaky!
```

---

## RULE: CORS Configuration: Dev vs Production

**Alcance:** API security

**Justificación:**
Dev uses `allow_origins=["*"]` for convenience. Production must restrict origins. Prevents unauthorized API access.

**Evidencia:**
- `backend/app/main.py:7-12` - Current config allows all origins
- Comment in RULES.md: "CORS abierto en dev, restrictivo en prod (TODO)"

**Guía Accionable:**
1. Development: `allow_origins=["*"]` (current state - OK for dev)
2. Production: `allow_origins=["https://your-domain.com"]` (restrict to your frontend)
3. Future: Implement env var to switch configs
4. Never: Leave `allow_origins=["*"]` in production
5. Testing: Verify CORS headers in requests

**Ejemplo Correcto (Production):**
```python
# backend/app/main.py (production)
import os

ALLOWED_ORIGINS = os.getenv("ALLOWED_ORIGINS", "http://localhost:5173").split(",")

app.add_middleware(
    CORSMiddleware,
    allow_origins=ALLOWED_ORIGINS,
    allow_methods=["GET", "POST", "PUT"],
    allow_headers=["Content-Type"],
)
```

**Ejemplo Incorrecto:**
```python
# ❌ WRONG - Leaving dev config in production
allow_origins=["*"]  # Exposes API to any origin!
```

