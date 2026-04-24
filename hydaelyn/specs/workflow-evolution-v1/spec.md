---
ticket: workflow-evolution-v1
module: .claude (hydaelyn system)
created: 2026-04-22
status: planning
intake: .mirror/intake.md
---

# Workflow Evolution — Self-Evolving Workflow + Personal Brand (Side Effect)

## Context

A hackathon rejection triggered the reflection: before the next application, the GitHub / LinkedIn / Twitter trio needs to be presentable enough to show reviewers concrete work experience and technical foresight. Mirror intake expanded the scope from "tweet drafts at the output layer" to "the upstream workflow self-evolution mechanism" — the trunk is letting `.claude` periodically benchmark itself against external academic / industry progress, and personal brand is the operational side effect. See `.mirror/intake.md` for details.

## Current Behavior

- No periodic external audit mechanism — paper / library investigation is sporadic, dependent on accidental encounters
- `/reflect` (inward behavior review), `/evolve` (rule evolution) already exist, but the "outward comparison review" angle is missing
- Personal brand has no systematic flow; GitHub / Twitter / LinkedIn are largely empty
- Workflow optimization opportunities (e.g., back-translation for translation scoring) are not actively detected and tested
- Standalone tool extraction (promoting a rule / skill into its own repo) has neither criteria nor flow

## Target Behavior

Three-tier architecture in operation:

**Upstream: `/scout` skill once per week**
- Outward technical scan against `.claude` skills/rules/agents/current memory pain points
- Use researcher subagent to find "verified" emerging libraries or papers in academia / industry
- Output candidate list "behavior A could be optimized via strategy X, source Y, verification Z"
- User picks candidates worth further experimentation

**Midstream: live experimentation + practitioner notes**
- Selected candidates get tested in real workflow (could be AI tools, could be `.claude` itself)
- Results accumulate as raw material, written to `hydaelyn/experiments/{topic}/` or project memory

**Downstream: output layer**
- Standalone extraction: rules / skills meeting "≥100 lines + stranger-usable + has standalone name" threshold get promoted to standalone GitHub repos (named `rainoff/claude-xxx`); the `hydaelyn` README maintains the index
- `/draft-tweet` skill: scans memory + session + rules changelog + scout output + experiments material, produces bilingual Twitter drafts; LinkedIn syncs to Twitter content
- Cadence: 1-2 posts biweekly to start, stability-first

## Files to Change

**New**
- `skills/scout/SKILL.md` — periodic external audit skill
- `skills/draft-tweet/SKILL.md` — bilingual tweet draft generator
- `agents/researcher.md` — external technical investigation subagent
- `rules/workflow-evolution.md` — three-tier flow rules (scout → experiment → extraction → publish)
- `hydaelyn/scout/` directory — scout artifact storage
- `hydaelyn/experiments/` directory — experiment artifact storage

**Modified**
- `rules/subagent-strategy.md` — add researcher row + model recommendation to agent list
- `rules/public-repo-sync.md` — add "tweet material" sanitization category; sync scope decisions for `hydaelyn/scout/` and `hydaelyn/experiments/`
- `rules/CHANGELOG.md` — log workflow-evolution entry
- `projects/PROJ/memory/MEMORY.md` — add workflow-evolution pointer (private only)
- `projects/PROJ/memory/pattern-constitution.md` — established during planning phase (private only)

**External (M3+)**
- `rainoff/hydaelyn` README — standalone tools index section
- Future: `rainoff/claude-xxx` standalone repos (when threshold is met)

## Pattern Compliance

Based on `pattern-constitution.md` (workflow-evolution-v1 first established).

### Areas this spec touches

| Area | Mainstream practice | Count | This implementation |
|------|---------|------|---------|
| Skill Structure | Positioning declaration + H2 sections (trigger / flow / artifact / Gotchas / related rule) | 6 | scout, draft-tweet follow |
| Agent Frontmatter | `name` / `description` (multi-line) / `model` / `tools` | 6 | researcher follows |
| Agent Structure | Positioning declaration → Input → Processing → Output → Rules | 6 | researcher follows |
| Agent Verdict Severity | 🔴 / 🟡 two-tier | 6 | researcher "candidate list" does not use verdict; finding severity convention preserved |
| Rule Structure | frontmatter → activation conditions → spec → Examples → Gotchas | many | workflow-evolution.md follows |
| CHANGELOG entry | one line per rule modification | >20 | this ticket adds entry |
| Artifact directory naming | date-based + `---` separator for same-day multiple | 2 | scout output `{date}.md` follows |

