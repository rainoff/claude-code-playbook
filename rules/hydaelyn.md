# Hydaelyn — System Naming and Positioning

> Hydaelyn is the core name of this methodology system, not a sub-project.

## Positioning

Hydaelyn is the umbrella name of the AI collaboration methodology represented by this `~/.claude/` setup. It's not an extra project or a feature module — it is the system itself.

| Aspect | Path / Location |
|--------|-----------------|
| Public repo | `rainoff/hydaelyn` (originally `claude-code-playbook`, renamed on 2026-04-17) |
| Private dev directory | `~/.claude/hydaelyn/` (peer to `rules/`, `skills/`, `agents/`, not a sub-project) |
| Sub-skill | `/hyd-mirror` (Mirror — understanding check at critical cold-start moments) |
| Future skill | `/hyd-spar` (Sparring — habit check across time, V2) |

## Core Intent: Hear / Feel / Think

Hydaelyn's architectural intent is **Hear / Feel / Think**. These are loops that AI and the user each run during dialogue, with the two loops pulling each other forward.

### Dual Loop Structure

```
         AI's loop                   User's loop
    ┌─────────────────┐          ┌─────────────────┐
    │  Hear the user  │ ←──────  │  State the need │
    │      ↓          │          │                 │
    │  Feel / probe   │ ──────→  │  Hear AI        │
    │      ↓          │          │      ↓          │
    │  Think (plan)   │          │  Feel same/diff │
    │      ↓          │          │      ↓          │
    │  Probe the plan │ ──────→  │  Think          │
    └─────────────────┘ ←──────  └─────────────────┘
           ↑                              │
           └──────────────────────────────┘
     Two loops pull each other forward in dialogue
```

### What the Verbs Mean

- **Hear**: AI receives the user's needs; the user hears AI's questions and responses
- **Feel**: AI probes and builds confidence from the probe's return; when AI senses its framing differs from the user's, they talk until both sides align. The user feels where AI's framing matches and where it diverges
- **Think**: not a separate stage — it's the state that emerges during the feel-phase conversation, plus the planning after a goal is set. AI probes the plan, listens to feedback, feels and thinks, then builds according to the confirmed plan

### Provoke — and the User Becomes Willing

- **Provocation is AI's action**: creating contrast, asking specific questions, restating to expose mismatches — so the user naturally compares and wants to respond
- **Willingness is the user's reaction**: running their own loop because the exchange carries meaning
- "Invite" is too passive; "guide" lets the user follow along without engaging; **provoke** is what kicks off the user's own loop
- If the user isn't willing, forcing doesn't help. The skip mechanism and the audience scoped to "people who want to reflect" both say the same thing: Hydaelyn serves only where willingness is already present

### Behavior Principles

- Not forced, not judgmental — respect the user's rhythm and choice
- AI actively reminds the user that tools exist; the user actively decides whether to enter
- Provoke the user to run their loop alongside — AI running alone isn't enough. Only then will the output match what the user actually wants

## Audience

Hydaelyn is designed for **users who want to reflect during AI collaboration**. Reflection is the capacity to willingly run one's own loop. Hydaelyn's tools (Mirror, the future Sparring) lower the friction of reflection and provoke willingness, but they don't run the loop for the user.

Respect every choice to enter or skip. Don't treat reflection as a mandatory gate. Tone: inclusive, inviting, respecting choice — no exclusionary language.

## Reference Conventions

When AI refers to "this methodology system" in dialogue, docs, or rules:
- Use the word "Hydaelyn" (capitalized)
- Avoid vague alternatives like "playbook", "framework", or "this setup"
- For skills in the system, use the `hyd-` prefix (e.g., `/hyd-mirror`)

## Relation to Other Rules

- `rules/sdd.md`: the spec-generation flow integrates Mirror Trigger 1
- `rules/session-management.md`: cross-day resumption detection integrates Mirror Trigger 2
- `rules/writing-style.md`: the six anti-patterns apply to all reader-facing prose, including Hydaelyn's README and docs
