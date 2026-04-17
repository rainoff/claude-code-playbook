---
ticket: hydaelyn-mirror-v1
created: 2026-04-17
updated: 2026-04-17
module: hydaelyn
status: draft
---

# hydaelyn-mirror-v1: Hydaelyn Mirror v1

## Context

Hydaelyn is the core name of this methodology system — not a new sub-project, but the umbrella name for the entire AI collaboration methodology. The public repo is `rainoff/hydaelyn` (originally `claude-code-playbook`, renamed during V1). Hydaelyn's philosophy: provide **space to think** at critical cold-start moments in user–AI collaboration — not forced, not judgmental, only guiding.

Mirror is the first concrete skill under Hydaelyn, handling **understanding checks on a single artifact in the moment**. Its counterpart Sparring handles habit checks across time and is deferred to V2.

**Audience**: Mirror is designed for users who want to reflect during AI collaboration. It respects every choice to enter or skip, and doesn't treat reflection as a mandatory gate — everyone has a different rhythm, and Mirror's job is to make space.

This spec defines the minimum viable shape of Mirror v1: two automatic reminder triggers, a user-led entry flow, mutually calibrated output, and a writable feedback loop.

## Current Behavior

Users currently lack a reflection tool in these moments:

1. **Before spec generation**: user invokes `/plan` or explicitly asks to write a change spec, and AI proceeds directly to production. The user's understanding state (whether they've really thought through what needs solving, whether AI caught the user's intent) is unchecked
2. **Resuming work across days**: at session startup, the previous session note is read as passive resumption, with no mechanism to check "after a night away, do I still remember why I'm doing this"

Result: when the user presses YES, some uncertainty often remains, and the spec can degrade into AI-authored / user-rubber-stamped sugar. Implementation drift snowballs from this starting point.

## Target Behavior

Mirror provides a **Q&A space that can go as deep or shallow as needed**, appearing at two automatic reminder moments. Core principle: **AI actively reminds, user actively enters**. The system doesn't force interruption, but doesn't rely on the user remembering Mirror exists either.

### Trigger 1: Spec Intake (before spec generation)

1. User signals intent to open a change spec (invokes `/plan`, says "write a spec for X", etc.)
2. AI **actively prompts (conversational)**: "Want to do a Mirror intake first? I'll ask about 3 key questions around {brief direction}." V1 tries the conversational form; if this feels too heavy in practice, V1.1 can switch to a non-blocking one-line reminder
3. User accepts → enter intake Q&A flow
4. User declines → skip (flow continues). Today's session note briefly records the skip (one line: "{time} skipped Mirror at {trigger}"). **At the next session startup, AI has the duty to actively remind**: "You skipped Mirror at {trigger} last time — want to do it now?" — no pressure, but the reminder is owed
5. **First-time users can't skip** at each trigger — they must walk through once before unlocking the skip option

### Trigger 2: Cross-Day Resumption

