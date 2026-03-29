# Shared: Persistence Contract

## Overview

Defines how SDD artifacts are persisted and retrieved across sessions. Supports multiple persistence modes with deterministic fallback.

## Persistence Modes

| Mode | Storage | Cross-Session | Use Case |
|------|---------|---------------|-----------|
| `engram` | Engram memory | Yes | Default, maximum persistence |
| `openspec` | Filesystem (.atl/) | Yes | When Engram unavailable |
| `hybrid` | Both | Yes | Redundant, maximum safety |
| `none` | None | No | Stateless, testing |

## Mode Resolution

Resolution order (first wins):

1. **Environment Variable**: `PERSISTENCE_MODE`
2. **Project Config**: `.opencode/persistence.yaml`
3. **Global Config**: `~/.config/opencode/persistence.yaml`
4. **Default**: `engram`

### Resolution Example

```typescript
function resolvePersistenceMode(): string {
  // 1. Check env var
  const envMode = process.env.PERSISTENCE_MODE;
  if (envMode && ['engram', 'openspec', 'hybrid', 'none'].includes(envMode)) {
    return envMode;
  }
  
  // 2. Check project config
  const projectConfig = read('.opencode/persistence.yaml');
  if (projectConfig?.mode) {
    return projectConfig.mode;
  }
  
  // 3. Check global config
  const globalConfig = read('~/.config/opencode/persistence.yaml');
  if (globalConfig?.mode) {
    return globalConfig.mode;
  }
  
  // 4. Default
  return 'engram';
}
```

## Sub-Agent Context Protocol

### Starting a Sub-Agent

Every sub-agent should:

1. **Load Persistence Mode**: Check config to know where to read/write
2. **Load Skill Registry**: First call to discover available skills
3. **Check for Existing Artifacts**: Before creating new ones

### Artifact Discovery

**Priority order:**
1. Engram: `mem_search(query: "sdd/{change-name}/{artifact-type}")`
2. Filesystem: Check `.atl/changes/{change-name}/{artifact-type}.md`
3. If neither: Create new

### Saving Artifacts

**For Engram mode:**
```typescript
mem_save(
  title: "sdd/{change-name}/{artifact-type}",
  content: "Full artifact content",
  type: "artifact",
  project: "{project}",
  topic_key: "sdd/{change-name}/{artifact-type}"
);
```

**For Openspec mode:**
```bash
write(
  filePath: ".atl/changes/{change-name}/{artifact-type}.md",
  content: "Full artifact content"
);
```

**For Hybrid mode:**
Do both - redundancy ensures safety

**For None mode:**
- Read: Check memory only (no disk)
- Write: Log to console only
- Useful for testing

## Fallback Chain

When an operation fails:

```
Primary storage fails → Fallback to secondary → Fallback to none
```

### Example

```typescript
async function saveArtifact(artifact: Artifact): Promise<void> {
  const mode = resolvePersistenceMode();
  
  if (mode === 'engram' || mode === 'hybrid') {
    try {
      await mem_save(artifact);
      return;
    } catch (e) {
      console.warn('Engram save failed, falling back:', e);
    }
  }
  
  if (mode === 'openspec' || mode === 'hybrid') {
    try {
      await writeToFilesystem(artifact);
      return;
    } catch (e) {
      console.warn('Filesystem save failed:', e);
    }
  }
  
  if (mode === 'none') {
    console.log('Artifact (not saved):', artifact);
    return;
  }
  
  throw new Error('All persistence methods failed');
}
```

## Cross-Model Compatibility

### For Claude Models

Use Engram calls directly - native support:
- `mem_search()`
- `mem_save()`
- `mem_get_observation()`

### For Non-Claude Models

**Inline critical calls** - don't chain:

❌ BAD (multi-hop):
```
Read: engram-convention.md
Read: memory-engram/SKILL.md
Call: mem_save()
```

✅ GOOD (inline):
```
mem_save(
  title: "sdd/{change-name}/proposal",
  content: "...",
  type: "artifact"
)
```

### Convention Files Reference

When you need to reference this contract:

**For Claude models:**
- This file is automatically loaded
- Direct Engram calls work

**For other models:**
- Copy relevant sections into your skill
- Inline all Engram calls
- Don't assume multi-hop file reads work

## Mode-Specific Behavior

| Operation | engram | openspec | hybrid | none |
|-----------|--------|----------|--------|------|
| Read artifact | Engram API | File read | Both | Memory only |
| Save artifact | Engram API | File write | Both | Console log |
| Search | Engram FTS | Grep | Both | None |
| Get observation | Engram API | File read | Both | None |

## Configuration Files

### Project: `.opencode/persistence.yaml`
```yaml
mode: engram  # engram | openspec | hybrid | none
```

### Global: `~/.config/opencode/persistence.yaml`
```yaml
mode: engram
```

## Environment Variables

| Variable | Values | Description |
|----------|--------|-------------|
| `PERSISTENCE_MODE` | engram/openspec/hybrid/none | Override all configs |

## Testing

To test persistence without affecting real data:

```bash
PERSISTENCE_MODE=none your-command
```

This runs completely stateless - nothing is saved.
