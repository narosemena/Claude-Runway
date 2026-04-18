# Ecosystem Scanner — Environment Audit & Tooling Recommendations

This module scans the user's local Claude Code environment, identifies gaps
in their tooling setup, and searches GitHub for skills and MCP servers that
match the project's needs. Recommendations are quality-gated and tiered.

**When to load:** During Phase 4 of the intake protocol, after the project's
stack, integrations, and complexity tier are established. Also load if the
user explicitly asks about available skills, MCP servers, or plugins.

---

## Step 1: Environment Audit

Before recommending anything, establish what's already installed. Run these
commands and parse the output:

### Detect Installed Skills

```bash
# Check user-level skills
ls -1 ~/.claude/skills/ 2>/dev/null || echo "NO_USER_SKILLS"

# Check project-level skills
ls -1 .claude/skills/ 2>/dev/null || echo "NO_PROJECT_SKILLS"
```

### Detect Configured MCP Servers

```bash
# Check user-level MCP config
cat ~/.claude/settings.json 2>/dev/null | grep -A 5 '"mcpServers"' || echo "NO_USER_MCP"

# Check project-level MCP config
cat .claude/settings.json 2>/dev/null | grep -A 5 '"mcpServers"' || echo "NO_PROJECT_MCP"
```

### Detect Available CLI Tools

```bash
# Check for common development tools the project might need
for tool in git node npm npx python3 pip docker docker-compose psql redis-cli; do
  command -v "$tool" >/dev/null 2>&1 && echo "FOUND: $tool" || echo "MISSING: $tool"
done
```

### Summarize the Audit

Present findings to the user before searching for recommendations:

```
## Environment Audit

### Installed Skills
- [list or "None detected"]

### Configured MCP Servers
- [list or "None configured"]

### Available CLI Tools
- [list found tools]
- Missing: [list missing tools relevant to the project stack]
```

If the environment is well-equipped for the project's needs, say so and skip
the GitHub search. Don't recommend tools for the sake of recommending tools.

---

## Step 2: Environment Health Diagnostic

After auditing what's installed, check for conflicts, redundancies, and
misconfigurations that could degrade agent performance. A cluttered or
conflicting environment is worse than an empty one — it actively injects
noise into the context window and causes unpredictable behavior.

Run this diagnostic before looking for new tools. There is no point adding
capability to a broken foundation.

### 2.1 — Skill Conflict Detection

**Duplicate skills (same skill at multiple paths):**

```bash
# Compare skill names across user-level and project-level directories
USER_SKILLS=$(find ~/.claude/skills/ -name "SKILL.md" -exec grep -l "^name:" {} \; 2>/dev/null)
PROJECT_SKILLS=$(find .claude/skills/ -name "SKILL.md" -exec grep -l "^name:" {} \; 2>/dev/null)

# Extract names and find duplicates
for f in $USER_SKILLS $PROJECT_SKILLS; do
  grep "^name:" "$f" | sed 's/name: *//'
done | sort | uniq -d
```

If duplicates exist, flag them. Project-level skills override user-level for
the same name, but having both creates confusion about which version is active.

Mitigation:
> 🔧 Compare version numbers (or last-modified dates if no version). Keep the
> newer version at the scope you want (user-level for global, project-level for
> project-specific). Remove the other:
> `rm -rf ~/.claude/skills/[duplicate-skill-name]` or
> `rm -rf .claude/skills/[duplicate-skill-name]`

**Overlapping trigger descriptions:**

Read the `description` field from every installed SKILL.md frontmatter. Flag
any pair of skills where:
- Both claim to handle the same technology (e.g., two skills both mention "React")
- Both claim to handle the same task type (e.g., two skills both trigger on "create a component")
- One skill's description is a superset of another's (e.g., a "frontend" skill and a "React" skill where the frontend skill also covers React)

For each overlap, assess severity and provide the specific mitigation:

- **Hard conflict** 🔴: Two skills that would produce contradictory output for the same prompt (e.g., one enforces Tailwind, the other enforces CSS modules).
  > 🔧 Determine which skill's approach matches the current project. Deactivate
  > the conflicting skill by either removing it (`rm -rf [skill-path]`) or
  > renaming its SKILL.md to SKILL.md.disabled so it can be restored later.
  > If both skills have value but for different projects, move the non-applicable
  > one out of user-level and into the specific project where it belongs.