1. Session starts
2. AI detects cross-day conditions:
   - Previous session note exists and has unchecked "Next Steps" items
   - Previous session note's date is earlier than today
   - User signals intent to continue those items (or mentions yesterday's work in the startup conversation)
3. All conditions met → AI actively prompts: "Want to do a Mirror first?"
4. Rest of the flow same as Trigger 1

### Intake Q&A Flow (shared structure; questions generated dynamically by AI based on context)

1. **Key questions (required)**: about 3 core alignment questions, generated dynamically from the current context (**no fixed question bank — fixed question banks lose contextual flexibility**). The "Suggested question directions" below list angles each trigger should cover; AI picks and rewrites them to fit, and may substitute a better angle when one exists
2. **Mutual calibration**: after each answer, AI restates or extracts in its own words (**not literal repetition**). User replies "you got it" or "you missed it, it's actually X"
3. **Optional deep extension**: after the key questions wrap, AI lightly asks "anything else you want to talk through?". If the user wants to go deeper, AI probes further; when the user gives a closing signal (e.g., "let's get to work", "that's enough", "let's go with this", "OK"), AI reads as closure — **this is a context-aware read, not string matching**. Within the Mirror execution context, these phrases naturally mean closure; outside the Mirror context they don't trigger
4. **Artifact output**: the full conversation gets written to a file
   - Trigger 1 → `specs/{spec-id}/.mirror/intake.md`
   - Trigger 2 → a standalone `## Mirror Resumption` section in today's session note

### Trigger 1 suggested question directions (generated dynamically, not a fixed bank)

- **Problem definition**: who and what problem does this spec solve? (Anchor to concrete situations — avoid philosophical framings)
- **Failure imagination**: if this feature goes wrong or fails, how is it most likely to fail?
- **Implementation picture**: how clear is your mental image of what this looks like? Or are you waiting for AI to propose? (Checks depth of active understanding)

### Trigger 2 suggested directions

- **Memory continuity**: where did you leave off yesterday, and why did you stop there?
- **Goal alignment**: what's the reason to continue today? Is the goal still the same?
- **Risk reconsideration**: after a night away, is anything starting to feel off or worth reconsidering?

### Feedback Loop (post-hoc writeback)

Mirror's core idea is "thinking isn't done in one pass". New insights from intake / review / implementation must be writable back to the relevant artifact:

1. **Intake supplement**: after intake ends, if the user thinks of something new, they can append a section via `/hyd-mirror append intake {spec-id}` (with timestamp)
2. **Review writeback**: issues found in post-spec review can directly edit spec.md (manually or with AI assistance); the review record lives at `specs/{spec-id}/.mirror/review-{date}.md`
3. **Mid-implementation insight writeback**: when the user finds the spec is wrong or intake missed something during implementation, they can `/hyd-mirror append spec {spec-id}` or `/hyd-mirror append intake {spec-id}` — no need to open a full change spec flow

Mirror artifact directory structure:

```
specs/{spec-id}/.mirror/
├── intake.md              # Initial intake
├── intake.append.md       # Post-intake supplements (may or may not exist)
├── review-{date}.md       # Post-spec review notes
└── execution-notes.md     # Implementation-phase insight accumulation
```

## Files to Change

- `~/.claude/skills/hyd-mirror/SKILL.md` (new) — defines how `/hyd-mirror` is invoked, intake flow, artifact format, mutual calibration output requirements, `/hyd-mirror append` subcommand
- `~/.claude/rules/sdd.md` (modify) — add rule: "before spec generation, AI must actively (conversationally) remind Mirror is available and wait for user response"
- `~/.claude/rules/session-management.md` (modify) — add: "on session startup with cross-day resumption detected, AI actively reminds Mirror"; "if last session skipped Mirror, next session startup prompts follow-up"; skip trace format
- `~/.claude/rules/CHANGELOG.md` (append) — record Hydaelyn Mirror v1 introduction
- `~/.claude/CLAUDE.md` or new rule `~/.claude/rules/hydaelyn.md` (new) — explicitly declare Hydaelyn is the core name of this methodology system (not a sub-project); the public repo uses the Hydaelyn name
- `~/.claude/hydaelyn/` (new minimal directory) — Hydaelyn's own SDD workspace (peer to `rules/`, `skills/`, `agents/`; not under the default-gitignored `projects/`). V1 only creates `specs/`; `memory/`, `sessions/` etc. are created when needed
- `~/.claude/skills/sync-public/SKILL.md` (modify) — when processing `hydaelyn/specs/` paths, add a light tone-filter pass (V1 syncs only spec.md; `.mirror/` stays private)

## Acceptance Criteria

- [ ] AC-1: `/hyd-mirror` skill exists at `~/.claude/skills/hyd-mirror/`, invocable via slash command
- [ ] AC-2: `sdd.md` clearly specifies "before spec generation, AI must actively (conversationally) remind Mirror is available and wait for user response"
- [ ] AC-3: `session-management.md` clearly specifies "on session startup with cross-day resumption detected, AI actively reminds Mirror" and defines the detection conditions
- [ ] AC-4: each Mirror intake trigger includes: AI dynamically generates ~3 key questions → mutual calibration → optional deep extension (user's closing signal ends the flow via AI's context-aware read) → artifact written; all four steps complete
- [ ] AC-5: first-time users can't skip each trigger; tracking via `~/.claude/projects/{project}/.hydaelyn/onboarded.json`, recording whether each trigger has been first-completed
- [ ] AC-6: non-first-time users can skip; skip events are recorded as one line in today's session note; next session startup, AI actively prompts the follow-up
- [ ] AC-7: Trigger 1's artifact path is `specs/{spec-id}/.mirror/intake.md`; Trigger 2's artifact is the `## Mirror Resumption` section in today's session note
- [ ] AC-8: SKILL.md opens with an explicit audience statement: "Mirror is designed for users who want to reflect during AI collaboration; respects every choice to enter or skip" — tone must be neutral, inclusive, non-exclusive
- [ ] AC-9: AI restatements in mutual calibration must be **extraction or rephrasing**, not literal repetition (SKILL.md provides good/bad examples)
- [ ] AC-10: `/hyd-mirror append` subcommand supports three targets (`intake`, `spec`, `review`), writing post-hoc insights back to the corresponding artifact
- [ ] AC-11: Hydaelyn's positioning is clearly declared in `CLAUDE.md` or `rules/hydaelyn.md`: it is the core name of this methodology system (not a sub-project); the public repo uses this name
- [ ] AC-12: before public sync of `hydaelyn/specs/`, spec.md must go through a light tone-filter — AI does initial pass, user does final confirmation. Check directions stay light (absolute assertions, residual first-person emotion, exclusionary phrasing), no mandatory auto-audit. `.mirror/` subdirectories (intake / review) stay private and don't sync. sync-public SKILL.md records this

## Testing Strategy

- **Manual dog-food**: the author is the first user. During the first week, every spec intake and every cross-day resumption runs Mirror; artifacts are reviewed immediately to verify they actually helped clarify understanding
- **First-week check-in**: on day 7, do a short review of all `.mirror/intake.md` produced that week, judging:
  - Whether the dynamically generated questions actually caught the key issues, or became formulaic
  - Whether the AI restatements in mutual calibration actually added extraction value
  - Whether the skip option got abused
  - Whether Feedback Loop (`append` mechanism) actually got used
- **Skip path verification**: deliberately test a skip once; confirm the trace appears and the next session startup AI prompts
- **No automated tests**: this is a behavioral rule + skill, verified primarily by hand

## Known Pitfalls

1. **Too much friction → pure cost**: the top failure mode noted by the user. Mitigation: preserve the skip option with trace, respect the user's rhythm, don't turn reflection into judgment
2. **Abstract key questions → empty answers**: questions must anchor to concrete situations ("go back to the moment you pressed YES yesterday") — avoid philosophical framings like "pause and think"
3. **Mutual calibration becomes AI self-indulgent parroting**: AI's "what I'm hearing is..." has no value if it's just literal repetition. Must extract, rephrase, name a latent insight — so the user can judge whether AI actually got it or is just repeating form
4. **AI forgets to actively prompt**: rule-level behavior instructions are easy to skip; if observed in V1.1, upgrade to hardcode an invocation at the end of `/plan` skill
5. **Cross-day false trigger**: Trigger 2 relies on detecting whether the user wants to continue yesterday's work — conditions must stay conservative; missed triggers are better than false ones (interruption is worse than omission)
6. **"First-time" definition**: means the first time for each trigger, not a global first time. Trigger 1 and Trigger 2 each need a mandatory walkthrough
7. **Intake becomes an exam**: if question tone feels like audit or exam, the user enters answer-mode instead of reflect-mode. SKILL.md must require Mirror's tone to be "a serious-but-not-interrogating peer", not an examiner
8. **Closing signal context-aware misread**: "let's get to work" / "OK" read as closure within Mirror execution; if AI misreads similar phrases outside Mirror as closure, Mirror breaks. Mitigation: `/hyd-mirror` has clear "enter" and "leave" boundaries — closing signals detected only within Mirror execution, not outside
9. **`/hyd-mirror append` abuse → Feedback Loop becomes noise**: if every passing thought gets appended, the artifact becomes cluttered. SKILL.md should suggest: "before appending, ask — is this a clarification or correction to the spec's intent, or a passing thought? The latter goes in the session note"

## Public Distribution Strategy

Hydaelyn's publication plan is split into two stages. V1 itself doesn't execute any publication actions; it only records the strategy to give future restructuring a direction.

### Stage A: Internal Sharing (colleagues trying it out)

Goal: let colleagues try, reference, or fork without going through the full plugin marketplace flow.

Approach:
- Public repo uses `rainoff/hydaelyn` (renamed from `claude-code-playbook` during V1)
- Complete README covering Hydaelyn's philosophy, Mirror usage, SDD integration, manual install steps
- Lightweight `./install.sh` that copies `skills/hyd-mirror/`, `rules/hydaelyn.md` etc. to the user's `~/.claude/`
- Users choose read-only reference, partial copy, or full install

### Stage B: Formal Release (open community)

Goal: zero-friction install, aligned with Claude Code official conventions; modelled after BMAD's validated route.

Approach:
- Standalone repo (continuing from Stage A) with `.claude-plugin/marketplace.json`
- Skills uniformly prefixed `hyd-` (`/hyd-mirror`, future `/hyd-spar`) to avoid clashing with official or other plugins
- Plugin install: `/plugin marketplace add rainoff/hydaelyn` + `/plugin install hydaelyn-mirror@rainoff`
- **Rules can't be bundled into plugins**: maintain a separate `rules/` directory as drop-in fragments for manual copy or CLAUDE.md include; README explicitly states "rules need manual install"
- Separate `src/` (dog-food in progress) from `installed` (published snapshot), learned from BMAD

### V1 Groundwork for Future Release

- Skill name uses `/hyd-mirror` directly in V1 (original plan was V1 `/mirror` → Stage B rename to `/hyd-mirror`; during the 2026-04-17 Mirror Resumption, the migration step was skipped to avoid muscle-memory migration costs)
- Rules change path stays unchanged (V1 modifies existing rules); Stage B extracts them into standalone `.md` drop-ins
- Dog-food directory `~/.claude/hydaelyn/` can be extracted into its own repo later
- Naming consistency: all V1 documents, specs, session notes use Hydaelyn as the system name — avoiding future large-scale rename

### Spec Publication Policy

In V1, only `hydaelyn/specs/**/spec.md` is synced to the public repo. The `.mirror/` subdirectories (intake.md, review-{date}.md) stay private — their first-person density generates more friction than value for an outside audience.

spec.md goes through a light tone-filter pass (independent of translation):
- **Check directions (light, not a hard checklist)**: absolute assertions, company-internal context phrases, residual first-person
- **Execution**: AI does initial filter, user does final confirmation
- **Preservation judgment**: rather leave rough edges than over-sanitize; each file gets its own judgment

If the dog-food narrative (showing how Mirror was used to write Mirror) wants external presentation later, use hand-picked quotations in README / blog / talks — not automated sync.

### Key References

- **BMAD-METHOD** (10.4k stars) — plugin marketplace + `bmad-` prefix + src/installed separation; a validated case study
- **Claude official plugin mechanism** — `.claude-plugin/manifest.json`, `marketplace.json`, automatic namespace to avoid clashes

## Resolved Decisions (traceback from the initial spec to now)

- **Dynamic generation vs fixed question bank** → dynamic. Fixed question banks lose contextual flexibility. SKILL.md provides "suggested question directions" for AI to pick from based on context
- **Active reminder form** → conversational prompt. V1 tries it; if too heavy, V1.1 switches to non-blocking
- **Closing signal for deep extension** → trust AI's context-aware read, no string matching; detect only within Mirror execution context
- **Hydaelyn project directory build approach** → V1 only creates `specs/`, others added when needed
- **Skip trace format** → one-line note in session note + next session startup, AI actively prompts the follow-up
- **Hydaelyn positioning** → the core name of this methodology system (not a sub-project); the public repo uses the Hydaelyn name
- **Audience tone** → inclusive, inviting, respecting choice; no exclusionary language (considering future public scenarios)
- **Process file publication handling** (emergent from 2026-04-17 Mirror Resumption Trigger 2 first dog-food; revised later the same day) → V1 syncs only `spec.md` with a light filter; `.mirror/` subdirectories (intake / review) stay private. Reason: first-person density in process files produces more friction than value for outside readers, and mechanical filtering would erase dog-food authenticity — hand-picked quotes in blog / README suit narrative sharing better
- **Skill name finalized as `/hyd-mirror` in V1** (from the same Mirror Resumption) → skip the original two-stage naming (V1 `/mirror` → Stage B `/hyd-mirror`), use the final name directly. Reason: the public repo is already named `hydaelyn`; consistent skill prefix helps external communication; muscle-memory migration cost outweighs the "4 extra characters" during dog-food

## Open Questions (genuinely unresolved)

- **Structure of `.hydaelyn/onboarded.json`** — tracking first-time completion for each trigger; finalize during skill implementation
- **Concrete mutual-calibration example prompts in SKILL.md** — draft good/bad examples when implementing the skill, for AI reference
- **`/hyd-mirror append` artifact merge logic** — section format when appending to existing files? Timestamp style? Decide during implementation
