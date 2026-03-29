---
name: team-orchestrator
description: Enhanced team orchestration with reasoning, retry, fallback, and task ledger. Use when you need to coordinate multiple agents with error handling and planning capabilities.
---

# Team Orchestrator Skill

Enhanced delegation capabilities with reasoning, error handling, and task tracking.

---

## 1. Reasoning Pattern (Before Delegating)

Before delegating any task, ALWAYS show reasoning:

```
## Reasoning
[Thought 1]: Analyze the task - what is being asked?
[Thought 2]: What agents are needed? Which ones are available?
[Thought 3]: What's the best approach - parallel or sequential?
[Thought 4]: Any risks or considerations?
[Decision]: I'll delegate to [agent] because [reason]
```

**Example:**
```
## Reasoning
[Thought 1]: User wants to implement a new API endpoint - this requires understanding the existing codebase structure first.
[Thought 2]: I need @exploration to analyze the codebase, then @dev to implement, and finally @qa to verify.
[Thought 3]: Sequential approach makes sense here - exploration must complete before implementation can start.
[Thought 4]: Risk: The codebase might be complex; fallback to direct file analysis if exploration fails.
[Decision]: I'll delegate to @exploration first because understanding the existing patterns is critical before any implementation.
```

---

## 2. Retry with Exponential Backoff

When an agent fails, retry with increasing delays:

| Attempt | Wait Time |
|---------|-----------|
| 1 | Immediate |
| 2 | 2 seconds |
| 3 | 4 seconds |
| 4 | 8 seconds |

**Max attempts: 4**

```python
## Retry Logic
if agent fails:
  for attempt in range(1, max_retries + 1):
    wait_time = 2 ** attempt  # exponential: 2, 4, 8, 16
    wait(wait_time)
    retry
  if all fail: use fallback
```

**Example:**
```
## Retry Execution
Attempt 1 (immediate): @dev failed - timeout
Attempt 2 (2s wait): Retrying @dev...
Attempt 2 failed - partial response
Attempt 3 (4s wait): Retrying @dev...
Attempt 3 succeeded ✓

Proceeding with results...
```

---

## 3. Fallback Strategy

Define fallback agents for each primary:

```
## Fallback Chain
| Primary Agent | Fallback 1 | Fallback 2 |
|--------------|------------|------------|
| @dev | @exploration | manual code review |
| @qa | @exploration | user validation |
| @security | @dev | skip and warn |
| @exploration | direct file analysis | skip task |
```

**Example:**
```
## Fallback Execution
Primary: @dev (implement feature)
Result: Failed - insufficient context

Fallback 1: @exploration (analyze codebase)
Result: Succeeded ✓

Continuing with implementation using new context...
```

---

## 4. Task Ledger (Basic)

Track task state in each delegation:

```
## Task Ledger
| Task ID | Description | Agent | Status | Result |
|---------|-------------|-------|--------|--------|
| T1 | Explore codebase | @exploration | ✓ done | Found X files |
| T2 | Implement feature | @dev | ⟳ in_progress | - |
| T3 | Verify quality | @qa | ○ pending | - |
```

Use todowrite to maintain this ledger for complex tasks.

**Example:**
```
## Task Ledger Update
| Task ID | Description | Agent | Status | Result |
|---------|-------------|-------|--------|--------|
| T1 | Explore codebase | @exploration | ✓ done | Found 12 API files |
| T2 | Implement endpoint | @dev | ⟳ in_progress | Adding /users/:id route |
| T3 | Write tests | @qa | ○ pending | - |
| T4 | Security review | @security | ○ pending | - |
```

---

## 5. Execution Template

For each delegation, follow this flow:

```
## Before Delegation
1. Show reasoning (what, why, how)
2. Check task ledger for dependencies
3. Select agent with fallback plan

## During Execution
4. Track status in ledger
5. Handle errors with retry logic

## After Completion
6. Update ledger with results
7. Verify result matches expectations
8. Proceed to next task or fallback
```

**Complete Example:**
```
## Delegation: Implement User API

## Reasoning
[Thought 1]: Need to add a new user profile endpoint
[Thought 2]: Requires @exploration first to find existing patterns, then @dev for implementation
[Thought 3]: Sequential - exploration must complete first
[Thought 4]: Risk: May need fallback to direct file reading
[Decision]: Start with @exploration

## Task Ledger
| Task ID | Description | Agent | Status |
|---------|-------------|-------|--------|
| T1 | Find user patterns | @exploration | ⟳ in_progress |
| T2 | Implement endpoint | @dev | ○ pending |
| T3 | Test implementation | @qa | ○ pending |

## Execution
[T1] Delegating to @exploration...
  - Query: Find all user-related API files and patterns
  - Retry: Attempt 1/4
  - Result: Found 5 relevant files ✓

[T1] Update: ✓ done - Found user.ts, auth.ts, and 3 controller files

[T2] Checking dependencies: T1 complete ✓
[T2] Delegating to @dev...
  - Context: Previous findings from T1
  - Retry: Attempt 1/4
  - Result: Implementation complete ✓

## Final Ledger
| Task ID | Description | Agent | Status | Result |
|---------|-------------|-------|--------|--------|
| T1 | Find user patterns | @exploration | ✓ done | 5 files found |
| T2 | Implement endpoint | @dev | ✓ done | /users/:id added |
| T3 | Test implementation | @qa | ✓ done | All tests pass |
```

