# Session Management Rules

## Auto-Writing Session Notes

Only write a session note at these moments (path: `~/.claude/projects/{project}/sessions/{date}.md`; **in a worktree, write to the parent** — see `worktree-memory.md`):

1. **Explicit user request** (/session, or phrases like "done", "that's enough for today", "continue tomorrow") — write a complete handoff
2. **After a commit — session writeback** (two actions together):
   - Append a line `- ✅ {hash} {message}` to today's session note's "Completed" section (if the file already exists)
   - Check the previous session note's "Next Steps" section: if the commit completed one of the items, flip it from `[ ]` to `[x]` and annotate the commit hash

Don't proactively write session notes at other times.

## Session Note Format

```markdown
# Session: {date}

## Completed
- ✅ {commit hash} {commit message}
- ⚠️ {uncommitted} {describe work done but not yet committed}

> Every completed item must be marked ✅ (committed) or ⚠️ (uncommitted). Uncommitted work should be the first thing handled next session.

## Key Decisions (if any)
- {only decisions that will affect future judgment}

## Next Steps
- [ ] {specific next action}
```

## Starting a New Session

At the start of every new conversation, if the user asks "where were we", "next step", "continue", or similar resumption cues:

1. First read the current project's `sessions/` and `memory/MEMORY.md`
2. **Worktree awareness**: if the project key contains `-worktrees-`, resolve the parent project key (take the part before `-worktrees-`) and read from the parent's `memory/` and `sessions/`. See `worktree-memory.md`
3. **If the path can't be found, you must use `glob **/MEMORY.md` and `glob **/sessions/**/*.md` to search downward from `~/.claude/projects/`**
4. Never declare "no records found" after trying just one path — the provided project key may differ from the actual directory name
5. Once located, answer based on what you found

### Pending-Progress Check

Read the most recent session note's "Next Steps". If there are unchecked items (`- [ ]`) more than 3 days old:
- List them and ask the user: "Which of these leftover tasks have you finished?"
- After the user answers, update the session note (check off completed items) and sync the relevant memory

**Why:** tasks that got done but never got written back is the most common source of memory staleness. Asking once costs far less than auditing to fix it later.

### Mirror Resumption (Trigger 2 — cross-day intake)

#### Step 1: Cross-day detection (all three must be true)

- A previous session note exists and has unchecked "Next Steps" items
- The previous session note's date is earlier than today
- The user signals intent to continue those pending items (or mentions yesterday's work in the conversation)

**Triggering examples**: the user mentions yesterday's ticket ID, asks "where did I leave off", or explicitly says "continue X"
**Non-triggering examples**: simple greeting ("morning"), or a completely unrelated new topic

**False triggers are worse than missed triggers** — when conditions aren't clearly met, skip the prompt rather than interrupt.

#### Step 2: Read onboarded state

**Before actively prompting, AI must first read** the `trigger_2_cross_day_resumption.onboarded` value in `~/.claude/projects/{project}/.hydaelyn/onboarded.json` (treat missing file as `false`).

#### Step 3: Active (conversational) prompt

**Non-first-time (`onboarded: true`)**:

> Want to do a Mirror first? (To realign after the overnight gap.) (Or skip for now if you'd rather.)

**First-time (`onboarded: false` or missing file) — no skip option**:

> This is your first time hitting Mirror cross-day resumption. Walk through the flow once, then you can decide whether to keep using it. I'll ask 3 questions.

#### Step 4: Branch on user response

- **Accept** → enter `/hyd-mirror` skill's intake flow (the skill updates `onboarded.json` on completion)
- **Decline** (only available when non-first-time) → **the rule layer (main-session AI) writes a skip trace line to today's session note on the spot**:

  ```
  - ⏭ {HH:MM} Skipped Mirror at cross-day-resumption
  ```

The trace write is the rule layer's responsibility (the skill isn't invoked when the user declines, so it can't write).

See `skills/hyd-mirror/SKILL.md` for the detailed flow, mutual calibration, and artifact format.

### Mirror Skip-Followup Reminder

After session startup, scan the most recent session note for entries matching:

```
- ⏭ {HH:MM} Skipped Mirror at {trigger-name}
```

If found, AI must actively remind:

> Last time you skipped Mirror at {trigger}. Want to run it now? (No pressure.)

Once the user decides — whether to run it or not — this item isn't reminded again (unless skipped again next time).

**Why:** skipping is the user's right, but AI has a duty to remind. Trace + follow-up reminder prevents "skip becoming silent disappearance".

### MR Convergence Reminder

When "Next Steps" contains an "open MR" or "push" item still unfinished after 5 days:
- Actively remind: "The MR item has been pending for more than 5 days. Suggest scope freeze: open the MR with the current changes and file new findings as separate tickets."
- This is a required reminder, not a suggestion — scope creep is the most common cause of MR delay.

### External-Pending Tracking

Items that need response from external people (design, PM, backend) to move forward should be marked with `⏳` in "Next Steps":
```
- ⏳ {item} — waiting on {who}'s reply ({date raised})
```
After 7 days with no progress, session startup prompts the user to decide: fix, postpone, or mark wontfix. Don't let external-pending items vanish silently.

### Rule-Change Check

At every session startup, read the latest entry date in `~/.claude/rules/CHANGELOG.md` and compare with the current project's most recent session note date:
- New rule changes → remind: "Global rules updated on {date}: {summary}. Want to check whether this project's memory needs alignment?"
- No new changes → no reminder

## Division of Labor with memory/

- **Session notes**: short-term handoff (valid within a few days), record "what I did" and "what's next"
- **memory/**: long-term knowledge (valid across weeks), record "why this way" and "how the system works"
- Don't maintain todos in both session notes and memory
- Multiple sessions on the same day share one note (append, don't overwrite) — format below

### Same-Day Multi-Session Append Format

Use `---` to separate sub-sessions. But the whole file keeps **only one** "Next Steps" block (at the very bottom):

- When adding a sub-session, separate with `---`
- New completed items go directly below the separator (no need to repeat the `## Completed` header)
- New key decisions go into the **existing** "Key Decisions" block (don't open a new one)
- Completed items from the old "Next Steps" get checked and removed; unfinished items are merged into the newest "Next Steps"
- The final file has only one "Next Steps" block at the bottom
