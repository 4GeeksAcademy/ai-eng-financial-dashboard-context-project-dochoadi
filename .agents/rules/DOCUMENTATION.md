# DOCUMENTATION & DX RULES
**Alcance:** Documentation, types, developer experience

---

## RULE: VERIFICATION.md is Single Source of Truth

**Alcance:** Project documentation

**Justificación:**
Multiple docs create conflicts (README says X, ARCHITECTURE.md says Y). One master doc reduces confusion, easier maintenance. All agents and contributors read one place.

**Evidencia:**
- `VERIFICATION.md` exists and is comprehensive (742 lines)
- `README.md` references VERIFICATION.md implicitly
- No separate `ARCHITECTURE.md`, `DESIGN.md`, `DOCS/*.md` parallel docs

**Guía Accionable:**
1. Single documentation: `VERIFICATION.md` is master document
2. Never create: `ARCHITECTURE.md`, `DESIGN.md`, separate `/docs` folder
3. New documentation: ADD section to VERIFICATION.md (don't create new file)
4. README.md: Brief description + reference to VERIFICATION.md
5. Phase summaries (PHASE1_SUMMARY, etc.): Separate files OK (not core docs)

**Ejemplo Correcto:**
```markdown
# VERIFICATION.md
## Architecture
... comprehensive details ...

## API Documentation
... all endpoints ...
```

**Ejemplo Incorrecto:**
```markdown
# ARCHITECTURE.md (❌ Creates conflict with VERIFICATION.md)
# docs/API.md (❌ Separate docs increase confusion)
# README.md (contains everything) (❌ Bloats README)
```

---

## RULE: Types Document Their Own Contract

**Alcance:** TypeScript type definitions, Pydantic models

**Justificación:**
Type definitions ARE documentation. Comments on ambiguous fields prevent bugs (e.g., "is this ISO date or timestamp?" answered by comment).

**Evidencia:**
- `frontend/src/lib/financial-types.ts:6` - `create_date: string // ISO date`
- `frontend/src/lib/financial-types.ts` - All types include field documentation
- No separate type documentation file needed

**Guía Accionable:**
1. Every interface/type: Include JSDoc or inline comment explaining purpose
2. Ambiguous fields: Add comment clarifying format/range (e.g., "ISO date", "0-100", "enum values: X,Y,Z")
3. Examples: Add comment if format not obvious
4. Pydantic models (Python): Use docstrings for class, comments for complex fields

**Ejemplo Correcto:**
```typescript
export interface FinancialMovement {
  create_date: string // ISO date (YYYY-MM-DD)
  amount: number // USD cents, e.g., 1050 = $10.50
  operation_type: OperationType // 'income' or 'outcome'
  category: Category // See enum definition above
  business_type: BusinessType // 'B2B' or 'B2C'
}
```

**Ejemplo Incorrecto:**
```typescript
// ❌ No comments, unclear format
export interface FinancialMovement {
  create_date: string
  amount: number
  operation_type: OperationType
}

// ❌ Separate documentation file (types should be self-documenting)
// docs/types.md
```

