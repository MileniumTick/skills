# Shared: Engram Convention

## Overview

Defines naming conventions and recovery protocols for Engram artifacts. Ensures deterministic artifact naming and reliable cross-session recovery.

## Artifact Naming Format

### Standard Format

```
sdd/{change-name}/{artifact-type}
```

### Components

| Component | Description | Example |
|-----------|-------------|---------|
| `sdd/` | Prefix for SDD artifacts | Always `sdd/` |
| `{change-name}` | Kebab-case name of the change | `add-oauth-flow` |
| `{artifact-type}` | Type of artifact | `proposal`, `spec`, `design`, `tasks` |

### Artifact Types

| Type | Description | Example |
|------|-------------|---------|
| `proposal` | Proposal document | `sdd/add-oauth-flow/proposal` |
| `spec` | Delta specification | `sdd/add-oauth-flow/spec` |
| `design` | Architecture design | `sdd/add-oauth-flow/design` |
| `tasks` | Task checklist | `sdd/add-oauth-flow/tasks` |
| `verification` | Test results | `sdd/add-oauth-flow/verification` |
| `archive` | Final merged artifact | `sdd/add-oauth-flow/archive` |

## Topic Key Convention

### Purpose

`topic_key` enables updating evolving artifacts:
- Different topics = different observations
- Same topic_key = updates same observation

### Usage

```typescript
// First save - creates new observation
mem_save(
  title: "sdd/add-oauth-flow/proposal",
  content: "Proposal content...",
  type: "artifact",
  topic_key: "sdd/add-oauth-flow/proposal"
);

// Second save with same topic_key - updates observation
mem_save(
  title: "sdd/add-oauth-flow/proposal",
  content: "Updated proposal content...",
  type: "artifact",
  topic_key: "sdd/add-oauth-flow/proposal"
);
```

### Topic Key vs Title

| Field | Purpose | Uniqueness |
|-------|---------|-------------|
| `title` | Display name, searchable | Can be duplicated |
| `topic_key` | Stable ID for updates | Must be unique per scope |

## Two-Step Recovery Protocol

### Overview

When you need to find an artifact created by another skill:

### Step 1: Search

```typescript
// Search by change name
mem_search(
  query: "sdd/add-oauth-flow",
  project: "{project}",
  type: "artifact"
);
```

### Step 2: Get Full Content

```typescript
// Get observation by ID (from search results)
mem_get_observation(
  id: {observation-id}
);
```

### Recovery Example

```typescript
async function loadSpec(changeName: string) {
  // Step 1: Find the spec
  const results = await mem_search({
    query: `sdd/${changeName}/spec`,
    project: currentProject,
    type: "artifact",
    limit: 5
  });
  
  if (results.length === 0) {
    throw new Error(`No spec found for ${changeName}`);
  }
  
  // Step 2: Get full content
  const spec = await mem_get_observation({
    id: results[0].id
  });
  
  return spec.content;
}
```

## Inline Calls for Non-Claude Models

### Critical Calls to Inline

These calls should NOT reference this file - inline them directly:

```typescript
// ❌ DON'T do this:
// Read: engram-convention.md
// Then: mem_save(...)

// ✅ DO this instead - inline the call:
mem_save({
  title: "sdd/my-change/proposal",
  content: "Full proposal...",
  type: "artifact",
  project: myProject,
  topic_key: "sdd/my-change/proposal"
});
```

### Why Inline?

Multi-hop chains fail on non-Claude models:
- Model reads this file
- Model reads memory-engram skill
- Model calls mem_save()

Too many hops = failures

### Pattern to Follow

In every skill, include inline versions of:
- `mem_search()`
- `mem_save()`
- `mem_get_observation()`

## Save Contract

### Required Fields

```typescript
mem_save({
  title: string,        // "sdd/{change}/{type}"
  content: string,      // Full artifact content
  type: "artifact",     // Always "artifact" for SDD
  project: string,      // Current project name
  topic_key: string,   // Same as title for updates
});
```

### Optional Fields

```typescript
mem_save({
  title: string,
  content: string,
  type: "artifact",
  project: string,
  topic_key: string,
  session_id: string,   // Optional: specific session
  scope: "project",     // "project" or "personal"
});
```

## Search Patterns

### Find All Artifacts for a Change

```typescript
mem_search({
  query: "sdd/add-oauth-flow",
  project: currentProject,
  type: "artifact",
  limit: 20
});
```

### Find Specific Artifact Type

```typescript
mem_search({
  query: "sdd/add-oauth-flow proposal",
  project: currentProject,
  type: "artifact",
  limit: 5
});
```

### Find Recent Artifacts

```typescript
mem_search({
  query: "sdd",
  project: currentProject,
  type: "artifact",
  limit: 10
});
```

## Cross-Session Persistence

### How It Works

- Engram persists across sessions automatically
- Artifacts are searchable by any agent
- Session ID links artifacts to creation context

### Checking Persistence

```typescript
// Verify Engram is available
mem_context({ limit: 1 });

// If it fails, fallback to filesystem
// (see persistence-contract.md)
```

## Migration from Old Format

If you have artifacts with old naming:

| Old Format | New Format |
|------------|------------|
| `proposal-add-oauth` | `sdd/add-oauth/proposal` |
| `spec-feature-x` | `sdd/feature-x/spec` |

When found, update to new format:

```typescript
const old = await mem_get_observation({ id: oldId });
await mem_save({
  title: `sdd/${changeName}/${type}`,
  content: old.content,
  type: "artifact",
  topic_key: `sdd/${changeName}/${type}`
});
```
