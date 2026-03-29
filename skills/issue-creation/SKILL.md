# Skill: issue-creation

## Overview

Creates and manages issues in project management tools (Plane, Gitea) from SDD tasks. Links issues to SDD artifacts for traceability.

## When to Use This Skill

- Converting SDD tasks to trackable issues
- Creating issues for bugs or features
- Syncing SDD state with project management
- Updating issue status based on task progress

## Prerequisites

- Tasks should be defined (from sdd-tasks)
- Plane or Gitea integration should be available

## Step 1: Load Skill Registry

First, load available skills:
1. Try: `mem_search(query: "skill-registry", project: "{project}")`

## Step 2: Determine Issue Tracker

### 2.1 Check Available Tools

**For Plane:**
```bash
plane_list_projects(workspace_slug: "{workspace}")
```

**For Gitea:**
```bash
gitea_list_issues(owner: "{owner}", repo: "{repo}")
```

### 2.2 Select Target
- Use Plane for internal tracking
- Use Gitea for public/open source projects

## Step 3: Prepare Issue Data

### 3.1 Gather Task Information

From SDD tasks:
- Task title
- Task description
- Priority
- Dependencies

### 3.2 Format for Target Platform

**Plane format:**
```typescript
{
  name: "Task title",
  description_html: "<p>Description</p>",
  priority: "urgent" | "high" | "medium" | "low",
  state: "{state-uuid}",
  labels: ["label-id-1"],
  assignees: ["user-uuid"]
}
```

**Gitea format:**
```typescript
{
  title: "Task title",
  body: "Description",
  labels: ["label1,label2"]
}
```

### 3.3 Include SDD Context

Add references to SDD artifacts:
- Proposal: `.atl/changes/{change-name}/proposal.md`
- Spec: `.atl/changes/{change-name}/spec.md`
- Design: `.atl/changes/{change-name}/design.md`

## Step 4: Create Issues

### 4.1 For Plane

```bash
plane_create_work_item(
  project_id: "{project-id}",
  name: "{title}",
  description_html: "{description}",
  priority: "{priority}",
  labels: ["{label-ids}"]
)
```

### 4.2 For Gitea

```bash
gitea_issue_write(
  method: "create",
  owner: "{owner}",
  repo: "{repo}",
  title: "{title}",
  body: "{description}"
)
```

### 4.3 Handle Batch Creation

For multiple tasks:
1. Create issues sequentially or in small batches
2. Track created issue IDs
3. Update task with issue reference

## Step 5: Link SDD Artifacts

### 5.1 Add Links to Issue

**For Plane:**
```bash
plane_create_work_item_link(
  project_id: "{project-id}",
  work_item_id: "{issue-id}",
  url: "path/to/artifact.md"
)
```

**For Gitea:**
Add to issue body:
```
## SDD Artifacts
- [Proposal](./proposal.md)
- [Spec](./spec.md)
- [Design](./design.md)
```

### 5.2 Update Task Status

If system supports it, link task to issue:
- Update task checklist to include issue ID
- Mark task as "linked to issue"

## Step 6: Track Progress

### 6.1 Monitor Issue State

**For Plane:**
```bash
plane_retrieve_work_item(
  project_id: "{project-id}",
  work_item_id: "{issue-id}"
)
```

**For Gitea:**
```bash
gitea_issue_read(
  method: "get",
  owner: "{owner}",
  repo: "{repo}",
  index: "{issue-number}"
)
```

### 6.2 Sync Status Changes

When issue status changes:
- Update SDD task status
- Log status change to Engram

## Step 7: Persist Links

### 7.1 Save Issue Mapping

```bash
mem_save(
  title: "issue-mapping-{change-name}",
  content: "Task ID -> Issue ID mappings",
  type: "config",
  project: "{project}"
)
```

### 7.2 Update Work Items

```bash
team_git_work_item_upsert(
  team_id: "{team_id}",
  task_id: "{task_id}",
  external_id: "{issue-id}",
  external_source: "plane" | "gitea"
)
```

## Result Contract

**Status**: success | partial | blocked
**Summary**: Created {n} issues in {platform}. Linked to SDD tasks.
**Artifacts**: 
- Created issues with IDs
- Issue mapping saved
- Work items updated
**Next**: Continue implementation or update tasks
**Risks**: API rate limits, authentication issues

## Tips

- Create issues from tasks, not the other way around
- Use consistent naming conventions
- Include SDD artifact links for traceability
- Set appropriate labels and priorities
- Don't spam - create what's needed
- Update status as work progresses
- Close issues when tasks are complete
