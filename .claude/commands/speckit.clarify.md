---
description: Identify underspecified areas in the current feature spec by asking up to 5 highly targeted clarification questions and encoding answers back into the spec. Pure skill version - no CLI installation required.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

**Goal**: Detect and reduce ambiguity or missing decision points in the active feature specification and record the clarifications directly in the spec file.

**Note**: This clarification workflow is expected to run (and be completed) BEFORE invoking `/speckit.plan`. If the user explicitly states they are skipping clarification (e.g., exploratory spike), you may proceed, but must warn that downstream rework risk increases.

### Step 1: Discover the active feature

```bash
git branch --show-current
```

- If branch matches `[0-9]+-[a-z0-9-]+`: feature directory is `specs/{branch-name}/`
- Otherwise: scan `specs/` for subdirectories or ask user

Establish:
- `FEATURE_DIR` = `specs/{branch-name}/` (absolute path)
- `FEATURE_SPEC` = `FEATURE_DIR/spec.md`

If `FEATURE_SPEC` doesn't exist, instruct the user to run `/speckit.specify` first.

### Step 2: Load and scan the spec

Read `FEATURE_SPEC` and perform a structured ambiguity & coverage scan using this taxonomy. For each category, mark status: **Clear** / **Partial** / **Missing**. Produce an internal coverage map (do not output raw map unless no questions will be asked):

**Functional Scope & Behavior**:
- Core user goals & success criteria
- Explicit out-of-scope declarations
- User roles / personas differentiation

**Domain & Data Model**:
- Entities, attributes, relationships
- Identity & uniqueness rules
- Lifecycle/state transitions
- Data volume / scale assumptions

**Interaction & UX Flow**:
- Critical user journeys / sequences
- Error/empty/loading states
- Accessibility or localization notes

**Non-Functional Quality Attributes**:
- Performance (latency, throughput targets)
- Scalability (horizontal/vertical, limits)
- Reliability & availability (uptime, recovery expectations)
- Observability (logging, metrics, tracing signals)
- Security & privacy (authN/Z, data protection, threat assumptions)
- Compliance / regulatory constraints (if any)

**Integration & External Dependencies**:
- External services/APIs and failure modes
- Data import/export formats
- Protocol/versioning assumptions

**Edge Cases & Failure Handling**:
- Negative scenarios
- Rate limiting / throttling
- Conflict resolution (e.g., concurrent edits)

**Constraints & Tradeoffs**:
- Technical constraints (language, storage, hosting)
- Explicit tradeoffs or rejected alternatives

**Terminology & Consistency**:
- Canonical glossary terms
- Avoided synonyms / deprecated terms

**Completion Signals**:
- Acceptance criteria testability
- Measurable Definition of Done indicators

**Misc / Placeholders**:
- TODO markers / unresolved decisions
- Ambiguous adjectives ("robust", "intuitive") lacking quantification

For each category with Partial or Missing status, add a candidate question opportunity unless:
- Clarification would not materially change implementation or validation strategy
- Information is better deferred to planning phase

### Step 3: Prioritize clarification questions

Generate (internally) a prioritized queue of **maximum 5** candidate clarification questions. Apply these constraints:
- Each question must be answerable with EITHER:
  - A short multiple-choice selection (2–5 distinct, mutually exclusive options), OR
  - A one-word / short-phrase answer ("Answer in ≤5 words")
- Only include questions whose answers materially impact architecture, data modeling, task decomposition, test design, UX behavior, operational readiness, or compliance validation
- Ensure category coverage balance: highest-impact unresolved categories first
- Exclude questions already answered, trivial stylistic preferences, or plan-level execution details
- Favor clarifications that reduce downstream rework risk
- If more than 5 categories remain unresolved: select the top 5 by (Impact × Uncertainty) heuristic

### Step 4: Sequential questioning loop (interactive)

Present **EXACTLY ONE question at a time**:

**For multiple-choice questions**:
1. Analyze all options and determine the **most suitable option** based on best practices, risk reduction, and project context
2. Present your recommended option prominently:
   - `**Recommended:** Option [X] - <reasoning>`
