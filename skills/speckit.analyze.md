---
description: Read-only cross-artifact consistency analysis across spec.md, plan.md, and tasks.md. No CLI installation required.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Goal

Identify inconsistencies, duplications, ambiguities, and underspecified items across `spec.md`, `plan.md`, and `tasks.md` before implementation. Run after `/speckit.tasks`.

## Operating Constraints

**STRICTLY READ-ONLY**: Do not modify any files. Output a structured analysis report only.

**Constitution Authority**: `.specify/memory/constitution.md` is non-negotiable. Constitution conflicts are automatically CRITICAL. To amend a principle, use `/speckit.constitution` separately.

## Execution Steps

### Step 1: Discover the active feature

```bash
git branch --show-current
```

- Branch matches `[0-9]+-[a-z0-9-]+` → `FEATURE_DIR` = `specs/{BRANCH}`
- Otherwise: scan `specs/` or ask user

Absolute paths:
- `SPEC` = `{FEATURE_DIR}/spec.md`
- `PLAN` = `{FEATURE_DIR}/plan.md`
- `TASKS` = `{FEATURE_DIR}/tasks.md`

If any required file is missing: abort with error and instruct user to run the prerequisite command.

### Step 2: Load artifacts (minimal context)

**From spec.md**: Overview, Functional Requirements, Non-Functional Requirements, User Stories, Edge Cases

**From plan.md**: Architecture/stack, Data Model references, Phases, Technical constraints

**From tasks.md**: Task IDs, descriptions, phase grouping, `[P]` markers, referenced file paths

**From constitution** (if `.specify/memory/constitution.md` exists): principle names and MUST/SHOULD statements

### Step 3: Build semantic models (internal only, do not output)

- **Requirements inventory**: each FR/NFR with a stable slug (e.g. "User can upload file" → `user-can-upload-file`)
- **User story inventory**: discrete actions with acceptance criteria
- **Task coverage map**: each task mapped to requirement(s) or story by keyword/reference
- **Constitution rule set**: extracted MUST/SHOULD normative statements

### Step 4: Detection passes (max 50 findings)

**A. Duplication**: near-duplicate requirements; flag lower-quality phrasing

**B. Ambiguity**: vague adjectives (fast, scalable, secure, intuitive, robust) without measurable criteria; unresolved placeholders (TODO, ???, `<placeholder>`)

**C. Underspecification**: requirements missing object or measurable outcome; user stories missing acceptance criteria; tasks referencing undefined files/components

**D. Constitution Alignment**: requirements or plan elements conflicting with MUST principles; missing mandated sections/gates

**E. Coverage Gaps**: requirements with zero tasks; tasks with no mapped requirement; NFRs not reflected in tasks

**F. Inconsistency**: terminology drift; entities in plan absent from spec (or vice versa); conflicting requirements; task ordering contradictions

### Step 5: Severity

| Level | Criteria |
|-------|----------|
| CRITICAL | Violates constitution MUST; missing core artifact; zero-coverage requirement blocking baseline |
| HIGH | Duplicate/conflicting requirement; ambiguous security/performance; untestable acceptance criterion |
| MEDIUM | Terminology drift; missing NFR task coverage; underspecified edge case |
| LOW | Style/wording; minor redundancy |

### Step 6: Analysis Report

Output Markdown (no file writes):

---

## Specification Analysis Report

| ID | Category | Severity | Location(s) | Summary | Recommendation |
|----|----------|----------|-------------|---------|----------------|
| A1 | Ambiguity | HIGH | spec.md:L45 | "fast" without metric | Quantify with p95 latency target |

*(IDs prefixed by category: A=Ambiguity, C=Conflict, D=Duplication, G=Gap, I=Inconsistency, U=Underspec)*

**Coverage Summary**:

| Requirement | Has Task? | Task IDs | Notes |
|-------------|-----------|----------|-------|

**Constitution Issues** *(if any)*:

**Unmapped Tasks** *(if any)*:

**Metrics**:
- Total Requirements: N
- Total Tasks: N
- Coverage: N% (requirements with ≥1 task)
- CRITICAL: N | HIGH: N | MEDIUM: N | LOW: N

---

### Step 7: Next Actions

- CRITICAL issues → must resolve before `/speckit.implement`
- LOW/MEDIUM only → may proceed; list suggestions
- Provide explicit commands (e.g., "edit spec.md §FR-3 to quantify 'fast'")

### Step 8: Offer Remediation

Ask: *"Would you like me to suggest concrete edits for the top N issues?"* — do NOT apply automatically.

## Context

$ARGUMENTS
