# Shape Up Workflow Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Replace Scrum-based sprint planning with Shape Up methodology in the BMAD plugin.

**Architecture:** Pure Markdown skill files — no build, no tests, no CI. All changes are in `.md` and `.json` files within `plugin/`. Version bump to 0.11.0.

**Tech Stack:** Markdown, JSON (plugin.json)

**Design doc:** `docs/plans/2026-03-09-shape-up-workflow-design.md`

---

### Task 1: Create `bmad-cycle` skill

**Files:**
- Create: `plugin/skills/bmad-cycle/SKILL.md`

**Reference:** Read `plugin/skills/bmad-sprint/SKILL.md` for structure, then rewrite with Shape Up concepts.

**Step 1: Create the directory**

```bash
mkdir -p plugin/skills/bmad-cycle
```

**Step 2: Write `SKILL.md`**

Create `plugin/skills/bmad-cycle/SKILL.md` with:

Frontmatter:
```yaml
---
name: bmad-cycle
description: Interactive cycle planning ceremony. 4-step Shape Up process from shaping review to cycle commitment. Appetite-based sizing (cappuccino/sandwich/hutspot). Resumable.
allowed-tools: Read, Write, Grep, Glob, Bash
metadata:
  context: same
  agent: general-purpose
---
```

Body structure (rewrite from bmad-sprint):
- Title: `# BMAD Cycle Planning Ceremony`
- Role intro: "You are the **Cycle Planning Orchestrator** of the BMAD circle. You facilitate an interactive cycle planning ceremony using Shape Up methodology."
- Soul: same pattern as bmad-sprint (`${CLAUDE_PLUGIN_ROOT}/resources/soul.md`)
- Key reminders: "Appetite over estimates. No permanent backlog. Team autonomy."
- Ceremony Structure: 4 steps instead of 6:
  ```
  shaping_review → appetite_sizing → cycle_commitment → quality_notes
  ```
- Commands: `/bmad:bmad-cycle`, `/bmad:bmad-cycle resume`, `/bmad:bmad-cycle status`
- Prerequisites: reads from `prioritize/PRD-*.md` and `scope/requirements*.md`. No velocity reference.
  - If no PRD found: "No PRD or pitch found. Run `/bmad:bmad-prioritize` to create one, or describe your ideas directly."
- State Management: session-state.json with `"type": "cycle"` and ceremony_data:
  ```json
  {
    "workflow": {
      "type": "cycle",
      "current_step": "appetite_sizing",
      "completed_steps": ["shaping_review"],
      "ceremony_data": {
        "ideas": [
          { "id": "IDEA-001", "title": "...", "shaped": true, "priority": "Must" }
        ],
        "bets": [],
        "cycle_goal": null,
        "committed": false
      }
    }
  }
  ```

**Step 1/4 — Shaping Review:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Step 1/4: Shaping Review
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
1. Read PRD/pitches and extract features
2. If shards exist in stories/, list them
3. Display ideas with shaped status:
   ```
   Candidate Ideas:
     IDEA-001 [Must]  [shaped]  Mobile MCP integration
     IDEA-002 [Should] [raw]    Android knowledge pack
     IDEA-003 [Could]  [shaped] Guided onboarding

   Add ideas? Type an idea or 'done' to proceed.
   ```
4. User can add ideas. Each gets IDEA-{NNN} ID.
5. On `done`, save to ceremony_data and advance.

**Step 2/4 — Appetite Sizing:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Step 2/4: Appetite Sizing
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
For each idea from shaping review, ask interactively:
```
IDEA-001: Mobile MCP integration [Must] [shaped]

  Appetite? [cappuccino / sandwich / hutspot]
  → (user picks)

  Rabbit holes? (known risks that could derail — or 'none')
  → (user answers)

  No-gos? (explicitly out of scope — or 'none')
  → (user answers)
```

Appetite reference (show once at start of step):
```
Appetite Guide:
  ☕ Cappuccino  — One person, within 2 weeks
  🥪 Sandwich    — A few people, within one cycle (4 weeks)
  🍲 Hutspot     — Many people, more than one cycle
```