3. Render all options as a Markdown table:

| Option | Description |
|--------|-------------|
| A | <Option A description> |
| B | <Option B description> |
| C | <Option C description> |
| Short | Provide a different short answer (≤5 words) |

4. Add: `You can reply with the option letter (e.g., "A"), accept the recommendation by saying "yes" or "recommended", or provide your own short answer.`

**For short-answer style** (no meaningful discrete options):
1. Provide your **suggested answer** based on best practices
2. Format as: `**Suggested:** <your proposed answer> - <brief reasoning>`
3. Output: `Format: Short answer (≤5 words). You can accept the suggestion by saying "yes" or "suggested", or provide your own answer.`

**After the user answers**:
- If user replies with "yes", "recommended", or "suggested": use your stated recommendation/suggestion
- Otherwise: validate the answer maps to one option or fits the ≤5 word constraint
- If ambiguous: ask for quick disambiguation (count still belongs to same question; do not advance)
- Once satisfactory: record it in working memory and move to the next queued question

**Stop asking when**:
- All critical ambiguities resolved early, OR
- User signals completion ("done", "good", "no more"), OR
- You reach 5 asked questions

Never reveal future queued questions in advance.

### Step 5: Incremental spec updates (after EACH accepted answer)

Maintain an in-memory representation of the spec:

**For the first integrated answer in this session**:
- Ensure a `## Clarifications` section exists (create it just after the overview section if missing)
- Under it, create: `### Session YYYY-MM-DD` subheading for today

**For each answer**:
1. Append: `- Q: <question> → A: <final answer>`
2. Apply the clarification to the most appropriate section:
   - Functional ambiguity → Update/add bullet in Functional Requirements
   - User interaction / actor → Update User Stories or Actors subsection
   - Data shape / entities → Update Data Model (add fields, types, relationships)
   - Non-functional constraint → Add/modify criteria in Non-Functional / Quality Attributes
   - Edge case / negative flow → Add bullet under Edge Cases / Error Handling
   - Terminology conflict → Normalize term across spec; note `(formerly referred to as "X")` once
3. If clarification invalidates an earlier ambiguous statement: replace that statement instead of duplicating
4. Save the spec file AFTER each integration (atomic overwrite)
5. Preserve formatting: do not reorder unrelated sections; keep heading hierarchy intact
6. Keep each inserted clarification minimal and testable

### Step 6: Validation (after EACH write plus final pass)

- Clarifications session contains exactly one bullet per accepted answer (no duplicates)
- Total asked (accepted) questions ≤ 5
- Updated sections contain no lingering vague placeholders the new answer was meant to resolve
- No contradictory earlier statement remains
- Markdown structure valid; only allowed new headings: `## Clarifications`, `### Session YYYY-MM-DD`
- Terminology consistency: same canonical term used across all updated sections

### Step 7: Write the updated spec back to FEATURE_SPEC

### Step 8: Report completion

After questioning loop ends or early termination:
- Number of questions asked & answered
- Path to updated spec
- Sections touched (list names)
- Coverage summary table:

| Category | Status |
|----------|--------|
| Functional Scope | Resolved / Deferred / Clear / Outstanding |
| Domain & Data Model | ... |
| Interaction & UX | ... |
| Non-Functional | ... |
| Integration | ... |
| Edge Cases | ... |
| Constraints | ... |
| Terminology | ... |

- If any Outstanding or Deferred remain: recommend whether to proceed to `/speckit.plan` or run `/speckit.clarify` again later
- Suggested next command

## Behavior Rules

- If no meaningful ambiguities found: respond "No critical ambiguities detected worth formal clarification." and suggest proceeding
- If spec file missing: instruct user to run `/speckit.specify` first (do not create a new spec here)
- Never exceed 5 total asked questions (clarification retries for a single question do not count as new questions)
- Avoid speculative tech stack questions unless the absence blocks functional clarity
- Respect user early termination signals ("stop", "done", "proceed")
- If quota reached with unresolved high-impact categories remaining: explicitly flag them under Deferred with rationale

Context for prioritization: $ARGUMENTS
