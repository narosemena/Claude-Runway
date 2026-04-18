# Intake State Management — Checkpointing & Rehydration

Handles mid-process interruption recovery. The protocol writes incremental state
to `.claude/intake-state.json` at each phase boundary so sessions that die
unexpectedly can resume from the last checkpoint rather than restarting.

---

## Core Principle

**Never rely on conversation history to persist intake progress.** Conversations
are volatile — they die on disconnect, evaporate on `/clear`, and degrade on
`/compact`. Disk state is durable. Write progress to disk at every phase
boundary so the next session can rehydrate without re-interviewing.

---

## State File Location and Structure

Write to: `.claude/intake-state.json`

```json
{
  "version": "1.0",
  "project_name": "string",
  "last_updated": "ISO-8601 timestamp",
  "current_phase": "phase_0 | phase_1 | phase_2 | phase_3 | phase_4 | phase_5 | phase_6 | complete",
  "completed_phases": ["phase_0", "phase_1"],
  "environment": "claude-code-vscode | claude-code-terminal | claude-ai-container | claude-desktop | claude-web-chat",
  "proficiency_level": "L1 | L2 | L3",
  "tier": "lightweight | standard | complex | null",

  "phase_1_results": {
    "description": "string — what the user said they're building",
    "purpose": "string — problem it solves or goal it serves",
    "audience": "personal | client | public",
    "production": true,
    "stack_languages": ["string"],
    "stack_frameworks": ["string"],
    "project_type": "frontend | backend | fullstack | cli | library | other",
    "integrations": ["string"],
    "greenfield": true
  },

  "phase_2_results": {
    "repo_name": "string",
    "package_manager": "string",
    "pinned_versions": {"node": "20", "python": "3.12"},
    "testing_framework": "string",
    "deployment_target": "string",
    "ci_cd": "string | null",
    "constraints": ["string — fixed architectural decisions"],
    "anti_patterns": ["string — explicit do-nots"],
    "definition_of_done": "string — verbatim from user",
    "risks": [
      {"type": "technical | scope | security | data | dependency", "description": "string", "mitigation": "string"}
    ],
    "terminology": {"domain_term": "code_meaning"}
  },

  "phase_3_results": {
    "claude_md_status": "not_started | drafted | reviewed | approved",
    "claudeignore_status": "not_started | written",
    "security_perimeter_status": "not_needed | written",
    "hooks_status": "not_needed | offered | accepted | written | declined"
  },

  "phase_4_results": {
    "execution_mode": "direct | plan-first | agent-teams",
    "effort_level": "low | standard | high | max",
    "mcp_servers": ["string"],
    "ecosystem_health": "clean | warnings | critical_resolved",
    "recommended_installs": ["string"],
    "user_install_decisions": {"tool_name": "approved | declined"}
  },

  "phase_5_results": {
    "plan_written": false,
    "plan_path": "string | null",
    "plan_approved": false
  },

  "interruption_context": "string — optional note about what was happening when the session ended"
}
```

### State File Rules

- **Write after every phase completion.** Not after every question — after
  each phase boundary (1.3, 2.5, 3.4, 4.5, 5.3, 6.4).
- **Update `current_phase` BEFORE starting the next phase.** If the session
  dies during Phase 3, the state file should show `current_phase: "phase_3"`
  with Phase 2 in `completed_phases`. This tells rehydration: "Phase 2 is
  done, Phase 3 was in progress."
- **Null fields are fine.** Not every field will be populated at every checkpoint.
  A Phase 1 checkpoint won't have `phase_2_results`.
- **Overwrite, don't append.** Each write replaces the entire file with current
  state. This keeps the file small and avoids corruption from partial writes.
- **The state file is ephemeral.** Delete it when the protocol completes
  successfully (Phase 6 handover done). It's not a permanent artifact — it's
  crash recovery.

---

## Writing Checkpoints

At the end of each phase, before proceeding to the next:

```bash
# Write intake state (example — the actual JSON is generated dynamically)
mkdir -p .claude
cat > .claude/intake-state.json << 'EOF'
{generated JSON with current state}
EOF
```

For non-Claude-Code environments (Claude.ai container, Claude Desktop, web chat),
where filesystem writes may not persist or may not be in the user's project:
present the state as a JSON block the user can save locally if they want
session recovery. This is opt-in for non-CLI environments.

---

## Rehydration Protocol

At the very start of the protocol — before Phase 0 — check for an existing
state file:

```bash
if [ -f ".claude/intake-state.json" ]; then
  echo "INTAKE_STATE_FOUND"
  cat .claude/intake-state.json
else
  echo "NO_INTAKE_STATE"
fi
```

### If State File Found

Read the state file. Determine the last completed phase. Present the user with
options:

**Proficiency-calibrated rehydration offer:**

- **L3:**
  > "Found intake state from [timestamp]. Last completed: Phase [N].
  > Resume from Phase [N+1], or start fresh?"

- **L2:**
  > "It looks like we started setting up this project before but didn't finish.
  > Here's where we left off: [brief summary of completed decisions]. I can
  > pick up from where we stopped, or we can start over if things have changed.
  > Which do you prefer?"

- **L1:**
  > "I found notes from a previous setup session for this project. It looks like
  > we got through [plain-language description of what was completed] before the
  > session ended. I still have all the decisions we made. Would you like me to
  > continue from where we stopped, or would you rather start fresh?"

### Resume Flow

If the user chooses to resume:

1. Load all completed phase results into working context
2. Set proficiency level from the saved state
3. Briefly confirm key decisions haven't changed:
   > "Quick confirmation before we continue: you said you're building [description]
   > with [stack], targeting [deployment]. Still accurate?"
4. If confirmed: skip to the first incomplete phase
5. If anything changed: update the state file and re-run the affected phase

### Restart Flow

If the user chooses to start fresh:

1. Delete the state file: `rm .claude/intake-state.json`
2. Begin from Phase 0 as normal
3. Do NOT carry over any decisions from the old state

### Stale State Detection

If the state file is older than 7 days, treat it as potentially stale:
> "I found setup notes from [date], which is [N] days ago. Things may have
> changed since then. Want me to use these as a starting point and verify
> the key decisions, or start completely fresh?"

If older than 30 days, recommend starting fresh but don't force it.

---

## Mid-Phase Recovery

The state file records which phase was in progress (`current_phase`) vs. which
phases were fully completed (`completed_phases`). This handles the case where
the session died mid-phase:

- **Phase in progress but not completed:** Re-run the entire phase from the
  beginning. Partial phase results are unreliable — a half-generated CLAUDE.md
  is worse than no CLAUDE.md.
- **Exception — Phase 3 artifacts:** If a CLAUDE.md or .claudeignore file
  exists on disk from a prior session, check it rather than blindly overwriting.
  Ask: "I found an existing CLAUDE.md — was this from our previous session?
  Should I use it as-is, update it, or regenerate from scratch?"

---

## Cleanup

When the protocol completes successfully (Phase 6 handover executed):

```bash
# Remove the intake state file — no longer needed
rm -f .claude/intake-state.json
```

The handover brief (`.claude/handover.md`) takes over as the session continuity
artifact. The intake state file served its purpose as crash recovery and is no
longer relevant.

If the user explicitly abandons the intake ("forget it, I'll just start coding"):
also delete the state file to prevent stale state from confusing a future session.
