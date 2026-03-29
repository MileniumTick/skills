# Skill: skill-registry

## Overview

Auto-discovers and catalogs available skills for the orchestrator. Sub-agents load this registry to know what skills are available without manual configuration.

## When to Use This Skill

- **SDD Phases**: Every SDD sub-agent should load this first
- **Task Planning**: When you need to know what domain skills exist
- **Skill Discovery**: When user mentions a domain without an explicit skill
- **Onboarding**: Run `/skill-registry` after installing/removing skills

## Step 1: Load Skill Registry

### Priority Order:
1. Try Engram: `mem_search(query: "skill-registry", project: "{project}")`
2. If not found, try filesystem: read `.atl/skill-registry.md`
3. If neither exists, run registry scan (this skill)

### If Running Registry Scan:

## Step 2: Scan Installed Skills

### 2.1 Scan Global Skills Directory
```bash
ls -la ~/.agents/skills/
```

### 2.2 Scan Project Skills
Check for:
- `.agents/skills/` in project root
- `./skills/` in project root
- Any skill directories in project

### 2.3 Detect Project Conventions
Look for these files in project root:
- `AGENTS.md` - Orchestrator configuration
- `CLAUDE.md` - Claude-specific instructions
- `.cursorrules` - Cursor rules
- `opencode.json` - OpenCode config
- `.opencode/` - OpenCode directory
- `agents/` - Custom agents

## Step 3: Build Registry

### 3.1 Create Skill Table

Format each skill as:

```
| Trigger | Skill Name | Path | Description |
|---------|------------|------|-------------|
| react-* | frontend-react | ~/.agents/skills/frontend-react | React + TanStack + Tailwind |
| backend-* | backend-elysia | ~/.agents/skills/backend-elysia | Elysia + Drizzle + Better Auth |
```

### 3.2 Common Triggers Mapping

| Domain | Trigger Pattern | Skill |
|--------|-----------------|-------|
| React | react, frontend, ui, component | frontend-react |
| Backend | backend, api, server, elysia | backend-elysia |
| Database | db, database, drizzle, postgres | drizzle-orm |
| Auth | auth, authentication, login | better-auth-best-practices |
| Testing | test, testing, vitest, jest | testing-workflow, vitest |
| Design | ui, ux, design, component | ui-ux-pro-max |
| DevOps | docker, deploy, ci, cd | docker-openserve |
| SQL | sql, query, database | postgresql-optimization |

## Step 4: Validate Skills

### 4.1 Check Each Skill is Accessible
For each skill in registry:
1. Verify the path exists
2. Verify SKILL.md exists inside
3. If either fails, exclude from registry and log warning

### 4.2 Test Skill Loading
Optional: Try loading one skill to verify the system works

## Step 5: Persist Registry

### 5.1 Write to Filesystem
Write `.atl/skill-registry.md` in project root with:

```markdown
# Skill Registry

Generated: {timestamp}

## Skills Table

| Trigger | Skill Name | Path | Description |
|---------|------------|------|-------------|
| ... | ... | ... | ... |

## Project Conventions

- AGENTS.md: {found/not found}
- CLAUDE.md: {found/not found}
- .cursorrules: {found/not found}
- opencode.json: {found/not found}

## Usage

Sub-agents should load this registry at Step 1.
```

### 5.2 Save to Engram (if available)
```bash
mem_save(
  title: "skill-registry",
  content: "Full registry content",
  type: "config",
  project: "{project}"
)
```

## Result Contract

**Status**: success | partial | blocked
**Summary**: What the registry contains and where it's stored
**Artifacts**: 
- `.atl/skill-registry.md` (filesystem)
- Engram artifact if available
**Next**: sdd-explore or the original task
**Risks**: Any skills that couldn't be validated

## Tips

- Always load skill-registry FIRST in any SDD phase
- Use triggers to match user requests to skills
- Update registry after installing new skills
- The orchestrator does NOT need to pre-resolve skill paths - sub-agents discover them
