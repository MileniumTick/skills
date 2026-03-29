# Skill: sdd-design

## Overview

Creates detailed architectural design for a change. Documents architecture decisions, data models, API contracts, and the rationale behind key choices.

## When to Use This Skill

- After proposal is approved
- When architecture decisions need to be formalized
- Before task planning
- When multiple implementation approaches exist

## Prerequisites

- Proposal (`sdd-propose`) should be completed
- Specs (`sdd-spec`) should exist or be in progress

## Step 1: Load Skill Registry

First, load available skills:
1. Try: `mem_search(query: "skill-registry", project: "{project}")`
2. Load relevant domain skills:
   - `contract-typebox` for schema decisions
   - `backend-elysia` or `frontend-react` as needed

## Step 2: Review Existing Artifacts

### 2.1 Load Proposal
Read: `.atl/changes/{change-name}/proposal.md`

### 2.2 Load Specs
Read: `.atl/changes/{change-name}/spec.md`
Or check Engram: `mem_search(query: "sdd/{change-name}/spec")`

### 2.3 Understand Constraints
- What existing patterns must be followed?
- What are the non-functional requirements?
- Any technical limitations?

## Step 3: Architecture Design

### 3.1 High-Level Architecture
Define:
- Components/modules involved
- How they interact
- New services or APIs needed

### 3.2 Data Model
Document:
- New database tables/collections
- Schema changes
- Migrations needed

### 3.3 API Design
For each new endpoint:
- HTTP method and path
- Request/response schema
- Error cases
- Authentication/authorization

### 3.4 Component Design
For each new component:
- Props/interface
- State management
- Dependencies

## Step 4: Document Decisions

### 4.1 Architecture Decision Records (ADRs)

For each significant decision, document:
- **Context**: What was the situation?
- **Decision**: What did you choose?
- **Consequences**: What are the trade-offs?
- **Alternatives**: What was considered?

### 4.2 Technology Choices
- Why this library/framework over others?
- Any learning curve concerns?

## Step 5: Write Design Artifact

### 5.1 Format
Create a design document:

```markdown
# Design: {change-name}

## Overview
{Brief description of the architecture}

## Architecture

### Components
| Component | Responsibility | Public API |
|-----------|----------------|------------|
| ... | ... | ... |

### Data Flow
{Flow diagram or description}

## Data Model

### New Entities
```typescript
// Entity definition
```

### Schema Changes
- Table: ...
- Field: ...

## API Design

### New Endpoints

#### POST /api/resource
**Request:**
```typescript
{ schema }
```

**Response:**
```typescript
{ schema }
```

## Key Decisions

| Decision | Choice | Rationale | Trade-off |
|----------|--------|-----------|------------|
| ... | ... | ... | ... |

## Security Considerations
- ...


## Performance Considerations
- ...

## Testing Strategy
- Unit tests for: ...
- Integration tests for: ...
- E2E scenarios: ...

## Migration Plan (if applicable)
1. ...
2. ...
```

### 5.2 Save Location
- Filesystem: `.atl/changes/{change-name}/design.md`
- Engram: Topic key `sdd/{change-name}/design`

## Result Contract

**Status**: success | partial | blocked
**Summary**: Created design for {change-name}. Documented {n} components, {n} API endpoints, {n} architecture decisions.
**Artifacts**: 
- `.atl/changes/{change-name}/design.md`
- Engram `sdd/{change-name}/design`
**Next**: sdd-tasks or sdd-apply (after user approves design)
**Risks**: Any design concerns or open questions

## Tips

- Don't over-design - focus on what's needed
- Reference existing patterns in the codebase
- Include rationale, not just decisions
- Consider security, performance, and testability
- Keep design aligned with approved proposal scope
