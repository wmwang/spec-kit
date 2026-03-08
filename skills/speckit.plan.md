---
description: Generate implementation planning artifacts (research.md, data-model.md, contracts/) from a feature spec. No CLI installation required.
handoffs:
  - label: Create Tasks
    agent: speckit.tasks
    prompt: Break the plan into tasks
    send: true
  - label: Create Checklist
    agent: speckit.checklist
    prompt: Create a checklist for the following domain...
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

### Step 1: Discover the active feature

```bash
git branch --show-current
```

- Branch matches `[0-9]+-[a-z0-9-]+` → `FEATURE_DIR` = `specs/{BRANCH}`
- Otherwise: scan `specs/` for most recently modified subdirectory, or ask user

Variables:
- `BRANCH` = current branch
- `FEATURE_DIR` = `specs/{BRANCH}` (absolute path)
- `FEATURE_SPEC` = `{FEATURE_DIR}/spec.md`
- `IMPL_PLAN` = `{FEATURE_DIR}/plan.md`

Verify `FEATURE_SPEC` exists. If not: instruct user to run `/speckit.specify` first.

Create directories:
```bash
mkdir -p specs/{BRANCH}/contracts
```

### Step 2: Initialize plan.md

If `IMPL_PLAN` does not exist, create it with this structure:

```markdown
# Implementation Plan: {FEATURE NAME}

**Branch**: `{BRANCH}` | **Date**: {TODAY} | **Spec**: [spec.md](spec.md)

## Summary

{primary requirement + chosen technical approach}

## Technical Context

**Language/Version**: {e.g. Python 3.11 | NEEDS CLARIFICATION}
**Primary Dependencies**: {e.g. FastAPI, SQLAlchemy | NEEDS CLARIFICATION}
**Storage**: {e.g. PostgreSQL | N/A}
**Testing**: {e.g. pytest | NEEDS CLARIFICATION}
**Target Platform**: {e.g. Linux server, web browser | NEEDS CLARIFICATION}
**Project Type**: {library / cli / web-service / mobile-app / desktop-app | NEEDS CLARIFICATION}
**Performance Goals**: {e.g. <200ms p95 | NEEDS CLARIFICATION}
**Constraints**: {e.g. offline-capable, <100MB memory | NEEDS CLARIFICATION}
**Scale/Scope**: {e.g. 10k users | NEEDS CLARIFICATION}

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

{gates from .specify/memory/constitution.md, or "No constitution defined"}

## Project Structure

### Documentation

\`\`\`
specs/{BRANCH}/
├── plan.md         ← this file
├── research.md     ← Phase 0 output
├── data-model.md   ← Phase 1 output
├── quickstart.md   ← Phase 1 output
├── contracts/      ← Phase 1 output
└── tasks.md        ← /speckit.tasks output
\`\`\`

### Source Code

\`\`\`
{concrete directory tree based on tech stack}
\`\`\`

## Complexity Tracking

> Fill ONLY if Constitution Check has violations that must be justified

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|--------------------------------------|
```

### Step 3: Fill Technical Context

Detect from existing repo files:
- `package.json` → Node.js; check `dependencies` for framework
- `pyproject.toml` / `setup.py` → Python; check `[tool.poetry.dependencies]` or `install_requires`
- `go.mod` → Go
- `Cargo.toml` → Rust
- `*.csproj` / `*.sln` → .NET
- `build.gradle` / `pom.xml` → Java/Kotlin

Mark unresolvable items as `NEEDS CLARIFICATION`.

### Step 4: Constitution Check

Read `.specify/memory/constitution.md` if it exists. Evaluate each principle's gates. Mark PASS / FAIL / N/A. ERROR if FAIL and not justified in Complexity Tracking.

### Step 5: Phase 0 — Research

For each `NEEDS CLARIFICATION` in Technical Context, research and resolve it using repo analysis and best-practice reasoning.

Write `{FEATURE_DIR}/research.md`:

```markdown
# Research: {FEATURE NAME}

**Date**: {TODAY}

## Decisions

### {Decision Topic}

- **Decision**: {chosen option}
- **Rationale**: {why}
- **Alternatives considered**: {what else was evaluated}

## Technology Choices

{resolved tech stack summary}

## Key Findings

{constraints, patterns, or risks discovered}
```

### Step 6: Phase 1 — Design

**Prerequisite**: `research.md` complete.

**a. Data model** — write `{FEATURE_DIR}/data-model.md`:

```markdown
# Data Model: {FEATURE NAME}

## Entities

### {Entity Name}

**Description**: {what this represents}

**Fields**:
- `{field}` ({type}): {description}

**Relationships**:
- {relationship description}

**Validation Rules**:
- {rule}

**State Transitions** (if applicable):
- {StateA} → {StateB}: {trigger}
```

**b. Interface contracts** — only if the project exposes external interfaces.

Determine appropriate format from project type:
- Web service → REST or GraphQL endpoint docs in `contracts/`
- Library → public API signatures in `contracts/api.md`
- CLI tool → command schema in `contracts/cli.md`
- Skip entirely for purely internal tools

**c. Agent context update** — detect active AI agent and update its context file:

```bash
# Check in this order:
ls CLAUDE.md 2>/dev/null          # → Claude Code
ls .cursor/rules/ 2>/dev/null     # → Cursor
ls .github/copilot-instructions.md 2>/dev/null  # → GitHub Copilot
ls .windsurfrules 2>/dev/null     # → Windsurf
ls .roo/rules/ 2>/dev/null        # → Roo Code
ls AGENTS.md 2>/dev/null          # → generic fallback
```

Add only new technology from the current plan. Preserve existing content.

### Step 7: Update plan.md

Fill all placeholders with resolved values. Update Constitution Check with actual pass/fail.

### Step 8: Report

- Branch: `{BRANCH}`
- Plan: `{IMPL_PLAN}`
- Generated: `research.md` ✅, `data-model.md` ✅ (or N/A), `contracts/` ✅ (or N/A)
- Next: `/speckit.tasks`
