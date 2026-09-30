# 📋 REGLAS DE INGENIERÍA - Financial Metrics Dashboard

**Versión:** 1.0  
**Fecha:** 2026-09-30  
**Aplicable a:** Agents, Contribuidores, Desarrolladores

---

## 🏗️ ARQUITECTURA

### Hallazgo 1: Separación Frontend/Backend con Vite Proxy
**Evidencia:**
- `frontend/vite.config.ts:12-16` - Proxy configuration
- `docker-compose.yml:11-15` - Containers separated
- `frontend/src/App.tsx:13-16` - API consumption via relative path

**Patrón Observado:**
Frontend hace fetch a `/api/metrics` (ruta relativa), y Vite proxy automáticamente lo redirige a `http://backend:8000/api/metrics`. No hay hardcoding de URLs.

**Riesgo:**
Si un agent hardcodea URLs o ignora el proxy, romperá:
- Desarrollo local (localhost no tiene backend en 5173)
- Docker Compose (frontend container no puede resolver `localhost:8000`)
- Testing en diferentes entornos

**Regla Propuesta:**
```
REGLA: Proxy-First API Communication
- Todos los fetch() a /api/* desde frontend DEBEN ser relativos (sin dominio)
- Nunca hardcodear localhost, http://, o dominio en URLs internas
- El proxy Vite reescribe /api → http://backend:8000 automáticamente
- Excepción: Variables de entorno VITE_API_BASE_URL (si falta, usar "")
  Ref: frontend/vite.config.ts línea 10, App.tsx línea 13
```

---

### Hallazgo 2: Backend como Módulo Importable
**Evidencia:**
- `backend/app/__init__.py` - Package marker (vacío)
- `backend/app/main.py:1-4` - Importa router desde routes
- `backend/app/main.py:14` - app.include_router(router)
- `backend/tests/test_routes.py:5-6` - Imports app directly

**Patrón Observado:**
Backend NO usa un único archivo monolítico. Routes está separado, main solo orquesta. Tests importan `from app.main import app` sin montar servidor.

**Riesgo:**
Si un agent agrega funcionalidad directamente a `main.py` en lugar de crear rutas en `routes.py`, los tests no podrán acceder fácilmente y violará la separación.

**Regla Propuesta:**
```
REGLA: Backend Modular - main.py es solo orquestador
- main.py: SOLO define FastAPI app, middleware (CORS), e incluye router
- routes.py: TODOS los @router.get(), @router.post(), etc.
- main.py NO debe contener lógica de negocio ni endpoints
- Nueva funcionalidad → crear función auxiliar + decorador en routes.py
- Tests siempre importan desde app.main, no crean nueva app
  Ref: backend/app/main.py líneas 1-14, backend/app/routes.py líneas 19+
```

---

## 📝 NAMING & CONVENCIÓN DE ARCHIVOS

### Hallazgo 1: Kebab-Case para Componentes React
**Evidencia:**
- `frontend/src/components/dashboard/dashboard-header.tsx`
- `frontend/src/components/dashboard/income-outcome-chart.tsx`
- `frontend/src/components/dashboard/kpi-card.tsx`
- `frontend/src/components/dashboard/kpi-row.tsx`
- `frontend/src/components/dashboard/profit-percent-chart.tsx`

**Patrón Observado:**
100% de archivos de componentes usan `kebab-case.tsx`. No hay CamelCase para archivos.

**Riesgo:**
Si agent usa `KpiCard.tsx` o `KpiCard.tsx`, violará convención. Además:
- Inconsistencia visual en git logs
- Problemas en sistemas case-sensitive (Linux) vs case-insensitive (macOS/Windows)
- Importes inconsistentes (`from ./kpi-card` vs `from ./KpiCard`)