---

## 6. Success Score (Reinforce)

Before finalizing any plan, calculate:

| Criterion | Score |
|-----------|-------|
| Task decomposition clarity | ?/10 |
| Agent selection appropriateness | ?/10 |
| Context completeness | ?/10 |
| Risk identification | ?/10 |

**Average = Success Score**

If average < 8: **RECONSIDER** the approach.

**Example:**
```
## Success Score Calculation
- Task decomposition clarity: 9/10 (clear 3-step plan)
- Agent selection appropriateness: 8/10 (right agents, good fallback)
- Context completeness: 7/10 (could include more examples)
- Risk identification: 8/10 (identified fallback paths)

Average: 8/10 ✓ - Proceed with delegation
```

---

## Summary

This skill enables robust multi-agent coordination:

1. **Reason first** - Always explain your delegation decisions
2. **Retry smartly** - Use exponential backoff (2s, 4s, 8s, 16s)
3. **Fallback gracefully** - Define backup agents in advance
4. **Track everything** - Maintain a task ledger for visibility
5. **Verify success** - Calculate scores and reconsider if < 8/10

---

## Level 2: Advanced Orchestration Patterns

### 2.1 Plan-and-Execute Pattern

This pattern separates PLANNING from EXECUTION. Use for complex tasks.

**Workflow:**
```
┌─────────────────┐     ┌─────────────────┐
│    PLANNER      │     │   EXECUTORS     │
│ (you/team-lead) │────▶│ (specialized)   │
│                 │     │                 │
│ 1. Analyze task │     │ Step 1 ──►      │
│ 2. Create plan  │     │ Step 2 ──►      │
│ 3. Define steps │     │ Step 3 ──►      │
└─────────────────┘     └─────────────────┘
```

**When to use:**
- Multi-file changes
- Tasks with clear sequential steps
- When you need to validate the plan before executing

**Template:**
```
## PLAN: {task-name}

### Step 1: {action}
- Agent: @X
- Input: {what to give}
- Expected: {what to get back}

### Step 2: {action}
- Agent: @Y  
- Input: {what to give}
- Expected: {what to get back}

...continue for each step

### Validation: How will I know this worked?
- {criterion 1}
- {criterion 2}
```

**Security benefit:** Once plan is created, tool data cannot modify it. Protects against prompt injection.

---

### 2.2 Hierarchical Pattern (Multi-Level)

Structure agents in levels, never more than 2-3 levels deep:

```
Level 1: ORCHESTRATOR (@team-lead)
  └─ Level 2: DOMAIN LEADS
       ├─ @backend-lead (Elysia, Node, Python)
       ├─ @frontend-lead (React, Vue, Svelte)
       ├─ @data-lead (PostgreSQL, Drizzle, Redis)
       └─ @security-lead (Auth, OWASP, Compliance)
            └─ Level 3: WORKERS
                 ├─ @dev
                 ├─ @qa
                 ├─ @exploration
                 └─ @security
```

**When to use:**
- Large projects with multiple domains
- When different domains need different expertise
- When you want to distribute cognitive load

**Routing rules:**
- Level 1 → Level 2: Strategic decisions (what needs to be done)
- Level 2 → Level 3: Tactical execution (how to do it)
- Never skip levels unless it's a simple task

---

### 2.3 Agent Selection Matrix

For each task type, know which domain lead to route to:

| Task Type | Route to | If Failed → |
|-----------|----------|-------------|
| Backend API, DB, Auth | @backend-lead | @dev |
| Frontend UI, Components | @frontend-lead | @dev |
| Queries, Performance | @data-lead | @exploration |
| Security Review, Auth | @security-lead | @security |
| Investigation, Debugging | @exploration | direct file read |
| Testing, QA | @qa | manual verification |

---

### 2.4 Complete Execution Flow

For complex tasks, use this full flow:

