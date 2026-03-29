# Shared: Openspec Convention

## Overview

Defines filesystem paths, directory structure, and conventions for SDD artifacts stored on disk. Used when persistence mode is `openspec` or `hybrid`.

## Directory Structure

### Root: `.atl/`

All SDD artifacts live under `.atl/` (Agent Teams Lite) in project root.

```
project-root/
├── .atl/                    # Root for all SDD artifacts
│   ├── skill-registry.md    # Skill registry (auto-generated)
│   ├── changes/             # Change-specific artifacts
│   │   └── {change-name}/
│   │       ├── proposal.md
│   │       ├── spec.md
│   │       ├── design.md
│   │       ├── tasks.md
│   │       ├── verification.md
│   │       └── archive.md
│   └── archives/            # Archived completed changes
│       └── {date}/
│           └── {change-name}/
│               └── ...
```

## File Paths

### Change Directory

```
.atl/changes/{change-name}/
```

### Artifact Files

| Artifact | Path | Description |
|----------|------|-------------|
| Proposal | `.atl/changes/{change-name}/proposal.md` | Proposal document |
| Spec | `.atl/changes/{change-name}/spec.md` | Delta specification |
| Design | `.atl/changes/{change-name}/design.md` | Architecture design |
| Tasks | `.atl/changes/{change-name}/tasks.md` | Task checklist |
| Verification | `.atl/changes/{change-name}/verification.md` | Test results |
| Archive | `.atl/changes/{change-name}/archive.md` | Final merged artifact |

### Change Name Format

- **Lowercase**: `add-oauth-flow`
- **Kebab-case**: Replace spaces with hyphens
- **No special characters**: Letters, numbers, hyphens only

## Archive Layout

### Structure

```
.atl/archives/{YYYY-MM-DD}/{change-name}/
├── proposal.md          # Original proposal
├── spec.md             # Final spec (merged)
├── design.md           # Architecture design
├── tasks.md            # Completed tasks
├── verification.md     # Test results
├── artifacts/          # Supporting files
│   ├── diagrams/
│   ├── api-schemas/
│   └── screenshots/
└── metadata.json        # Archive metadata
```

### Archive Metadata

```json
{
  "change-name": "add-oauth-flow",
  "archived-date": "2026-03-25",
  "original-proposal-date": "2026-03-20",
  "status": "completed",
  "pr-url": "https://github.com/...",
  "artifacts": [
    "proposal.md",
    "spec.md",
    "design.md"
  ]
}
```

## Configuration

### Project Config

`.opencode/persistence.yaml`:
```yaml
mode: openspec
```

### Custom Root (optional)

```yaml
mode: openspec
root: .sdd-artifacts  # Custom directory instead of .atl
```

## File Templates

### Proposal Template

```markdown
# Proposal: {change-name}

## Intent
...

## Scope
### In Scope
- ...

### Out of Scope
- ...

### Dependencies
- ...

## Approach
...

## Rollback Plan
...

## Risks
| Risk | Impact | Mitigation |
|------|--------|------------|
| ... | ... | ... |

---
*Created: {timestamp}*
*Last Updated: {timestamp}*
```

### Spec Template

```markdown
# Spec: {change-name}

## Overview
...

## Changes

### ADDED
```diff
+ New component or feature
```

### MODIFIED
```diff
- Old code
+ New code
```

### REMOVED
```diff
- Old code
```

## Scenarios

### Scenario 1: {description}
**Given** {precondition}
**When** {action}
**Then** {result}

---
*Spec Version: {version}*
*Created: {timestamp}*
```

### Tasks Template

```markdown
# Tasks: {change-name}

## Phase 1: {phase-name}
- [ ] Task 1
- [ ] Task 2

## Phase 2: {phase-name}
- [ ] Task 3
- [ ] Task 4

---
*Last Updated: {timestamp}*
```

## Reading Artifacts

### From Filesystem

```typescript
import { read } from 'fs/promises';

async function loadProposal(changeName: string) {
  const path = `.atl/changes/${changeName}/proposal.md`;
  const content = await read(path, 'utf-8');
  return content;
}
```

### From Engram (Hybrid Mode)

```typescript
// Check Engram first
const results = await mem_search({
  query: `sdd/${changeName}/proposal`,
  type: 'artifact'
});

if (results.length > 0) {
  return await mem_get_observation({ id: results[0].id });
}

// Fallback to filesystem
return await loadProposal(changeName);
```

## Writing Artifacts

### To Filesystem

```typescript
import { writeFile, mkdir } from 'fs/promises';
import { dirname } from 'path';

async function saveProposal(changeName: string, content: string) {
  const dir = `.atl/changes/${changeName}`;
  await mkdir(dir, { recursive: true });
  
  const path = `${dir}/proposal.md`;
  await writeFile(path, content, 'utf-8');
  
  return path;
}
```

### To Both (Hybrid)

```typescript
async function saveArtifact(changeName: string, type: string, content: string) {
  // Save to filesystem
  const path = await saveToFilesystem(changeName, type, content);
  
  // Also save to Engram if available
  try {
    await mem_save({
      title: `sdd/${changeName}/${type}`,
      content,
      type: 'artifact',
      topic_key: `sdd/${changeName}/${type}`
    });
  } catch (e) {
    console.warn('Engram save failed:', e);
  }
  
  return path;
}
```

## Migration

### From Old Structure

If you have artifacts in old locations:

| Old | New |
|-----|-----|
| `proposals/{name}.md` | `.atl/changes/{name}/proposal.md` |
| `specs/{name}.md` | `.atl/changes/{name}/spec.md` |
| `.specs/{name}/` | `.atl/changes/{name}/` |

### Migration Script

```bash
# Example migration
mkdir -p .atl/changes/{name}
mv proposals/{name}.md .atl/changes/{name}/proposal.md
mv specs/{name}.md .atl/changes/{name}/spec.md
rmdir proposals specs 2>/dev/null || true
```

## Index Files

### Change Index

`.atl/changes/index.md`:
```markdown
# Changes Index

| Change | Status | Created | Updated |
|--------|--------|---------|---------|
| [add-oauth-flow](./add-oauth-flow/proposal.md) | completed | 2026-03-20 | 2026-03-25 |
| [fix-login-bug](./fix-login-bug/proposal.md) | in-progress | 2026-03-24 | 2026-03-24 |

---
*Generated: {timestamp}*
```

## Cleanup

### Remove Old Archives

```bash
# Remove archives older than 90 days
find .atl/archives -type d -mtime +90 -exec rm -rf {} +
```

### Verify Integrity

```bash
# Check all artifacts have required files
for dir in .atl/changes/*/; do
  for file in proposal.md spec.md design.md; do
    if [ ! -f "$dir/$file" ]; then
      echo "Missing: $dir/$file"
    fi
  done
done
```