**Regla Propuesta:**
```
REGLA: Kebab-Case para todos los archivos TypeScript/React
- Archivos componentes: dashboard-header.tsx (NO DashboardHeader.tsx)
- Archivos utilidades: financial-utils.ts (NO financialUtils.ts)
- Archivos tipos: financial-types.ts (NO financialTypes.ts)
Excepción: SOLO tsconfig.json, vite.config.ts (tools, sin componentes)
  Ref: frontend/src/components/dashboard/ - todos los archivos
  Ref: frontend/src/lib/financial-*.ts - todos los archivos
```

---

### Hallazgo 2: Snake_Case para Funciones Python
**Evidencia:**
- `backend/app/routes.py:71` - `def _build_movement(...)`
- `backend/app/routes.py:94` - `def generate_mock_movements(...)`
- `backend/app/routes.py:125` - `def filter_movements(...)`
- `backend/app/routes.py:150` - `def build_metrics_facets(...)`

**Patrón Observado:**
100% de funciones Python usan `snake_case`. Private helpers tienen prefijo `_`.

**Riesgo:**
Si agent usa `generateMockMovements()` o `GenerateMockMovements`, PEP8 falla y linterna fallará (si hay uno configurado).

**Regla Propuesta:**
```
REGLA: Snake_Case para funciones Python
- Funciones públicas: def generate_mock_movements(...)
- Funciones privadas (helpers): def _build_movement(...)
- Clases: FinancialMovement (PascalCase para classes, normal en Python)
- Constantes: OUTCOME_CATEGORIES = [...] (si las hay)
Sigue PEP 8: https://www.python.org/dev/peps/pep-0008/#naming-conventions
  Ref: backend/app/routes.py líneas 71, 94, 125, 150 (todas snake_case)
```

---

### Hallazgo 3: Pascal Case para TypeScript Types/Interfaces
**Evidencia:**
- `frontend/src/lib/financial-types.ts:1-25` - Types y interfaces
  - `export type OperationType = ...`
  - `export interface FinancialMovement { ... }`
  - `export interface KPIMetrics { ... }`

**Patrón Observado:**
Todos los types e interfaces usan PascalCase. También aplica a clases.

**Riesgo:**
Si agent usa `operationType` (camelCase) para type, violaría convención. TypeScript compilará pero es incorrecto semánticamente.

**Regla Propuesta:**
```
REGLA: PascalCase para Types e Interfaces TypeScript
- Type names: type OperationType = ...
- Interface names: interface FinancialMovement { ... }
- Class names (si las hay): class MyService { ... }
- Variable names (valores): camelCase → const myVar = ...
  Ref: frontend/src/lib/financial-types.ts líneas 1-25 (todos PascalCase)
```

---

### Hallazgo 4: Pydantic Models = Python Classes PascalCase
**Evidencia:**
- `backend/app/routes.py:22` - `class FinancialMovement(BaseModel):`
- `backend/app/routes.py:30` - `class MetricsFacets(BaseModel):`
- `backend/app/routes.py:38` - `class MetricsSummaryItem(BaseModel):`
- `backend/app/routes.py:51` - `class MetricsComparison(BaseModel):`

**Patrón Observado:**
Todos los Pydantic models usan PascalCase.

**Riesgo:**
Si agent crea `metrics_facets` en lugar de `MetricsFacets`, romperá:
- Convención del proyecto
- Documentación automática de Swagger (que usa nombre de clase)
- Type consistency

**Regla Propuesta:**
```
REGLA: PascalCase para Pydantic Models
- Modelo: class MetricsFacets(BaseModel):
- Campo dentro: operation_types: list[OperationType]
- Modelo = PascalCase, campo = snake_case (respeta JSON API)
  Ref: backend/app/routes.py líneas 22-62 (todos PascalCase)
```

---

## 🧪 TESTING

### Hallazgo 1: Test Colocación = Junto al Código
**Evidencia:**
- Frontend test: `frontend/src/lib/financial-utils.test.ts` (junto a financial-utils.ts)
- Backend tests: `backend/tests/test_routes.py` (directorio separado pero mismo nivel que app/)

**Patrón Observado:**
- Frontend: test files collocated (mismo directorio que source)
- Backend: test files en directorio `tests/` dedicado

