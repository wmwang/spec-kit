---
description: Generate a custom checklist for the current feature based on user requirements. Checklists are "unit tests for requirements" — they validate quality, clarity, and completeness of requirements. Pure skill version - no CLI installation required.
---

## Checklist Purpose: "Unit Tests for English"

**CRITICAL CONCEPT**: Checklists are **UNIT TESTS FOR REQUIREMENTS WRITING** — they validate the quality, clarity, and completeness of requirements in a given domain.

**NOT for verification/testing**:
- ❌ NOT "Verify the button clicks correctly"
- ❌ NOT "Test error handling works"
- ❌ NOT "Confirm the API returns 200"

**FOR requirements quality validation**:
- ✅ "Are visual hierarchy requirements defined for all card types?" (completeness)
- ✅ "Is 'prominent display' quantified with specific sizing/positioning?" (clarity)
- ✅ "Are hover state requirements consistent across all interactive elements?" (consistency)

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

- If branch matches `[0-9]+-[a-z0-9-]+`: feature directory is `specs/{branch-name}/`
- Otherwise: scan `specs/` or ask user

Establish:
- `FEATURE_DIR` = `specs/{branch-name}/` (absolute path)

### Step 2: Clarify intent (dynamic)

Derive up to **THREE** initial contextual clarifying questions. They MUST:
- Be generated from the user's phrasing + extracted signals from spec/plan/tasks
- Only ask about information that materially changes checklist content
- Be skipped individually if already unambiguous in `$ARGUMENTS`
- Prefer precision over breadth

**Generation algorithm**:
1. Extract signals: feature domain keywords (auth, latency, UX, API), risk indicators ("critical", "must", "compliance"), stakeholder hints ("QA", "review", "security team"), and explicit deliverables ("a11y", "rollback", "contracts")
2. Cluster signals into candidate focus areas (max 4) ranked by relevance
3. Identify probable audience & timing (author, reviewer, QA, release) if not explicit
4. Detect missing dimensions: scope breadth, depth/rigor, risk emphasis, exclusion boundaries

**Question formatting rules**:
- If presenting options, use compact table: Option | Candidate | Why It Matters
- Limit to A–E options; omit table if free-form answer is clearer
- Never ask the user to restate what they already said

**Defaults when interaction impossible**:
- Depth: Standard
- Audience: Reviewer (PR) if code-related; Author otherwise
- Focus: Top 2 relevance clusters

After initial answers: if ≥2 scenario classes remain unclear, you MAY ask up to TWO more targeted follow-ups (Q4/Q5). Do not exceed five total questions.

### Step 3: Understand user request

Combine `$ARGUMENTS` + clarifying answers:
- Derive checklist theme (e.g., security, review, deploy, ux)
- Consolidate explicit must-have items mentioned by user
- Map focus selections to category scaffolding
- Infer missing context from spec/plan/tasks (do NOT hallucinate)

### Step 4: Load feature context

Read from `FEATURE_DIR`:
- `spec.md`: Feature requirements and scope
- `plan.md` (if exists): Technical details, dependencies
- `tasks.md` (if exists): Implementation tasks

**Context Loading Strategy**:
- Load only necessary portions relevant to active focus areas
- Prefer summarizing long sections into concise bullets
- Use progressive disclosure: add follow-on retrieval only if gaps detected

### Step 5: Generate checklist

**Create/find the checklist file**:
```bash
mkdir -p specs/{branch-name}/checklists
```

Generate unique checklist filename:
- Use short, descriptive name based on domain (e.g., `ux.md`, `api.md`, `security.md`)
- Format: `[domain].md`

**File handling behavior**:
- If file does NOT exist: Create new file and number items starting from CHK001
- If file exists: Append new items to existing file, continuing from the last CHK ID (e.g., if last item is CHK015, start new items at CHK016)
- Never delete or replace existing checklist content — always preserve and append

**Use this template structure**:

```markdown
# [CHECKLIST TYPE] Checklist: [FEATURE NAME]

**Purpose**: [Brief description of what this checklist covers]
**Created**: [TODAY'S DATE]
**Feature**: [Link to spec.md]

## [Category 1]

- [ ] CHK001 [Requirement quality question] [Completeness/Clarity/Consistency/etc., Spec §X.Y]
- [ ] CHK002 [Requirement quality question] [Gap]

## [Category 2]

- [ ] CHK003 [Requirement quality question] [Ambiguity, Spec §X.Y]

## Notes

- Check items off as completed: `[x]`
- Add comments or findings inline
```

**CORE PRINCIPLE — Test the Requirements, Not the Implementation**:

