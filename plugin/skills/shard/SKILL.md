---
name: shard
description: Splits large documents (PRD, architecture) into atomic shards for context management. Reduces token usage by 90%. Use when PRD or architecture exceeds 3000 tokens.
allowed-tools: Read, Write, Grep, Glob, Bash
metadata:
  context: same
  agent: general-purpose
---

# Circle Document Sharding

Implements Circle context sharding: splits large documents into atomic task files that roles can load individually, dramatically reducing token usage.

## Why Sharding Matters

When the Implementer works on TASK-001, it doesn't need to load the entire PRD. Sharding splits documents into focused atomic files so each role invocation loads only what's relevant.

- **Without sharding**: The Implementer loads full PRD (~5000+ tokens) every time
- **With sharding**: The Implementer loads one shard (~200-400 tokens) per task
- **Result**: ~90% token reduction per role invocation

## Input

Automatically detect documents to shard in `~/.claude/circle/projects/{project}/output/`:

| Source | Path |
|---|---|
| PRD | `prioritize/PRD-*.md` |
| Architecture | `arch/architecture.md` |
| Requirements | `scope/requirements.md` |

If no documents found: "No documents to shard. Run `/circle:prioritize` or `/circle:scope` first."

## Process

1. **Derive project paths**:
   ```bash
   PROJECT_NAME=$(basename "$PWD" | tr '[:upper:]' '[:lower:]')
   BASE=~/.claude/circle/projects/$PROJECT_NAME
   mkdir -p $BASE/shards/{requirements,architecture,tasks}
   ```

2. **Analyze documents**: Identify independent sections
   - Functional requirements → individual requirement shards
   - Work items / initiatives → individual task shards
   - Architecture decisions → individual ADR shards
   - Non-functional requirements → grouped shard

3. **Create a shard for each section**:

   **Requirements shards** → `$BASE/shards/requirements/`
   ```markdown
   # FR-1.1: User Authentication

   **Type**: Requirement
   **Priority**: High
   **Dependencies**: [ADR-001]

   ## Description
   [extracted section content]

   ## Acceptance Criteria
   - [ ] Criterion 1
   - [ ] Criterion 2

   ## References
   - [ADR-001] Authentication architecture decision
   ```

   **Architecture shards** → `$BASE/shards/architecture/`
   ```markdown
   # ADR-001: Authentication Strategy

   **Type**: Architecture Decision
   **Status**: Proposed
   **Related Requirements**: [FR-1.1, FR-1.2]

   ## Context
   [extracted from architecture document]

   ## Decision
   [the decision made]

   ## Consequences
   [impact on the system]
   ```

   **Task shards** → `$BASE/shards/tasks/`
   ```markdown
   # TASK-001: Implement User Login

   **Type**: Work Item
   **Priority**: Must Have
   **Dependencies**: [ADR-001, FR-1.1]

   ## Description
   Enable users to log in with credentials to access their dashboard.

   ## Acceptance Criteria
   - [ ] User can enter email and password
   - [ ] Invalid credentials show error message
   - [ ] Successful login navigates to dashboard

   ## Technical Notes
   - Uses AuthenticationManager from ADR-001
   - Follow MVVM pattern from architecture

   ## Related Shards
   - [FR-1.1] User Authentication requirement
   - [ADR-001] Authentication architecture
   ```

4. **Naming convention**: `{TYPE}-{ID}-{slug}.md`
   - `FR-1.1-user-authentication.md`
   - `ADR-001-auth-strategy.md`
   - `TASK-001-implement-user-login.md`

5. **Update session state**:
   ```json
   {
     "sharding": {
       "enabled": true,
       "shards_count": 15,
       "last_shard_date": "{ISO-8601}",
       "sources": ["prioritize/PRD.md", "arch/architecture.md"]
     }
   }
   ```

6. **Display summary**:
   ```
   Sharding Complete
   =================
   Requirements: 8 shards → ~/.claude/circle/projects/{project}/shards/requirements/
   Architecture: 4 shards → ~/.claude/circle/projects/{project}/shards/architecture/
   Tasks:        6 shards → ~/.claude/circle/projects/{project}/shards/tasks/

   Usage:
   /circle:impl TASK-001    ← Implements only TASK-001
   /circle:impl TASK-002    ← Implements only TASK-002

   Each invocation loads only the relevant shard (~300 tokens instead of ~5000).
   ```

## Post-Sharding Usage

```bash
# The Implementer works on only TASK-001
/circle:impl TASK-001

# It will read ONLY:
# - ~/.claude/circle/projects/{project}/shards/tasks/TASK-001.md
# - Any dependencies referenced in the shard (loaded on demand)
# - NOT: other tasks, full PRD, future work items
```

## Circle Principles
- Progressive disclosure: show only what's needed for the current task
- Token efficiency: every token saved is context preserved for reasoning
- Atomic units: each shard is self-contained with declared dependencies
- Traceability: shards reference their source documents and related shards