**Riesgo:**
Si agent crea tests en lugar incorrecto:
- Frontend tests en `__tests__/` en lugar de colocados → CI no los encuentra
- Backend tests fuera de `backend/tests/` → descubierta fallará

**Regla Propuesta:**
```
REGLA: Colocación de Tests según lenguaje
Frontend:
  - Test files: junto al source (financial-utils.ts → financial-utils.test.ts)
  - Framework: Vitest (package.json:12 "test": "vitest run")
  - Patrón: describe(), it(), expect()
    Ref: frontend/src/lib/financial-utils.test.ts línea 1

Backend:
  - Test files: backend/tests/test_*.py (directorio dedicado)
  - Framework: pytest (requirements.txt)
  - Patrón: def test_*(), assert statements
    Ref: backend/tests/test_routes.py línea 1
```

---

### Hallazgo 2: Tests Importan Modelos Reales, No Mocks
**Evidencia:**
- `backend/tests/test_routes.py:5-6`
  ```python
  from app.main import app
  from app.routes import filter_movements_by_date, generate_mock_movements
  ```
- No hay carpeta `mocks/` o `fixtures/`
- Tests usan `TestClient(app)` real

**Patrón Observado:**
Tests prueban el código real, no mocks. Reutilizan `generate_mock_movements(seed=42)` para datos consistentes.

**Riesgo:**
Si agent crea tests con mocks/stubs, puede:
- Falsear resultados (test pasa pero código falla en producción)
- Aumentar mantenimiento (dos versiones de la lógica)
- Romper CI (test !== realidad)

**Regla Propuesta:**
```
REGLA: Tests Prueban Código Real, No Mocks
- Importar código real: from app.main import app
- Usar TestClient(app) en lugar de crear mock app
- Reutilizar generate_mock_movements(seed=42) para datos reproducibles
- Si necesitas fixture, usa conftest.py (pytest) o vitest setup
- Evitar mocking de funciones internas; testear comportamiento
  Ref: backend/tests/test_routes.py líneas 5-9 (imports reales)
  Ref: backend/app/routes.py:96 (seed=42 para reproducibilidad)
```

---

### Hallazgo 3: Test Naming = Patrón test_<function>_<scenario>
**Evidencia:**
- `backend/tests/test_routes.py:12` - `def test_generate_mock_movements_returns_full_year_sorted_data():`
- `backend/tests/test_routes.py:19` - `def test_filter_movements_by_date_includes_range_edges():`
- `backend/tests/test_routes.py:29` - `def test_health_endpoint_returns_ok():`

**Patrón Observado:**
Nombre = `test_<función>_<resultado/escenario>`. Muy explícito, no ambiguo.

**Riesgo:**
Si agent crea test con nombre vago como `test_movements()` o `test_1()`:
- Difícil saber qué testea
- Difícil debuggear si falla
- No refleja intención

**Regla Propuesta:**
```
REGLA: Test Naming = test_<function>_<scenario>
- test_generate_mock_movements_returns_full_year_sorted_data()
- test_health_endpoint_returns_ok()
- test_metrics_endpoint_filters_by_category()
Estructura: test_[WHAT]_[EXPECTED RESULT]
  Ref: backend/tests/test_routes.py líneas 12, 19, 29, 36+
```

---

## 📖 DOCUMENTACIÓN & DX

### Hallazgo 1: Archivo VERIFICATION.md = Single Source of Truth
**Evidencia:**
- `VERIFICATION.md` (742 líneas, existe en root)
- Contiene: arquitectura, verificación, APIs, stack, comandos
- README.md apunta a documentación pero no duplica

**Patrón Observado:**
Hay un documento maestro (VERIFICATION.md) que consolida TODO. No hay múltiples README/DOCS conflictivos.

**Riesgo:**
Si agent crea más archivos de documentación redundantes:
- Inconsistencia (X en README, Y en DOCS/architecture.md)
- Confusión para nuevos contribuidores
- Mantenimiento duplicado

