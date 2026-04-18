# Claude Runway — Operator Reference Guide

**Purpose:** This is your playbook for preparing Claude Code sessions before
writing code. The companion Skill (SKILL.md) is what Claude Code reads and
executes. This document is what YOU read to understand the system, know when
to intervene, and recognize when the protocol is working or failing.

---

## The Core Thesis

The article you read makes one argument that matters above all the enterprise
vocabulary: **the quality of an agentic coding session is decided before the
first line of code is written.**

This protocol is grounded in the Software Development Life Cycle — not as an
abstract methodology, but as the operational backbone that governs every phase.
The SDLC disciplines at work here:

- **Requirements before design** (Phases 1-2): You don't build until you know
  what you're building, who it's for, and what "done" looks like.
- **Shift-left security** (Phase 3): Security controls are embedded during
  environment setup, not patched in after deployment. Catching issues here
  costs 30-50× less than fixing them in production.
- **Risk identification during planning** (Phase 2.4): Surface technical,
  scope, security, data, and dependency risks while they're still cheap to
  address. Not after they've cascaded into production bugs.
- **Testing strategy established before code** (Phase 3, CLAUDE.md): The
  testing approach, framework, and expectations are decided during intake —
  not improvised during implementation.
- **Specification-driven development** (Phase 5): For complex work, the
  architectural plan is a reviewable artifact — "version control for your
  thinking" — not a verbal agreement that drifts.
- **Human-in-the-loop governance** (Cross-cutting): The agent proposes, the
  human approves. Ownership of architectural tradeoffs, security decisions,
  and production consequences stays with you.
- **Living documentation** (Session hygiene): CLAUDE.md and @docs/ files
  update as the code evolves. Stale documentation is worse than no
  documentation because the agent will follow it next session.
- **Continuous feedback loops** (Session hygiene): /compact, /rewind, and
  /clear create the iterative checkpoint rhythm that separates high-performing
  teams from those that accumulate drift.

Claude Code is stateless. Every session starts from zero. The CLAUDE.md file,
the ignore rules, the hooks, the MCP connections — these aren't bureaucratic
overhead. They are the agent's entire understanding of your project. Skip them
and you're asking a brilliant engineer to build your house blindfolded.

But there's a critical nuance the article underplays: **over-preparation is
also a failure mode.** A 500-line CLAUDE.md for a 50-line script wastes tokens
and clutters the agent's reasoning. The protocol must scale to match the project.

---

## What the Skill Does

When activated, the Skill runs a structured intake interview with 5 phases:

| Phase | Purpose | When It Runs |
|-------|---------|-------------|
| 1. Classification | Understand what's being built, assign a complexity tier | Always |
| 2. Architecture & Constraints | Discover tech stack, boundaries, terminology, definition of done | Always |
| 3. Environment Configuration | Generate CLAUDE.md, .claudeignore, hooks, security perimeter | Standard + Complex |
| 4. Session Strategy | Choose execution mode, effort level, MCP servers, hygiene rhythm | Standard + Complex |
| 5. Specification-Driven Planning | Enforce Plan Mode, UI-first design, decomposition for parallelism | Complex only |

The three complexity tiers:

- **Lightweight** (scripts, experiments, single-file tools): Phases 1-2 only
- **Standard** (apps, APIs, multi-file systems): Phases 1-4
- **Complex** (multi-service, production, professional): All 5 phases

---

## Your Role During Intake

You are NOT passive during this process. The Skill asks questions. You answer
them. But you also:

**Challenge the classification.** If the Skill says "Standard" but you know
this is going to grow into something complex, override upward. Your local
knowledge always outranks the protocol's inference.

**Define "done" precisely.** The single most important answer you give during
intake is the definition of done. "Get it working" is not a definition. "The
API returns correct JSON for all 4 endpoint types and passes the test suite"
is a definition.

**Name your anti-patterns.** If you know from experience that Claude Code
tends to reach for a pattern you hate (class components, ORMs, CSS-in-JS,
over-abstraction), say so during intake. It goes into the CLAUDE.md as a
hard constraint.

**Map your vocabulary.** If your project uses domain terms that could be
misinterpreted ("widget" meaning a product, not a UI element; "event"
meaning a user action, not a JS event), surface them. This prevents the
single most common agent error.

