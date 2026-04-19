# Claude Runway

![version](https://img.shields.io/badge/version-1.1.0-blue) ![license](https://img.shields.io/badge/license-CC%20BY--NC--ND%204.0-green) ![Claude Code](https://img.shields.io/badge/Claude%20Code-skill-orange)

### The hardest part of Claude Code isn't the coding. It's knowing how to start.

You installed Claude Code. You opened your terminal. And then you sat there, cursor blinking, wondering: *What do I type? How much does it need to know? What if I set it up wrong and it makes a mess of my project?*

You don't need to know what a `CLAUDE.md` file does, how to configure MCP servers, or why hooks matter. You just need to know what you want to build. Runway handles the rest.

You're not alone. And you're not doing anything wrong. The problem isn't you — it's that Claude Code is a powerful engine with no onboarding. It expects you to know what a `CLAUDE.md` file is, how to structure your project context, which safety rails to set up, and how to manage your session. Before you write a single line, Claude Code expects you to already know things it never told you.

**Claude Runway changes that.** It sits between you and Claude Code and asks the questions that need asking: *What are you building? What tools are you using? What should I stay away from? What does "done" look like?* Then it sets everything up — the configuration files, the safety boundaries, the session strategy — and hands you a clear first step.

---

## What Happens When You Use It

You open Claude Code and say something like: *"I want to build a workout tracker app."*

Here's what follows:

```
You:     "I want to build a workout tracker app"

Runway:  "Got it. What does done look like for you — are we building
          a working prototype, or something you'll actually ship?"

You:     "I want to actually ship it. Mobile-friendly, user accounts,
          workout logging."

Runway:  "This is a Standard-tier project — multi-file, auth, a database.
          I'll set up your full environment. A couple more questions first..."

          [5 minutes later]

          ✅ CLAUDE.md generated (your project rules, stack, verification commands)
          ✅ .claudeignore configured
          ✅ Environment scanned — 1 conflict resolved
          ✅ Handover brief saved to .claude/handover.md

          Your first action: scaffold the Next.js project with the
          directory structure we just defined.

          Ready to build now, or saving this for later?
```

**A 5-minute conversation replaces the hours you'd spend learning what `CLAUDE.md` is, what `.claudeignore` does, why `permissions.deny` matters, and how to keep the agent from going off the rails.**

---

## Install in 30 Seconds

**Recommended — works on Mac, Linux, and Windows:**

```bash
# Install globally (available in every project)
git clone https://github.com/narosemena/Claude-Runway ~/.claude/skills/claude-runway
```

**Windows (PowerShell or Git Bash):**
```powershell
git clone https://github.com/narosemena/Claude-Runway "$env:USERPROFILE\.claude\skills\claude-runway"
```

**Verify it installed:**
```bash
ls ~/.claude/skills/claude-runway/SKILL.md
```

You should see `SKILL.md` listed. If you don't, the clone went somewhere unexpected — check your current directory and try again.

**To update later:**
```bash
cd ~/.claude/skills/claude-runway && git pull
```

Then open Claude Code and say:

> *"I want to start a new project"*

Runway activates automatically. No commands to memorize. No flags to set.

## Built for Everyone — Calibrated to You

Most AI developer tools assume you already know what you're doing. Runway doesn't.

It reads your responses during the interview and silently adapts. Not a quiz. Not a self-assessment. It just listens to how you talk about your project and adjusts accordingly:

**If you say** *"Next.js 15 with App Router, RSC for data fetching, Zustand for client state"* — Runway skips the basics, challenges your architectural choices, and gets out of your way fast.

**If you say** *"I want to build a website for my business"* — Runway recommends the right technology, explains each piece in plain language, and walks you through every decision so you understand what's being set up and why.

**The output is the same quality either way.** An experienced developer's configuration files are structurally identical to a beginner's. Runway just does more of the heavy lifting when you need it — and more of the explaining so you learn as you go.

---

## You Don't Need to Be a Developer

The rise of AI coding tools is bringing new people to software development every day — entrepreneurs building their own products, designers who want to prototype, professionals automating their workflows, hobbyists turning ideas into apps. Claude Code is one of the most powerful tools available, but its terminal-first interface creates a wall that keeps many of these people out.

Claude Runway lowers that wall.

- **No terminal expertise required.** Runway asks you questions in plain English and translates your answers into the technical configuration Claude Code needs.
- **No configuration knowledge required.** You don't need to know what a `CLAUDE.md` file is, how hooks work, or what an MCP server does. Runway builds it all and explains what each piece does if you want to know.
- **No architecture degree required.** Runway recommends project structures, testing approaches, and safety boundaries based on what you're building. You confirm or redirect — you don't have to design from scratch.
- **Mistakes are recoverable.** If your session crashes or you close the window, Runway saves your progress automatically. Come back tomorrow and pick up where you left off.

The door to AI-assisted development is open. Runway keeps it that way.

---

## For the Experienced Developer

You don't need hand-holding. You need speed.

Runway detects that in your first two responses and shifts to peer-level conversation — terse questions, architectural challenges, minimal commentary on generated artifacts. What you get:

- A project-specific `CLAUDE.md` with zero generic advice, generated in minutes instead of hand-authored over an hour
- An environment health check that catches conflicting skills, stale MCP configs, and broken hooks you forgot about
- A harness validation pass that verifies every generated artifact works with every other — your CLAUDE.md references tools that are installed, your hooks match your stack, your ignore rules don't block files the agent needs
- Quality-gated tooling recommendations from GitHub — not a firehose of repos, but filtered results you can trust (50+ stars, actively maintained, documented)
- Plan Mode enforcement and architectural contracts for complex projects
- A clean handover with a starter prompt you can paste in any future session

The protocol scales to your project. A weekend script gets a 2-minute triage. A production system gets security perimeters, pre-execution hooks, and specification-driven planning.

You've done this preparation manually before. Runway makes it systematic and consistent.

---

## What Runway Sets Up for You

Depending on your project's complexity, Runway generates some or all of these:

| Artifact | What It Does | Why It Matters |
|----------|-------------|---------------|
| **CLAUDE.md** | Tells Claude Code about your project — the stack, the structure, the rules | Without this, the agent guesses. With it, the agent knows. |
| **.claudeignore + security perimeter** | Tells Claude Code which files to skip and hard-blocks secrets, API keys, and credentials | Prevents wasted time and protects sensitive data |
| **Hooks** | Automatically blocks dangerous commands and formats code after every edit | Safety and consistency without you having to think about it |
| **Harness validation** | Verifies every generated file works with every other before you start coding | Catches mismatches — like a CLAUDE.md referencing a tool that isn't installed — before they become mid-session bugs |
| **Handover brief + starter prompt** | Saves your project plan and gives you a copy-pasteable command to resume anytime | Eliminates the "what do I type?" moment whether you continue now or come back tomorrow |

---

## When Things Go Wrong

### Your internet drops mid-setup
Runway saves progress at every step. When you reconnect, it detects the saved state and offers to resume from where you left off.

### You close the window by accident
Same as above. Your progress is on disk, not in the conversation.

### You come back a week later
Runway notices the saved state is getting stale and asks you to verify that your plans haven't changed before continuing.

### You want to start over
Just say so. Runway clears the old state and begins fresh. No judgment.

---

## Where Runway Works

Claude Runway adapts to wherever you're working:

| Where You Are | What You Get |
|---------------|-------------|
| **Claude Code** (terminal or VS Code) | Full experience — everything automated |
| **Claude.ai** (web with computer use) | Full interview, config files generated for you to copy to your project |
| **Claude Desktop** (Mac or Windows) | Interview + configuration as copy-paste blocks |
| **Claude.ai** (web or mobile chat) | Guided planning conversation — you create the files yourself with clear instructions |

---

## What's in the Box

```
claude-runway/
├── SKILL.md                     # Core protocol — 415 lines
├── OPERATOR-REFERENCE.md        # Your companion guide
├── README.md
└── references/                  # Loaded only when needed
    ├── claude-md-examples.md    # Templates by technology stack
    ├── complexity-decision-tree.md
    ├── ecosystem-scanner.md     # Environment audit + tool search
    ├── environment-detection.md
    ├── handover-templates.md
    ├── harness-validation.md    # Cross-artifact integrity checks
    ├── hook-templates.md
    ├── intake-state.md          # Crash recovery + rehydration
    ├── mcp-catalog.md
    ├── proficiency-calibration.md
    └── session-hygiene.md
```

---

## How Runway Thinks

A few principles behind every decision the skill makes:

- **Understand before you build.** Discovery isn't overhead. It's the highest-leverage activity.
- **Protect by default.** Security and safety go in during preparation — not patched in after something breaks.
- **Surface risks early.** A problem found in planning costs pennies. Found in production, it costs thousands.
- **Scale to the work.** A script doesn't need an architectural review. A production API does.
- **Meet you where you are.** Expertise isn't a prerequisite. Curiosity is enough.
- **You decide.** The agent proposes, recommends, and generates. You approve, redirect, and own the outcome.

And what Runway won't do, no matter how you ask:

- **Write your code.** It prepares the environment so coding goes well.
- **Replace your judgment.** It gives the agent better context, not better taste.
- **Lock you in.** Every artifact it generates is a plain text file you can edit, move, or delete.

---

## What You Need

- **Best experience:** Claude Code CLI or VS Code extension
- **For hooks:** Node.js
- **Works without Claude Code** in advisory mode on Claude.ai, Claude Desktop, and web chat

---

## Feedback

The best feedback comes from real use. If the intake asks a question that doesn't make sense for your project, or generates a config that needs immediate editing, or makes you feel stupid — that last one especially — open an issue:

1. What you were trying to build
2. What happened
3. What you expected

This skill is for everyone. If it's not working for you, that's a bug.

---

## License

CC BY-NC-ND 4.0
