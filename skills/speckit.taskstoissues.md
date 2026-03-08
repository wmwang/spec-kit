---
description: Convert tasks.md into GitHub Issues on the remote repository. No CLI installation required.
tools: ['github/github-mcp-server/issue_write']
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

### Step 1: Discover the active feature and tasks

```bash
git branch --show-current
```

- Branch matches `[0-9]+-[a-z0-9-]+` → `FEATURE_DIR` = `specs/{BRANCH}`
- Otherwise: scan `specs/` or ask user

Tasks file: `{FEATURE_DIR}/tasks.md` (absolute path). If missing: instruct user to run `/speckit.tasks` first.

### Step 2: Verify GitHub remote

```bash
git config --get remote.origin.url
```

> [!CAUTION]
> **ONLY PROCEED IF THE REMOTE IS A GITHUB URL** (contains `github.com`).
> If the remote is not GitHub, stop immediately.

### Step 3: Create GitHub Issues

For each task in `tasks.md`, use the GitHub MCP server to create a new issue in the repository that matches the remote URL.

> [!CAUTION]
> **UNDER NO CIRCUMSTANCES CREATE ISSUES IN REPOSITORIES THAT DO NOT MATCH THE REMOTE URL.**
