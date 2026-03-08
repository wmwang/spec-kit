---
description: Generate a "unit tests for requirements" checklist — validates quality, clarity, and completeness of requirements, NOT implementation behavior. No CLI installation required.
---

## Concept: "Unit Tests for English"

Checklists validate **requirements quality**, not implementation behavior.

**NOT**:
- ❌ "Verify the button clicks correctly"
- ❌ "Test error handling works"

**YES**:
- ✅ "Are visual hierarchy requirements defined with measurable criteria? [Clarity]"
- ✅ "Is 'prominent display' quantified with specific sizing? [Clarity, Spec §FR-4]"
- ✅ "Are hover states consistently defined for all interactive elements? [Consistency]"

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Execution Steps

### Step 1: Discover the active feature

```bash
git branch --show-current
```

- Branch matches `[0-9]+-[a-z0-9-]+` → `FEATURE_DIR` = `specs/{BRANCH}`
- Otherwise: scan `specs/` or ask user

### Step 2: Clarify intent (up to 3 questions)

Derive clarifying questions from user's phrasing + signals in spec/plan/tasks. Only ask if the answer materially changes checklist content. Skip questions already answered in `$ARGUMENTS`.

Extract signals: domain keywords (auth, latency, UX, API), risk indicators ("critical", "compliance"), audience hints ("QA", "security team"), deliverables ("a11y", "rollback").

Question archetypes:
- Scope: "Should this include X or stay limited to Y?"
- Depth: "Lightweight sanity list or formal release gate?"
- Audience: "For the author or PR reviewers?"
- Risk emphasis: "Which risk areas should have mandatory gates?"

After initial answers: if ≥2 scenario classes remain unclear, ask up to 2 more (Q4/Q5 max). Never exceed 5 total.

**Defaults when interaction impossible**: Standard depth, Reviewer audience, top 2 relevance clusters.

### Step 3: Load feature context (minimal)

From `FEATURE_DIR`: `spec.md` (required), `plan.md` (if exists), `tasks.md` (if exists).

Load only portions relevant to the focus area. Summarize long sections.

### Step 4: Determine checklist file

Use a short descriptive name based on domain:
- UX/interaction → `ux.md`
- API/endpoints → `api.md`
- Security → `security.md`
- Performance → `performance.md`
- Testing/QA → `qa.md`

```bash
mkdir -p specs/{BRANCH}/checklists
```

- File does NOT exist → create new, start IDs at CHK001
- File EXISTS → append, continue from last CHK ID (e.g., last is CHK015 → next is CHK016)
- **Never delete or overwrite existing items**

### Step 5: Generate checklist items

Use this file structure:

```markdown
# {Type} Checklist: {FEATURE NAME}

**Purpose**: {what this validates}
**Created**: {TODAY}
**Feature**: [spec.md](../spec.md)

## Requirement Completeness

- [ ] CHK001 Are {requirement type} defined for all {scenarios}? [Completeness, Spec §X.Y]
- [ ] CHK002 Are {requirement type} specified for {edge case}? [Gap]

## Requirement Clarity

- [ ] CHK003 Is '{vague term}' quantified with specific criteria? [Clarity, Spec §X.Y]
- [ ] CHK004 Can '{requirement}' be objectively measured? [Measurability, Spec §X.Y]

## Requirement Consistency

- [ ] CHK005 Are requirements consistent between {section A} and {section B}? [Consistency]

## Scenario Coverage

- [ ] CHK006 Are {alternate/exception/recovery} flows addressed in requirements? [Coverage, Gap]

## Non-Functional Requirements

- [ ] CHK007 Are {performance/security/accessibility} requirements specified? [Coverage, Gap]

## Dependencies & Assumptions

- [ ] CHK008 Are external dependencies documented with failure modes? [Dependency, Gap]
```

**Writing rules**:

Every item must:
- Be in question form asking about requirement quality
- Include a quality dimension: `[Completeness]`, `[Clarity]`, `[Consistency]`, `[Measurability]`, `[Coverage]`, `[Gap]`, `[Ambiguity]`, `[Conflict]`, `[Assumption]`
- Reference spec section `[Spec §X.Y]` when checking existing requirements
- Use `[Gap]` when checking for missing requirements
- Minimum 80% of items must have at least one traceability reference

**Required item patterns**:
- ✅ "Are {requirement type} defined/specified/documented for {scenario}?"
- ✅ "Is '{vague term}' quantified/clarified with specific criteria?"
- ✅ "Are requirements consistent between {section A} and {section B}?"
- ✅ "Can '{requirement}' be objectively measured/verified?"
- ✅ "Are {edge cases/scenarios} addressed in requirements?"
- ✅ "Does the spec define {missing aspect}?"

**Absolutely prohibited**:
- ❌ Items starting with "Verify", "Test", "Confirm" + implementation behavior
- ❌ References to code execution, user clicks, system rendering
- ❌ "Displays correctly", "works properly", "functions as expected"
- ❌ Implementation details (frameworks, APIs, algorithms)

**Content consolidation**: soft cap 40 items; if >5 low-impact edge cases, merge into one item.

### Step 6: Report

- Full path to checklist file
- Item count (total, new)
- New file or appended
- Focus areas selected
- Audience/depth level

## Domain Examples

**UX** (`ux.md`):
- "Are visual hierarchy requirements defined with measurable criteria? [Clarity, Spec §FR-1]"
- "Are interaction state requirements (hover, focus, active) consistently defined? [Consistency]"
- "Is fallback behavior specified when images fail to load? [Edge Case, Gap]"

**API** (`api.md`):
- "Are error response formats specified for all failure scenarios? [Completeness]"
- "Are rate limiting requirements quantified with specific thresholds? [Clarity]"

**Security** (`security.md`):
- "Are authentication requirements specified for all protected resources? [Coverage]"
- "Is the threat model documented and requirements aligned to it? [Traceability]"
- "Are security failure/breach response requirements defined? [Gap, Exception Flow]"
