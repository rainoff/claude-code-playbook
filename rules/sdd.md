# SDD — Spec-Driven Development

> Every module has a ground truth spec. Read the spec before changing anything — don't just scan the code.

## When to Apply

Applies to all projects. Introduce gradually — only write a spec for a module when you touch it; missing specs don't block work.

## Two Kinds of Spec

| Type | Purpose | Description | Template |
|------|---------|-------------|----------|
| **System Spec** | as-is | How the system looks today (ground truth) | Below |
| **Change Spec** | to-be | What's being changed (delta) | `rules/spec-template.md` |

## Where System Specs Live

Resolution order (first match wins):

1. **In-repo OpenSpec**: `{project}/openspec/specs/{module}.md`
2. **Private specs**: `~/.claude/projects/{key}/specs/system/{module}.md`
3. **Memory fallback**: relevant memory files under `~/.claude/projects/{key}/memory/`
4. **Prompt to create**: none of the above exist → propose creating one (user can skip)

For projects where an `openspec/` directory can't live in the repo (shared monorepos, restricted repo permissions, or not wanting non-code directories in the repo), use Private specs.

## System Spec Template

Frontmatter:

```yaml
---
module: {module name}
scope: {directories or file patterns covered}
last_verified: {YYYY-MM-DD}
---
```

Body:

```markdown
# {Module Name}

## Intent
<!-- One paragraph: why this module exists, what problem it solves -->

## Public API
<!-- External-facing interfaces: function signatures, endpoints, events, CLI commands -->

## Internal Structure
<!-- Inner composition: key files, class/function relationships, data flow -->

## Extension Points
<!-- How to add functionality without changing the core: hooks, plugins, config -->

## Dependencies
<!-- Upstream: what this depends on. Downstream: who depends on this module -->

## Gotchas
<!-- Traps when using/modifying. Same format as CLAUDE.md Gotchas -->

## Patterns
<!-- Code patterns specific to this module, complementing the Pattern Constitution -->
```

### Section Notes

- **Intent**: one paragraph. Not a feature list, but "why it exists"
- **Public API**: external-facing interfaces. Changes here = breaking changes
- **Internal Structure**: how it's organized internally. Doesn't need line-level precision, but should give the reader a grasp of the module architecture
- **Extension Points**: how to extend. "Adding a new X" should modify what
- **Dependencies**: upstream and downstream. Who breaks if this changes
- **Gotchas**: traps. Accumulate from memory feedback and actual experience
- **Patterns**: code patterns unique to this module. Cross-module patterns go in the Pattern Constitution. If a pattern is already covered by the Constitution (with a corresponding Lint Assertion), don't repeat — use `→ see constitution: {domain}` as a pointer

## Mirror Intake (Before Spec Generation)

When the user signals intent to produce a change spec (triggering `/plan`, saying "write a spec for X", etc.), AI follows this flow:

### Step 1: Read onboarded state

**Before actively prompting, AI must first read** the `trigger_1_spec_intake.onboarded` value in `~/.claude/projects/{project}/.hydaelyn/onboarded.json` (treat missing file as `false`). This value determines the next prompt phrasing and whether skip is offered.

### Step 2: Active (conversational) prompt

**Non-first-time (`onboarded: true`)**:

> Want to do a Mirror intake first? I'll ask about 3 key questions around {brief direction}. (Or skip for now if you'd rather.)

**First-time (`onboarded: false` or missing file) — no skip option**:

> This is your first time hitting the spec-intake trigger. Walk through the flow once, then you can decide whether to keep using it. I'll ask 3 questions.

### Step 3: Branch on user response

- **Accept** → enter `/hyd-mirror` skill's intake flow (the skill updates `onboarded.json` on completion)
- **Decline** (only available when non-first-time) → **the rule layer (main-session AI) writes a skip trace line to today's session note on the spot**:

  ```
  - ⏭ {HH:MM} Skipped Mirror at spec-intake
  ```

  Then spec generation proceeds normally. The trace write is the rule layer's responsibility (the skill isn't invoked when the user declines, so the skill can't write it).

See `skills/hyd-mirror/SKILL.md` for the detailed intake flow, mutual calibration, and artifact format.

## Trigger Timing

### Auto-prompt to create (non-blocking)

- `/plan` touches a module with no system spec → propose creating one
- User can skip ("don't need it this time") → doesn't block the flow
- Accept → create during the /plan flow as Step 3.2 output

### Update timing

- After implementing a change spec → check whether the corresponding system spec needs updating
- Modified the module's Public API → must update
- Modified Internal Structure → suggested update
- `/session` knowledge extraction flags specs needing update
- When updating a spec, also update `last_verified` to today's date

### Staleness check

- `/housekeeping` scans specs with `last_verified` older than 30 days → prompts update
- Doesn't auto-update, only prompts

## Division of Labor with Memory

| Knowledge type | Location | Example |
|----------------|----------|---------|
| Structural | System Spec | API signatures, module dependencies, extension points |
| Experiential | Memory (feedback) | "Last time I changed X and Y broke" |
| State-like | Memory (project) | "Currently working on Z feature" |
| Normative | Rules | "All projects must do a context check" |

**Migration principle**: if memory contains content describing module structure (API, dependencies, internal organization), migrate it into the system spec when creating one; memory should retain only experiential content.

## OpenSpec CLI Integration

Use the CLI if available for speed; otherwise, manual Read/Write works. Format is identical.

```bash
# Initialize (optional)
openspec init

# Create a system spec
openspec spec create {module}

# Create a change spec
openspec change create {ticket-id}

# Archive after completion
openspec change apply {ticket-id}
```

**Not a dependency** — the CLI is just an accelerator. The core is the spec files themselves; any way of creating them works.
