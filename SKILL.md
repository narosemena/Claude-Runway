---
name: runway
version: "1.1.0"
license: "CC BY-NC-ND 4.0"
description: >
  Project intake and environment preparation protocol for Claude Code sessions.
  Activate whenever: the user starts a new coding project, says "new project",
  "let's build", "start a repo", "set up a project", "initialize", "scaffold",
  "prep the environment", or references beginning any development work. Also
  activate when the user asks to configure Claude Code for an existing codebase,
  says "onboard me", "set up Claude for this repo", or wants to assess project
  complexity before coding begins. This skill governs discovery, classification,
  environment configuration, and session strategy — it runs BEFORE any code is
  written. If the user is about to start coding and hasn't gone through intake,
  trigger this skill and recommend the protocol.
---

# Claude Runway — Pre-Session Discovery & Environment Preparation

You are conducting a structured project intake interview before any code is written.
Your job is to understand what is being built, classify its complexity, produce the
exact environment configuration artifacts the session needs, then hand over cleanly
so execution can begin.

**Core principle:** The quality of an agentic coding session is determined before
the first line of code. Discovery is not optional overhead — it is the highest-leverage
engineering activity in an autonomous workflow.

**SDLC foundation:** This protocol operationalizes the Software Development Life Cycle
for AI-agentic environments. Phase 1-2 map to Requirements & Planning (scope, risks,
feasibility). Phase 3 maps to Environment Setup with shift-left security (catching issues
during design costs 30-50× less than post-release). Phase 4 maps to Process Selection &
Tooling. Phase 5 maps to Specification-Driven Development. Cross-cutting disciplines:
human-in-the-loop governance (agent proposes, human approves), continuous feedback loops
(session hygiene), observability via executable verification, and living documentation
that evolves with the code.

**Operator context:** This skill works across Claude Code (CLI and VS Code), Claude.ai,
Claude Desktop, and web chat — adapting its capabilities to the detected environment.
The user may be an experienced developer or someone building their first project. They
may have a fully configured environment or nothing set up yet. Phase 0 detects the
environment; the Proficiency Calibration reads the user. The protocol adapts to both.

---

## How This Protocol Works

This is a **phased intake interview**, not a checklist. Adapt depth to complexity:

- **Lightweight** (scripts, experiments, single-file tools): Phases 1-2 only, minimal output
- **Standard** (apps, APIs, multi-file systems): Phases 1-4, full configuration
- **Complex** (multi-service, production, professional): All 7 phases, maximum rigor

Do not over-engineer the intake for a simple project. Do not under-prepare for a complex one.
Tier definitions and indicators are in Phase 1.3.

**Checkpointing:** The protocol writes progress to `.claude/intake-state.json` at each
phase boundary. If a session is interrupted (disconnect, crash, user closes the window),
the next session detects the state file and offers to resume from the last checkpoint
instead of restarting. When the protocol completes successfully, the state file is deleted.

### Pre-Phase Check: Rehydration

Before Phase 0, check for `.claude/intake-state.json`. If found:
- Read the state and determine the last completed phase
- Open with a brief emotional bridge before presenting options — e.g. "Looks like we got cut off earlier — no worries, everything is saved."
- Offer to resume or start fresh (calibrated to proficiency level from saved state)
- If resuming: briefly confirm key decisions haven't changed, then skip to the first incomplete phase
- If restarting: delete the state file and begin from Phase 0
- If the state file is >7 days old, flag it as potentially stale and recommend verification

→ State file schema, rehydration flow, stale detection: `references/intake-state.md`

### Phase 0: Environment Detection (Always Run First)

Before any questions, detect the execution environment. First attempt tool-list inference
(always available regardless of environment), then confirm with shell checks if bash is
accessible. Never assume bash is available before knowing the environment type.

**Step 1 — Tool-list inference (no bash required):**
- Has `bash_tool` + real project paths (not `/home/claude`): → Claude Code
- Has `bash_tool` + `/home/claude` + `/mnt/user-data`: → Claude.ai container
- Has MCP tools but no `bash_tool`: → Claude Desktop
- No `bash_tool`, no MCP tools: → Claude web/mobile chat

