---
description: Execute the implementation plan by processing all tasks in tasks.md. No CLI installation required.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Pre-Execution: Extension Hooks

Check `.specify/extensions.yml` for `hooks.before_implement`. For each enabled hook with no `condition`:
- `optional: true` → display and offer to run
- `optional: false` → display `EXECUTE_COMMAND: {command}` and wait for result before proceeding

Skip silently if file missing or unparseable.

## Outline

### Step 1: Discover the active feature

```bash
git branch --show-current
```

- Branch matches `[0-9]+-[a-z0-9-]+` → `FEATURE_DIR` = `specs/{BRANCH}`
- Otherwise: scan `specs/` or ask user

List available docs:
```bash
ls specs/{BRANCH}/
```

Verify `{FEATURE_DIR}/tasks.md` exists. If missing: instruct user to run `/speckit.tasks` first.

### Step 2: Check checklist gate

If `{FEATURE_DIR}/checklists/` exists, scan every `.md` file in it:

```
| Checklist    | Total | Done | Remaining | Status  |
|--------------|-------|------|-----------|---------|
| ux.md        | 12    | 12   | 0         | ✓ PASS  |
| security.md  | 8     | 5    | 3         | ✗ FAIL  |
```

- All pass → proceed automatically
- Any fail → **STOP** and ask: *"Some checklists are incomplete. Proceed anyway? (yes/no)"*
  - `no` / `wait` / `stop` → halt
  - `yes` / `proceed` → continue

### Step 3: Load implementation context

- **REQUIRED**: `tasks.md` — full task list and execution plan
- **REQUIRED**: `plan.md` — tech stack, architecture, file structure
- **IF EXISTS**: `data-model.md`, `contracts/`, `research.md`, `quickstart.md`

### Step 4: Verify project setup

Check and create ignore files based on the detected tech stack (from `plan.md`):

```bash
git rev-parse --git-dir 2>/dev/null  # → .gitignore needed?
ls Dockerfile* 2>/dev/null            # → .dockerignore needed?
ls .eslintrc* eslint.config.* 2>/dev/null  # → .eslintignore needed?
ls .prettierrc* 2>/dev/null           # → .prettierignore needed?
ls package.json 2>/dev/null           # → .npmignore needed?
```

Common patterns by language:
- **Node.js/TS**: `node_modules/`, `dist/`, `build/`, `*.log`, `.env*`
- **Python**: `__pycache__/`, `*.pyc`, `.venv/`, `venv/`, `dist/`, `*.egg-info/`
- **Java**: `target/`, `*.class`, `*.jar`, `.gradle/`, `build/`
- **C#/.NET**: `bin/`, `obj/`, `*.user`, `packages/`
- **Go**: `*.exe`, `*.test`, `vendor/`, `*.out`
- **Rust**: `target/`, `debug/`, `release/`, `*.rs.bk`
- **Swift**: `.build/`, `DerivedData/`, `*.swiftpm/`
- **Universal**: `.DS_Store`, `Thumbs.db`, `*.tmp`, `*.swp`

If a file exists: append only missing critical patterns. If missing: create with full set.

### Step 5: Execute tasks phase by phase

From `tasks.md`:
1. Parse all phases, tasks, dependencies, `[P]` markers
2. **Phase by phase**: complete each before the next
3. **Sequential tasks**: execute in order; halt on failure
4. **Parallel tasks `[P]`**: run concurrently (different files, no shared dependencies)
5. After each completed task: mark `[x]` in `tasks.md`
6. Report progress after each task

Execution order within a phase:
1. Setup / project structure
2. Tests (if TDD requested)
3. Models / data layer
4. Services / business logic
5. Endpoints / CLI / UI
6. Integration / middleware

### Step 6: Completion validation

- All required tasks marked `[x]`
- Implementation matches spec.md requirements
- Tests pass (if applicable)
- Final status report with summary of completed work

### Post-Execution: Extension Hooks

Check `.specify/extensions.yml` for `hooks.after_implement`. Process same as pre-execution hooks.