### Lint Assertions

```bash
# L1. New skill has SKILL.md
test -f ~/.claude/skills/scout/SKILL.md
test -f ~/.claude/skills/draft-tweet/SKILL.md

# L2. researcher agent frontmatter is complete
head -10 ~/.claude/agents/researcher.md | grep -cE '^(name|description|model|tools):'
# Expected: >= 4

# L3. scout SKILL.md has key sections
grep -cE '^## (When to trigger|Flow|Artifact|Gotchas|Related rule)' ~/.claude/skills/scout/SKILL.md
# Expected: >= 4 (section names may vary slightly)

# L4. workflow-evolution rule logged in CHANGELOG
grep -c 'workflow-evolution' ~/.claude/rules/CHANGELOG.md
# Expected: >= 1

# L5. subagent-strategy includes researcher
grep -c 'researcher' ~/.claude/rules/subagent-strategy.md
# Expected: >= 1

# L6. scout artifact structure matches convention (after M1 Step 5)
ls ~/.claude/hydaelyn/scout/ 2>/dev/null | grep -E '^\d{4}-\d{2}-\d{2}\.md$'
# Expected: at least one date-formatted file
```

### Deviations

None.

## Acceptance Criteria

### Milestone 1 — scout skill MVP + first experiment case (stability-first)

- [ ] **AC-M1-1**: `skills/scout/SKILL.md` exists, structure matches Pattern Compliance
- [ ] **AC-M1-2**: scout skill is manually invokable (`/scout` or `/scout {focus-area}`), produces candidate-list artifact at `hydaelyn/scout/{date}.md`, each candidate has "expected confidence" (high/medium/low) field, sorted with **medium** prioritized (high = doing it produces no signal, low = often wasted time)
- [ ] **AC-M1-3**: `agents/researcher.md` exists, frontmatter complete, can be invoked by scout for external research (WebSearch / WebFetch)
- [ ] **AC-M1-4**: `rules/workflow-evolution.md` exists, describes complete three-tier path + scout periodic trigger rules
- [ ] **AC-M1-5**: `rules/subagent-strategy.md` adds researcher row (model recommendation + trigger timing)
- [ ] **AC-M1-6**: First experiment case (translation scoring + back-translation) runs full closed loop scout → candidate confirm → experiment → **publishable insight**, **positive or negative outcome both count as success** (see "Experiment Outcome Strategy"), artifact produced at `hydaelyn/experiments/translation-back-translation/`
- [ ] **AC-M1-7**: Pattern Constitution established (completed during planning)
- [ ] **AC-M1-8**: `rules/CHANGELOG.md` logs workflow-evolution entry

### Milestone 2 — draft-tweet skill (output layer)

- [ ] **AC-M2-1**: `skills/draft-tweet/SKILL.md` exists, structure matches Pattern Compliance
- [ ] **AC-M2-2**: scan scope covers project memory + session notes + rules changelog + scout artifacts + experiments artifacts
- [ ] **AC-M2-3**: supports four material pattern templates (work experience / work-product extraction / emerging investigation + practice / convergence), **each with positive/negative outcome variants** (negative outcomes have an independent tone discipline to avoid the "I learned X" inspirational genre)
- [ ] **AC-M2-4**: produces bilingual (Chinese / English) drafts, sanitization rules follow `public-repo-sync.md`, each language written independently rather than sentence-by-sentence translation; **failure cases differentiated in bilingual framing** (English can directly say "Why X didn't work", Chinese softens to "actual experiment shows X only works under condition Y" — same fact, tone adjusted to culture)
- [ ] **AC-M2-5**: produces at least 2 drafts (could come from M1 experiment case), user reviews and confirms usable

### Milestone 3 — GitHub standalone-extraction flow (packaging layer)

- [ ] **AC-M3-1**: `rules/workflow-evolution.md` adds "standalone-extraction threshold" section (≥100 lines + stranger-usable + has standalone name)
- [ ] **AC-M3-2**: standalone repo naming convention (`rainoff/claude-xxx`) documented
- [ ] **AC-M3-3**: `hydaelyn` README adds "Independent Tools" index section
- [ ] **AC-M3-4**: first actual extraction example (pick from existing rule / skill meeting threshold) completes repo creation + README + license + Twitter announcement