After all ideas sized, display summary:
```
Sized Ideas:
  IDEA-001 🥪 sandwich   Mobile MCP integration    [2 rabbit holes]
  IDEA-002 ☕ cappuccino  Android knowledge pack     [no risks]
  IDEA-003 🍲 hutspot     Guided onboarding          [1 rabbit hole]

Select which ideas to bet on: type IDEA-ID to toggle, or 'done'.
```

Overload warning: if more than 1 sandwich + 2 cappuccinos selected (rough heuristic for a 4-week cycle), warn:
```
This cycle looks overloaded. A healthy 4-week cycle typically fits
~1 sandwich + a few cappuccinos. Proceed anyway? [y/n]
```

**Step 3/4 — Cycle Commitment:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Step 3/4: Cycle Commitment
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
1. Based on selected bets, propose a cycle goal (one sentence)
2. Show final summary:
   ```
   Cycle Goal: "{goal}"
   Duration: 4 weeks

   Bets:
     ✓ BET-001 🥪 sandwich   Mobile MCP integration
     ✓ BET-002 ☕ cappuccino  Android knowledge pack

   No-Gos:
     - No Android for MCP integration
     - No Figma comparison mode

   Commit to this cycle? [y/n/edit]
   ```
3. If `y`: save and advance. If `n`: back to sizing. If `edit`: modify goal.

**Step 4/4 — Quality Notes:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Step 4/4: Quality Notes
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
Optional — ask:
```
Any quality items for this cycle? (bugs, tech debt, spikes for next cycle)
Type items line by line, or 'done' to finish.
```

Save cycle plan to `facilitate/cycle-plan-{date}.md` using the template from the design doc.
Display completion:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Cycle Planning — COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Cycle plan saved to:
~/.claude/bmad/projects/{project}/output/facilitate/cycle-plan-{date}.md

Start implementation: /bmad:bmad-impl BET-001
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Navigation commands: same as bmad-sprint (done, back, pause, exit, status).

MCP Integration:
- **Linear**: "Want me to help create a Linear cycle with these bets?" (interactive, never automatic)
- **claude-mem**: Search for past cycle plans. Save cycle commitment at completion.

BMAD Principles:
- Appetite over estimates: time is fixed, scope is variable
- No permanent backlog: ideas that matter resurface
- Team autonomy: the team owns scope and delivery within appetite
- Transparency: show appetite totals, make trade-offs visible

**Step 3: Verify the file**

```bash
head -10 plugin/skills/bmad-cycle/SKILL.md
```

Expected: frontmatter with `name: bmad-cycle`

**Step 4: Commit**

```bash
git add plugin/skills/bmad-cycle/SKILL.md
git commit -m "feat: add bmad-cycle skill (Shape Up cycle planning)"
```

---

### Task 2: Delete `bmad-sprint` skill

**Files:**
- Delete: `plugin/skills/bmad-sprint/SKILL.md`

**Step 1: Remove the directory**

```bash
rm -rf plugin/skills/bmad-sprint
```

**Step 2: Verify it's gone**

```bash
ls plugin/skills/bmad-sprint 2>&1
```

Expected: "No such file or directory"

**Step 3: Commit**

```bash
git add -A plugin/skills/bmad-sprint
git commit -m "feat: remove bmad-sprint (replaced by bmad-cycle)"
```

---

### Task 3: Rewrite `bmad-facilitate`

**Files:**
- Modify: `plugin/skills/bmad-facilitate/SKILL.md`

**Step 1: Rewrite the file**

Replace the entire content of `plugin/skills/bmad-facilitate/SKILL.md`.

Frontmatter changes:
```yaml
---
name: bmad-facilitate
description: Facilitator — Plans cycles, coordinates team, removes blockers. Use for cycle planning, retrospectives, or workflow coordination.
allowed-tools: Read, Grep, Glob, Bash
metadata:
  context: fork
  agent: general-purpose
  model: haiku
---
```

