# Handover Templates — Brief, Starter Prompts, Summary

Reference templates for the Phase 6 handover. Load when executing the
intake-to-execution transition.

---

## Handover Brief Template

Write this to `.claude/handover.md` as the final act of intake.

```markdown
# Handover Brief — [Project Name]
Generated: [timestamp]

## What We're Building
[2-3 sentence summary of the project from the user's own description]

## Immediate Next Actions
1. **First:** [the single most important thing to do next — be specific]
2. **Then:** [second priority]
3. **Then:** [third priority, if applicable]

## Key Decisions Made During Intake
- [Decision 1 — e.g., "Using App Router, not Pages Router"]
- [Decision 2 — e.g., "Zustand for state, not Redux"]
- [Decision 3 — e.g., "Test-first for API routes, test-after for UI"]

## Risks Identified
- [Risk 1 + how we're addressing it]
- [Risk 2 + mitigation]

## Session Strategy
- Mode: [direct | plan-first | agent teams]
- Effort: [level]
- Hygiene: [when to /compact, when to /clear]

## Definition of Done
[Exact criteria from the user — copy verbatim, not paraphrased]

## If You're a Fresh Claude Code Instance Reading This
The project is configured. Read CLAUDE.md for architecture and verification
commands. This file tells you what the user wants to accomplish in the current
work phase. Start with the first item under "Immediate Next Actions."
```

---

## Starter Prompts by Proficiency Level

Generate one of these and present it as a copy-pasteable command.

**L3 (Practitioner):**
> "Here's your starter prompt — paste this after /compact or in a fresh session:"
> ```
> Read .claude/handover.md. Execute the first action item.
> ```

**L2 (Builder):**
> "Here's a starter prompt you can use to kick things off. Paste it here after
> I compact the conversation, or save it for when you start a new session:"
> ```
> Read .claude/handover.md for context on what we're building. Start with:
> [first action item, stated concretely]. Use plan mode first, then execute.
> ```

**L1 (Explorer):**
> "I've saved a file called handover.md that tells the next version of me
> everything we just discussed. When you're ready to start building — either
> right now or later — just paste this into the chat:"
> ```
> I'm starting a new project. Read .claude/handover.md to understand what
> we're building and what to do first. Walk me through each step as you go.
> ```
> "You don't need to remember everything from our conversation. That file
> has it all."

---

## Intake Summary Template

Present conversationally — not as a raw dump. Adapt tone to proficiency level.
Only show artifacts that were actually generated. Lead with what happens next.

```
## Intake Complete

**Project:** [name]
**Tier:** [Lightweight | Standard | Complex]
**Stack:** [languages, frameworks, key libraries]

### What I've Set Up
- [List only artifacts actually generated]

### Identified Risks
- [Risk → mitigation, or "None identified" for Lightweight]

### What Happens Next
1. [First action — concrete and specific]
2. [Second action]
3. [Third action, if applicable]

### Your Starter Prompt
[The copy-pasteable prompt from above]

### Definition of Done
[Verbatim from the user]
```

After presenting: "I've written a handover brief to `.claude/handover.md` so
nothing from this conversation gets lost."

---

## Context Transition Logic

**Same-session flow:**
1. Ask: "Want to start building now, or are you calling it here for today?"
2. If continuing:
   - Run `/compact` to purge interview transcript but preserve intent
   - Immediately execute the first action item — don't wait for re-prompt
3. If stopping:
   - Confirm: "Everything is saved. When you come back, paste the starter prompt
     in a fresh session."

**Fresh-session flow:**
- User pastes the starter prompt
- New agent reads `.claude/handover.md` and CLAUDE.md
- Execution begins from the first action item

---

## Handover Anti-Patterns

- **Summary as endpoint**: Presenting the summary and stopping without a next action → Always generate a starter prompt and ask whether to continue or pause
- **Context carryover without compaction**: Starting execution with full interview transcript still in context → Always /compact between intake and execution
- **Handover brief as CLAUDE.md clone**: Duplicating architecture info → Handover captures session intent; CLAUDE.md captures project identity
- **Vague first action**: "Start building the app" → Must be specific enough for the agent to execute without clarification
- **Losing the definition of done**: Burying it in a long summary → Copy verbatim into the handover brief