---

## How Proficiency Calibration Works

The Skill reads your responses and silently adjusts its interview style to
match your technical proficiency. It does this per-domain — you might get
expert-level questions about frontend architecture and beginner-friendly
guidance on database design, all in the same intake session.

**Three levels (internal labels you won't see announced):**

- **L3 Practitioner**: You get terse, peer-level questions. The Skill challenges
  your assumptions and asks "why" more than "what." Artifacts come with minimal
  commentary because you can read them directly.

- **L2 Builder**: You get structured choices with brief rationale. The Skill
  explains tradeoffs when presenting options and confirms understanding of
  key concepts. Artifacts include "here's why" annotations on non-obvious decisions.

- **L1 Explorer**: You get warm, concrete, closed-ended questions. The Skill
  uses analogies, defines terms on first use, and makes single clear recommendations
  instead of open-ended architectural questions. Artifacts come with a companion
  explanation of what each configuration does and why it matters.

**What you should know about this:**

The calibration starts at L2 and adjusts based on signal. If you know your stuff,
just answer naturally — the Skill will detect it and stop over-explaining. If you're
in unfamiliar territory, the Skill will pick up on that too and provide more context
without you having to ask.

The critical constraint: **your proficiency level never changes what gets built.**
An L1 user's CLAUDE.md has the same structural quality as an L3 user's. The Skill
just does more of the architectural thinking for you and explains more along the way.

If you feel the Skill is over-explaining or under-explaining, you can always say
so directly. It will recalibrate immediately.

---

## The Artifacts You Get

After intake, you'll have some or all of these sitting in your repo:

### CLAUDE.md
The agent's primary briefing document. Under 300 lines. Contains:
- Project identity (what this is)
- Stack with exact versions
- Architecture decisions (yours, not framework defaults)
- Verification commands (executable, not aspirational)
- Terminology map
- Anti-patterns
- Progressive disclosure pointers to deeper docs

**Your job:** Review this before approving. If it contains generic advice
("follow best practices", "write clean code"), delete those lines. Every
line must be project-specific.

### .claudeignore
Soft filter. Prevents the agent from indexing build artifacts, logs, and
large files during exploration. The agent CAN bypass this if you explicitly
point it at an ignored file.

### .claude/settings.json (permissions.deny)
Hard block. Files listed here are completely inaccessible to the agent,
even with explicit instructions. Use for .env files, private keys,
certificates, production credentials.

### Hooks (Complex projects)
Pre-execution hooks block destructive commands before they run.
Post-execution hooks auto-format files after the agent writes them.
These are deterministic — they don't consume tokens and they don't
rely on the agent "remembering" to format its output.

### Ecosystem Scan Results (Standard + Complex projects)
The protocol now runs a three-stage scan: audit, health check, then gap analysis.

**Stage 1 — Audit:** Inventories your installed skills, MCP servers, and CLI tools.

**Stage 2 — Health Check:** This is the critical new piece. Before recommending
any new tools, the protocol checks your existing environment for problems:

- 🔴 **Critical issues** that will degrade session quality: conflicting skills
  (two skills claiming the same trigger territory with contradictory instructions),
  duplicate MCP servers injecting redundant tool schemas, hooks pointing to scripts
  that don't exist, MCP servers configured with missing environment variables,
  and contradictory CLAUDE.md files at different directory levels. Critical issues
  block the intake from proceeding until resolved or explicitly accepted.

- 🟡 **Warnings** that signal drift: overly broad skill descriptions that
  activate on irrelevant prompts, redundant ignore rules, soft overlaps between
  skills that aren't contradictory but consume extra context.

- 🟢 **Clean** areas that passed inspection.

The principle: there's no point adding new capability to a broken foundation.
A single conflicting skill that injects wrong instructions into every session
does more damage than having no skills at all.

**Stage 3 — Gap Analysis + Recommendations:** Same as before — identifies
missing tooling, searches GitHub with quality gates, presents tiered
recommendations for your approval.

---

## Session Commands You Need to Know

| Command | What It Does | When to Use |
|---------|-------------|-------------|
| `/compact` | Compresses conversation history into a dense summary | After completing each sub-task |
| `/rewind` | Rolls back to a prior checkpoint | When the agent enters a failure loop — do NOT try to prompt your way out |
| `/clear` | Wipes conversation entirely, codebase stays | After major milestones, or when switching task domains |
| `/context` | Shows token usage breakdown | After connecting MCP servers, or when the agent seems to lose track |
| `/effort` | Sets cognitive depth (low/standard/high/max) | Match to task complexity — max for architecture, low for boilerplate |

### The Most Important Rule

**When the agent starts compounding errors — 3+ consecutive failures on
the same problem — use /rewind. Do not send more corrective messages.**

Every corrective message adds tokens to an already confused context window.
The agent is not going to reason its way out with more text. Roll back to
the last clean state and re-approach with a different framing.

---

## When to Run This Protocol

**Always run it for:**
- Any new project
- Any existing project you haven't used Claude Code on before
- Any session where the scope has significantly changed since last time

**Skip it for:**
- Quick one-off questions ("what does this error mean?")
- Simple file edits where you already have a configured environment
- Continuation of a session where intake was already completed

---

## How to Install the Skill

Copy the `claude-runway/` directory into your Claude Code skills location:

```bash
# Global installation (available in all projects)
cp -r claude-runway/ ~/.claude/skills/claude-runway/

# Per-project installation
cp -r claude-runway/ .claude/skills/claude-runway/
```

The Skill auto-activates when you start a new project or use trigger phrases
like "new project", "set up", "initialize", "scaffold", or "prep the environment."

---

## What This Protocol Does NOT Do

- It does not write your code. It prepares the environment so the code-writing session is high-quality.
- It does not replace your judgment. It gives Claude Code the context to exercise better judgment.
- It does not guarantee perfect output. It raises the floor and narrows the failure modes.
- It does not manage multi-day project continuity beyond the handover brief — for longer arcs, maintain your CLAUDE.md and @docs/ files as the project evolves.

---

## The Handover Moment

The transition from intake to execution is the most important moment in the
protocol. Here's what happens and what to expect:

**The Skill writes a handover brief** to `.claude/handover.md`. This is NOT a
duplicate of CLAUDE.md — it captures your *current session's intent*: what to
build first, key decisions from the interview, identified risks, and your
exact definition of done. It's designed to be read by either the current agent
(after `/compact`) or a fresh agent (if you come back later).

**You get a starter prompt.** A copy-pasteable command that kicks off execution.
Save it somewhere — in your notes, a pinned tab, wherever. If you come back
tomorrow, paste it into a new Claude Code session and it picks up right where
the intake left off.

**If you continue in the same session:** The Skill will `/compact` the interview
transcript to free up context window, then immediately start on the first action
item. You don't need to re-explain anything.

**If you come back later:** Start a fresh session, paste the starter prompt. The
new agent reads `.claude/handover.md` and CLAUDE.md, and it knows exactly what
to do. No re-explaining, no re-interviewing.

**The handover brief is ephemeral by design.** It captures *this phase's* work
priorities. Once you complete the actions in it, you can delete it or let the
next intake cycle overwrite it. CLAUDE.md is the permanent artifact; the
handover brief is the session plan.

### What If the Session Dies Mid-Intake?

The protocol saves progress to `.claude/intake-state.json` at every phase
boundary. If your session disconnects, crashes, or you close the window in
the middle of intake, nothing is lost.

When you start a new session, the protocol detects the state file and offers
to resume from where you left off. It will briefly confirm that the key decisions
from earlier phases haven't changed, then pick up at the next incomplete phase.

If more than 7 days have passed, it'll flag the state as potentially stale and
ask you to verify. If more than 30 days, it'll recommend starting fresh.

You can always choose to start over instead of resuming — the protocol asks,
it doesn't assume. And once the intake completes successfully, the state file
is automatically deleted.

---

## Quick Start for Your Next Project

1. Open VS Code terminal with Claude Code
2. Say: "I want to start a new project" (triggers the Skill)
3. Answer the intake questions — especially "what does done look like?"
4. Review the generated CLAUDE.md before approving it
5. Save the starter prompt the Skill gives you
6. Either continue building immediately or come back later and paste the starter prompt

That's it. The protocol handles the rest.
