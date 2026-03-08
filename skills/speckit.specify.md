---
description: Create a feature specification from a natural language description. Creates git branch, directory structure, and writes spec.md. No CLI installation required.
handoffs:
  - label: Build Technical Plan
    agent: speckit.plan
    prompt: Create a plan for the spec. I am building with...
  - label: Clarify Spec Requirements
    agent: speckit.clarify
    prompt: Clarify specification requirements
    send: true
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

The text after `/speckit.specify` **is** the feature description. Do not ask the user to repeat it unless they gave an empty command.

### Step 1: Generate a short branch name

From the feature description, create a 2–4 word kebab-case name:

- "Add user authentication" → `user-auth`
- "Implement OAuth2 for the API" → `oauth2-api-integration`
- "Analytics dashboard" → `analytics-dashboard`
- "Fix payment timeout bug" → `fix-payment-timeout`

Preserve technical terms (OAuth2, JWT, API). Use lowercase and hyphens only.

### Step 2: Determine next feature number

```bash
git fetch --all --prune 2>/dev/null || true
```

Search all three sources for the highest existing number matching `<short-name>`:

```bash
# Remote branches
git ls-remote --heads origin 2>/dev/null | grep -oE '[0-9]+-<short-name>$' | grep -oE '^[0-9]+' | sort -n | tail -1

# Local branches
git branch 2>/dev/null | grep -oE '[0-9]+-<short-name>' | grep -oE '^[0-9]+' | sort -n | tail -1

# Existing specs dirs
ls specs/ 2>/dev/null | grep -E '^[0-9]+-<short-name>$' | grep -oE '^[0-9]+' | sort -n | tail -1
```

Take the highest N across all three sources. Use N+1 (start at 1 if none found).

### Step 3: Create branch and directory

```bash
git checkout -b {N}-{SHORT_NAME}
mkdir -p specs/{N}-{SHORT_NAME}/checklists
```

Establish:
- `BRANCH_NAME` = `{N}-{SHORT_NAME}`
- `FEATURE_DIR` = `specs/{N}-{SHORT_NAME}`
- `SPEC_FILE` = `specs/{N}-{SHORT_NAME}/spec.md`

### Step 4: Write the spec

If description is empty: ERROR "No feature description provided"

Extract: actors, actions, data, constraints. Make informed guesses for unknowns. Only add `[NEEDS CLARIFICATION: <question>]` when the choice materially changes scope, security, or UX **and** no reasonable default exists. **Maximum 3 such markers.**

Write `SPEC_FILE` using this structure:

```markdown
# Feature Specification: {FEATURE NAME}

**Feature Branch**: `{BRANCH_NAME}`
**Created**: {TODAY}
**Status**: Draft
**Input**: {USER DESCRIPTION}

## User Scenarios & Testing

### User Story 1 – {Brief Title} (Priority: P1)

{User journey in plain language}

**Why this priority**: {business value}

**Independent Test**: {how to test this story alone}

**Acceptance Scenarios**:

1. **Given** {state}, **When** {action}, **Then** {outcome}
2. **Given** {state}, **When** {action}, **Then** {outcome}

---

### User Story 2 – {Brief Title} (Priority: P2)

{Continue pattern}

---

### Edge Cases

- What happens when {boundary condition}?
- How does the system handle {error scenario}?

## Requirements

### Functional Requirements

- **FR-001**: System MUST {specific capability}
- **FR-002**: System MUST {specific capability}

### Key Entities *(include only if feature involves data)*

- **{Entity}**: {what it represents, key attributes}

## Success Criteria

- **SC-001**: {measurable, technology-agnostic metric}
- **SC-002**: {measurable, technology-agnostic metric}
```

**Success criteria rules**: specific metrics (time/%, count/rate), no frameworks/languages/DBs, user-facing outcomes, verifiable without knowing implementation.

### Step 5: Create quality checklist

Write `FEATURE_DIR/checklists/requirements.md`:

```markdown
# Specification Quality Checklist: {FEATURE NAME}

**Purpose**: Validate specification completeness before planning
**Created**: {TODAY}
**Feature**: [spec.md](../spec.md)

## Content Quality

- [ ] No implementation details (languages, frameworks, APIs)
- [ ] Focused on user value and business needs
- [ ] Written for non-technical stakeholders
- [ ] All mandatory sections completed

## Requirement Completeness

- [ ] No [NEEDS CLARIFICATION] markers remain
- [ ] Requirements are testable and unambiguous
- [ ] Success criteria are measurable and technology-agnostic
- [ ] All acceptance scenarios are defined
- [ ] Edge cases are identified
- [ ] Scope is clearly bounded

## Feature Readiness

- [ ] All functional requirements have clear acceptance criteria
- [ ] User scenarios cover primary flows
- [ ] No implementation details leak into specification

## Notes

- Items marked incomplete require spec updates before `/speckit.clarify` or `/speckit.plan`
```

### Step 6: Handle [NEEDS CLARIFICATION] markers

If any remain, present them (max 3) as:

```markdown
## Question {N}: {Topic}

**Context**: {quote relevant spec section}
**What we need to know**: {specific question}

**Options**:

| Option | Answer | Implications |
|--------|--------|--------------|
| A | {answer} | {what this means} |
| B | {answer} | {what this means} |
| C | {answer} | {what this means} |
```

Wait for responses, replace markers with chosen answers, re-validate checklist.

### Step 7: Report

- Branch created: `{BRANCH_NAME}`
- Spec: `{SPEC_FILE}`
- Checklist: `{FEATURE_DIR}/checklists/requirements.md`
- Next: `/speckit.clarify` or `/speckit.plan`
