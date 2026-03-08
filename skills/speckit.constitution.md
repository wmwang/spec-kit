---
description: Create or update the project constitution — architectural principles that govern all planning and implementation. No CLI installation required.
handoffs:
  - label: Build Specification
    agent: speckit.specify
    prompt: Implement the feature specification based on the updated constitution. I want to build...
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

You are creating or updating the project constitution at `.specify/memory/constitution.md`.

### Step 1: Initialize if needed

```bash
ls .specify/memory/constitution.md 2>/dev/null || echo "missing"
```

If missing, create directory and bootstrap from this template:

```bash
mkdir -p .specify/memory
```

Then create `.specify/memory/constitution.md` with:

```markdown
# [PROJECT_NAME] Constitution

## Core Principles

### I. [PRINCIPLE_1_NAME]

[PRINCIPLE_1_DESCRIPTION]

### II. [PRINCIPLE_2_NAME]

[PRINCIPLE_2_DESCRIPTION]

### III. [PRINCIPLE_3_NAME]

[PRINCIPLE_3_DESCRIPTION]

### IV. [PRINCIPLE_4_NAME]

[PRINCIPLE_4_DESCRIPTION]

### V. [PRINCIPLE_5_NAME]

[PRINCIPLE_5_DESCRIPTION]

## Development Standards

[STANDARDS_CONTENT]

## Governance

[GOVERNANCE_RULES]

**Version**: [CONSTITUTION_VERSION] | **Ratified**: [RATIFICATION_DATE] | **Last Amended**: [LAST_AMENDED_DATE]
```

The user may specify fewer or more principles — follow their intent.

### Step 2: Collect values for all placeholders

- Use user input if supplied; otherwise infer from repo context (README, existing code patterns)
- `RATIFICATION_DATE`: original adoption date (use today if unknown or mark `TODO(RATIFICATION_DATE): unknown`)
- `LAST_AMENDED_DATE`: today if making changes
- `CONSTITUTION_VERSION`: semantic versioning:
  - **MAJOR**: principle removed or redefined in a breaking way
  - **MINOR**: new principle or section added
  - **PATCH**: clarifications, wording, typos

Common principle patterns (adapt to project context):
- **Library-First**: every feature starts as a standalone library
- **Test-First** (NON-NEGOTIABLE): TDD mandatory — tests written → fail → implement
- **Simplicity**: start simple, YAGNI, no premature abstraction
- **Observability**: structured logging required, debuggability via text I/O
- **Versioning**: MAJOR.MINOR.BUILD with backward compatibility commitment

### Step 3: Draft updated constitution

- Replace every `[PLACEHOLDER]` with concrete text — no brackets remaining
- Each Principle section must have: succinct name, non-negotiable rules (bullets or paragraph), explicit rationale
- Governance section must include: amendment procedure, versioning policy, compliance review cadence
- Replace "should" with MUST/SHOULD where the normative strength is clear

### Step 4: Propagate changes to dependent files

Check which of these files exist in the project root and update any that reference principles:

```bash
ls README.md 2>/dev/null
ls docs/ 2>/dev/null
ls AGENTS.md CLAUDE.md .cursor/rules/ .github/copilot-instructions.md 2>/dev/null
```

For each file found, update any references to renamed, added, or removed principles.

If the project also uses spec-kit's template files (`templates/plan-template.md`, `templates/spec-template.md`, `templates/tasks-template.md`), check and update them too — but this is only applicable in spec-kit repos. Skip silently if not present.

### Step 5: Sync Impact Report

Prepend as HTML comment at top of constitution file:

```html
<!--
## Sync Impact Report

**Version change**: {OLD} → {NEW}
**Modified principles**: {list or none}
**Added sections**: {list or none}
**Removed sections**: {list or none}
**Files updated**:
- README.md: ✅ updated / ⚠ pending / N/A
- AGENTS.md / agent context file: ✅ updated / ⚠ pending / N/A
- templates/ (if spec-kit project): ✅ updated / ⚠ pending / N/A
**Deferred items**: {list or none}
-->
```

### Step 6: Validate before writing

- No remaining `[PLACEHOLDER]` tokens (unless explicitly deferred with `TODO`)
- Version matches Sync Impact Report
- Dates in ISO format (YYYY-MM-DD)
- All principles are declarative and free of vague language

### Step 7: Write the constitution

Overwrite `.specify/memory/constitution.md` with completed content.

### Step 8: Report

- New version and bump rationale
- Files flagged for manual follow-up
- Suggested commit message (e.g., `docs: amend constitution to vX.Y.Z`)