**Regla Propuesta:**
```
REGLA: Single Documentation File - VERIFICATION.md
- VERIFICATION.md = documento maestro
- README.md = breve descripción + link a VERIFICATION.md
- No crear ARCHITECTURE.md, DESIGN.md, etc. paralelos
- Si necesitas nueva doc: agregar sección a VERIFICATION.md
- Versionar VERIFICATION.md en git
  Ref: VERIFICATION.md existe y es completo
  Ref: README.md línea 24 apunta a VERIFICATION.md implícitamente
```

---

### Hallazgo 2: Tipos TypeScript = Documentación Viva
**Evidencia:**
- `frontend/src/lib/financial-types.ts` - Interfaces con comentarios
  ```typescript
  export interface FinancialMovement {
    create_date: string // ISO date
    amount: number
    operation_type: OperationType
    category: Category
    business_type: BusinessType
  }
  ```

**Patrón Observado:**
Tipos documentan su propio contrato. No hay "tipos oscuros" sin comentarios.

**Riesgo:**
Si agent agrega tipo sin documentar:
- Confusión sobre qué es cada campo
- Errores en consumidores (frontend espera ISO date, backend da timestamp)

**Regla Propuesta:**
```
REGLA: Tipos = Documentación Embedded
- Cada interfaz/type tiene descripción de qué es
- Campos no-obvios tienen comentarios inline
- Valores de tipos literales se documentan (ej: 'income' | 'outcome')
- Backend (Pydantic models) también sigue patrón
  Ref: frontend/src/lib/financial-types.ts líneas 5-10
```

---

## 🔧 DEPENDENCIAS & CONFIGURACIÓN

### Hallazgo 1: Vite Proxy Centraliza Conexión Frontend-Backend
**Evidencia:**
- `frontend/vite.config.ts:10-16`
  ```typescript
  server: {
    host: "0.0.0.0",
    proxy: {
      "/api": {
        target: "http://backend:8000",
        changeOrigin: true,
      },
    },
  },
```

**Patrón Observado:**
Proxy es la ÚNICA forma de comunicación frontend-backend. Hardcoded en vite.config.ts, no en código.

**Riesgo:**
Si agent:
- Agrega nueva ruta que NO empieza con `/api/` → bypassea proxy
- Intenta patchear proxy en runtime → solo funciona en Vite, no en build
- Ignora `changeOrigin: true` → rompe cookies/auth (futuro)

**Regla Propuesta:**
```
REGLA: Vite Proxy = Única Configuración frontend-backend
- URL de API: SIEMPRE /api/...
- Host/port: NUNCA hardcodear, siempre vía proxy
- Proxy config: SOLO en vite.config.ts:11-15
- Cambios proxy: REQUIERE cambio en vite.config.ts + restart dev server
- Producción: Si cambias backend URL, cambiar proxy target
  Ref: frontend/vite.config.ts líneas 10-16
  Ref: frontend/src/App.tsx línea 16 (fetch via /api/metrics)
```

---

### Hallazgo 2: Package Scripts = CLI Estándar
**Evidencia:**
Frontend `package.json:6-12`:
```json
"scripts": {
  "dev": "vite",
  "build": "tsc -b && vite build",
  "lint": "eslint .",
  "preview": "vite preview",
  "test": "vitest run",
  "test:watch": "vitest",
  "test:coverage": "vitest run --coverage"
}
```

**Patrón Observado:**
Scripts standardizados. No scripts personalizados raros. `build` siempre include `tsc -b` (type checking).

**Riesgo:**
Si agent crea script sin type checking (solo `vite build`):
- TypeErrors se detectan en producción, no en build time
- Breaking changes en tipos no se catch