### Milestone 4 (publishing closed loop, long-tail execution)

- [ ] **AC-M4-1**: first actual post published on Twitter (Chinese / English bilingual)
- [ ] **AC-M4-2**: LinkedIn synced
- [ ] **AC-M4-3**: maintains 1-2 posts biweekly cadence for 4 consecutive weeks

## AC-Test Mapping

| AC | Test method |
|----|---------|
| AC-M1-1 | L1 + L3 + manual review structure |
| AC-M1-2 | manual `/scout` invocation, verify artifact output (L6) |
| AC-M1-3 | L2 + manual review agent content |
| AC-M1-4 | `test -f rules/workflow-evolution.md` + manual review structure |
| AC-M1-5 | L5 |
| AC-M1-6 | manual: run full closed loop, user reviews artifact (scout candidate quality + experiment insight depth) |
| AC-M1-7 | `test -f pattern-constitution.md` ✅ (completed) |
| AC-M1-8 | L4 |
| AC-M2-1 | same as AC-M1-1 |
| AC-M2-2 | manual: sampling review on scan results |
| AC-M2-3 | drafts match one of four templates (manual review) |
| AC-M2-4 | sanitization lint: grep for company URLs / accounts / JIRA keys in drafts |
| AC-M2-5 | user review + confirm publishable |
| AC-M3-* | manual check |
| AC-M4-* | actual published URL + cadence tracking |

## Testing Strategy

**Automated**:
- Lint assertions L1-L6 (bash script)
- File existence tests (`test -f`)

**Manual**:
- Review of skill / agent / rule structure
- Review of first experiment case results (scout candidate quality / experiment insight quality)
- Review of draft sanitization + readability
- Sanity check before publishing

**Note**: This is a documentation / tooling repo; testing primarily relies on structured lint + manual review, no unit test framework.

**Stability validation timeline**: M1 ACs are essentially "MVP works" and cannot independently prove stability. Operational stability (scout runs weekly without breaking, researcher doesn't return empty, draft-tweet doesn't produce garbage) is only validated through M2-M4's repeated execution — particularly AC-M4-3 "4 consecutive weeks of cadence" is the real stability gate. M1 does not bear the stability proof responsibility, only proves "the closed loop exists and can run once".

## Experiment Outcome Strategy

When experiments produce conclusions like "strategy doesn't apply / made it worse / impossible to implement", **this is not failure, it is first-class material** — the rarity of failure cases makes the reviewer signal even stronger (honest validation + critical thinking > bandwagon hyping).

### Handling principles (combine option 1 + option 4)

**1. Closed-loop definition** — AC-M1-6 and other "experiment ACs" succeed by "running through scout → candidate confirm → experiment → publishable insight", **independent of whether the strategy holds**. Conclusions can be:
- ✅ Positive: strategy is effective under condition Y, data as follows
- ❌ Negative: strategy does not apply under condition Y, because Z
- ⚠️ Partial: strategy effective in sub-context A, ineffective in sub-context B
- 🚫 Aborted: implementation cost too high / technical limit, test not completed but decision is recorded

All types produce artifacts; all types can enter the draft-tweet material pool.

### Experiment Artifact Frontmatter Spec

Every `hydaelyn/experiments/{topic}/` directory's main artifact (e.g., `result.md`) must have frontmatter:

```yaml
---
experiment: {topic-slug}
outcome: positive | negative | partial | aborted
confidence_ex_ante: high | medium | low   # pre-test expected confidence (matches scout candidate field)
date_started: YYYY-MM-DD
date_concluded: YYYY-MM-DD
publishable: true | false                 # whether it meets publishable bar (independent of outcome)
---
```

draft-tweet skill filters material by `publishable: true`, picks template by `outcome` (positive / negative / partial / aborted). **outcome and publishable are two independent dimensions** — negative outcomes can absolutely be publishable, aborted outcomes can also be publishable ("why not continue" has its own value).

