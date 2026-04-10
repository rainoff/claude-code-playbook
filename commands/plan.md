---
description: Structured feature development workflow
---

Guide me through building a new feature:

1. Ask me exactly what I want to build
2. Clarify any ambiguities with 2-3 targeted questions
3. Ask: do I have a ticket ID? (e.g., COIN-75, WEB-71, OP-36)

### 3.5. Figma Pre-check (UI tickets only)

If the ticket involves UI implementation and a Figma design file exists:

a. **Check if spec already has a Node Map** — read `specs/{ticket-id}.md` and confirm the `## Figma Node Map` section exists
b. **If not → run Phase 0 first** — follow `figma-workflow.md` to execute Phase 0 (explore → screenshots → Node Map → user review)
c. **Check Design Tokens** — confirm the `## Design Tokens` section exists (at minimum, shared colors + typography for the desktop breakpoint)
d. **If not → run Phase 1 first** — extract Design Tokens for each component and write to spec
e. **Layout-level Sanity Check** — download screenshots of each section, do a high-level structure comparison with the current implementation (if any). Confirm flex direction, layout hierarchy, and component composition are correct before proceeding to subtask breakdown

> This checkpoint is mandatory. UI specs without both a Node Map and Design Tokens may not proceed to subtask breakdown.

4. Build a specification document following the spec-template rule format, and write it to `specs/{ticket-id}.md` in the project root (create `specs/` if it doesn't exist). If no ticket ID, use a descriptive slug (e.g., `specs/add-auth-middleware.md`). Required sections:
   - **Context** — why this change exists (1-3 sentences)
   - **Current Behavior** — what happens now
   - **Target Behavior** — what should happen after
   - **Files to Change** — each file with one-line description
   - **Acceptance Criteria** — checkboxes, this is the "done" definition
   - **Testing Strategy** — automated tests, manual steps, or both
   - **Known Pitfalls** — things that could go wrong (optional)
5. Create a detailed implementation plan as a numbered checklist, appended to the spec file under `## Implementation Plan`
6. Wait for my approval before starting implementation
7. After approval, work through the checklist step by step
8. Commit after each logical step
9. After all steps done, mark acceptance criteria as checked in the spec file

## Minimum Testable Step Breakdown Criteria

After producing the checklist and before starting implementation, each step must pass the following checks:

### Three Breakdown Principles

A step is "minimally testable" if and only if it simultaneously satisfies:

1. **Single AC principle** — corresponds to only 1-2 Acceptance Criteria from the spec. If a step needs to verify 3 or more ACs, it can be broken down further.

2. **Independently testable principle** — can be tested independently without depending on other incomplete steps. If Step B's tests require Step A to be complete first, Step B is not independently testable — consider merging or reordering.

3. **Single module principle** — the scope of changes is within one module. If a step requires changing both frontend and backend simultaneously, split it into two steps, each with their own ACs and tests.

### Breakdown Output Format

```markdown
## Task Breakdown

### Step 1: {title}
- **AC**: AC-1
- **Scope**: `src/components/Modal.tsx`, `__tests__/Modal.test.tsx`
- **Test**: `should open modal on button click`
- **Dependencies**: none

### Step 2: {title}
- **AC**: AC-2, AC-3
- **Scope**: `src/api/submit.ts`, `__tests__/api.test.ts`
- **Test**: `should submit form data`, `should handle 500 error`
- **Dependencies**: Step 1 (needs Modal's form data structure)
```

### Breakdown Quality Check

After breakdown is complete, auto-check:

```
Breakdown quality check

  {✅|❌} Each step has ≤ 2 ACs
  {✅|❌} Each step is independently testable
  {✅|❌} Each step is within a single module
  {✅|❌} No circular dependencies
  {✅|❌} All ACs are covered (none missed)
  {✅|❌} UI ticket: spec includes Figma Node Map + Design Tokens
  {✅|❌} UI ticket: layout-level sanity check passed (high-level structure matches Figma)
```