Body rewrite — key changes from current:
- Title: `# Facilitator` (keep)
- Role intro: "You energize the **Facilitator** role in the BMAD circle. You facilitate cycle planning, coordinate work, and remove blockers." (was "agile ceremonies")
- Model rationale: "Cycle coordination is structured and lightweight" (was "Sprint coordination")
- Your Role: "You push back on overcommitment and protect the team from scope creep mid-cycle." (was "mid-sprint"). Remove "burning out the team for a deadline" — Shape Up is appetite-based, not deadline-based. Replace with: "You care about sustainable pace — overloading a cycle defeats the purpose of appetite-based planning."
- Domain Detection: keep as-is
- Input Prerequisites:
  - PRD: keep `prioritize/PRD-*.md`
  - Architecture: keep
  - Change: `Previous sprint: facilitate/sprint-plan-*.md` → `Previous cycle: facilitate/cycle-plan-*.md`
  - Missing PRD message: "PRD or pitch needed for cycle planning. Run `/bmad:bmad-prioritize` first."
- Process:
  - Step 1: keep init
  - Step 2: keep "Review available work"
  - Step 3: Replace sprint plan template entirely with cycle plan template:
    ```markdown
    # Cycle Plan: {Cycle Name}

    ## Cycle Goal
    {One clear sentence describing what this cycle delivers}

    ## Duration
    4 weeks ({start} → {end})

    ## Bets
    | ID | Pitch | Appetite | Owner | Rabbit Holes |
    |----|-------|----------|-------|-------------|
    | BET-001 | {Pitch title} | ☕/🥪/🍲 | {who} | {risks} |

    ## No-Gos
    - {Explicitly excluded from this cycle}

    ## Quality Notes
    - {Known bugs, tech debt, spikes for next cycle}

    ## Definition of Done
    - [ ] Code implemented and self-reviewed
    - [ ] Tests written and passing
    - [ ] Architecture review passed
    - [ ] QA verification passed
    ```
  - Step 4: Save to `facilitate/cycle-plan-{date}.md`
  - Step 5: MCP Integration:
    - **Linear**: Create cycle, assign bets as issues (interactive)
    - **claude-mem**: Search for past cycle plans. Save cycle commitments.
  - Step 6: Handoff:
    > **Facilitator — Complete.**
    > Cycle plan saved to: `~/.claude/bmad/projects/{project}/output/facilitate/cycle-plan-{date}.md`
    > Bets committed: {count}
    > Next: Team begins implementation with `/bmad:bmad-impl`.
- BMAD Principles:
  - "Protect the team: push back on overcommitment" (keep)
  - "Sustainable pace: a cycle means focused, not exhausted" (was "sprint")
  - "Remove blockers: identify and escalate impediments early" (keep)
  - "Transparency: make progress and risks visible to everyone" (keep)

**Step 2: Verify the rewrite**

```bash
grep -c "sprint" plugin/skills/bmad-facilitate/SKILL.md
```

Expected: 0 (no sprint references remaining)

**Step 3: Commit**

```bash
git add plugin/skills/bmad-facilitate/SKILL.md
git commit -m "feat: rewrite bmad-facilitate for Shape Up cycles"
```

---

### Task 4: Modify `bmad-prioritize` template

**Files:**
- Modify: `plugin/skills/bmad-prioritize/SKILL.md`

**Step 1: Update the PRD template**

In `plugin/skills/bmad-prioritize/SKILL.md`, find the "Generate PRD" section (around line 61-90). Replace the template:

Replace the Prioritization table:
```markdown
   ## Prioritization
   | Feature | Priority | Effort | Value |
   |---|---|---|---|
   | {Feature} | Must/Should/Could | S/M/L | High/Med/Low |
```
With:
```markdown
   ## Prioritization
   | Feature | Priority | Appetite | Value | Dependency |
   |---|---|---|---|---|
   | {Feature} | Must/Should/Could | ☕/🥪/🍲 | High/Med/Low | {deps} |
```

Replace the Release Plan section:
```markdown
   ## Release Plan
   - **v1 (MVP)**: {Must Have features}
   - **v1.1**: {Should Have features}
   - **v2**: {Could Have features}
```
With:
```markdown
   ## Pitches

   ### Pitch: {FR-ID} — {Feature Name}
   - **Problem:** {what it solves}
   - **Appetite:** ☕ cappuccino / 🥪 sandwich / 🍲 hutspot
   - **Solution sketch:** {high-level approach, not wireframes}
   - **Rabbit holes:** {known risks that could derail}
   - **No-gos:** {explicitly out of scope}
```

**Step 2: Update the MCP Integration section** (line 94-96)