**2. Internal work vs external publishable separation** — two standards independently evaluated:
- Optimizations found by scout count if "useful for the workflow" (may only change `.claude` internally, not public)
- Only those meeting "publishable" bar (with viewpoint / data / comparison / thinking trace) enter draft-tweet
- Avoids forcing signal for the sake of posting, also avoids skipping useful internal improvements because "can't be published"

**3. 2-3 hr blocked → mini-pivot** (safety net) — if a single experiment makes zero progress in 2-3 hr, do not push through. Write "attempt + blocker + why paused" artifact (this is itself publishable material) and return to scout for another candidate.

### Tone Discipline for Negative Results

draft-tweet must avoid three anti-patterns when handling failure cases:

- ❌ **Inspirational platitudes**: "the most important lesson learned is to stay open-minded" — empty
- ❌ **Complaint mode**: "the paper said it would work, but it didn't" — unprofessional
- ❌ **Excessive humility**: "I just didn't understand it deeply enough" — evades technical fact

✅ **Correct framing** (5 sections):
1. **Hypothesis** — originally expected strategy X to produce effect Z under condition Y
2. **Test method** — concretely how (sample, metrics, control)
3. **Result** — data or observation (no cherry-picking)
4. **Why it didn't work / why not as expected** — technical explanation, do not retreat to "our context is special"
5. **Alternative direction** — what signal this round points to (could be a different strategy, could be redefining the problem)

## Repo Extraction Candidate Pool (M3)