**Regla Propuesta:**
```
REGLA: Package Scripts = Operaciones Estándar
Frontend scripts DEBEN ser:
  - dev: vite
  - build: tsc -b && vite build (NUNCA solo vite build)
  - lint: eslint .
  - test: vitest run
  - test:watch: vitest
  - test:coverage: vitest run --coverage

Backend equivalente (no package.json, pero mismo rigor):
  - dev: debugpy + uvicorn con --reload
  - test: pytest
  - lint: podría agregar pylint/flake8 en futuro

No agregar scripts ad-hoc sin consenso.
  Ref: frontend/package.json líneas 6-12
```

---

### Hallazgo 3: Dockerfile = Build Reproducible
**Evidencia:**
Frontend `frontend/Dockerfile:1-12`:
```dockerfile
FROM node:24-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 5173

CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0", "--port", "5173"]
```

Backend `backend/Dockerfile:1-12`:
```dockerfile
FROM python:3.13-slim

WORKDIR /app

COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000 5678

CMD ["python", "-m", "debugpy", "--listen", "0.0.0.0:5678", "-m", "uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--reload"]
```

**Patrón Observado:**
Dockerfiles siguen Best Practices:
- Copia deps primero (COPY package*.json), install, copia código
- Especifica port y host explícitamente
- Usa slim/alpine para tamaño mínimo
- `--reload` para desarrollo

**Riesgo:**
Si agent cambia Dockerfile sin respetar orden (COPY . . antes de deps):
- Cada cambio de código invalida cache de deps
- Build slow, desarrollo frustrante

**Regla Propuesta:**
```
REGLA: Dockerfile = Orden de Capas Optimizado
- Capa 1: FROM [imagen oficial]
- Capa 2: WORKDIR /app
- Capa 3: COPY [deps files] ./  (package.json, requirements.txt)
- Capa 4: RUN [install]
- Capa 5: COPY . .  (código fuente)
- Capa 6: EXPOSE [ports]
- Capa 7: CMD [comando]

Razón: Invalida solo layers de código, reutiliza cache de deps.
Puertos: Host 0.0.0.0 (escuchar todas las interfaces en Docker).
Flags: --reload para dev, se quita en producción.
  Ref: frontend/Dockerfile líneas 1-12 (sigue patrón exacto)
  Ref: backend/Dockerfile líneas 1-12 (sigue patrón exacto)
```

---

### Hallazgo 4: docker-compose.yml = Servicios con Nombrado Explícito
**Evidencia:**
`docker-compose.yml:1-23`:
```yaml
services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "5173:5173"
    volumes:
      - ./frontend:/app
      - /app/node_modules
    depends_on:
      - backend

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "8000:8000"
      - "5678:5678"
    volumes:
      - ./backend:/app
```

**Patrón Observado:**
- Nombres de servicios = `frontend`, `backend` (simples, no `web_app_container_1`)
- `depends_on` declara orden startup
- Volúmenes anónimos para `node_modules` (evita conflictos)
- Puertos claros (5173, 8000, 5678)

**Riesgo:**
Si agent renombra servicios o ignora depends_on:
- Frontend inicializa antes que backend → falla
- Código dentro contenedores no puede resolver `http://frontend:5173` si nombre es diferente

**Regla Propuesta:**
```
REGLA: docker-compose.yml = Servicios Nombrados, Orden Explícito
- Servicios: frontend, backend (NUNCA web_1, app_2, etc.)
- depends_on: backend siempre < frontend
- Volúmenes: code en /app, node_modules anónimo
- Puertos: mapping explícito (5173:5173, 8000:8000, 5678:5678)
- Network: default bridge (automático, no personalizar)

Si cambias nombres servicios, actualiza:
  - docker-compose.yml (services, depends_on)
  - vite.config.ts proxy target (http://backend:8000)
  - Documentación (README, VERIFICATION.md)
  Ref: docker-compose.yml líneas 1-23
```

---

### Hallazgo 5: TypeScript Config = Project References
**Evidencia:**
`frontend/tsconfig.json`:
```json
{
  "files": [],
  "references": [
    { "path": "./tsconfig.app.json" },
    { "path": "./tsconfig.node.json" }
  ]
}
```

**Patrón Observado:**
TSConfig usa Project References. Separa config de app y build tools.