- **Soft overlap** 🟡: Two skills that cover adjacent territory but could coexist (e.g., a general frontend skill and a specialized accessibility skill).
  > 🔧 Narrow the broader skill's trigger description to exclude the territory
  > covered by the specialized skill. For example, add "Do NOT trigger for
  > accessibility-specific tasks — defer to [other-skill-name]" to the
  > broader skill's description field.

- **Harmless duplication** 🟢: Same skill at different versions.
  > 🔧 Keep the newer version. Remove the older:
  > `rm -rf [path-to-older-version]`

**Broken skill references:**

```bash
# For each SKILL.md, check that referenced files exist
for skill_dir in ~/.claude/skills/*/ .claude/skills/*/; do
  [ -f "$skill_dir/SKILL.md" ] || continue
  # Extract file references (patterns like references/foo.md, scripts/bar.py)
  grep -oP '`(references|scripts|assets)/[^`]+`' "$skill_dir/SKILL.md" | tr -d '`' | while read ref; do
    [ -f "$skill_dir/$ref" ] || echo "BROKEN REF: $skill_dir → $ref"
  done
done
```

Broken references mean the skill's progressive disclosure is silently failing —
the model is told to read a file that doesn't exist, which wastes a tool call
and degrades trust in the skill's instructions.

Mitigation (choose based on situation):
> 🔧 **If the file was accidentally deleted or never downloaded:** Restore it
> from the skill's source repo: `git checkout [skill-repo] -- [missing-file-path]`
> or re-download the skill package.
>
> 🔧 **If the reference is to a file that was renamed:** Update the pointer in
> SKILL.md to the current filename.
>
> 🔧 **If the file was intentionally removed and the reference is stale:**
> Remove the reference line from SKILL.md so the model doesn't attempt to load it.
>
> 🔧 **If the entire skill appears incomplete or corrupted:** Remove and reinstall:
> `rm -rf [skill-dir] && claude install-skill [source]`

**Overly broad skill descriptions:**

Flag any skill whose description contains fewer than 2 specific technology
names or task types. A description like "Use for all coding tasks" will
trigger on nearly every prompt, consuming context budget on tasks where it
adds no value.

Mitigation:
> 🔧 Rewrite the skill's `description` frontmatter field to include specific
> trigger conditions. Replace vague language with concrete technology names,
> file types, or task patterns. Example — change "Use for coding tasks" to
> "Use when building React components with TypeScript, or when the user asks
> about component architecture, prop typing, or hook patterns."
> Run the updated description against 3-4 test prompts to verify it triggers
> correctly and doesn't over-fire.

### 2.2 — MCP Server Conflict Detection

**Duplicate tool providers:**

Parse the `mcpServers` configuration from both user-level and project-level
settings files. Flag if:
- Two servers provide the same category of tools (e.g., two different database
  introspection servers both configured)
- The same server is configured at both user and project level with different
  parameters (which config wins is ambiguous)

Mitigation:
> 🔧 **Same-category duplicates:** Determine which server is better suited for
> the current project (check documentation, features, token footprint). Remove
> the other from the active settings file:
> Edit `.claude/settings.json` → delete the redundant entry from `mcpServers`.
>
> 🔧 **Same server at both levels with different configs:** Decide which scope
> is correct. If project-specific config is intentional, remove the user-level
> entry to eliminate ambiguity. If user-level is the canonical config, remove
> the project-level override.

**Missing environment variables:**

```bash
# For each MCP server config, check if referenced env vars are set
cat ~/.claude/settings.json .claude/settings.json 2>/dev/null | \
  grep -oP '\$\{(\w+)\}' | tr -d '${}\n' | sort -u | while read var; do
    [ -z "${!var}" ] && echo "MISSING ENV: $var (required by MCP server)"
  done
```

A configured MCP server with a missing environment variable will either fail
silently or throw errors that consume context window with useless stack traces.

Mitigation:
> 🔧 **If the variable should be set:** Add it to your shell profile
> (`~/.bashrc`, `~/.zshrc`) or `.env` file:
> `export VARIABLE_NAME="your-value-here"`
> Then reload: `source ~/.bashrc` (or restart the terminal).
>
> 🔧 **If the server is no longer needed:** Remove the server entry from
> `.claude/settings.json` entirely rather than leaving a broken config in place.
>
> 🔧 **If the API key needs to be obtained:** Note which service the server
> connects to and direct the user to the provider's API key page. Do NOT
> offer to generate or guess credentials.

**Stale server configurations:**

Flag MCP server entries where the referenced npm package or command doesn't
exist locally:

```bash
# Check if MCP server commands are available
cat ~/.claude/settings.json .claude/settings.json 2>/dev/null | \
  python3 -c "
import json, sys, shutil
for line in sys.stdin:
    try:
        cfg = json.loads(line) if '{' in line else {}
        for name, srv in cfg.get('mcpServers', {}).items():
            cmd = srv.get('command', '')
            if cmd and not shutil.which(cmd):
                print(f'STALE SERVER: {name} → command \"{cmd}\" not found')
    except: pass
" 2>/dev/null
```

Mitigation:
> 🔧 **If the server is still needed:** Install the missing command:
> `npm install -g [package-name]` or `npx -y [package-name]` (if the config
> uses npx, the package will auto-install on first use — verify with a test run).
>
> 🔧 **If the server is no longer needed:** Remove the stale entry from
> `.claude/settings.json` to prevent silent errors and wasted context tokens.

### 2.3 — Configuration Consistency Check

**CLAUDE.md contradictions:**

If multiple CLAUDE.md files exist (root + subdirectory), read both and flag:
- Contradictory technology mandates (root says "use Zustand," subdirectory says "use Redux")
- Contradictory anti-patterns (root says "no ORMs," subdirectory uses Prisma patterns)
- Contradictory verification commands (different test runners specified at different levels)

Subdirectory CLAUDE.md files should *add specificity*, not contradict the root.

Mitigation:
> 🔧 **If the subdirectory context is intentionally different** (e.g., a monorepo
> where different packages use different state management): Add an explicit
> scope note to both files explaining the boundary. Example — root CLAUDE.md:
> "Note: packages/legacy uses Redux. See packages/legacy/CLAUDE.md for that context."
>
> 🔧 **If the contradiction is unintentional drift:** Update the stale file to
> match the current architectural decisions. Determine which file reflects the
> actual codebase by checking the code — the source of truth is always what's
> implemented, not what's documented.
>
> 🔧 **If neither file is clearly authoritative:** Consolidate into the root
> CLAUDE.md and delete the subdirectory file. One source of truth is better
> than two conflicting ones.

**Hook integrity:**

```bash
# Check that configured hook scripts actually exist
cat .claude/settings.json 2>/dev/null | \
  grep -oP '"command":\s*"[^"]*"' | sed 's/"command":\s*"//' | sed 's/"//' | \
  while read cmd; do
    script=$(echo "$cmd" | awk '{print $NF}')
    [ -f "$script" ] || echo "BROKEN HOOK: $cmd → script not found"
  done
```

Mitigation:
> 🔧 **If the hook script was deleted or moved:** Either restore/recreate the
> script at the expected path (see `references/hook-templates.md` for templates),
> or remove the hook entry from `.claude/settings.json` → `hooks` section.
>
> 🔧 **If the hook was from an old project and doesn't apply here:** Remove the
> hook entry from the project-level `.claude/settings.json`. Leaving broken hooks
> configured causes silent failures that the agent may interpret as permission
> denials, leading to confused workaround attempts.

**Redundant ignore rules:**

Flag if `.claudeignore` and `permissions.deny` both target the same path.
Not harmful, but signals configuration drift — the operator may not realize
one layer is redundant.

Mitigation:
> 🔧 Keep the `permissions.deny` entry (hard block) and remove the matching
> line from `.claudeignore` (soft filter). The hard block is the stronger
> protection — the soft filter is redundant beneath it. This simplifies
> maintenance and makes the security model easier to reason about.

### 2.4 — Health Report Format

Present the diagnostic results before proceeding to gap analysis. Every line
item includes the issue, its severity, and the specific remediation action.

```
## Environment Health Check

### 🔴 Critical Issues (resolve before proceeding)

1. **[Issue type]: [specific finding]**
   Impact: [what goes wrong if this isn't fixed]
   Fix: [exact remediation command or action]

2. **[Issue type]: [specific finding]**
   Impact: [what goes wrong]
   Fix: [exact action]

### 🟡 Warnings (recommend addressing)

1. **[Issue type]: [specific finding]**
   Risk: [what could go wrong]
   Suggested fix: [recommended action]

### 🟢 Clean
- Skills: [no conflicts detected | N skills, no overlaps]
- MCP servers: [all configured servers healthy | N servers verified]
- Configuration: [CLAUDE.md consistent | hooks intact | ignore rules clean]
```

**After presenting the report, offer to execute the mitigations:**
> "I found [N] issues. Want me to fix them now, or would you prefer to
> review and handle them yourself?"

For approved mitigations, execute the fix, verify the result, and update the
health report to reflect the resolved state before proceeding.

**Proficiency-calibrated presentation:**

- L3: Show the report as-is — issue, impact, fix command. They'll execute or
  direct you to execute.
- L2: Show the report with a brief explanation of each issue's impact on
  session quality. Offer to execute the fixes one at a time with confirmation.
- L1: Walk through each issue conversationally before showing the report:
  "I found a couple of things we should clean up before we start. The first
  is [plain-language description of the problem]. If we leave it as-is,
  [plain-language consequence]. The fix is straightforward — [describe the
  action in non-technical terms]. Want me to handle that?"

**Critical issues block the intake from proceeding to gap analysis.** The user
must resolve them (or explicitly accept the risk) before the protocol recommends
adding more tools. Warnings are informational and don't block progress.

---

## Step 3: Identify Gaps

Based on the project's stack (from Phase 2) and the environment audit, identify
categories where tooling would provide concrete value:

| Project Need | Gap Indicator | Search Category |
|-------------|---------------|-----------------|
| Frontend UI development | No design/component skills installed | UI/component generation skills |
| Database interactions | No DB MCP server configured | Database MCP servers |
| External API consumption | No documentation search MCP | Documentation/search MCP servers |
| Testing and QA | No testing tools or skills | Testing automation skills |
| Deployment | No CI/CD or deployment skills | Deployment/DevOps skills |
| Version control integration | No GitHub/Git MCP | Version control MCP servers |
| Code quality | No formatting/linting hooks | Quality assurance skills |

Only search for categories where a genuine gap exists AND the project would
benefit from filling it. A lightweight script project does not need a
deployment skill.

---

## Step 4: GitHub Search Protocol

For each identified gap, search GitHub using the following approach.

### Search Queries

Construct targeted queries. The GitHub search API responds best to specific terms:

```bash
# For Claude Code skills
curl -s "https://api.github.com/search/repositories?q=claude+code+skill+[CATEGORY]+in:name,description,readme&sort=stars&order=desc&per_page=10"

# For MCP servers
curl -s "https://api.github.com/search/repositories?q=mcp+server+[TECHNOLOGY]+in:name,description,readme&sort=stars&order=desc&per_page=10"

# Alternative: search the awesome-mcp-servers list
curl -s "https://api.github.com/search/repositories?q=awesome+mcp+servers&sort=stars&order=desc&per_page=5"
```

Replace `[CATEGORY]` and `[TECHNOLOGY]` with terms from the gap analysis.

### Search Fallback

If the GitHub API is unavailable (rate limits, network restrictions), fall back
to known community aggregation points:

- `https://github.com/punkpeye/awesome-mcp-servers` — curated MCP server list
- `https://github.com/anthropics/claude-code` — official Claude Code repo
- `https://skills.sh` — community skill marketplace (if available)
- `https://mcp.so` — MCP server directory (if available)
- `https://glama.ai/mcp/servers` — MCP server index (if available)

Use `curl` to fetch README content from these repos and extract relevant entries.

---

## Step 5: Quality Gate — Tiered Filtering

Every search result is evaluated against two tiers of quality criteria.
A result must pass ALL criteria in a tier to qualify for that tier.

### Tier 1: Recommended (high confidence)

These are presented as "I'd recommend installing this for your project."

ALL of the following must be true:
- **Recency:** Last commit within 60 days
- **Adoption:** 50+ stars
- **Maintenance:** Open-to-closed issue ratio ≤ 3:1 (maintainer engages with issues), OR fewer than 5 total issues (too new to judge, but other signals are strong)
- **Hygiene:** Has a LICENSE file
- **Documentation:** README includes installation instructions AND usage examples
- **Relevance:** Description or README explicitly references the user's stack or use case

### Tier 2: Explore (worth investigating)

These are presented as "You might also want to look at these."

ALL of the following must be true:
- **Recency:** Last commit within 6 months
- **Adoption:** 10+ stars
- **Hygiene:** Has a README with at least a description of what it does
- **Relevance:** Description or README is related to the user's stack or use case

### Rejected (not shown to the user)

Any result that fails Tier 2 criteria is silently discarded. Do not present
abandoned, undocumented, or irrelevant results under any circumstances.

### Extracting Quality Signals from GitHub API

The GitHub search API returns these relevant fields per result:

```
stargazers_count        → star count
pushed_at               → last commit/push date
open_issues_count       → open issues (note: includes PRs)
license                 → license info (null = no license file)
description             → repo description
html_url                → link to repo
```

For deeper evaluation (issue ratio, README content), make follow-up API calls:

```bash
# Get closed issues count for ratio calculation
curl -s "https://api.github.com/repos/OWNER/REPO/issues?state=closed&per_page=1" \
  -I | grep -i 'link:' # Parse the last page number for total count

# Get README content
curl -s "https://api.github.com/repos/OWNER/REPO/readme" \
  -H "Accept: application/vnd.github.raw" | head -100
```

**Rate limit awareness:** The GitHub API allows 60 unauthenticated requests/hour.
A typical scan uses 5-15 requests. If the user has a `GITHUB_TOKEN` environment
variable set, use it for authenticated requests (5,000/hour):

```bash
# Check for available token
if [ -n "$GITHUB_TOKEN" ]; then
  AUTH_HEADER="-H \"Authorization: Bearer $GITHUB_TOKEN\""
else
  AUTH_HEADER=""
  echo "NOTE: Using unauthenticated GitHub API (60 req/hr limit)"
fi
```

---

## Step 6: Presentation Format

Present results to the user in a clear, scannable format. Group by category,
separate tiers visually, and include enough information for the user to make
an informed decision without clicking through to every repo.

### Presentation Template

```
## Tooling Recommendations for [Project Name]

Based on your project's stack ([stack summary]) and current environment,
here are tools that could strengthen your setup.

### Recommended (high confidence)

**[Category: e.g., Database Integration]**

📦 **[repo-name]** — [one-line description from README]
   ⭐ [star count] | Last updated: [date] | License: [license]
   Why: [1 sentence explaining why this is relevant to THEIR project]
   Install: `[install command if available]`
   → [repo URL]

**[Category: e.g., Documentation Search]**

📦 **[repo-name]** — [one-line description]
   ⭐ [star count] | Last updated: [date] | License: [license]
   Why: [relevance to their project]
   Install: `[install command]`
   → [repo URL]

---

### Worth Exploring (investigate before installing)

📎 **[repo-name]** — [one-line description]
   ⭐ [star count] | Last updated: [date]
   Note: [caveat — e.g., "Smaller community, but actively maintained"]
   → [repo URL]

📎 **[repo-name]** — [one-line description]
   ⭐ [star count] | Last updated: [date]
   Note: [caveat]
   → [repo URL]

---

Would you like me to install any of these? I'll configure them in your
.claude/settings.json and verify they're working.
```

### Proficiency-Calibrated Presentation

- **L3 (Practitioner):** Show the table, skip the explanations of what MCP
  servers or skills are. Trust them to evaluate the repos themselves.

- **L2 (Builder):** Show the table with the "Why" explanations. Briefly note
  what skills and MCP servers are if they haven't encountered them before.

- **L1 (Explorer):** Before showing recommendations, explain the concept:
  "Skills are like specialized instruction manuals I can load to handle
  specific tasks better. MCP servers are connections to external tools —
  like giving me the ability to look at your database structure directly
  instead of you having to describe it to me. Here's what I'd recommend
  adding for your project, and why each one helps."

---

## Step 7: Installation Assistance

If the user approves a recommendation, handle installation:

### For MCP Servers

Add the server configuration to `.claude/settings.json`:

```bash
# Read existing settings (or create new)
SETTINGS_FILE=".claude/settings.json"
mkdir -p .claude

# The Skill should generate the appropriate JSON configuration
# based on the server's documented setup, then write or merge
# it into the settings file.
```

After installation, run `/context` to verify the MCP server is loaded and
measure its token impact. Report the impact to the user.

### For Skills

```bash
# If a .skill package file is available
claude install-skill [path-or-url]

# If it's a raw GitHub repo with SKILL.md structure
git clone [repo-url] .claude/skills/[skill-name]
```

After installation, verify the skill is detected by checking that it appears
in the available skills list during the next prompt.

### Post-Installation Verification

After any installation:
1. Confirm the tool is detected and functional
2. Report context window impact (for MCP servers)
3. Note any required environment variables or API keys the user needs to set
4. Update the intake summary to reflect the new tooling

---

## When NOT to Recommend

Do not run the ecosystem scan if:

- The project is Lightweight tier (overhead exceeds benefit)
- The environment is already well-equipped for the project's needs
- The user explicitly says they don't want additional tooling
- Network access is unavailable and the curated catalog doesn't cover the gap
- The project is a quick experiment where setup time would exceed coding time

When in doubt, ask: "I can scan for skills and MCP servers that might help
with this project. Want me to do that, or would you rather keep the setup lean?"