Candidate pool for the first standalone repo in M3 (counters Known Pitfalls #3, avoids never-starting due to overly strict threshold):

### Candidate A | `sync-public` skill (recommended first)
- **Maturity**: high (used several months, multiple times, stable)
- **Generality**: high ("private dotfiles → public repo" is a general problem, not limited to `.claude`)
- **Lines**: SKILL.md >300 lines ✅
- **Stranger usability**: high (concept self-evident, with config file replacing `.claude`-specific logic after abstraction)
- **Extraction challenge**: needs to decouple Hydaelyn-specific tone-filter logic into a pluggable form

### Candidate B | `hyd-mirror` skill
- **Maturity**: medium (established 2026-04-17, several actual uses + onboarded mechanism validated)
- **Generality**: medium-high ("AI collaboration cold-start reflection" is a general concept, but implementation binds to Trigger 1/2)
- **Lines**: SKILL.md >200 lines ✅
- **Stranger usability**: medium (requires understanding bidirectional calibration concept first)
- **Unique value**: Hydaelyn brand flagship, can serve as system narrative anchor
- **Extraction challenge**: depends on `sdd.md` + `session-management.md` Triggers; replacement mechanism needed when extracted

### Candidate C | Pattern Discovery + Constitution flow
- **Maturity**: low (workflow-evolution-v1 is the first run)
- **Generality**: extremely high ("auto-generate lint rules from code" is a hot question)
- **Lines**: should reach threshold after restructuring
- **Stranger usability**: high (has practical value proposition)
- **Extraction challenge**: currently scattered in `plan` skill Step 4.5, needs extraction refactor first

### Recommended order

A (sync-public) > B (hyd-mirror) > C (Pattern Discovery)

Decision timing: pick A or B as the first actual extraction example after M3 Step 1 completes "standalone-extraction threshold" definition (AC-M3-4).

## Known Pitfalls

1. **Scout output quality determines the entire system's value** — if researcher returns only shallow summaries with no verification signal, the downstream material pool will be empty. The researcher prompt must emphasize "only list with replication / industry practice", and confidence tags in the verdict
2. **Experiment cost may be underestimated** — "test back-translation in AI tools" sounds simple, may actually take several hours. Milestone time estimates should be conservative; allow M1 Step 5 to span weeks
3. **Standalone repo extraction threshold too strict → never happens** — ≥100 lines + stranger-usable, if too strict, may yield zero in 6 months. Backup candidate pool in "Repo Extraction Candidate Pool (M3)" section, first pick `sync-public` skill
4. **Tweet draft reviewer signal insufficient** — draft-tweet may produce "today I learned X" stream-of-consciousness. Skill must explicitly reject pointless mode, only produce drafts with "viewpoint / data / comparison"
5. **Researcher agent's WebFetch / WebSearch cost** — weekly scout × multiple focus areas can burn quota. Design quota-aware scan scope (limit 1-2 focus areas per week)
6. **Writing-style six anti-patterns easily violated in bilingual drafts** — English version often becomes stiff translation. draft-tweet must enforce "write each language independently, do not translate sentence-by-sentence"; both versions must pass `rules/writing-style.md` check
7. **Personal brand mindset drift** — original "workflow side effect" can mutate into "things done for the sake of posting". Need checkpoint reminders for trunk priority, e.g., scout asks first "does this investigation have value to the work itself" before "could this become a tweet"
8. **Boundaries between scout and `/housekeeping`, `/reflect`, `/evolve` blurry** — periodic skills may overlap. `rules/workflow-evolution.md` must explicitly draw boundaries:
   - `/reflect` = inward review of own behavior patterns
   - `/evolve` = rule evolution (from reflect output)
   - `/scout` = outward review against external technology
   - `/housekeeping` = memory cleanup and archiving

## Implementation Plan

Detailed planning only for **Milestone 1** (stability-first principle). M2/M3/M4 keep high-level; re-`/plan` after M1 validates the closed loop.

### M1 Step 1: `agents/researcher.md`
- **AC**: AC-M1-3
- **Scope**: `agents/researcher.md`
- **Test**: L2 + manual review Input/Output format follows agent pattern
- **Dependencies**: none
- **Estimate**: 60 min
- **Key design**: Input is "focus area + context snippet"; Output is "candidate list, each with source / verification evidence / actionable recommendation for current workflow"; Tools: WebSearch + WebFetch + Read

### M1 Step 2: `rules/workflow-evolution.md`
- **AC**: AC-M1-4
- **Scope**: `rules/workflow-evolution.md`
- **Test**: frontmatter + activation conditions + three-tier flow + scout periodic rules + Examples + Gotchas complete
- **Dependencies**: none (can run in parallel with Step 1)
- **Estimate**: 60 min
- **Key content**: scout trigger timing (weekly + manual), experiment start criteria, standalone-extraction threshold (initial, hardened in M3), boundaries with existing reflect/evolve/housekeeping

### M1 Step 3: `rules/subagent-strategy.md` update
- **AC**: AC-M1-5
- **Scope**: `rules/subagent-strategy.md` (add researcher row)
- **Test**: L5
- **Dependencies**: Step 1 (need researcher positioning and model first)
- **Estimate**: 15 min

### M1 Step 4: `skills/scout/SKILL.md`
- **AC**: AC-M1-1, AC-M1-2
- **Scope**: `skills/scout/SKILL.md` + `hydaelyn/scout/` directory spec
- **Test**: L1 + L3 + manual `/scout` invocation produces artifact
- **Dependencies**: Step 1 (researcher) + Step 2 (rules)
- **Estimate**: 90 min
- **Key design**: manual vs periodic trigger, focus area passing mechanism, scan scope (skills/rules/agents/memory pain points), artifact format (each candidate has expected confidence field), collaboration flow with researcher agent

### M1 Step 5: First experiment case (translation scoring + back-translation)
- **AC**: AC-M1-6
- **Scope**: `hydaelyn/experiments/translation-back-translation/` (includes scout output / experiment design / results / insights)
- **Test**: complete artifact + user review pass
- **Dependencies**: Step 4
- **Estimate**: 3-4 hours (including actual testing in AI tools, may span sessions)

### M1 Step 6: `rules/CHANGELOG.md` log + MEMORY.md update
- **AC**: AC-M1-8
- **Scope**: `rules/CHANGELOG.md` + project memory
- **Test**: L4 + grep workflow-evolution in MEMORY.md
- **Dependencies**: Step 5 complete (log only after entire M1 closed loop validated, avoid half-done)
- **Estimate**: 10 min

### Cut quality check

- ✅ Each step ≤ 2 ACs (Step 4 binds AC-M1-1 + AC-M1-2)
- ✅ Each step independently testable
- ✅ Each step in single module
- ✅ No circular dependencies (Step 1,2 → 3,4 → 5 → 6)
- ✅ All M1 ACs covered (M1-1 to M1-8, where M1-7 completed during planning)
- ⬜ UI ticket: not applicable
- ⬜ Figma: not applicable
- ⬜ system spec: not applicable (new module)
- ✅ Pattern Compliance: filled + user pending confirmation
- ✅ Spec-Test Debate: AC-Test mapping complete

## Post-Hoc Insights

<!-- reserved append section -->