```
## COMPLETE FLOW: {task-name}

### PHASE 1: PLANNING (Plan-and-Execute)
1. Analyze the request
2. Determine required domains
3. Route to domain leads for input
4. Create consolidated plan

### PHASE 2: EXECUTION  
5. Execute each step in order
6. Track in Task Ledger
7. Handle failures with retry/fallback

### PHASE 3: VALIDATION
8. Verify each step succeeded
9. Consolidate results
10. Report to user

### Task Ledger (Full)
| ID | Step | Domain Lead | Worker | Status | Result |
|----|------|-------------|--------|--------|--------|
| P1 | Explore | @team-lead | @exploration | ✓ | Found X |
| P2 | Plan | @team-lead | - | ✓ | 3 steps |
| E1 | Backend | @backend-lead | @dev | ⟳ | - |
| E2 | Frontend | @frontend-lead | @dev | ○ | - |
| E3 | Verify | @team-lead | @qa | ○ | - |
```

---

### 2.5 Plan Example: "Add OAuth to the app"

```
## PLAN: Add OAuth Authentication

### Context
- Current: Email/password auth
- Goal: Add Google + GitHub OAuth

### Step 1: Exploration (Assign → @backend-lead)
Input: "Analyze current auth implementation"
Expected: auth.ts structure, sessions handling, DB schema

### Step 2: Security Review (Assign → @security-lead)  
Input: "Review OAuth security requirements"
Expected: Security checklist, OAuth flow validation

### Step 3: Implementation (Assign → @backend-lead)
Input: "Add OAuth with Better Auth"
Expected: Working Google + GitHub login

### Step 4: Frontend Update (Assign → @frontend-lead)
Input: "Add OAuth buttons to login page"
Expected: Login page with OAuth options

### Step 5: Testing (Assign → @qa)
Input: "Verify OAuth flow works"
Expected: Test report

### Validation:
- [ ] OAuth login works for Google
- [ ] OAuth login works for GitHub
- [ ] Session persists correctly
- [ ] No security vulnerabilities
```

---

## Level 2 Summary

Advanced orchestration patterns for complex tasks:

1. **Plan-and-Execute** - Separate planning from execution for complex multi-step tasks
2. **Hierarchical Pattern** - Structure agents in 2-3 levels with domain leads
3. **Agent Selection Matrix** - Route tasks to the right domain lead
4. **Complete Execution Flow** - Full planning → execution → validation cycle
5. **Plan Examples** - Templates for common complex scenarios

---

## Level 3: Production-Ready Patterns

### 3.1 Error Handling (Three Types)

AI systems have three error types you must handle:

| Error Type | Description | How to Handle |
|------------|-------------|---------------|
| **Execution** | Tool failed, API down, timeout | Retry with exponential backoff |
| **Semantic** | Valid params but wrong values | Validate output structure |
| **Planning** | Plan has impossible steps | Re-plan from scratch |

**Concrete rules:**
```
- Retry policy: 3-5 attempts for external APIs (5s delay + jitter)
- Retry policy: 2-3 attempts for LLMs (respect rate limits)
- Circuit breaker: If tool fails X times → stop calling for a period
- Fallback LLMs: Claude fails → GPT-4 → local model
```

### 3.2 Observability Metrics

Track these four categories:

**Performance:**
- Total response time
- Latency per stage (planning, execution, synthesis)
- Time to first token

**Reliability:**
- Error rate by type
- Completion rate
- Retry rate
- Circuit breaker trips

**Quality:**
- Task success rate
- User feedback (if available)

**Cost:**
- Token usage per request
- Estimated cost per task

**Template for tracking:**
```
## Execution Metrics

| Metric | Value |
|--------|-------|
| Total Time | Xs |
| Planning Time | Xs |
| Execution Time | Xs |
| Retries | X |
| Errors | X |
| Tokens Est. | X |
```

### 3.3 MCP Ecosystem (For Future)

Current MCP clients (consuming):
- context7: Codebase context
- gitea: Git hosting
- plane: Project management
- engram: Memory

**Could add MCP servers (exposing):**
- Custom tools for other AI apps
- Project-specific capabilities
- Internal API endpoints

### 3.4 Clean Architecture Principles (For Reference)

If building AI apps, structure:
```
src/
├── domain/           # Pure, no AI deps
├── application/      # Orchestration + Use Cases
│   ├── agents/       # AI Agents
│   └── orchestration/
└── infrastructure/   # LLM Adapters, MCP Clients
```

**Sacred principle:** LLMs interpret and orchestrate. The domain validates and executes. Never let AI make business decisions without validation.

---

## Summary: All Levels

| Level | Patterns | When to Use |
|-------|----------|-------------|
| 1 | Reasoning + Retry + Fallback + Task Ledger | Simple delegations |
| 2 | Plan-and-Execute + Hierarchical | Complex multi-domain tasks |
| 3 | Error Handling + Observability + MCP | Production systems |

---

## Quick Reference Card

```
Delegation Flow:
1. Reasoning (show thought process)
2. Plan (if complex) → Execute
3. Retry on failure (2, 4, 8, 16s)
4. Fallback if all retries fail
5. Track in Task Ledger
6. Report with metrics

Levels:
- Simple → Level 1
- Multi-step → Level 2  
- Production → Level 3
```

(End of file)
