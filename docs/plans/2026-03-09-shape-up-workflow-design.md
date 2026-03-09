# Design: Shape Up Workflow for BMAD

**Date:** 2026-03-09
**Status:** Approved
**Approach:** A — Rename & Rewrite (clean break from Scrum)

## Problem

BMAD's workflow skills (`bmad-sprint`, `bmad-facilitate`) use Scrum concepts (sprints, story points, velocity, backlog) that don't match how the team actually works. Luscii uses Shape Up with Linear cycles and appetite-based sizing (cappuccino/sandwich/hutspot). The plugin should reflect the real workflow.

## Appetite Scale (from Luscii)

| Appetite | Meaning |
|----------|---------|
| ☕ **Cappuccino** | One person, within 2 weeks |
| 🥪 **Sandwich** | A few people, within one cycle (4 weeks) |
| 🍲 **Hutspot** | Many people, more than one cycle |

## Key Shape Up Concepts Adopted

- **Cycles** (4 weeks) instead of sprints (2 weeks)
- **Appetite** (how much time we want to spend) instead of estimates (how long it takes)
- **Pitches** instead of sprint backlogs — shaped work with problem, appetite, solution sketch, rabbit holes, no-gos
- **No permanent backlog** — good ideas resurface naturally
- **Team autonomy** — team owns scope and delivery within appetite
- **No cooldown period** — replaced by Luscii's quality standard process
- **Betting table = the dev** — in BMAD/solo-dev context, the developer decides what to bet on

## Changes

### 1. New: `bmad-cycle` (replaces `bmad-sprint`)

Interactive ceremony for planning a cycle. 4 steps (down from 6 in sprint):

```
shaping_review → appetite_sizing → cycle_commitment → quality_notes
```

**Step 1: Shaping Review**
- Reads pitches from `prioritize/` and requirements from `scope/`
- Lists candidate ideas — shaped (ready) vs raw (needs work)
- No permanent backlog

**Step 2: Appetite Sizing**
Per idea:
```
IDEA-001: Mobile MCP integration
  Appetite? [cappuccino / sandwich / hutspot]
  → sandwich

  Rabbit holes?
  → Device farm availability

  No-gos?
  → No Android this cycle
```

**Step 3: Cycle Commitment**
Summary of selected ideas with appetites. Sanity check that the cycle isn't overloaded (e.g., max ~1 sandwich + a few cappuccinos for 4 weeks). User confirms.

**Step 4: Quality Notes** (replaces cooldown)
Space for: known bugs, tech debt, spikes for next cycle. Integrates with quality standard process.

**Output:** `facilitate/cycle-plan-{date}.md`
**Linear MCP:** Interactive — "Want me to help create a Linear cycle with these bets?"

### 2. Rewrite: `bmad-facilitate`

From sprint facilitator to cycle facilitator.

**Changes:**
- "Sprint" → "Cycle" everywhere
- Story points → appetite (cappuccino/sandwich/hutspot)
- Velocity tracking → removed
- Task breakdown with assignees → removed (team self-organizes)

**Stays the same:**
- Lightweight role (haiku model)
- Reads PRD/pitch and architecture as input
- Produces a cycle plan
- Linear MCP section (interactive)
- Push back on overcommitment

**Output template:**
```markdown
# Cycle Plan: {date}

## Cycle Goal
{One clear sentence}

## Duration
4 weeks ({start} → {end})

## Bets
| ID | Pitch | Appetite | Owner | Rabbit Holes |
|----|-------|----------|-------|-------------|
| BET-001 | ... | 🥪 sandwich | team | ... |
| BET-002 | ... | ☕ cappuccino | dev | ... |

## No-Gos
- {explicitly excluded}

## Quality Notes
- {bugs, tech debt, spikes for next cycle}
```

**Difference with `bmad-cycle`:** facilitate produces a quick plan from an existing pitch. `bmad-cycle` is the full interactive ceremony (4 steps with shaping review and sizing).

### 3. Modify: `bmad-prioritize` (template change)

Role stays the same. Output evolves:

- "Release Plan" → removed (the cycle decides what gets in)
- Story points → appetite
- MoSCoW priorities → stay (orthogonal to Shape Up)
- Added "Pitch" section per prioritized feature

**Output structure (pitch-style PRD):**
```markdown
# PRD: {Name}

## Vision
{unchanged}

## Goals & Success Metrics
{unchanged}

## User Stories
{unchanged, but without story points}

## Prioritization
| Feature | Priority | Appetite | Value | Dependency |
|---------|----------|----------|-------|------------|
| FR-1 | Must | ☕ cappuccino | High | None |
| FR-2 | Must | 🥪 sandwich | High | FR-1 |

## Pitches

### Pitch: FR-1 — {Name}
- **Problem:** {what it solves}
- **Appetite:** ☕ cappuccino
- **Solution sketch:** {high-level, not wireframes}
- **Rabbit holes:** {known risks}
- **No-gos:** {out of scope}

## Dependencies & Risks
{unchanged}
```

### 4. Modify: `bmad-greenfield`

- Optional step "Facilitator (Sprint Planning)" → "Cycle Planning"
- `/bmad:bmad-sprint` → `/bmad:bmad-cycle`
- Output path `facilitate/sprint-plan-*.md` → `facilitate/cycle-plan-*.md`
- Completion checklist wording

### 5. Modify: `plugin/commands/bmad.md`

- `bmad-sprint` listing → `bmad-cycle` with new description
- `bmad-facilitate` description updated

### 6. Modify: `README.md`

- Sprint → Cycle references (6 occurrences)
- `bmad-sprint` → `bmad-cycle` in skills table
- Linear description: "sprint management" → "cycle management"

### 7. Cascade: Linear MCP wording

Minimal wording changes across impacted roles (facilitate, cycle, prioritize, greenfield):
- "Create sprint/cycle" → "Create cycle"
- "sprint velocities" → removed
- "assign stories as issues" → "assign bets as issues"

## Files Summary

| File | Action |
|------|--------|
| `plugin/skills/bmad-cycle/SKILL.md` | **Create** (replaces bmad-sprint) |
| `plugin/skills/bmad-sprint/` | **Delete** |
| `plugin/skills/bmad-facilitate/SKILL.md` | **Rewrite** |
| `plugin/skills/bmad-prioritize/SKILL.md` | **Modify** template |
| `plugin/skills/bmad-greenfield/SKILL.md` | **Modify** references |
| `plugin/commands/bmad.md` | **Modify** listing |
| `plugin/.claude-plugin/plugin.json` | **Bump** to v0.11.0 |
| `docs/CHANGELOG.md` | **Add** v0.11.0 entry |
| `README.md` | **Modify** sprint → cycle wording |
| `CLAUDE.md` | **No change** (already clean) |

## Not Changed

scope, arch, security, impl, qa, ux, code-review, triage, docs, shard, validate-prd, init, soul.md, deps-manifest.yaml