Every checklist item MUST evaluate the REQUIREMENTS THEMSELVES for:
- **Completeness**: Are all necessary requirements present?
- **Clarity**: Are requirements unambiguous and specific?
- **Consistency**: Do requirements align with each other?
- **Measurability**: Can requirements be objectively verified?
- **Coverage**: Are all scenarios/edge cases addressed?

**Category Structure** — Group items by requirement quality dimensions:
- **Requirement Completeness**: Are all necessary requirements documented?
- **Requirement Clarity**: Are requirements specific and unambiguous?
- **Requirement Consistency**: Do requirements align without conflicts?
- **Acceptance Criteria Quality**: Are success criteria measurable?
- **Scenario Coverage**: Are all flows/cases addressed?
- **Edge Case Coverage**: Are boundary conditions defined?
- **Non-Functional Requirements**: Performance, Security, Accessibility — are they specified?
- **Dependencies & Assumptions**: Are they documented and validated?
- **Ambiguities & Conflicts**: What needs clarification?

**HOW TO WRITE CHECKLIST ITEMS**:

❌ **WRONG** (Testing implementation):
- "Verify landing page displays 3 episode cards"
- "Test hover states work on desktop"
- "Confirm logo click navigates home"

✅ **CORRECT** (Testing requirements quality):
- "Are the exact number and layout of featured episodes specified? [Completeness, Spec §FR-001]"
- "Is 'prominent display' quantified with specific sizing/positioning? [Clarity, Spec §FR-004]"
- "Are hover state requirements consistent across all interactive elements? [Consistency]"
- "Are keyboard navigation requirements defined for all interactive UI? [Coverage, Gap]"
- "Is the fallback behavior specified when logo image fails to load? [Edge Cases, Gap]"

**ITEM STRUCTURE**:
- Question format asking about requirement quality
- Focus on what's WRITTEN (or not written) in the spec/plan
- Include quality dimension in brackets [Completeness/Clarity/Consistency/etc.]
- Reference spec section `[Spec §X.Y]` when checking existing requirements
- Use `[Gap]` marker when checking for missing requirements

**Traceability Requirements**:
- MINIMUM: ≥80% of items MUST include at least one traceability reference
- Each item references: spec section `[Spec §X.Y]`, or markers: `[Gap]`, `[Ambiguity]`, `[Conflict]`, `[Assumption]`

**Content Consolidation**:
- Soft cap: If raw candidate items > 40, prioritize by risk/impact
- Merge near-duplicates checking the same requirement aspect
- If >5 low-impact edge cases: create one item "Are edge cases X, Y, Z addressed in requirements? [Coverage]"

**🚫 ABSOLUTELY PROHIBITED**:
- ❌ Any item starting with "Verify", "Test", "Confirm", "Check" + implementation behavior
- ❌ References to code execution, user actions, system behavior
- ❌ "Displays correctly", "works properly", "functions as expected"
- ❌ "Click", "navigate", "render", "load", "execute"
- ❌ Test cases, test plans, QA procedures
- ❌ Implementation details (frameworks, APIs, algorithms)

**✅ REQUIRED PATTERNS**:
- ✅ "Are [requirement type] defined/specified/documented for [scenario]?"
- ✅ "Is [vague term] quantified/clarified with specific criteria?"
- ✅ "Are requirements consistent between [section A] and [section B]?"
- ✅ "Can [requirement] be objectively measured/verified?"
- ✅ "Are [edge cases/scenarios] addressed in requirements?"
- ✅ "Does the spec define [missing aspect]?"

### Step 6: Report

Output:
- Full path to checklist file
- Item count
- Whether the run created a new file or appended to existing
- Summary:
  - Focus areas selected
  - Depth level
  - Audience/timing
  - Any explicit user-specified must-have items incorporated

## Checklist Type Examples

**UX Requirements Quality** → `ux.md`:
- "Are visual hierarchy requirements defined with measurable criteria? [Clarity, Spec §FR-1]"
- "Is the number and positioning of UI elements explicitly specified? [Completeness, Spec §FR-1]"
- "Are interaction state requirements (hover, focus, active) consistently defined? [Consistency]"

**API Requirements Quality** → `api.md`:
- "Are error response formats specified for all failure scenarios? [Completeness]"
- "Are rate limiting requirements quantified with specific thresholds? [Clarity]"
- "Are authentication requirements consistent across all endpoints? [Consistency]"

**Security Requirements Quality** → `security.md`:
- "Are authentication requirements specified for all protected resources? [Coverage]"
- "Is the threat model documented and requirements aligned to it? [Traceability]"
- "Are security failure/breach response requirements defined? [Gap, Exception Flow]"

**Performance Requirements Quality** → `performance.md`:
- "Are performance requirements quantified with specific metrics? [Clarity]"
- "Are performance targets defined for all critical user journeys? [Coverage]"
- "Can performance requirements be objectively measured? [Measurability]"