Change:
```
- **Linear**: Create issues from user stories, set priorities, plan milestones. Full access to issue management.
```
To:
```
- **Linear**: Create issues from pitches, set priorities. Full access to issue management.
```

**Step 3: Update the handoff** (line 98-102)

Change:
```
> User stories: {count}, Must Have: {count}, Should Have: {count}
```
To:
```
> Pitches: {count}, Must Have: {count}, Should Have: {count}
```

**Step 4: Verify no stale references**

```bash
grep -n "story.point\|velocity\|Release Plan\|Effort" plugin/skills/bmad-prioritize/SKILL.md
```

Expected: no matches

**Step 5: Commit**

```bash
git add plugin/skills/bmad-prioritize/SKILL.md
git commit -m "feat: update bmad-prioritize template for Shape Up pitches"
```

---

### Task 5: Modify `bmad-greenfield` references

**Files:**
- Modify: `plugin/skills/bmad-greenfield/SKILL.md`

**Step 1: Update optional phase prompt** (around line 113)

Change:
```
2. Facilitator (Sprint Planning) — Include? [y/n]
```
To:
```
2. Facilitator (Cycle Planning) — Include? [y/n]
```

**Step 2: Update Role Sequence Detail table** (around line 199)

Change:
```
| 7* | **Facilitator** | haiku | Sprint planning | PRD + Architecture | `facilitate/sprint-plan.md` |
```
To:
```
| 7* | **Facilitator** | haiku | Cycle planning | PRD + Architecture | `facilitate/cycle-plan.md` |
```

**Step 3: Update Completion Phase** (around line 349-361)

Change:
```
| Sprint Plan | Facilitator | ✓/skipped | sprint-plan.md |
```
To:
```
| Cycle Plan | Facilitator | ✓/skipped | cycle-plan.md |
```

Change:
```
- [ ] Update Linear issues
```
To:
```
- [ ] Update Linear cycle
```

**Step 4: Update Active workflow display** (around line 86)

Change `Active workflow: <greenfield/sprint/none>` to `Active workflow: <greenfield/cycle/none>`

**Step 5: Verify**

```bash
grep -n "sprint" plugin/skills/bmad-greenfield/SKILL.md
```

Expected: 0 matches (no sprint references remaining)

**Step 6: Commit**

```bash
git add plugin/skills/bmad-greenfield/SKILL.md
git commit -m "feat: update bmad-greenfield references from sprint to cycle"
```

---

### Task 6: Modify dashboard (`bmad.md`)

**Files:**
- Modify: `plugin/commands/bmad.md`

**Step 1: Update role listing** (around line 43)

Change:
```
  /bmad-facilitate  — Facilitator (sprint planning)
```
To:
```
  /bmad-facilitate  — Facilitator (cycle planning)
```

**Step 2: Update workflows section** (around line 49-50)

Change:
```
  /bmad-sprint     — Sprint planning session
```
To:
```
  /bmad-cycle      — Cycle planning session (Shape Up)
```

**Step 3: Update active workflow display** (around line 86)

Change:
```
Active workflow: <greenfield/sprint/none>
```
To:
```
Active workflow: <greenfield/cycle/none>
```

**Step 4: Commit**

```bash
git add plugin/commands/bmad.md
git commit -m "feat: update bmad dashboard for Shape Up workflow"
```

---

### Task 7: Modify `README.md`

**Files:**
- Modify: `README.md`

**Step 1: Update the Circle table** (line 23)

Change:
```
| `/bmad:bmad-facilitate` | Facilitator | Plans sprints, coordinates the team |
```
To:
```
| `/bmad:bmad-facilitate` | Facilitator | Plans cycles, coordinates the team |
```

**Step 2: Update Orchestrators table** (lines 43-44)

Change:
```
| `/bmad:bmad-greenfield` | Runs the full workflow: ... → Facilitator (sprint plan) → ... |
| `/bmad:bmad-sprint` | Interactive sprint planning ceremony — 6 steps from backlog review to sprint commitment |
```
To:
```
| `/bmad:bmad-greenfield` | Runs the full workflow: ... → Facilitator (cycle plan) → ... |
| `/bmad:bmad-cycle` | Interactive cycle planning ceremony — 4-step Shape Up process from shaping review to cycle commitment |
```

**Step 3: Update Dependencies table** (line 84)

