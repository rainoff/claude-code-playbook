---
name: hyd-mirror
description: Hydaelyn Mirror — provides space to think at critical cold-start moments. Actively reminds the user when they're about to generate a spec (via /plan, "write a spec for X"), or when session startup detects cross-day resumption. User can also manually invoke /hyd-mirror, /hyd-mirror append intake|spec|review {spec-id}.
---

# Hydaelyn Mirror

> Audience: Mirror is designed for users who want to reflect during AI collaboration. It respects every choice to enter or skip, and doesn't treat reflection as a mandatory gate. Tone must stay neutral, inclusive, non-exclusive — never an examiner, but a serious-but-not-interrogating peer.

## Scope (Mirror ≠ /plan)

Mirror intake's purpose is **understanding calibration**, not producing a structured spec. `/plan` is the skill that structures already-understood content into a change spec. They're sequential, not interchangeable:

`/hyd-mirror` intake (calibrate understanding) → `/plan` (structure) → spec output

**Common misunderstanding**: treating Mirror intake as a substitute for `/plan`. The result is intake becomes "half-writing the spec", and `/plan` becomes formulaic restatement. Mirror's output is **aligned understanding**, not the spec artifact itself.

## When to Trigger

### Trigger 1 — Spec Intake (before spec generation)

1. User signals intent to produce a change spec (triggers `/plan`, says "write a spec for X", etc.)
2. AI actively (conversationally) asks: "Want to do a Mirror intake first? I'll ask about 3 key questions around {brief direction}."
3. User accepts → enter intake Q&A flow (below)
4. User declines → skip handling (see "Skip Mechanism")

### Trigger 2 — Cross-Day Resumption

