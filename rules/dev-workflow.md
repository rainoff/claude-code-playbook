# Development Workflow Rules (Always Enforce)

## Environment

<!-- Adjust for your system: Python command (python3 / py), shell (bash / zsh) -->

- Response language: English

## Task Startup

- Start every task in Plan Mode. Do not write code until the plan is approved.
- If the user's task description is fewer than 3 sentences, ask 2-3 clarifying questions before planning.
- When the user gives a vague instruction ("fix this", "make it better"), clarify the specific goal first.

## Commit Discipline

- Commit after each logical section (DB layer, API, frontend separately)
- Commit message format follows the project skill convention (default: Commitizen)
- If the project has a git-commit skill, always commit through that skill
- After large changes pass tests, commit immediately — do not accumulate unpushed changes
- Do not push proactively unless the user requests it

## Deployment Discipline

- **Always ask before deploying**: "Should we test on staging first, or deploy directly to prod?"
- Do not independently decide to deploy to prod, even for small changes
- After staging passes, still verify that prod has all prerequisites in place

## Test First

- Every batch should prioritize testability
- Run automated tests first when available
- Explicitly list steps for anything requiring manual testing
- For projects without protobuf/backend specs, default to TDD: write tests first → confirm they fail → implement → confirm they pass.

## Quality

- Do not review code you just wrote in the same context. Suggest using a subagent or new session.
- For core business logic changes, explain each modification and wait for confirmation.
- If stuck on the same error 3+ times, stop and suggest: "/clear and restart with a better approach."

## No Over-Engineering

- Only make changes that are directly requested or clearly necessary
- Do not add unnecessary docstrings, comments, or type annotations
- Three lines of duplication is better than premature abstraction

## Task Completion

- After finishing a task, remind: "Run /review before pushing."
- Then suggest /clear before starting the next task.
- Before /clear, write a /session handoff so the next session can pick up seamlessly.

## Context Management

- Use @ file references instead of pasting file contents.
- Prefer reading files with tools rather than asking the user to copy content in.
- If context usage exceeds 50%, proactively suggest: "/clear + paste task summary to continue with fresh context."
