---
description: Create or update the feature specification from a natural language feature description. Pure skill version - no CLI installation required.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

The text the user typed after `/speckit.specify` **is** the feature description. Do not ask the user to repeat it unless they provided an empty command.

Given that feature description, do this:

### Step 1: Generate a concise short name (2-4 words) for the branch

- Analyze the feature description and extract the most meaningful keywords
- Create a 2-4 word short name that captures the essence of the feature
- Use kebab-case (e.g., `user-auth`, `oauth2-api-integration`, `analytics-dashboard`)
- Preserve technical terms and acronyms (OAuth2, API, JWT, etc.)
- Examples:
  - "I want to add user authentication" → `user-auth`
  - "Implement OAuth2 integration for the API" → `oauth2-api-integration`
  - "Create a dashboard for analytics" → `analytics-dashboard`
  - "Fix payment processing timeout bug" → `fix-payment-timeout`

### Step 2: Determine the next available feature number

Run the following commands to find the highest existing number for this short-name:

```bash
git fetch --all --prune 2>/dev/null || true
```

Then check all three sources:

```bash
# Remote branches matching the short-name
git ls-remote --heads origin 2>/dev/null | grep -oE '[0-9]+-<short-name>$' | grep -oE '^[0-9]+' | sort -n | tail -1

# Local branches matching the short-name
git branch 2>/dev/null | grep -oE '[0-9]+-<short-name>' | grep -oE '^[0-9]+' | sort -n | tail -1

# Existing specs directories matching the short-name
ls specs/ 2>/dev/null | grep -E '^[0-9]+-<short-name>$' | grep -oE '^[0-9]+' | sort -n | tail -1
```

(Replace `<short-name>` with the actual short name you generated.)

- Extract the highest number N from all three sources combined
- Use N+1 as the new feature number
- If no existing entries found for this short-name, start at 1

### Step 3: Create branch and directory structure

```bash
# Create the feature branch
git checkout -b {NUMBER}-{SHORT_NAME}

# Create the spec directory structure
mkdir -p specs/{NUMBER}-{SHORT_NAME}/checklists
```

Where `{NUMBER}` is the determined number and `{SHORT_NAME}` is the kebab-case short name.

**Variables established**:
- `BRANCH_NAME` = `{NUMBER}-{SHORT_NAME}`
- `FEATURE_DIR` = `specs/{NUMBER}-{SHORT_NAME}`
- `SPEC_FILE` = `specs/{NUMBER}-{SHORT_NAME}/spec.md`

### Step 4: Execute the specification workflow

Parse the user description from Input. If empty: ERROR "No feature description provided"

Follow this execution flow:

1. Extract key concepts from description: actors, actions, data, constraints
2. For unclear aspects:
   - Make informed guesses based on context and industry standards
   - Mark with `[NEEDS CLARIFICATION: specific question]` **only** if:
     - The choice significantly impacts feature scope or user experience
     - Multiple reasonable interpretations exist with different implications
     - No reasonable default exists
   - **LIMIT: Maximum 3 [NEEDS CLARIFICATION] markers total**
   - Prioritize: scope > security/privacy > user experience > technical details
3. Fill User Scenarios & Testing section
4. Generate Functional Requirements (each must be testable)
5. Define Success Criteria (measurable, technology-agnostic outcomes)
6. Identify Key Entities (if data involved)

### Step 5: Write the specification to SPEC_FILE

Use this template structure:

```markdown
# Feature Specification: [FEATURE NAME]

**Feature Branch**: `{NUMBER}-{SHORT_NAME}`
**Created**: [TODAY'S DATE]
**Status**: Draft
**Input**: User description: "$ARGUMENTS"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - [Brief Title] (Priority: P1)

[Describe this user journey in plain language]

**Why this priority**: [Explain the value and why it has this priority level]

**Independent Test**: [How this can be tested independently]

**Acceptance Scenarios**:

1. **Given** [initial state], **When** [action], **Then** [expected outcome]
2. **Given** [initial state], **When** [action], **Then** [expected outcome]

---

### User Story 2 - [Brief Title] (Priority: P2)

[Continue pattern for additional stories]

---

### Edge Cases

- What happens when [boundary condition]?
- How does system handle [error scenario]?

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST [specific capability]
- **FR-002**: System MUST [specific capability]

### Key Entities *(include if feature involves data)*

- **[Entity 1]**: [What it represents, key attributes]

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: [Measurable metric, technology-agnostic]
- **SC-002**: [Measurable metric]
```

### Step 6: Specification Quality Validation

After writing the initial spec, validate it:

**a. Create Spec Quality Checklist** at `FEATURE_DIR/checklists/requirements.md`:

```markdown
# Specification Quality Checklist: [FEATURE NAME]

**Purpose**: Validate specification completeness and quality before proceeding to planning
**Created**: [TODAY'S DATE]
**Feature**: [Link to spec.md]

## Content Quality

- [ ] No implementation details (languages, frameworks, APIs)
- [ ] Focused on user value and business needs
- [ ] Written for non-technical stakeholders
- [ ] All mandatory sections completed

## Requirement Completeness

- [ ] No [NEEDS CLARIFICATION] markers remain
- [ ] Requirements are testable and unambiguous
- [ ] Success criteria are measurable
- [ ] Success criteria are technology-agnostic (no implementation details)
- [ ] All acceptance scenarios are defined
- [ ] Edge cases are identified
- [ ] Scope is clearly bounded
- [ ] Dependencies and assumptions identified

## Feature Readiness

- [ ] All functional requirements have clear acceptance criteria
- [ ] User scenarios cover primary flows
- [ ] Feature meets measurable outcomes defined in Success Criteria
- [ ] No implementation details leak into specification

## Notes

- Items marked incomplete require spec updates before `/speckit.clarify` or `/speckit.plan`
```

**b. Run Validation Check**: Review the spec against each checklist item and update pass/fail status.

**c. Handle [NEEDS CLARIFICATION] markers** (if any):

1. Extract all markers from the spec
2. If more than 3 exist: keep only the 3 most critical, make informed guesses for the rest
3. Present questions to user in this format:

```markdown
## Question [N]: [Topic]

**Context**: [Quote relevant spec section]

**What we need to know**: [Specific question]

**Suggested Answers**:

| Option | Answer | Implications |
|--------|--------|--------------|
| A      | [First answer] | [What this means] |
| B      | [Second answer] | [What this means] |
| C      | [Third answer] | [What this means] |

**Your choice**: _[Wait for user response]_
```

4. Wait for user responses, then update the spec with chosen answers

### Step 7: Report completion

Report:
- Branch name created
- Spec file path
- Checklist status
- Readiness for next phase (`/speckit.clarify` or `/speckit.plan`)

## Guidelines

- Focus on **WHAT** users need and **WHY** — not HOW to implement
- Avoid technology-specific details (no frameworks, APIs, code structure)
- Write for business stakeholders, not developers
- **Reasonable defaults** (don't ask about these):
  - Data retention: Industry-standard practices for the domain
  - Performance targets: Standard web/mobile app expectations unless specified
  - Error handling: User-friendly messages with appropriate fallbacks
  - Authentication method: Standard session-based or OAuth2 for web apps
- **Success criteria must be**:
  - Measurable (specific metrics: time, percentage, count, rate)
  - Technology-agnostic (no frameworks, languages, databases)
  - User-focused (outcomes from user/business perspective)
  - Verifiable (testable without knowing implementation details)
