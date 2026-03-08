---
description: Create or update the project constitution from interactive or provided principle inputs, ensuring all dependent templates stay in sync. Pure skill version - no CLI installation required.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

You are creating or updating the project constitution at `.specify/memory/constitution.md`. This file governs the project's architectural principles, development practices, and quality standards.

Follow this execution flow:

### Step 1: Load or initialize the constitution

```bash
ls .specify/memory/constitution.md 2>/dev/null || echo "not found"
```

- If `.specify/memory/constitution.md` exists: read it and identify every placeholder token of the form `[ALL_CAPS_IDENTIFIER]`
- If it doesn't exist: create the directory and initialize from the template below

**Initialize directory if needed**:
```bash
mkdir -p .specify/memory
mkdir -p .specify/templates/commands
```

**Constitution Template** (use this when creating a new constitution):

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

**IMPORTANT**: The user might require fewer or more principles than the template. Respect what they specify and update the document accordingly.

### Step 2: Collect/derive values for placeholders

- If user input supplies a value, use it
- Otherwise infer from existing repo context (README, docs, existing code patterns)
- For governance dates:
  - `RATIFICATION_DATE`: the original adoption date (if unknown, use today or mark TODO)
  - `LAST_AMENDED_DATE`: today if changes are made, otherwise keep previous
- `CONSTITUTION_VERSION` must increment according to semantic versioning:
  - **MAJOR**: Backward incompatible governance/principle removals or redefinitions
  - **MINOR**: New principle/section added or materially expanded guidance
  - **PATCH**: Clarifications, wording, typo fixes, non-semantic refinements
- If version bump type is ambiguous, propose reasoning before finalizing

**Common principle patterns** (use as inspiration, derive from user/codebase context):
- Library-First: Every feature starts as a standalone library
- Test-First (NON-NEGOTIABLE): TDD mandatory — tests written → tests fail → implement
- Simplicity: Start simple, YAGNI principles, no premature abstraction
- Observability: Structured logging required, debuggability through text I/O
- Integration Testing: Focus on contract tests, inter-service communication, shared schemas
- CLI Interface: Every library exposes functionality via CLI; text in/out protocol
- Versioning: MAJOR.MINOR.BUILD format with backward compatibility commitment

### Step 3: Draft the updated constitution content

- Replace every placeholder with concrete text (no bracketed tokens left)
- Preserve heading hierarchy
- Ensure each Principle section has:
  - Succinct name
  - Non-negotiable rules (bullet list or paragraph)
  - Explicit rationale if not obvious
- Ensure Governance section includes:
  - Amendment procedure
  - Versioning policy
  - Compliance review expectations

### Step 4: Consistency propagation

Check if these files exist and update if constitution changes affect them:

```bash
ls .specify/templates/ 2>/dev/null
ls .claude/commands/ 2>/dev/null
```

- Check `plan-template.md`: ensure "Constitution Check" sections align with updated principles
- Check `spec-template.md`: scope/requirements alignment
- Check `tasks-template.md`: task categorization reflects new principle-driven task types
- Check command files: verify no outdated principle references remain
- Check `README.md` if it references principles

### Step 5: Produce a Sync Impact Report

Prepend as an HTML comment at the top of the constitution file after update:

```html
<!--
## Sync Impact Report

**Version change**: [OLD] → [NEW]
**Modified principles**: [list]
**Added sections**: [list or none]
**Removed sections**: [list or none]
**Templates requiring updates**:
- .specify/templates/plan-template.md: ✅ updated / ⚠ pending
- .specify/templates/spec-template.md: ✅ updated / ⚠ pending
- .specify/templates/tasks-template.md: ✅ updated / ⚠ pending
**Deferred items**: [list or none]
-->
```

### Step 6: Validation before final output

- No remaining unexplained bracket tokens
- Version line matches Sync Impact Report
- Dates in ISO format (YYYY-MM-DD)
- Principles are declarative, testable, and free of vague language
  - Replace "should" with MUST/SHOULD where appropriate

### Step 7: Write the completed constitution

Write the completed constitution back to `.specify/memory/constitution.md` (overwrite).

### Step 8: Output a final summary

Report to the user:
- New version and bump rationale
- Any files flagged for manual follow-up
- Suggested commit message (e.g., `docs: amend constitution to vX.Y.Z (principle additions + governance update)`)

## Formatting & Style Requirements

- Use Markdown headings exactly as in the template (do not demote/promote levels)
- Keep lines under 100 chars for readability
- Keep a single blank line between sections
- Avoid trailing whitespace

## Special Cases

- If the user supplies partial updates (e.g., only one principle revision): still perform validation and version decision steps
- If critical info is missing (e.g., ratification date truly unknown): insert `TODO(<FIELD_NAME>): explanation` and include in Sync Impact Report under deferred items
- Do not create a new template; always operate on the existing `.specify/memory/constitution.md` file (once initialized)
