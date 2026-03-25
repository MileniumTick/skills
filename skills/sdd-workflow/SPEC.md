# Spec: Mejoras al Sistema SDD

## Overview

Mejorar el sistema de sub-agentes SDD agregando routing rules, invocation protocol, y model routing basados en mejores prácticas de agent-teams-lite y patrones modernos de Claude Code.

## Scope

### In Scope
- Agregar routing rules (parallel vs sequential)
- Crear invocation protocol de 4 componentes
- Agregar model routing
- Mejorar documentación

### Out of Scope
- Reescribir completamente las fases
- Cambiar estructura de archivos

---

## Delta Spec

## MODIFIED

### sdd-workflow/SKILL.md

#### Changed: Agregar Routing Rules
**From**: Sin routing rules, ejecución secuencial por defecto
**To**: Routing rules que determinan parallel vs sequential

```
## Routing Rules

### Parallel Dispatch (ALL conditions):
- 3+ tareas independientes
- Sin estado compartido
- Sin archivos compartidos

### Sequential Dispatch (ANY trigger):
- Tareas tienen dependencias
- Archivos compartidos
- Scope unclear
```

#### Changed: Agregar Invocation Protocol
**From**: Sin protocolo estandarizado
**To**: Protocolo de 4 componentes para cada invocación

```
## Invocation Protocol

Cada sub-agente debe recibir:
1. Role: Qué hace el sub-agente
2. Context: Información del proyecto
3. Task: Qué hacer específicamente
4. Success Criteria: Cómo saber que terminó bien
```

#### Changed: Agregar Model Routing
**From**: Un modelo para todo
**To**: Modelo según complejidad

| Task Type | Model |
|-----------|-------|
| Explore/Research | fast/cheap |
| Spec/Design | sonnet |
| Apply/Verify | opus |

---

## ADDED

### Feature: Enhanced Quick Delegate

#### Scenario: Routing automático
**Given**: Una tarea del usuario
**When**: Se invoca quick-delegate
**Then**: Detecta dominio y routing correcto automáticamente

#### Scenario: Parallel execution
**Given**: Múltiples tareas independientes
**When**: quick-delegate detecta independence
**Then**: Ejecuta en parallel si es óptimo

---

### Feature: Token Optimization

#### Scenario: Context isolation
**Given**: Sub-agente ejecutándose
**When**: Comienza tarea
**Then**: Recibe solo contexto necesario, no historial completo

---

## Acceptance Criteria

- [ ] sdd-workflow/SKILL.md tiene routing rules
- [ ] sdd-workflow/SKILL.md tiene invocation protocol
- [ ] sdd-workflow/SKILL.md tiene model routing
- [ ] quick-delegate actualizado con parallel routing
- [ ] README.md refleja los cambios