**Riesgo:**
Si agent modifica tsconfig.json raíz:
- Puede romper compilación de app o build tools
- Si todo es monolítico, cambios de Vite afectan app types

**Regla Propuesta:**
```
REGLA: TypeScript = Project References para Separación
- tsconfig.json (raíz): SOLO references, files=[]
- tsconfig.app.json: Configuración de app (compilerOptions)
- tsconfig.node.json: Configuración de tools (vite.config.ts, eslint.config.js)

Cambios:
  - Tipos de app → tsconfig.app.json
  - Tipos de tools → tsconfig.node.json
  - Nunca modifiques tsconfig.json raíz (solo references)

Build:
  - npm run build → tsc -b (build todas las referencias)
  - Falla si alguna ref tiene error
  Ref: frontend/tsconfig.json líneas 1-6
```

---

## 🔐 CORS & SEGURIDAD

### Hallazgo 1: CORS Abierto Intencionalmente (Desarrollo)
**Evidencia:**
`backend/app/main.py:7-12`:
```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

**Patrón Observado:**
CORS permite TODO (`allow_origins=["*"]`). Es intencional para desarrollo.

**Riesgo:**
Si agent deja `allow_origins=["*"]` en producción:
- Cualquier sitio puede hacer requests
- Vulnerabilidad CSRF si hay cookies
- Data expuesta

**Regla Propuesta:**
```
REGLA: CORS Abierto en Dev, Restrictivo en Prod
Desarrollo:
  - allow_origins=["*"]
  - allow_credentials=True
  - allow_methods=["*"]

Producción (FUTURO - no implementado yet):
  - allow_origins=["https://mi-dominio.com"]
  - allow_credentials=True
  - allow_methods=["GET", "POST", "PUT"] (específicos)
  - allow_headers=["Content-Type", "Authorization"]

TODO: Implementar env var para cambiar config CORS
  Ref: backend/app/main.py líneas 7-12 (dev config)
  Risk: Cambiar sin considerar implicaciones de seguridad
```

---

## 🚀 DEPLOYMENT & REPRODUCIBILIDAD

### Hallazgo 1: Seed Determinístico para Mock Data
**Evidencia:**
`backend/app/routes.py:94-96`:
```python
def generate_mock_movements(seed: int | None = None) -> list[FinancialMovement]:
    if seed is not None:
        random.seed(seed)
```

`backend/tests/test_routes.py:12-16`:
```python
def test_generate_mock_movements_returns_full_year_sorted_data():
    movements = generate_mock_movements(seed=42)
    assert len(movements) == 360
    assert movements == sorted(movements, key=lambda item: item.create_date)
```

**Patrón Observado:**
Todos los tests usan `seed=42`, haciendo datos reproducibles. Necesario para CI y debugging.

**Riesgo:**
Si agent omite seed en tests:
- Datos aleatoriamente diferentes cada run
- Tests fallan "sometimes" (flaky tests)
- Imposible debuggear en CI

**Regla Propuesta:**
```
REGLA: Seed Determinístico en Tests
- generate_mock_movements(seed=42) - SIEMPRE usar seed en tests
- Seed permite reproducibilidad exacta
- Si cambias seed, TODOS los tests que dependen deben cambiar también
- Documentar por qué se eligió valor específico (ej: "42 es número mágico de proyecto")

API en Desarrollo:
  - Backend devuelve seed=42 siempre
  - Permite comparar datos entre sesiones
  - TODO: Parameterizar seed vía query param para testing

  Ref: backend/app/routes.py línea 96 (default seed=None, pero routes usan 42)
  Ref: backend/tests/test_routes.py línea 13 (seed=42)
