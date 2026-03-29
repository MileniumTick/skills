# Skill: sdd-propose

## Overview

Creates a formal proposal for a change. Defines intent, scope, approach, and rollback plan before writing specs.

## When to Use This Skill

- Starting a new feature that needs approval
- Architectural changes requiring discussion
- Any substantial change that benefits from upfront planning
- After `/sdd-explore` when user approves proceeding

## Step 1: Load Skill Registry

First, load available skills:
1. Try: `mem_search(query: "skill-registry", project: "{project}")`
2. If found, use the skills listed
3. Also check for domain-specific skills (frontend-react, backend-elysia, etc.)

## Step 2: Understand the Change

### 2.1 Gather Context
- What problem does this solve?
- Who requested it and why?
- What's the priority/urgency?

### 2.2 Research Existing Work
- Check if similar work exists
- Look at related specs, proposals, or implementations
- Identify dependencies

## Step 3: Define Proposal

### 3.1 Intent Section
Write a clear statement of:
- **What**: The change being proposed
- **Why**: The problem it solves or value it adds
- **Who**: Who requested it and who will benefit

### 3.2 Scope Section
Define:
- **In Scope**: What will be built/delivered
- **Out of Scope**: What is explicitly NOT included
- **Dependencies**: What needs to happen first

### 3.3 Approach Section
Describe:
- **Primary Approach**: How you plan to implement it
- **Alternatives Considered**: Why this approach was chosen over others
- **Trade-offs**: What you're trading off with this choice

### 3.4 Rollback Plan
Define:
- How to undo this change if it fails
- What state the system should be in after rollback
- Any data migration concerns

## Step 4: Identify Risks

### 4.1 Technical Risks
- Complexity, performance, security concerns

### 4.2 Process Risks
- Dependencies on other teams, delayed reviews

### 4.3 Mitigation Strategies
For each risk, define how to reduce or manage it

## Step 5: Write Proposal Artifact

### 5.1 Format
Create a proposal document:

```markdown
# Proposal: {change-name}

## Intent
{What, Why, Who}

## Scope
### In Scope
- ...

### Out of Scope
- ...

### Dependencies
- ...

## Approach
### Primary Approach
...

### Alternatives Considered
1. ...
2. ...

### Trade-offs
- ...

## Rollback Plan
...

## Risks
| Risk | Impact | Mitigation |
|------|--------|------------|
| ... | ... | ... |

## Timeline Estimate
- Phase 1: ...
- Phase 2: ...
```

### 5.2 Save Location
- Filesystem: `.atl/changes/{change-name}/proposal.md`
- Engram: Topic key `sdd/{change-name}/proposal`

## Result Contract

**Status**: success | partial | blocked
**Summary**: Created proposal for {change-name}. Defined intent, scope, approach, and rollback plan.
**Artifacts**: 
- `.atl/changes/{change-name}/proposal.md`
- Engram `sdd/{change-name}/proposal`
**Next**: sdd-spec (after user approves proposal)
**Risks**: Identified {n} risks with mitigation strategies

## Tips

- Keep proposal focused - don't over-specify
- Be explicit about what's OUT of scope
- Always include rollback plan
- Update proposal if scope changes during implementation
