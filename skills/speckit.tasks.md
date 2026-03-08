---
description: Generate a dependency-ordered tasks.md from existing planning artifacts. No CLI installation required.
handoffs:
  - label: Analyze For Consistency
    agent: speckit.analyze
    prompt: Run a project analysis for consistency
    send: true
  - label: Implement Project
    agent: speckit.implement
    prompt: Start the implementation in phases
    send: true
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Pre-Execution: Extension Hooks

Check `.specify/extensions.yml` for `hooks.before_tasks`. For each enabled hook with no `condition`:
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

### Step 2: Load planning documents

From `FEATURE_DIR`:
- **Required**: `plan.md` (tech stack, libraries, project structure)
- **Required**: `spec.md` (user stories with priorities P1/P2/P3...)
- **Optional**: `data-model.md`, `contracts/`, `research.md`, `quickstart.md`

If `plan.md` or `spec.md` missing, stop and instruct user to run the prerequisite command.

### Step 3: Generate task breakdown

1. From `plan.md`: extract tech stack, libraries, project structure
2. From `spec.md`: extract user stories and their priorities
3. If `data-model.md`: map entities to user stories
4. If `contracts/`: map contracts to user stories
5. If `research.md`: extract decisions for setup tasks

### Step 4: Write tasks.md

Write `{FEATURE_DIR}/tasks.md`:

```markdown
# Tasks: {FEATURE NAME}

**Input**: `specs/{BRANCH}/` — plan.md + spec.md required
**Format**: `- [ ] T{NNN} [P?] [US{N}?] Description — path/to/file.ext`
- `[P]` = can run in parallel (different files, no dependencies on incomplete tasks)
- `[US1]` = belongs to User Story 1 (maps to spec.md priorities)

---

## Phase 1: Setup

**Purpose**: Project initialization and shared infrastructure

- [ ] T001 Create project structure per plan.md
- [ ] T002 Initialize dependencies and configuration
- [ ] T003 [P] Configure linting/formatting

---

## Phase 2: Foundational

**Purpose**: Blocking prerequisites — MUST be complete before any user story

⚠️ No user story work begins until this phase is done.

- [ ] T004 {foundational task}
- [ ] T005 [P] {foundational task}

**Checkpoint**: Foundation complete

---

## Phase 3: User Story 1 — {Title} (Priority: P1) 🎯 MVP

**Goal**: {what this story delivers}
**Independent Test**: {how to verify this story alone}

- [ ] T010 [P] [US1] {description} — {path/to/file}
- [ ] T011 [US1] {description} — {path/to/file}

**Checkpoint**: User Story 1 independently testable

---

## Phase 4: User Story 2 — {Title} (Priority: P2)

{repeat pattern}

---

## Phase N: Polish & Cross-Cutting

- [ ] TXXX [P] Documentation updates
- [ ] TXXX Code cleanup and refactoring

---

## Dependencies

| Phase | Depends On | Notes |
|-------|-----------|-------|
| Setup (Ph1) | — | Start immediately |
| Foundational (Ph2) | Ph1 | Blocks all user stories |
| US1 (Ph3) | Ph2 | Independent of other stories |
| US2 (Ph4) | Ph2 | May integrate with US1 |
| Polish | All desired stories | Final phase |

## Implementation Strategy

**MVP** (User Story 1 only):
1. Phase 1: Setup → Phase 2: Foundational → Phase 3: US1
2. Validate independently, then deploy/demo

**Incremental**: each story adds value without breaking previous stories.
```

**Task format rules (REQUIRED)**:

Every task must follow exactly:
```
- [ ] T{NNN} [P?] [US{N}?] Description — path/to/file.ext
```

- ✅ `- [ ] T001 Create project structure`
- ✅ `- [ ] T005 [P] Auth middleware — src/middleware/auth.py`
- ✅ `- [ ] T012 [P] [US1] User model — src/models/user.py`
- ❌ `- [ ] Create User model` (missing ID)
- ❌ `T001 [US1] Create model` (missing checkbox)
- ❌ `- [ ] T001 [US1] Create model` (missing file path)

Setup/Foundational/Polish phases: no `[US{N}]` label.
User Story phases: `[US{N}]` label required.

### Step 5: Report

- Path to `tasks.md`
- Total tasks, tasks per story
- Parallel opportunities
- Suggested MVP scope (typically Phase 1+2+3)

### Post-Execution: Extension Hooks

Check `.specify/extensions.yml` for `hooks.after_tasks`. Process same as pre-execution hooks.
