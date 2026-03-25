# OpenCode Agent Skills Repository

Este directorio contiene el conjunto de habilidades especializadas (skills) para el asistente de IA **OpenCode**. El repositorio proporciona patrones, flujos de trabajo y mejores prácticas para automatizar tareas de desarrollo de software.

## Que es OpenCode?

OpenCode es un asistente de IA especializado en desarrollo de software que sigue el principio de **"Nunca hacer trabajo real directamente"**. En su lugar, delega todas las tareas a sub-agentes especializados que utilizan skills domain-specific.

Este repositorio es el nucleo de esa delegacion: contiene **30+ skills** que cubren todo el ciclo de desarrollo, desde la exploracion del codebase hasta el deployment.

## Estructura del Repositorio

```
.agents/
├── skills/                    # Directorio principal de skills
│   ├── frontend-react/         # React + TanStack + Tailwind
│   ├── frontend-design/        # Diseno UI/UX de alta calidad
│   ├── backend-elysia/         # Elysia + Drizzle + Better Auth
│   ├── drizzle-orm/            # ORM TypeScript
│   ├── sdd-workflow/           # Orquestacion SDD (6 fases)
│   ├── testing-workflow/       # Testing (Vitest + Playwright)
│   ├── docker-openserve/       # Docker + OpenTelemetry
│   └── ... (25+ mas)
└── .skill-lock.json            # Registro de skills instalados
```

## Skills Disponibles

### Frontend
| Skill | Descripcion |
|-------|-------------|
| `frontend-react` | Componentes React con TanStack, Tailwind y Eden API |
| `frontend-design` | Interfaces de produccion con alta calidad de diseno |
| `ui-ux-pro-max` | 50+ estilos, 161 paletas de colores, patrones de UX |
| `tailwind-design-system` | Design tokens y componentes escalables |
| `tanstack-query-best-practices` | Server state management |
| `tanstack-router-best-practices` | Type-safe routing |
| `vercel-react-best-practices` | Optimizacion de rendimiento React/Next.js |
| `web-design-guidelines` | Auditoria de accesibilidad y UX |

### Backend
| Skill | Descripcion |
|-------|-------------|
| `backend-elysia` | Elysia + Drizzle + Better Auth |
| `elysiajs` | Patrones del framework Elysia |
| `architecture-patterns` | Clean Architecture, Hexagonal, DDD |

### Base de Datos
| Skill | Descripcion |
|-------|-------------|
| `drizzle-orm` | ORM type-safe con zero runtime overhead |
| `postgresql-optimization` | Caracteristicas avanzadas de Postgres |
| `sql-optimization` | Query tuning e indexacion |
| `postgresql-code-review` | Mejores prácticas de Postgres |
| `sql-code-review` | Revision de seguridad SQL |

### Testing
| Skill | Descripcion |
|-------|-------------|
| `vitest` | Testing unitario con Vite |
| `playwright-best-practices` | E2E testing, accesibilidad, rendimiento |
| `testing-workflow` | Estrategia de testing completa |

### DevOps & Auth
| Skill | Descripcion |
|-------|-------------|
| `docker-openserve` | Docker + OpenTelemetry + OpenObserve |
| `multi-stage-dockerfile` | Dockerfiles optimizados |
| `better-auth-best-practices` | Autenticacion con Better Auth |

### Workflow & Utilidades
| Skill | Descripcion |
|-------|-------------|
| `sdd-workflow` | Orquestacion de 6 fases SDD |
| `memory-engram` | Persistencia de memoria entre sesiones |
| `monorepo-bun` | Convenciones de monorepo con Bun |
| `architecture-blueprint-generator` | Documentacion arquitectonica |
| `refactor` | Refactoring cirurgico |
| `skill-creator` | Creacion de nuevas skills |
| `create-readme` | Generacion de archivos README |
| `create-agentsmd` | Generacion de AGENTS.md |
| `find-skills` | Descubrimiento de skills |
| `conventional-commit` | Estandar de commits |
| `contract-typebox` | Esquemas de validacion |
| `python-best-practices` | Patron Python con tipos |

## Flujo de Trabajo SDD

Para tareas sustanciales, OpenCode utiliza **Spec-Driven Development (SDD)**, un flujo de 6 fases:

```
explore → spec → tasks → apply → verify → archive
```

| Fase | Skill | Descripcion |
|------|-------|-------------|
| **explore** | `sdd-explore` | Investiga el codebase, identifica riesgos |
| **spec** | `sdd-spec` | Escribe specs delta con Given/When/Then |
| **tasks** | `sdd-tasks` | Lista de tareas en fases |
| **apply** | `sdd-apply` | Implementa codigo siguiendo specs |
| **verify** | `sdd-verify` | Valida contra specs con tests reales |
| **archive** | `sdd-archive** | Cierra cambio, persiste estado |

### Delegacion Rapida

Para tareas pequenas, se utiliza `quick-delegate` que:
- Auto-detecta el stack tecnologia del proyecto
- Selecciona el skill de dominio apropiado
- Ejecuta y retorna un resumen

## Como Funciona

1. **Detectar Stack**: Cuando inicializas un proyecto, OpenCode detecta automaticamente el stack (React, Next.js, Elysia, Django, etc.)

2. **Seleccionar Skill**: Segun la tarea, OpenCode carga el skill relevante (ej: "crear componente" → carga `frontend-react`)

3. **Ejecutar Patron**: El skill proporciona patrones probados, mejores prácticas y templates para completar la tarea

4. **Persistir Memoria**: Usando Engram, OpenCode recuerda decisiones y patrones entre sesiones

## Extension de Skills

Para agregar un nuevo skill:

1. Crea un directorio en `skills/`
2. Agrega un archivo `SKILL.md` con la estructura:
   - Descripcion del proposito
   - Cuándo usar este skill
   - Patron de implementacion
   - Ejemplos de codigo
3. Registralo en `.skill-lock.json`

Usa el skill `skill-creator` para guiate en el proceso.

---

*Este repositorio es parte de la configuración de OpenCode en `/home/josue/.config/opencode/`*