1. Session starts
2. AI detects cross-day conditions (all true simultaneously):
   - Previous session note exists and has unchecked "Next Steps" items
   - Previous session note's date is earlier than today
   - User signals intent to continue those items (or mentions yesterday's work)
3. AI actively asks: "Want to do a Mirror first?"
4. Everything else same as Trigger 1

### Manual Invocation

User can `/hyd-mirror` anytime to manually enter the intake flow. Context is user-specified (e.g., "redo an intake for spec X").

## Intake Q&A Flow (shared across triggers)

### Step 1: Three Key Questions (required)

**AI generates them dynamically from the current context** — no fixed question bank. Banks lose contextual flexibility.

The "suggested question directions" below list the angles each trigger should cover. AI picks and rewrites them to fit the situation, and may replace with a better angle when one exists.

#### Suggested directions for Trigger 1

- **Problem definition**: who and what problem does this spec solve? (Anchor to concrete situations — avoid philosophical framings)
- **Failure imagination**: if this feature goes wrong or fails, how is it most likely to fail?
- **Implementation picture**: how clear is your mental image of what this looks like? Or are you waiting for AI to propose? (Checks depth of active understanding)

#### Suggested directions for Trigger 2

- **Memory continuity**: where did you leave off yesterday, and why did you stop there?
- **Goal alignment**: what's the reason to continue today? Is the goal still the same?
- **Risk reconsideration**: after a night away, is anything starting to feel off or worth reconsidering?

### Step 2: Mutual Calibration

After each answer, AI **restates in its own words or extracts** what it heard — not literal repetition. User replies:
- "You got it" → move to the next question
- "You missed the point — it's actually X" → AI re-extracts from the correction, recalibrates

#### Examples (good and bad)

❌ **Literal repetition (no value)**:
> User: "I want to address that uncertain heart after pressing YES."
> AI: "Okay, you want to address the uncertain heart after pressing YES."

✅ **Extracting / rephrasing (has value)**:
> User: "I want to address that uncertain heart after pressing YES."
> AI: "What I'm hearing: this isn't an architectural problem, it's an emotional+cognitive mismatch — the user made a commitment (pressed YES), but internally hasn't caught up. So Mirror's intervention point is before the commitment, not after-the-fact rescue."

❌ **Self-indulgent extension (drifts from user's intent)**:
> AI: "This really is the core dilemma of the entire AI collaboration era..."

✅ **Naming a latent insight (lets the user confirm)**:
> AI: "This also seems to imply a design decision: Mirror's output can't be a user monologue — there needs to be AI response so the user can verify 'you got it'. Does that reading match?"

### Step 3: Optional Deep Extension

After the key questions wrap, AI lightly asks: "Anything else you want to talk through?"

- User wants to go deeper → AI probes, generates more rounds
- User gives a closing signal ("let's get to work", "that's enough", "let's go with this", "OK") → AI reads as closure

**The closing signal is a context-aware read, not string matching.** Within the Mirror execution context, these phrases naturally mean closure; outside the Mirror context, they don't trigger.

**The "execution context" boundary**: starts the moment the user agrees to enter the intake, ends when the artifact is written. Any dialogue between (including when the user switches windows and comes back) is still inside execution context. Once the artifact is written, closing-signal detection turns off — a later "OK" won't trigger Mirror-end behavior.

### Step 4: Artifact Output

The whole conversation gets written to a file:

| Trigger | Artifact path |
|---------|---------------|
| Trigger 1 | `{spec-base}/.mirror/intake.md` (path resolution below in the Subcommand section) |
| Trigger 2 | A standalone `## Mirror Resumption` section in today's session note |
| Manual | Depends on context (with spec-id → same as Trigger 1; otherwise same as Trigger 2) |

#### Trigger 2 write-location rules

The `## Mirror Resumption` section goes in today's session note **before the "Completed" section, after the `# Session:` header** — recorded as the session's "opening ritual". Rationale: Mirror is pre-work alignment, happens before any work semantically, so it belongs in the first block.

**Coordinating with "same-day multi-session append format"**:
- `session-management.md` specifies using `---` to separate sub-sessions
- Mirror Resumption only records **the first Trigger 2 of the day** — same-day second triggers are unlikely by the detection conditions (previous session note must be older than today)
- If a same-day re-trigger does happen (e.g., user manually runs `/hyd-mirror` cross-day style), append to the existing section with `---` — don't open a new `## Mirror Resumption`

Artifact format:

```markdown
---
spec: {spec-id} (Trigger 1)
stage: intake | resumption | manual
date: {YYYY-MM-DD}
executor: Claude (hyd-mirror skill)
---

# Mirror {Intake|Resumption} — {context}

## Q1 — {question}
### User's answer
> {original}
### AI calibration (extracted)
- {key points}
### Calibration result
{got it / corrected understanding}

## Q2 — ...
## Q3 — ...

## Core Insights
1. ...

## Direction for next step
- ...
```

## First-Time Enforcement + Skip Mechanism

### Tracking Structure

Each project has `~/.claude/projects/{project}/.hydaelyn/onboarded.json`:

```json
{
  "trigger_1_spec_intake": {
    "onboarded": true,
    "first_completed_at": "2026-04-17T13:00:00+08:00"
  },
  "trigger_2_cross_day_resumption": {
    "onboarded": true,
    "first_completed_at": "2026-04-17T15:30:00+08:00"
  }
}
```

- **Each trigger tracked independently** — Trigger 1 and Trigger 2 each need one completion before skip is unlocked
- Missing file = treat all triggers as not yet onboarded

### Read/Write Responsibility Layering (important)

| Action | Responsible party | Timing |
|--------|------------------|--------|
| **Read** `onboarded.json` to check first-time status | **Triggering rule layer** (`rules/sdd.md`, `rules/session-management.md`) | Before AI actively prompts |
| **Write** `onboarded.json` to mark completion | **This skill** | After the intake flow completes (artifact written) |
| **Write skip trace** to session note (when user **declines** the prompt) | **Triggering rule layer** | At the moment user declines (skill isn't invoked at this point) |
| **Write mid-abort trace** (user entered intake but left mid-flow) | **This skill** | When skill detects an abort signal |

**Why this split**: the skill is only invoked after the user agrees to enter; first-time checks and decline traces both happen before invocation, so they must be handled by the rule layer.

### First-Time Enforcement (judged by the rule layer, passed to the skill)

When the rule layer reads `onboarded: false`, it uses the no-skip phrasing (see `rules/sdd.md`, `rules/session-management.md`). The skill itself assumes onboarded status has already been checked by the rule layer when it's entered — it doesn't re-check.

After the skill completes the intake (artifact successfully written), it **updates** the corresponding trigger in `onboarded.json` to `onboarded: true`, `first_completed_at: {ISO 8601 now}`.

### Non-First-Time Skip (follow-up reminder)

When the user has declined a prompt at a non-first-time trigger, the rule layer records a trace in the session note. At the next session startup, the "Mirror Skip-Followup Reminder" rule in `rules/session-management.md` detects the trace and reminds the user to catch up. The skill itself doesn't handle this detection — it's only invoked if the user decides to run the follow-up.

## Subcommand: `/hyd-mirror append`

Mirror's core idea is "thinking isn't done in one pass". Post-hoc insights must be writable back to the relevant artifact.

### Path Resolution Rule

`{spec-id}` can correspond to a spec in one of two locations:

1. **Hydaelyn system dog-food spec**: `~/.claude/hydaelyn/specs/{spec-id}/spec.md` (directory style)
2. **General project change spec**: `{project-root}/specs/{spec-id}.md` (single-file style, per `rules/spec-template.md`)

When AI invokes append, it first determines the correct path from the current CWD or user-specified context. If both locations contain specs with the same spec-id, AI directly asks the user which one to append to.

### Three Targets

#### `/hyd-mirror append intake {spec-id}`

- **Target file**: `{spec-base}/.mirror/intake.md`
- **Action**: **append** a section to the end of the existing intake.md (see "Append format" below); don't create a new file
- **If the file doesn't exist**: tell the user "can't find an intake for {spec-id} — want to run a formal intake first?", don't auto-create
- **Same-day multiple invocations**: keep appending, each section with its own timestamp

#### `/hyd-mirror append spec {spec-id}`

- **Target file**: resolve via the path rule to find `spec.md` (system style) or `{spec-id}.md` (general style)
- **Action**: always append to the `## Post-Hoc Insights` section. If the section doesn't exist, auto-create it at the end of the file
- **Don't do "inline annotation near relevant sections"**: avoid having AI guess positions and break the spec's main structure
- **If you want to edit the main spec body**: user must explicitly state "this is a clarification, not a post-hoc note" — then they edit manually or AI assists editing the target section; this doesn't go through append

#### `/hyd-mirror append review {spec-id}`

- **Action semantics**: start a new review conversation round, output to `{spec-base}/.mirror/review-{YYYY-MM-DD}.md`
- **Naming**: the subcommand reuses `append` so the three actions stay semantically consistent (all "post-hoc writeback"), but **under the hood it's creating or appending to a file**
- **Same-day multiple invocations**: append to the existing same-day `review-{date}.md` at the end (separated by `---`); don't create `review-{date}-{N}.md`, which would balloon files

### Append Format (shared by all three)

Each appended section must have a timestamp marker:

```markdown
---

## Post-hoc insight — {YYYY-MM-DD HH:MM}

{content}
```

### Avoid Append Abuse

Before appending, ask yourself: **is this a clarification or correction to the spec's original intent, or just a passing thought?**

- Has impact on the spec → `append`
- Passing thought → write in the session note instead

The skill actively reminds the user of this judgment when `/hyd-mirror append` is invoked.

## Artifact Directory Structure

```
{spec-base}/.mirror/
├── intake.md              # Initial intake (Trigger 1); post-hoc additions also write here
└── review-{date}.md       # Post-spec review notes (same-day additions appended to same file)
```

V1 scope covers only `intake.md` and `review-{date}.md`. Other uses (like "accumulating insights during implementation") are deferred to V2 to evaluate whether a separate file is needed.

## Gotchas

1. **Too much friction → pure cost**: respect the user's rhythm; don't turn reflection into judgment. Skip leaves a trace and respects the choice
2. **Abstract questions → empty answers**: questions must anchor to concrete situations ("go back to the moment you pressed YES yesterday") — avoid philosophical framings like "pause and think"
3. **Mutual calibration becomes self-indulgent parroting**: AI's "what I'm hearing is..." carries no value if it's just literal repetition. Must extract, rephrase, name a latent insight
4. **AI forgets to actively prompt**: rule-level behavior instructions are easy to skip. In V1.1, if this is observed, upgrade to hardcode an invocation at the end of `/plan` skill
5. **Cross-day false triggers**: Trigger 2's conditions should stay conservative — missed triggers are better than false ones. Interruption is worse than omission
6. **"First time" definition**: means the first time for each trigger, not a global first time. Trigger 1 and Trigger 2 each need a walkthrough
7. **Intake becomes an exam**: if question tone feels like audit or exam, the user enters answer-mode instead of reflect-mode. Mirror's tone is "a serious-but-not-interrogating peer", not an examiner
8. **Closing signal context-aware misread**: "let's get to work" / "OK" are read as closure only inside the Mirror execution context. Don't trigger outside
9. **`append` abuse → Feedback Loop becomes noise**: passing thoughts that don't affect the spec's intent shouldn't be appended — put them in the session note

## Related Rules

- `rules/hydaelyn.md` — system naming and positioning
- `rules/sdd.md` — upstream rule for Trigger 1
- `rules/session-management.md` — Trigger 2 + skip-followup rules