```

---

## 📋 ERROR HANDLING & LOGGING

### Hallazgo 1: Frontend Error Handling = Try/Catch + User Message
**Evidencia:**
`frontend/src/App.tsx:29-43`:
```typescript
useEffect(() => {
  fetchFinancialData()
    .then((movements) => {
      setMetrics(computeKPIs(movements));
      setMonthlyData(computeMonthlyData(movements));
    })
    .catch(() => {
      setError(
        "No se pudo cargar la informacion financiera. Revisa la API de backend.",
      );
    })
    .finally(() => {
      setLoading(false);
    });
}, []);
```

**Patrón Observado:**
- `.then()` → actualizar state
- `.catch()` → setError con mensaje user-friendly (español)
- `.finally()` → setLoading(false) siempre

**Riesgo:**
Si agent:
- Omite `.catch()` → usuario no sabe por qué falló
- No setea Loading en finally → UI queda en "loading" perpetuo
- Logga error pero no muestra UI → usuario confundido

**Regla Propuesta:**
```
REGLA: Frontend Error = Visible al Usuario + Logged
Patrón:
  .then(data => setState(data))
  .catch(err => {
    console.error(err);  // Log para debugging
    setError("User-friendly message en español o inglés");
  })
  .finally(() => setLoading(false))

Mensaje Error:
  - Debe ser entendible para no-técnicos
  - Sugerencia de acción ("Revisa la API de backend")
  - Visible en UI (renderizar error state)

Ref: frontend/src/App.tsx líneas 29-43
```

---

### Hallazgo 2: Backend Error = HTTP Status + Response Model
**Evidencia:**
- Backend responde con HTTP status correcto (200, 400, 500)
- Pydantic valida tipos automáticamente
- Si dato inválido → FastAPI retorna 422 automáticamente

**Patrón Observado:**
No hay `try/except` explícito en routes. FastAPI + Pydantic hacen validación.

**Riesgo:**
Si agent agrega `try/except` sin re-lanzar:
- Error se "swallows", no llega al cliente
- Client recibe 200 OK pero datos inconsistentes

**Regla Propuesta:**
```
REGLA: Backend Error = Dejar que FastAPI maneje
Patrón:
  - Pydantic models validan automáticamente
  - FastAPI retorna 422 si validación falla
  - Si necesitas lógica error, re-lanzar: raise HTTPException(status_code=400, detail="...")
  - NO hacer try/except silencioso

Validación:
  - En route handler, confia en type hints
  - Pydantic hace el trabajo
  - Si endpoint recibe bad data, Swagger + Pydantic lo validan

Error Logging:
  - Agregar logging si necesitas debugging
  - Pero no ocultar errores del cliente

Ref: backend/app/routes.py (no hay try/except, confía en Pydantic)
```

---

## ✅ CHECKLIST PARA AGENTS

Antes de hacer cambios, agent DEBE verificar:

- [ ] **Arquitectura**: ¿Respeta separación frontend/backend? ¿Usa proxy Vite?
- [ ] **Naming**: ¿Archivos React en kebab-case? ¿Funciones Python en snake_case?
- [ ] **Types**: ¿Interfaces/types en PascalCase? ¿Documentadas?
- [ ] **Testing**: ¿Agrega tests en lugar correcto? ¿Usa seed=42?
- [ ] **Documentación**: ¿Actualiza VERIFICATION.md? ¿No crea archivos paralelos?
- [ ] **Configuración**: ¿Respeta docker-compose.yml? ¿Usa vite proxy?
- [ ] **Error Handling**: ¿Frontend setea error state? ¿Backend usa HTTPException?
- [ ] **Build**: ¿npm run build incluye tsc -b?

---

## 📞 REFERENCIAS

**Archivos Clave:**
- `frontend/src/` - Toda la lógica React
- `frontend/src/lib/` - Tipos, utilidades, tests
- `backend/app/` - FastAPI app y routes
- `backend/tests/` - Pytest tests
- `.agents/rules/RULES.md` - Este archivo
- `VERIFICATION.md` - Documentación maestro

**Commits Relacionados:**
- `655fb0c` - Consolidate documentation
- `1a8b11e` - Verify code against documentation

---

**Versión:** 1.0 | **Fecha:** 2026-09-30 | **Estado:** ✅ Activo
