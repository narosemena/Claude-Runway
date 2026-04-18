# Session Hygiene — Context Management Commands & Patterns

Reference for maintaining a clean, high-performance context window
throughout a Claude Code session.

---

## Core Commands

### /compact — Compress Conversation History

**What it does:** Spawns a separate thread to summarize the full conversation
into a dense summary. Purges the raw transcript but preserves intent,
decisions, and established parameters.

**When to use:**
- Immediately after completing a distinct sub-task
- When you notice the agent starting to repeat itself or lose track of earlier decisions
- Before starting a conceptually different phase of work
- When `/context` shows high token usage from conversation history

**When NOT to use:**
- In the middle of a complex multi-step operation (breaks the reasoning chain)
- When you need the agent to reference specific earlier output verbatim

---

### /rewind — Rollback to Prior Checkpoint

**What it does:** Reverts to a prior state. The system captures a checkpoint
at every user prompt. Checkpoints persist in `~/.claude/file-history/`.

**When to use:**
- Agent enters a failure loop (3+ consecutive errors on the same problem)
- Agent makes a fundamentally wrong architectural choice
- A file mutation cascades into broken dependencies
- You realize the approach is wrong and want to try a different direction

**Restoration options when rewinding:**
1. **Full revert**: Restore both conversation memory AND filesystem to prior state
2. **Code-only revert**: Restore filesystem but keep the conversation (preserves discussion of what went wrong)
3. **Summary revert**: Summarize the failed path to prevent repetition without keeping the bloated transcript

**Critical rule:** Do NOT try to prompt the agent out of a failure loop with
natural language corrections. This compounds token bloat and rarely produces
a clean solution. Rewind, then re-approach with a clearer instruction.

---

### /clear — Full Session Reset

**What it does:** Wipes the entire conversation and starts fresh. The codebase
on disk remains unchanged.

**When to use:**
- After completing a major feature or milestone
- When switching to an entirely different area of the codebase
- When context is so degraded that compaction won't help

**After clearing:** The agent has zero memory of prior conversation. It will
rely entirely on: (1) CLAUDE.md files, (2) the codebase itself, (3) your
next prompt. Make sure your CLAUDE.md is current before clearing.

---

### /context — Audit Token Usage

**What it does:** Shows current token consumption breakdown — how much is
used by system prompt, MCP server definitions, conversation history, and
available headroom.

**When to use:**
- After connecting MCP servers (to measure their impact)
- When the agent seems to be ignoring instructions (possible context overflow)
- Periodically during long sessions (every 20-30 minutes of active work)

---

## Session Rhythm Patterns

### For a 1-2 Hour Focused Session

```
[Start] → Run intake protocol → Generate artifacts
  ↓
[Phase 1] → Work on first sub-task
  ↓
[Checkpoint] → /compact after sub-task completion
  ↓
[Phase 2] → Work on second sub-task
  ↓
[Checkpoint] → /compact again
  ↓
[Wrap-up] → Final verification → /clear if done, or leave session for tomorrow
```

### For Multi-Day Projects

```
[Day 1] → Full intake → Plan Mode → Approve plan → Begin implementation → /compact at end of day

[Day 2] → Resume session OR /clear and re-orient from CLAUDE.md
        → Continue implementation → /compact after each milestone

[Day N] → Final integration → Full verification suite → /clear
```

### For Debugging Sessions

```
[Start] → Describe the bug precisely
  ↓
[Explore] → Let agent investigate (read-only is fine here)
  ↓
[Hypothesis] → Agent proposes a fix
  ↓
[Test] → Apply fix → Run verification
  ↓
[If fix works] → /compact and continue
[If fix fails] → /rewind (don't accumulate failed attempts in context)
  ↓
[Retry] → Re-approach with different framing
```

---

## Anti-Patterns

**Token hoarding:** Keeping the full conversation history "just in case."
The cost of a bloated context is degraded reasoning on every subsequent
response. Compact aggressively.

**Correction stacking:** Sending 5 consecutive messages trying to fix an
agent error. Each message adds tokens and often makes things worse.
Rewind after the first failed correction.

**Ghost context:** After /clear, giving a minimal prompt and expecting
the agent to "remember" the project. It doesn't. Either maintain the
session or write a good CLAUDE.md that survives the clear.

**Over-monitoring:** Checking /context every 2 minutes. Check at natural
breakpoints, not obsessively.

---

## Automatic Cleanup

All session artifacts (transcripts, tool spillover files, checkpoints) in
`~/.claude/` are subject to a 30-day automatic cleanup cycle. This is
configurable via `cleanupPeriodDays` in settings.

If you need to preserve a specific session's state beyond 30 days,
export or copy the relevant files before the cleanup window.