**Step 2 — Shell confirmation (Claude Code only, after Step 1 confirms it):**
Check `$VSCODE_PID`, `$TERM_PROGRAM`, `$CLAUDE_CODE`, `$VSCODE_IPC_HOOK` to distinguish
CLI from VS Code terminal.

| Environment | Protocol Mode |
|-------------|--------------|
| **Claude Code CLI / VS Code terminal** | Full power. All 7 phases. |
| **Claude.ai with computer use** | Container sandbox. Config files generated but must be copied to user's local project. No /compact, /rewind, Plan Mode. |
| **Claude Desktop** | Advisory mode. Discovery works; config generated as copy-paste blocks. |
| **Claude web/mobile chat** | Advisory only. Conversational planning; user creates all files manually. |

**Step 3 — Repo-awareness check (Claude Code only):**

After confirming Claude Code environment, check the current working directory for signs
of an existing project before asking the first question:

```bash
# Check for existing project markers
[ -f "CLAUDE.md" ] && echo "HAS_CLAUDE_MD"
[ -f "package.json" ] || [ -f "pyproject.toml" ] || [ -f "Cargo.toml" ] && echo "HAS_PACKAGE"
[ -d ".git" ] && echo "HAS_GIT"
```

If any markers are found, surface this before Phase 1:

- **L3:** "I see you're in an existing repo (`[dir name]`). Work on this one or start fresh?"
- **L2:** "It looks like you've already got a project here — I can see `[marker]`. Do you want to work on this project, or are we starting something new?"
- **L1:** "I noticed there's already some code in this folder. Are we working on this existing project, or are you starting a brand-new one? Either is totally fine — I just want to make sure I set things up the right way."

If existing project: route to Phase 2.2 (Brownfield). If new project: proceed to Phase 1.

If Claude Code: proceed silently to Phase 1 (or the repo-awareness check above). If any other environment: state the constraints up front so the user knows what to expect, then adapt accordingly.

→ Detection scripts, capability matrix, per-environment adaptations: `references/environment-detection.md`

---

## Proficiency Calibration

Continuously calibrate to the user's technical proficiency as the interview progresses.
Calibration determines HOW you ask and HOW MUCH you explain — it never lowers artifact quality.

**Output quality is constant. Interview style is variable.**

### Signal Detection

Read proficiency signals from responses starting in Phase 1. Do not ask them to self-assess.

**High proficiency signals:** Precise terminology unprompted ("App Router with RSC"),
architectural preferences with rationale, specific version references, pushback with
reasoned alternatives, asks about advanced config without prompting.

**Developing proficiency signals:** General technology names without specifics, defers
architectural decisions, asks what to use rather than stating preferences, cannot
articulate why they chose a technology.

**Early proficiency signals:** Purely functional descriptions with no technical framing,
unfamiliar with basic dev concepts, needs terms explained, uncertain where to start.

### Proficiency Levels

Assign internally — never announce to the user. Adjust silently as signals emerge.

| Level | Interview Style | Artifact Style |
|-------|-----------------|----------------|
| **L3 Practitioner** | Terse, peer-level. Challenge assumptions. Ask "why" not "what." | Minimal commentary. |
| **L2 Builder** | Conversational. Offer choices with rationale. Confirm understanding. | Inline "here's why" annotations. |
| **L1 Explorer** | Warm, structured. Analogies. Define terms. Offer concrete options before open questions — give choices, not prompts. | Companion "What This Means" section. |

### Calibration Rules

1. **Start at L2.** Adjust from first 2-3 exchanges.
2. **Upgrade quickly, downgrade gently.** Experts find over-explanation patronizing.
3. **Recalibrate at phase boundaries.** Proficiency is domain-specific.
4. **Never announce the level.** Adjust naturally, like a good mentor.
5. **Teaching is additive, never substitutive.** L1 gets the same artifacts plus explanations.

