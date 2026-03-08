---
description: Execute the implementation planning workflow to generate design artifacts (research.md, data-model.md, contracts/). Pure skill version - no CLI installation required.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

### Step 1: Discover the active feature

Determine the current feature by running:

```bash
git branch --show-current
```

- If the branch name matches the pattern `[0-9]+-[a-z0-9-]+` (e.g., `1-user-auth`, `3-oauth2-api`), the feature directory is `specs/{branch-name}/`
- If the branch is `main`, `master`, or another non-feature branch, scan `specs/` for subdirectories and pick the most recently modified one, or ask the user which feature to plan
- If user input specifies a feature name, use that instead

Establish variables:
- `BRANCH` = current git branch
- `FEATURE_DIR` = `specs/{branch-name}/` (absolute path from repo root)
- `FEATURE_SPEC` = `FEATURE_DIR/spec.md`
- `IMPL_PLAN` = `FEATURE_DIR/plan.md`

Verify `FEATURE_SPEC` exists. If not, instruct the user to run `/speckit.specify` first.

### Step 2: Set up plan file

Copy the plan template structure into `IMPL_PLAN` if it doesn't exist yet:

```markdown
# Implementation Plan: [FEATURE]

**Branch**: `[###-feature-name]` | **Date**: [TODAY] | **Spec**: [link to spec.md]
**Input**: Feature specification from `specs/[###-feature-name]/spec.md`

## Summary

[Extract from feature spec: primary requirement + technical approach from research]

## Technical Context

**Language/Version**: [e.g., Python 3.11, Node.js 20, Go 1.22 or NEEDS CLARIFICATION]
**Primary Dependencies**: [e.g., FastAPI, Express, Gin or NEEDS CLARIFICATION]
**Storage**: [e.g., PostgreSQL, SQLite, files or N/A]
**Testing**: [e.g., pytest, Jest, go test or NEEDS CLARIFICATION]
**Target Platform**: [e.g., Linux server, web browser, iOS 15+ or NEEDS CLARIFICATION]
**Project Type**: [e.g., library/cli/web-service/mobile-app/desktop-app or NEEDS CLARIFICATION]
**Performance Goals**: [e.g., 1000 req/s, sub-200ms response or NEEDS CLARIFICATION]
**Constraints**: [e.g., <200ms p95, <100MB memory, offline-capable or NEEDS CLARIFICATION]
**Scale/Scope**: [e.g., 10k users, 50 screens or NEEDS CLARIFICATION]

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

[Gates determined based on constitution file at `.specify/memory/constitution.md`]

## Project Structure

### Documentation (this feature)

\`\`\`text
specs/[###-feature]/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/           # Phase 1 output
└── tasks.md             # Phase 2 output (/speckit.tasks command)
\`\`\`

### Source Code (repository root)

\`\`\`text
[Concrete directory structure based on tech stack]
\`\`\`

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
```

Also create the `contracts/` directory:

```bash
mkdir -p specs/{BRANCH}/contracts
```

### Step 3: Load context

Read `FEATURE_SPEC` and `.specify/memory/constitution.md` (if it exists). Load the initialized `IMPL_PLAN`.

### Step 4: Fill Technical Context

Analyze the feature spec to determine:
- Language/runtime from existing repo files (check `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `*.csproj`, etc.)
- Frameworks and dependencies already in use
- Storage solutions already present
- Test frameworks in use
- Mark unknowns as "NEEDS CLARIFICATION"

### Step 5: Constitution Check

Read `.specify/memory/constitution.md` if it exists. Evaluate all gates (ERROR if violations unjustified). Fill the Constitution Check section in the plan.

### Step 6: Phase 0 — Outline & Research

1. **Extract unknowns from Technical Context**:
   - For each NEEDS CLARIFICATION → research task
   - For each dependency → best practices task
   - For each integration → patterns task

2. **Resolve all unknowns** through codebase analysis, reasoning about best practices, and considering the project context.

3. **Generate and write `FEATURE_DIR/research.md`**:

```markdown
# Research: [FEATURE NAME]

**Purpose**: Resolve unknowns and technical decisions before implementation
**Date**: [TODAY]

## Decisions

### [Decision Topic]

- **Decision**: [what was chosen]
- **Rationale**: [why chosen]
- **Alternatives considered**: [what else was evaluated]

[Repeat for each decision]

## Technology Choices

[Summary of resolved tech stack]

## Key Findings

[Important constraints, patterns, or risks discovered]
```

**Output**: `research.md` with all NEEDS CLARIFICATION resolved

### Step 7: Phase 1 — Design & Contracts

**Prerequisites:** `research.md` complete

1. **Extract entities from feature spec** → write `FEATURE_DIR/data-model.md`:

```markdown
# Data Model: [FEATURE NAME]

## Entities

### [Entity Name]

**Description**: [What this entity represents]

**Fields**:
- `[field_name]` ([type]): [description]
- `[field_name]` ([type]): [description]

**Relationships**:
- [Relationship description]

**Validation Rules**:
- [Rule 1]
- [Rule 2]

**State Transitions** (if applicable):
- [State A] → [State B]: [trigger]
```

2. **Define interface contracts** (if project has external interfaces) → `FEATURE_DIR/contracts/`:
   - Identify what interfaces the project exposes to users or other systems
   - Document the contract format appropriate for the project type
   - Examples: REST API endpoints for web services, CLI command schemas, public API for libraries
   - Skip if project is purely internal

3. **Detect which AI agent is active** by checking for these files in order:
   - `CLAUDE.md` → Claude Code
   - `.cursor/rules/` or `cursor.rules` → Cursor
   - `.github/copilot-instructions.md` → GitHub Copilot
   - `.windsurfrules` → Windsurf
   - `.roo/rules/` → Roo Code
   - `AGENTS.md` (generic) → Generic agent

   Then update the agent-specific context file with new technology from the current plan. Add only new technology; preserve existing content. Do not overwrite manual additions.

### Step 8: Update the plan file

Fill in all sections of `IMPL_PLAN` with concrete content derived from research and design:
- Replace Technical Context placeholders with resolved values
- Update Constitution Check with actual pass/fail status
- Fill Project Structure with concrete directory tree
- Write Summary from the feature spec's primary requirement

### Step 9: Report completion

Report:
- Branch name
- IMPL_PLAN path
- Generated artifacts:
  - `research.md` ✅
  - `data-model.md` ✅ (or N/A if no entities)
  - `contracts/` ✅ (or N/A if no external interfaces)
- Suggested next command: `/speckit.tasks`

## Key Rules

- Use absolute paths
- ERROR on gate failures or unresolved clarifications
- This command ends after Phase 1 planning — implementation tasks are handled by `/speckit.tasks`
