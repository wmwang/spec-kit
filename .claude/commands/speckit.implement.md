---
description: Execute the implementation plan by processing and executing all tasks defined in tasks.md. Pure skill version - no CLI installation required.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Pre-Execution Checks

**Check for extension hooks (before implementation)**:
- Check if `.specify/extensions.yml` exists in the project root.
- If it exists, read it and look for entries under the `hooks.before_implement` key.
- If the YAML cannot be parsed or is invalid, skip hook checking silently and continue normally.
- Filter to only hooks where `enabled: true`.
- For each hook without a `condition` field (or with empty condition), output based on its `optional` flag:
  - **Optional hook** (`optional: true`):
    ```
    ## Extension Hooks

    **Optional Pre-Hook**: {extension}
    Command: `/{command}`
    Description: {description}

    Prompt: {prompt}
    To execute: `/{command}`
    ```
  - **Mandatory hook** (`optional: false`):
    ```
    ## Extension Hooks

    **Automatic Pre-Hook**: {extension}
    Executing: `/{command}`
    EXECUTE_COMMAND: {command}

    Wait for the result of the hook command before proceeding to the Outline.
    ```
- If no hooks are registered or `.specify/extensions.yml` does not exist, skip silently.

## Outline

### Step 1: Discover the active feature

```bash
git branch --show-current
```

- If branch matches `[0-9]+-[a-z0-9-]+`: feature directory is `specs/{branch-name}/`
- Otherwise: scan `specs/` for subdirectories or ask user

Establish:
- `FEATURE_DIR` = `specs/{branch-name}/` (absolute path)

Verify `tasks.md` exists at `FEATURE_DIR/tasks.md`. If missing, instruct user to run `/speckit.tasks` first.

List available docs:
```bash
ls specs/{branch-name}/
```

### Step 2: Check checklists status

If `FEATURE_DIR/checklists/` exists, scan all checklist files:
- Count total items (lines matching `- [ ]` or `- [X]` or `- [x]`)
- Count completed items (lines matching `- [X]` or `- [x]`)
- Count incomplete items (lines matching `- [ ]`)

Create a status table:

```text
| Checklist    | Total | Completed | Incomplete | Status  |
|--------------|-------|-----------|------------|---------|
| ux.md        | 12    | 12        | 0          | ✓ PASS  |
| security.md  | 8     | 5         | 3          | ✗ FAIL  |
```

- **If any checklist is incomplete**:
  - Display the table
  - **STOP** and ask: "Some checklists are incomplete. Do you want to proceed with implementation anyway? (yes/no)"
  - Wait for user response before continuing
  - If user says "no" / "wait" / "stop": halt execution
  - If user says "yes" / "proceed" / "continue": proceed to Step 3

- **If all checklists are complete**: Display table and automatically proceed to Step 3

### Step 3: Load implementation context

- **REQUIRED**: Read `tasks.md` for the complete task list and execution plan
- **REQUIRED**: Read `plan.md` for tech stack, architecture, and file structure
- **IF EXISTS**: Read `data-model.md` for entities and relationships
- **IF EXISTS**: Read `contracts/` for API specifications and test requirements
- **IF EXISTS**: Read `research.md` for technical decisions and constraints
- **IF EXISTS**: Read `quickstart.md` for integration scenarios

### Step 4: Project Setup Verification

Check and create/verify ignore files based on actual project setup:

**Detection & Creation Logic**:

```bash
# Is it a git repo?
git rev-parse --git-dir 2>/dev/null
# Is there a Dockerfile?
ls Dockerfile* 2>/dev/null
# Is there ESLint config?
ls .eslintrc* eslint.config.* 2>/dev/null
# Is there Prettier config?
ls .prettierrc* 2>/dev/null
# Is there package.json?
ls package.json 2>/dev/null
```

Create appropriate ignore files if missing, or verify they contain essential patterns if they exist.

**Common Patterns by Technology** (from plan.md tech stack):
- **Node.js/JavaScript/TypeScript**: `node_modules/`, `dist/`, `build/`, `*.log`, `.env*`
- **Python**: `__pycache__/`, `*.pyc`, `.venv/`, `venv/`, `dist/`, `*.egg-info/`
- **Java**: `target/`, `*.class`, `*.jar`, `.gradle/`, `build/`
- **C#/.NET**: `bin/`, `obj/`, `*.user`, `*.suo`, `packages/`
- **Go**: `*.exe`, `*.test`, `vendor/`, `*.out`
- **Ruby**: `.bundle/`, `log/`, `tmp/`, `*.gem`, `vendor/bundle/`
- **Rust**: `target/`, `debug/`, `release/`, `*.rs.bk`
- **Universal**: `.DS_Store`, `Thumbs.db`, `*.tmp`, `*.swp`

### Step 5: Parse tasks.md

Extract from `tasks.md`:
- Task phases: Setup, Foundational, User Story phases, Polish
- Task dependencies: sequential vs parallel execution rules
- Task details: ID, description, file paths, parallel markers `[P]`
- Execution flow: order and dependency requirements

### Step 6: Execute implementation

Follow the task plan phase by phase:

- **Phase-by-phase execution**: Complete each phase before moving to the next
- **Respect dependencies**: Run sequential tasks in order, parallel tasks `[P]` can run together
- **Follow TDD approach if requested**: Execute test tasks before their corresponding implementation tasks
- **File-based coordination**: Tasks affecting the same files must run sequentially
- **Validation checkpoints**: Verify each phase completion before proceeding

**Implementation execution rules**:
1. **Setup first**: Initialize project structure, dependencies, configuration
2. **Tests before code** (if TDD): Write tests for contracts, entities, integration scenarios
3. **Core development**: Implement models, services, CLI commands, endpoints
4. **Integration work**: Database connections, middleware, logging, external services
5. **Polish and validation**: Unit tests, performance optimization, documentation

### Step 7: Progress tracking and error handling

- Report progress after each completed task
- Halt execution if any non-parallel task fails
- For parallel tasks `[P]`, continue with successful tasks, report failed ones
- Provide clear error messages with context for debugging
- Suggest next steps if implementation cannot proceed
- **IMPORTANT**: For completed tasks, mark the task off as `[x]` in the tasks file

### Step 8: Completion validation

- Verify all required tasks are completed
- Check that implemented features match the original specification
- Validate that tests pass and coverage meets requirements
- Confirm implementation follows the technical plan
- Report final status with summary of completed work

Note: This command assumes a complete task breakdown exists in tasks.md. If tasks are incomplete or missing, run `/speckit.tasks` first.

### Step 9: Check for after_implement extension hooks

Check `.specify/extensions.yml` for `hooks.after_implement` entries and process accordingly (same as Pre-Execution Checks pattern above).
