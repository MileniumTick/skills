---
name: sdd-workflow
description: >
  SDD Workflow Orchestrator - Delegate-first task execution.
  Detects stack, manages phases, delegates to sub-agents.
---

# SDD Workflow - Orchestrator Core

## Philosophy

**"Never do real work directly — delegate everything."**

El orquestador es un coordinador liviano que delega todo el trabajo real a sub-agentes especializados. Mantiene solo estado y resúmenes.

## Reglas de Delegación

### Tarea Pequeña (Quick Delegation)
Si la tarea es simple y enfocada:
```
→ skill(name="[dominio relevant]") 
→ Ejecuta directamente con contexto fresco
→ Retorna resultado
```

**Ejemplos de quick delegation:**
- "Fix el bug del login" → `skill(name="backend-elysia")`
- "Agrega un componente Button" → `skill(name="frontend-react")`
- "Optimiza esta query" → `skill(name="postgresql-optimization")`

### Tarea Sustancial (SDD Completo)
Si la tarea es compleja, multi-archivo, o necesita planeación:
```
→ /sdd-new <change-name>
→ Explora → Spec → Tasks → Apply → Verify → Archive
```

## Comandos SDD

| Comando | Qué hace |
|---------|----------|
| `/sdd-init` | Detecta stack, bootstraps persistencia, build skill registry |
| `/sdd-explore <topic>` | Investiga el codebase, identifica riesgos |
| `/sdd-new <name>` | Inicia nuevo cambio: explore → proposal → spec → design → tasks |
| `/sdd-continue` | Ejecuta la siguiente fase lista |
| `/sdd-spec <name>` | Escribe specs delta (ADDED/MODIFIED/REMOVED) |
| `/sdd-apply` | Implementa tareas en batches |
| `/sdd-verify` | Valida implementación contra specs con tests reales |
| `/sdd-archive` | CIerra cambio y persiste estado final |

## Flujo SDD

```
user: "quiero agregar autenticación OAuth"

ORCHESTRATOR (delegar todo):
  → /sdd-explore oauth-integration
  → Muestra resumen, user aprueba
  → /sdd-spec oauth-feature  
  → /sdd-design oauth-architecture
  → /sdd-tasks oauth-implementation
  → /sdd-apply (batch 1)
  → /sdd-apply (batch 2)
  → /sdd-verify
  → /sdd-archive
```

## Result Contract

Todo sub-agente debe retornar:

```markdown
## Result

**Status**: success | partial | blocked

**Summary**: 1-3 oraciones de lo que se hizo

**Artifacts**: 
- `engram:sdd/{change}/explore` | `file:docs/sdd/{change}/explore.md`

**Next**: siguiente fase o "none"

**Risks**: riesgos descubiertos o "None"
```

## Auto-detección de Skills

El orquestador detecta automáticamente qué skills cargar según el stack del proyecto:

1. Lee `package.json` o `pyproject.toml` → detecta stack
2. Carga skills relevantes automáticamente
3. Pasa los skills al sub-agente en el prompt

**No necesitas especificar skills manualmente.**

## Decision Matrix

| Tipo de tarea | Acción |
|---------------|--------|
| Bug fix simple | Quick delegation a skill de dominio |
| Nueva feature pequeña | Quick delegation + spec inline |
| Feature compleja | SDD completo |
| Refactor grande | SDD completo |
| Investigación | /sdd-explore |
| Code review | skill(name="sql-code-review") o similar |
| Optimización | skill(name="postgresql-optimization") |

## Routing Rules

### Parallel Dispatch (ALL conditions must be met):
- 3+ tareas independientes entre sí
- Sin estado compartido
- Sin archivos compartidos
- Diferentes dominios (frontend + backend + db)

### Sequential Dispatch (ANY trigger):
- Tareas tienen dependencias (B necesita output de A)
- Archivos compartidos
- Scope unclear (necesita entender antes de proceder)

### When to Use Each
| Pattern | Central Agent Uses When | Risk If Wrong |
|---------|------------------------|---------------|
| **Parallel** | Tareas independientes, sin file overlap | Merge conflicts, estado inconsistente |
| **Sequential** | Dependencias existen, recursos compartidos | Tiempo perdido en trabajo serializado |

## Phase Dependencies

```
explore ──┬──> spec ──> tasks ──> apply ──> verify ──> archive
          │                                         
          └- design ───────────────────────────────┘
```

- `spec` y `design` pueden correr en paralelo (sin dependencias)
- `apply` puede correr en batches
- `verify` siempre después de `apply`
- `archive` siempre al final

## Skill Registry

Skills de dominio disponibles en `~/.agents/skills/`:

| Dominio | Skill | Para qué |
|---------|-------|----------|
| Frontend | `frontend-react` | Componentes React, TanStack, Tailwind |
| Frontend | `tanstack-query` | Server state management |
| Frontend | `tanstack-router` | Type-safe routing |
| Frontend | `vercel-react` | Next.js performance |
| Backend | `backend-elysia` | Elysia + Drizzle + Auth |
| Backend | `elysiajs` | Framework patterns |
| DB | `drizzle-orm` | ORM patterns |
| DB | `postgresql-optimization` | Advanced Postgres |
| DB | `sql-optimization` | Query tuning |
| Testing | `vitest` | Unit tests |
| Testing | `playwright` | E2E tests |
| Testing | `testing-workflow` | Testing strategy |
| DevOps | `docker-openserve` | Docker + observability |
| Auth | `better-auth` | Authentication |
| UI/UX | `ui-ux-pro-max` | Design patterns |
| CSS | `tailwind-design-system` | Design tokens |
