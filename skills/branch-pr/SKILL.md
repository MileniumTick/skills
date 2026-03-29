# Skill: branch-pr

## Overview

Manages Git branches and Pull Requests. Creates branches from tasks, manages PR lifecycle, and synchronizes with git-work-items.

## When to Use This Skill

- Creating a new branch for implementation
- Creating a PR after completing tasks
- Syncing branch state with task tracking
- Managing PR reviews and merges

## Prerequisites

- Git repository should be initialized
- Tasks should be defined (from sdd-tasks)

## Step 1: Load Skill Registry

First, load available skills:
1. Try: `mem_search(query: "skill-registry", project: "{project}")`

## Step 2: Analyze Current State

### 2.1 Check Git Status
```bash
git status
git branch -a
```

### 2.2 Identify Current Branch
- What branch are we on?
- Is there a base branch configured?

### 2.3 Check Existing Work Items
Use team tools to check git-work-items:
```bash
team_git_work_item_list(team_id: "{team_id}")
```

## Step 3: Branch Operations

### 3.1 Create Branch

For new implementation:
1. Determine branch name format:
   - Feature: `feature/{issue-id}-{short-description}`
   - Bugfix: `fix/{issue-id}-{short-description}`
   - Hotfix: `hotfix/{issue-id}-{short-description}`

2. Create from appropriate base:
```bash
git checkout -b {branch-name} {base-branch}
```

3. Persist work item:
```bash
team_git_work_item_upsert(
  team_id: "{team_id}",
  task_id: "{task_id}",
  branch_name: "{branch-name}",
  commit_batching_mode: "single"
)
```

### 3.2 Switch Branch
```bash
git checkout {branch-name}
```

### 3.3 Sync Branch
If branch exists remotely but not locally:
```bash
git fetch
git checkout {branch-name}
```

## Step 4: PR Operations

### 4.1 Create PR

When tasks are complete and ready for review:

**Prepare PR metadata:**
- Title: `{issue-id}: {description}`
- Body: Include:
  - Summary of changes
  - Related issues/tasks
  - Testing performed
  - Screenshots (if UI)

**Create PR:**
- Use GitHub/Gitea API or CLI
- Add appropriate labels
- Request reviewers

### 4.2 Update PR

When commits are added to the branch:
- Push changes: `git push`
- Update PR description if needed

### 4.3 Merge PR

After approval:
- Use merge strategy: merge, squash, or rebase
- Delete branch after merge (if configured)

### 4.4 Close PR

If PR is rejected or abandoned:
- Close without merging
- Note reason in comments

## Step 5: Handle Conflicts

### 5.1 Detect Conflicts
```bash
git fetch origin {base-branch}
git merge origin/{base-branch}
```

### 5.2 Resolve
If conflicts exist:
1. Identify conflicting files
2. Resolve each conflict
3. Commit resolution

## Step 6: Persist State

### 6.1 Update Work Item
```bash
team_git_work_item_upsert(
  team_id: "{team_id}",
  task_id: "{task_id}",
  branch_name: "{branch-name}",
  base_branch: "{base-branch}",
  pr_target_branch: "{target}",
  status: "in_progress" | "ready_for_review" | "pr_open" | "merged"
)
```

### 6.2 Save to Engram (optional)
```bash
mem_save(
  title: "branch-pr-{branch-name}",
  content: "Branch and PR state",
  type: "config"
)
```

## Result Contract

**Status**: success | partial | blocked
**Summary**: Branch `{branch-name}` created/updated. PR #{pr-number} {state}.
**Artifacts**: 
- Git branch: `{branch-name}`
- PR: {pr-url}
- Work item updated
**Next**: Continue implementation or merge
**Risks**: Conflicts, CI failures, review delays

## Tips

- Use consistent branch naming
- Always base on latest main/develop
- Keep branches focused and small
- Write meaningful commit messages
- Update PR description as you go
- Don't merge your own PRs without review
