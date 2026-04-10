# claude-code-playbook

A battle-tested playbook for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — rules, agents, skills, hooks, and memory system to keep AI disciplined and productive.

This repo maps to the `~/.claude` directory. Clone it and adjust personal settings to get started.

> **Note**: This configuration was built incrementally from real usage, not designed all at once. Start with a few rules, add more as you encounter pain points.

## Directory Structure

```
~/.claude/
├── CLAUDE.md                 # 5 core principles + Context Check
├── settings.json             # Permissions, hooks, language settings
├── keybindings.json          # Custom keybindings
│
├── rules/                    # Always-active behavior rules (13)
│   ├── dev-workflow.md       # Task startup, commit discipline, testing, quality, context management
│   ├── session-management.md # Session notes, memory search fallback, stale todo check
│   ├── knowledge-index.md   # Three-tier memory system (Hot/Warm/Glacier)
│   ├── jira-sync.md         # 5-trigger JIRA sync
│   ├── verification-loop.md # Critic → Alignment Check → verification loop (max 2 fix rounds)
│   ├── ownership.md         # CODEOWNERS-based cross-module change checks
│   ├── spec-template.md     # Spec format and AC-Test mapping
│   ├── translation-system.md # Domain-specific: translation pipeline rules (example, replaceable)
│   ├── figma-workflow.md    # Figma MCP 4-phase workflow
│   ├── visual-ui-workflow.md # Visual change workflow (Playwright screenshot verification)
│   ├── subagent-strategy.md # Fork/Teammate/Worktree delegation strategy
│   ├── web-fetch-safety.md  # External content safety (anti prompt injection)
│   └── CHANGELOG.md         # Rule change log
│
├── agents/                   # Subagents (isolated context)
│   ├── task-executor.md     # Builder — implements single subtask (Sonnet)
│   ├── critic.md            # Adversarial review — pattern consistency as #1 dimension (Opus)
│   ├── alignment-checker.md # External reference alignment — Figma/schema/spec (Opus)
│   ├── code-reviewer.md     # Logic, error handling, code patterns (Opus)
│   ├── security-reviewer.md # OWASP, permissions, secrets, injection (Opus)
│   └── code-simplifier.md   # Remove dead code, over-abstraction (Sonnet)
│
├── skills/                   # Intent-triggered workflows (auto-detected)
│   ├── autopilot/           # Auto build→critic→alignment→fix loop
│   ├── git-commit/          # Generate and execute git commit
│   ├── pr/                  # Create MR/PR from commits
│   └── review/              # Parallel 3-agent review before push
│
├── commands/                 # Slash commands (manually invoked)
│   ├── plan.md              # /plan — structured feature planning, outputs spec
│   ├── project-init.md      # /project-init — initialize project setup
│   ├── session.md           # /session — progress snapshot + knowledge extraction
│   ├── housekeeping.md      # /housekeeping — memory cleanup and archival
│   ├── reflect.md           # /reflect — review sessions, detect violations
│   ├── evolve.md            # /evolve — modify rules based on reflect output
│   ├── careful.md           # /careful — enable destructive command blocking
│   └── freeze.md            # /freeze — lock edit scope to specified directory
│
└── scripts/                  # Hook scripts and tools
    ├── setup.sh             # One-click bootstrap
    ├── claude-notify-macos.sh  # macOS push notification (osascript)
    └── claude-notify-linux.sh  # Linux push notification (notify-send)
```

## Core Design Philosophy

### Five Core Principles (CLAUDE.md)

1. **Never assume** — Can't find the info? Ask or look it up. Never guess.
2. **Context check** — Before every code change, output: Modules / Memory / Existing patterns / Upstream / Unsure
3. **Follow existing patterns** — Consistency > cleverness. Don't introduce new patterns without approval.
4. **Context compaction preferences** — Guide the harness on information priority during compaction.
5. **Decide correctly** — Decision table for when to ask / look up / act.

### Dual Execution Modes

| Mode | Use Case | Pattern Consistency |
|------|----------|-------------------|
| `conservative` (default) | Existing projects | Enforced |
| `rapid` | New projects / fast iteration | Relaxed |

### Three-Tier Memory System

```
Hot    → MEMORY.md (always loaded, ≤50 lines, knowledge map + pointers)
Warm   → memory/*.md (loaded on demand, frontmatter description for quick scan)
Glacier → same directory, frontmatter archived: true (no file relocation)
```

### Verification Loop

```
task-executor (Sonnet) implements
        ↓
critic (Opus) — fresh context, no builder conversation history
        ↓ PASS
alignment-checker — external reference alignment (Figma/schema/spec)
        ↓ ALIGNED
main session verification → commit
```

- Max 2 fix rounds per step. More than that means the decomposition granularity is wrong.

### Self-Iteration Loop

```
/plan → /autopilot → /session → /reflect → /evolve
```

### Hooks

| Hook | Trigger | Action |
|------|---------|--------|
| PreToolUse (Edit/Write) | Before every file write | Remind Context Check + MEMORY.md read |
| PreCompact | Before context compaction | Remind to run /session for knowledge extraction |
| Stop (self-check) | When AI stops | Self-verify: is the task actually complete? |
| Stop (uncommitted) | When AI stops | Warn about uncommitted files |

Add push notification hooks as needed — `scripts/` has macOS and Linux versions. For Windows, use PowerShell's `New-BurntToastNotification` or similar.

## Usage

### Quick Start

```bash
# 1. Clone
git clone https://github.com/rainoff/claude-code-playbook.git ~/.claude

# 2. Customize
#    - settings.json: adjust permission allowlist, hooks
#    - rules/: remove rules you don't need (e.g., translation-system.md, jira-sync.md)
#    - CLAUDE.md: adjust principles (most can be used as-is)
```

### Recommended Onboarding Order

Don't enable everything at once. Recommended:

1. **Week 1**: `CLAUDE.md` + `rules/dev-workflow.md` + `rules/session-management.md` only
2. **Week 2**: Add `agents/` and `skills/git-commit/`
3. **As needed**: Gradually add other rules and commands

### Customization

| Need | What to Do |
|------|-----------|
| Don't use JIRA | Delete `rules/jira-sync.md` |
| Don't use Figma | Delete `rules/figma-workflow.md` |
| Have your own domain rules | Replace `rules/translation-system.md` |
| Use macOS | Notification hook: `claude-notify-macos.sh` (included) |
| Use Linux | Notification hook: `claude-notify-linux.sh` (included) |
| Use Windows | Write your own PowerShell notification script |
| Don't want Chinese responses | Change `language` in `settings.json` |

## Design Philosophy

- **Rules grow from mistakes** — Not designed upfront. Every rule has a failure case behind it.
- **Deterministic where possible** — Hooks use deterministic checks (git status, echo reminders), not AI self-discipline.
- **Critic must be context-isolated** — AI reviewing its own code will rationalize. A fresh-context critic is effective.
- **Three tiers solve stale memory** — Stale memory is worse than no memory. Tiering + archival controls memory quality.

## License

Personal configuration. MIT License. Fork and adapt freely.
