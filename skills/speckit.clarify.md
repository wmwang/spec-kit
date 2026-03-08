---
description: Ask up to 5 targeted clarification questions about the current feature spec and write the answers back into spec.md. No CLI installation required.
handoffs:
  - label: Build Technical Plan
    agent: speckit.plan
    prompt: Create a plan for the spec. I am building with...
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

**Goal**: Detect and reduce ambiguity in the active feature spec, recording answers directly in the file. Run BEFORE `/speckit.plan`. If skipping, warn that downstream rework risk increases.

### Step 1: Discover the active feature

```bash
git branch --show-current
```

- Branch matches `[0-9]+-[a-z0-9-]+` → `FEATURE_DIR` = `specs/{BRANCH}`
- Otherwise: scan `specs/` or ask user

Set `FEATURE_SPEC` = `{FEATURE_DIR}/spec.md`. If missing: ERROR, instruct user to run `/speckit.specify` first.

### Step 2: Structured ambiguity scan

Read `FEATURE_SPEC`. For each category below, mark **Clear / Partial / Missing**:

| Category | What to check |
|----------|--------------|
| Functional Scope | Core user goals, success criteria, explicit out-of-scope |
| Domain & Data Model | Entities, relationships, state transitions, scale |
| Interaction & UX | User journeys, error/empty/loading states, a11y |
| Non-Functional | Performance targets, reliability, security, compliance |
| Integration | External services, failure modes, data formats |
| Edge Cases | Negative scenarios, rate limiting, conflict resolution |
| Constraints | Technical constraints, explicit tradeoffs |
| Terminology | Canonical terms, avoided synonyms |
| Acceptance | Testability of criteria, measurable DoD |
| Placeholders | TODO markers, vague adjectives without quantification |

Build an internal priority queue of candidate questions. Do NOT output it.

### Step 3: Prioritize questions

Generate at most **5** questions. Each must be answerable with:
- A short multiple-choice (2–5 mutually exclusive options), OR
- A short phrase (≤5 words)

Only ask if the answer materially impacts architecture, data modeling, test design, UX, security, or compliance. Skip low-impact questions. Favor those that reduce downstream rework.

### Step 4: Interactive questioning — one at a time

Present **exactly one question at a time**.

**Multiple-choice format**:

1. Analyze all options; pick the best based on best practices and project context
2. Show recommendation first: `**Recommended:** Option {X} — {reasoning}`
3. Render options as a table:

| Option | Description |
|--------|-------------|
| A | {description} |
| B | {description} |
| C | {description} |
| Short | Provide your own short answer (≤5 words) |

4. Add: *You can reply with the letter, say "yes"/"recommended" to accept the recommendation, or give your own short answer.*

**Short-answer format** (no discrete options):

`**Suggested:** {proposed answer} — {brief reasoning}`

*Format: ≤5 words. Say "yes"/"suggested" to accept, or give your own.*

**After each answer**:
- "yes" / "recommended" / "suggested" → use stated recommendation
- Ambiguous → ask disambiguation (does not count as a new question)
- Valid → record and advance to next question

**Stop when**: all critical ambiguities resolved, user says "done"/"stop", or 5 questions reached.

### Step 5: Update spec after each accepted answer

After EACH answer:

1. If first answer this session: ensure `## Clarifications` section exists (add after overview section if missing), then add `### Session {YYYY-MM-DD}` subheading
2. Append: `- Q: {question} → A: {answer}`
3. Apply the clarification to the most relevant section:
   - Functional ambiguity → Functional Requirements
   - Actor/role → User Stories / Actors
   - Data shape → Data Model (add fields, types, constraints)
   - Non-functional → NFR section (convert vague adjective to metric)
   - Edge case → Edge Cases / Error Handling
   - Terminology → normalize term across entire spec
4. If the answer invalidates an earlier statement: replace it, do not duplicate
5. **Save the file after each integration** (atomic overwrite)

### Step 6: Validate after each write

- One bullet per accepted answer in Clarifications (no duplicates)
- Total asked ≤ 5
- Updated sections have no lingering vague placeholders the answer was meant to resolve
- No contradictory earlier statements remain
- Only new headings allowed: `## Clarifications`, `### Session YYYY-MM-DD`

### Step 7: Report

- Questions asked and answered
- Sections updated (list names)
- Coverage summary:

| Category | Status |
|----------|--------|
| Functional Scope | Resolved / Clear / Deferred / Outstanding |
| ... | ... |

- If any Outstanding/Deferred: recommend `/speckit.clarify` again or proceed to `/speckit.plan`

**If no meaningful ambiguities found**: say so and suggest proceeding.

Context for prioritization: $ARGUMENTS