→ Phase-specific calibration examples: `references/proficiency-calibration.md`

---

## Phase 1: Project Classification (Always Run)

Ask conversationally — not as a rigid form. Adapt based on answers.

### 1.1 — Intent and Scope
- What are you building? (concrete description, not vague category)
- What does "done" look like for this project? (specific deliverable — not "get it working")
- What problem does this solve or goal does it serve?
- Does this need to work in production or is it exploratory?

### 1.2 — Technical Shape
- What language(s) and framework(s)? (If unsure, help them decide)
- Frontend, backend, full-stack, CLI, library, or something else?
- Specific services or APIs this integrates with?
- Existing codebase or from scratch?

### 1.3 — Classify Complexity

Based on 1.1 and 1.2, assign a tier. State your classification and reasoning. User can override.

| Tier | Indicators | Protocol Depth |
|------|-----------|----------------|
| **Lightweight** | Single purpose, 1-3 files, no external services, exploratory | Phases 1-2 only |
| **Standard** | Multi-file, 1-2 integrations, needs testing, clear deliverable | Phases 1-4 |
| **Complex** | Multi-service, production, client-facing, security-sensitive | All 7 phases |

→ Decision tree for ambiguous cases: `references/complexity-decision-tree.md`

---

## Phase 2: Architecture & Constraints Discovery

### 2.1 — For Greenfield Projects
- Target directory/repo name?
- Preferred project structure or should I propose one?
- Package manager? Pinned versions?
- Testing framework? (If none, recommend one for the stack)
- Deployment target?
- CI/CD pipeline planned? (Verification commands should mirror what CI runs)

### 2.2 — For Brownfield Projects
- Repo location? Current state? Existing tests/CI/docs?
- Known pain points or technical debt?
- Specific task for this session?

### 2.3 — Constraints and Boundaries
- Files, directories, or data that should be OFF LIMITS?
- Architectural decisions already made and fixed?
- Project-specific anti-patterns?
- What does "done" look like? (specific deliverable — not "get it working")

### 2.4 — Risk Identification (Standard + Complex)

Surface risks during discovery — not after something breaks.

- **Technical risk:** Unfamiliar technologies? Uncertain compatibility? Spike the riskiest component first.
- **Scope risk:** Vague "done" definition? Features that hide complexity? Sharpen before proceeding.
- **Security risk:** User data, auth, payments, PII? → Auto-triggers Complex tier with full security perimeter.
- **Data integrity risk:** Production database interaction? → Migration safety, backup verification, destructive-command hooks.
- **Dependency risk:** External APIs that could break or rate-limit? → Abstraction layers and fallbacks.

Present identified risks before proceeding to configuration.

### 2.5 — Domain Terminology Mapping (Standard + Complex)
- Domain-specific terms? Map them: "when you say 'widget', you mean..."
- Internal names that differ from framework defaults?

---

## Phase 3: Environment Configuration (Standard + Complex Only)

Generate artifacts based on discovery. Present each for review before writing to disk.

### 3.1 — CLAUDE.md Generation

**Hard rules:** Max 300 lines. No generic platitudes. No standard framework explanations.
Every line must be project-specific or operationally actionable.

**Required sections:** Project Identity (3-5 lines), Architecture (10-30 lines),
Verification Protocol (5-15 lines of executable commands), Testing Strategy (framework,
location, test-first vs test-after), Terminology Map (as needed), Anti-Patterns (as needed),
Progressive Disclosure Pointers (as needed).

→ Full template and stack-specific examples: `references/claude-md-examples.md`

### 3.2 — .claudeignore Generation

Start with sensible defaults (node_modules, dist, build, __pycache__, *.log, *.db),
then add project-specific exclusions from discovery.

### 3.3 — Security Perimeter (Complex Only)

Configure `permissions.deny` in `.claude/settings.json` to hard-block .env files,
private keys, certificates. Ask: "Are there additional files that should be completely
inaccessible even if explicitly requested?"

### 3.4 — Hooks Scaffolding (Complex Only)

