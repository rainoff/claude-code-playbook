# Rules Changelog

> Record one line for every global rule change. At session startup, compare against the most recent session note date and notify the user if there are new entries.

- 2026-04-07: knowledge-index — L0 now uses frontmatter description, Glacier archiving uses archived flag (no file moves), MEMORY.md no longer stores current-state snapshots
- 2026-04-07: session-management — added "todo progress check": at session startup, proactively ask the user about todos unresolved for more than 3 days
- 2026-04-07: web-fetch-safety — added external content safety policy: three-step verification for WebFetch results + MCP installation safety checks
- 2026-04-10: scoped-rules — moved translation-system to project-level, added activation condition to public-repo-sync (only triggers in ~/.claude)
- 2026-04-10: model-quality — switched to 200k model + effort high + showThinkingSummaries + lowered /clear threshold to 40% (response to anthropics/claude-code#42796 analysis)
- 2026-04-10: session-management — added cross-session todo writeback rule + PostToolUse hook to detect commits and remind about writeback
- 2026-04-10: worktree-memory — added worktree memory strategy (parent resolution, read/write to parent, anti-trampling, subagent injection)
- 2026-04-10: context-compact — PreCompact hook upgrade (marker + log), mandatory 40%/60% thresholds, Post-Compact Recovery rule