Change:
```
| Linear | Cloud MCP | Core | Issue tracking and sprint management for all roles |
```
To:
```
| Linear | Cloud MCP | Core | Issue tracking and cycle management for all roles |
```

**Step 4: Update Architecture — Zero Footprint** (line 179)

Change:
```
│   ├── facilitate/   # Sprint plans
```
To:
```
│   ├── facilitate/   # Cycle plans
```

**Step 5: Update MCP Integration table** (line 222)

Change:
```
| Linear | All roles | Issue tracking, sprint management |
```
To:
```
| Linear | All roles | Issue tracking, cycle management |
```

**Step 6: Update intro paragraph** (line 9)

Change:
```
BMAD works for everyone on the team: product managers, designers, analysts, scrum masters, developers, and documentation writers.
```
To:
```
BMAD works for everyone on the team: product managers, designers, analysts, developers, and documentation writers.
```

**Step 7: Verify**

```bash
grep -ni "sprint\|scrum" README.md
```

Expected: 0 matches

**Step 8: Commit**

```bash
git add README.md
git commit -m "docs: update README for Shape Up workflow"
```

---

### Task 8: Update `plugin.json` and `CHANGELOG.md`

**Files:**
- Modify: `plugin/.claude-plugin/plugin.json`
- Modify: `docs/CHANGELOG.md`

**Step 1: Update plugin.json version and keywords**

In `plugin/.claude-plugin/plugin.json`:
- Change `"version": "0.10.0"` → `"version": "0.11.0"`
- Change `"sprint-planning"` → `"cycle-planning"` in keywords
- Add `"shape-up"` to keywords

**Step 2: Add CHANGELOG entry**

Prepend to `docs/CHANGELOG.md` (before the v0.10.0 entry):

```markdown
## v0.11.0 — Shape Up Workflow

### Shape Up Replaces Scrum

BMAD's workflow now follows Shape Up methodology instead of Scrum. Appetite-based sizing replaces story points. Cycles replace sprints. Pitches replace backlog items.

- **New skill: `bmad-cycle`** — Interactive 4-step cycle planning ceremony (shaping review → appetite sizing → cycle commitment → quality notes). Replaces `bmad-sprint`.
- **Appetite sizing**: ☕ Cappuccino (1 person, ≤2 weeks), 🥪 Sandwich (few people, ≤1 cycle), 🍲 Hutspot (many people, >1 cycle)
- **Pitch-based PRD**: `bmad-prioritize` now generates pitches with problem, appetite, solution sketch, rabbit holes, and no-gos
- **`bmad-facilitate` rewritten**: Produces cycle plans with bets and appetite instead of sprint plans with story points
- **`bmad-greenfield` updated**: References cycle planning instead of sprint planning
- **Removed: `bmad-sprint`** — Use `bmad-cycle` instead

### Breaking Changes

- `/bmad:bmad-sprint` no longer exists — use `/bmad:bmad-cycle`
- Cycle plan output moved from `facilitate/sprint-plan-*.md` to `facilitate/cycle-plan-*.md`
- PRD template no longer includes "Release Plan" section — replaced by "Pitches" section
```

**Step 3: Commit**

```bash
git add plugin/.claude-plugin/plugin.json docs/CHANGELOG.md
git commit -m "chore: bump to v0.11.0, add Shape Up changelog"
```

---

### Task 9: Final verification

**Step 1: Search for any remaining Scrum references across plugin/**

```bash
grep -rni "sprint\|story.point\|velocity\|backlog" plugin/ --include="*.md" --include="*.json"
```

Expected: 0 matches (except possibly in templates that are domain-agnostic)

**Step 2: Search README and docs/**

```bash
grep -rni "sprint\|scrum" README.md docs/ --include="*.md"
```

Expected: 0 matches (design docs may reference "sprint" in context of "replaced sprint" — that's fine)

**Step 3: Verify file structure**

```bash
ls plugin/skills/bmad-cycle/SKILL.md plugin/skills/bmad-facilitate/SKILL.md
ls plugin/skills/bmad-sprint 2>&1  # should not exist
```

**Step 4: Verify plugin.json**

```bash
cat plugin/.claude-plugin/plugin.json | grep version
```

Expected: `"version": "0.11.0"`