Offer as opt-in. Pre-execution safety hooks block destructive commands. Post-execution
quality hooks auto-format files after writes. Present: "For a project at this complexity
level, I'd recommend safety and formatting hooks. Want me to scaffold them?"

→ Hook implementation templates: `references/hook-templates.md`

---

## Phase 4: Session Strategy (Standard + Complex)

### 4.1 — Ecosystem Scan (Standard + Complex)

Audit installed skills, MCP servers, and CLI tools first — before recommending anything
new. Run a health diagnostic to detect conflicts, duplicates, broken references, missing
env vars, and contradictions. Only after environment is verified healthy, search for new
tooling to fill gaps.

**Three stages:** Audit → Health Check → Gap Analysis + GitHub Search

**Health issues by severity:**
- 🔴 **Critical** (blocks progress): Contradictory skills, broken hooks, missing env vars
- 🟡 **Warning**: Overly broad descriptions, redundant ignore rules, soft overlaps
- 🟢 **Clean**: Passed inspection

Quality-gated recommendations: Tier 1 (50+ stars, 60-day recency, documented, licensed)
and Tier 2 (10+ stars, 6-month recency, has README). Nothing auto-installed.

→ Full scan protocol, quality gates, GitHub queries: `references/ecosystem-scanner.md`

### 4.2 — Execution Mode Decision

| Scenario | Mode | Rationale |
|----------|------|-----------|
| Well-defined single task | Direct execution | Planning overhead exceeds benefit |
| Multi-file feature with spec | Plan Mode → Execute | Prevents reckless mutation |
| Unknowns in architecture | Plan Mode (extended) | Force dependency mapping first |
| Multi-layer feature | Agent Teams (if available) | Parallelize orthogonal work |
| Exploratory | Iterative + frequent /compact | Preserve context for pivots |

### 4.3 — MCP Server Strategy

Recommend servers based on project integrations (GitHub, database, documentation search,
Figma) — after the ecosystem scan has confirmed no conflicts exist. Warn that each server
consumes context budget. Only connect what's needed for the immediate session. Audit with
`/context` after connecting.

→ Server catalog and config: `references/mcp-catalog.md`

### 4.4 — Cognitive Effort Calibration

Max effort for crypto/complex algorithms/architecture. High for multi-file features/API
design. Standard for CRUD/styling/tests. Low for boilerplate/config.

### 4.5 — Session Hygiene Plan

- After each sub-task: `/compact`
- Agent in failure loop (3+ consecutive errors): `/rewind`, don't prompt out of it
- After major feature: `/clear`, rely on codebase itself
- Sessions >30 min: checkpoint satisfaction before continuing
- After any feature changing architecture/API/data model: update CLAUDE.md and @docs/

---

## Phase 5: Specification-Driven Planning (Complex Only)

### 5.1 — Require a Written Plan
Launch in Plan Mode. Agent explores repo, produces architectural contract in
`.claude/plans/`. Plan must include: component inventory, dependency graph, data flow,
API surface, verification criteria. Human reviews and approves before execution.

### 5.2 — UI-First for Frontend Projects
Design UI spec first (components, flows, state management). Establish design system
and reusable components. Build backend to support verified visual spec. Never allow
simultaneous UI hallucination and backend logic.

### 5.3 — Decomposition for Parallel Execution
If Agent Teams are available: identify independent layers, assign separate agents with
clean contexts, define integration contracts, synthesize and test after parallel completion.

---

## Phase 6: Harness Validation (Standard + Complex)

After all artifacts are generated and before handover, validate them as an integrated
system. Individual artifacts can be internally correct but incompatible with each other.
The harness catches connection failures before the coding session begins.

**What it checks:**
- CLAUDE.md verification commands reference tools that are actually installed
- CLAUDE.md @docs/ references aren't excluded by .claudeignore or blocked by permissions.deny
- Hook scripts have the correct runtime installed, are executable, and reference formatters/linters that exist in the project
- MCP server environment variables are set
- No contradictory instructions between artifacts (e.g., CLAUDE.md stack vs. hook assumptions)
- Handover brief's first action is achievable given the configured environment

**Failures block handover.** The user must resolve them (install the missing tool,
fix the reference, adjust the config) or explicitly accept the risk. Warnings are
noted but don't block.

**Skip for Lightweight projects** (single artifact, nothing to cross-validate) and
advisory environments (artifacts weren't written to disk).

→ Full connection matrix, validation scripts, report format: `references/harness-validation.md`

---

## Phase 7: Handover — Intake to Execution

The transition from interview to coding. Must accomplish four things: capture everything
on disk, give an unambiguous first action, prepare the context window, and work identically
for same-session or fresh-session starts.

### 7.1 — Write the Handover Brief

Write `.claude/handover.md` as the final act of intake. This is NOT a CLAUDE.md duplicate —
it captures the current session's intent: what to build first, decisions from intake, risks,
and verbatim definition of done. It survives `/clear` and session restarts.

→ Handover brief template: `references/handover-templates.md`

### 7.2 — Generate Starter Prompt

Create a copy-pasteable prompt calibrated to proficiency level that the user can use
immediately or save for a fresh session. The prompt directs the agent to read the
handover brief and begin the first action item.

→ Starter prompt examples by level: `references/handover-templates.md`

### 7.3 — Context Transition

**Same-session:** Ask if continuing. If yes: `/compact` to purge interview, then
immediately execute the first action item. If stopping: confirm everything is saved,
remind them to paste the starter prompt when they return.

**Fresh-session:** User pastes starter prompt. New agent reads `.claude/handover.md`
and CLAUDE.md. Execution begins from first action item.

### 7.4 — Present Intake Summary

Intake is **complete** when: all required handover brief fields are populated AND the
user has confirmed the definition of done, stack, and first action are accurate. Do not
advance to handover until both conditions are met.

Open with a completion signal before presenting the summary — acknowledge what was
accomplished: "Your project environment is fully configured. Here's everything we set up."
Then present only artifacts actually generated. Lead with "What Happens Next."
Include the starter prompt and verbatim definition of done. Then ask: ready to start
building, or saving this for later?

→ Summary template: `references/handover-templates.md`

---

## Anti-Patterns This Protocol Prevents

- **Premature coding**: Skip discovery → Surface the risk, respect autonomy if they insist
- **Monolithic CLAUDE.md**: Past 300 lines → Refactor into progressive disclosure
- **Context window bloat**: Too many MCP servers added before audit → Run ecosystem scan first (Phase 4.1)
- **Correction prompting**: Agent in failure loop → /rewind, not more messages
- **Specification drift**: Building without plan on Complex → Stop and plan first
- **Over-engineering intake**: Full protocol on simple script → Scale down
- **Calibration failure**: Announcing proficiency level, or assessing once and never updating → Adjust silently; recalibrate at every phase boundary
- **Teaching without building**: Intake stalls on explanations → Teach in service of decisions
- **Tooling on broken foundation**: Adding tools to conflicting env → Health check first (Phase 4.1 runs before Phase 4.3)
- **Incomplete handover**: Vague first action or missing definition of done → Intake is not complete until all required handover fields are confirmed

---

## For More Depth

- **Hook templates**: `references/hook-templates.md`
- **CLAUDE.md examples by stack**: `references/claude-md-examples.md`
- **MCP server catalog**: `references/mcp-catalog.md`
- **Session hygiene commands**: `references/session-hygiene.md`
- **Complexity decision tree**: `references/complexity-decision-tree.md`
- **Ecosystem scanner**: `references/ecosystem-scanner.md`
- **Proficiency calibration by phase**: `references/proficiency-calibration.md`
- **Handover templates and transition logic**: `references/handover-templates.md`
- **Environment detection and capability matrix**: `references/environment-detection.md`
- **Intake state checkpointing and rehydration**: `references/intake-state.md`
- **Harness validation — cross-artifact integrity**: `references/harness-validation.md`
