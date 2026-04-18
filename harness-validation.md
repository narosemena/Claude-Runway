# Harness Validation — Cross-Artifact Integrity Checks

The harness layer validates that all artifacts generated during intake function
as an integrated system. Individual artifacts can be internally correct but
incompatible with each other. The harness catches those connection failures
before the coding session begins.

**When to load:** After the last active phase for the project's tier (Phase 2
for Lightweight, Phase 4 for Standard, Phase 5 for Complex) and before Handover.

---

## Core Principle

Artifacts don't exist in isolation. A CLAUDE.md references test commands that
depend on installed packages. Hooks invoke formatters that must match the stack.
Ignore rules exclude paths that other artifacts reference. MCP servers require
environment variables that must be set.

The harness models these as a dependency graph and validates every edge.

---

## Connection Matrix

Every pair of artifacts has potential connection points. The harness checks each:

### CLAUDE.md ↔ Installed Packages

| Check | What Could Go Wrong | Validation |
|-------|-------------------|------------|
| Verification commands reference installed tools | `pnpm test` in CLAUDE.md but pnpm isn't installed | Run `command -v [tool]` for each command referenced in Verification section |
| Testing framework exists | CLAUDE.md says "Vitest" but vitest isn't in package.json | Check package.json devDependencies (or pip list, Cargo.toml, etc.) |
| Linter/formatter exists | CLAUDE.md references `eslint --fix` but eslint isn't configured | Check for config files (.eslintrc, .prettierrc, pyproject.toml) |
| Stack versions match reality | CLAUDE.md says "Node 20" but `node -v` returns 18 | Run version commands and compare |

```bash
# Extract verification commands from CLAUDE.md
grep -A 20 "## Verification" CLAUDE.md | grep '`' | tr -d '`-' | \
  awk '{print $1}' | sort -u | while read cmd; do
    command -v "$cmd" >/dev/null 2>&1 || echo "HARNESS FAIL: '$cmd' in CLAUDE.md Verification but not installed"
  done
```

### CLAUDE.md ↔ .claudeignore

| Check | What Could Go Wrong | Validation |
|-------|-------------------|------------|
| Progressive disclosure paths not ignored | CLAUDE.md says "read @docs/schema.md" but docs/ is in .claudeignore | Extract @-references from CLAUDE.md, check against .claudeignore patterns |
| Architecture references accessible | CLAUDE.md describes files in src/components/ but src/ is ignored | Extract directory references from Architecture section, verify not ignored |

```bash
# Extract @docs/ references from CLAUDE.md
grep -oP '@\S+' CLAUDE.md | sed 's/@//' | while read ref; do
  # Check if any .claudeignore pattern matches this path
  if [ -f .claudeignore ]; then
    dir=$(dirname "$ref")
    grep -q "^${dir}" .claudeignore && \
      echo "HARNESS FAIL: CLAUDE.md references '$ref' but '$dir' is in .claudeignore"
  fi
done
```

### CLAUDE.md ↔ permissions.deny

| Check | What Could Go Wrong | Validation |
|-------|-------------------|------------|
| Referenced files not hard-blocked | CLAUDE.md tells agent to read a file that permissions.deny blocks | Extract file references from CLAUDE.md, check against deny patterns |
| Terminology map targets accessible | Terminology says "widget → src/models/Widget.ts" but that path is blocked | Verify each terminology target path |

### Hooks ↔ Stack

| Check | What Could Go Wrong | Validation |
|-------|-------------------|------------|
| Hook runtime exists | Hook script is Node.js but node isn't installed | Check `command -v node` or `command -v python3` based on hook shebang |
| Formatter in post-hook exists | Post-hook runs `npx prettier` but prettier isn't in devDependencies | Check package.json or global installs |
| Linter in post-hook matches stack | Post-hook runs `eslint` on a Python project | Compare hook file extensions against project stack |
| Hook scripts are executable | Hook file exists but lacks execute permission | Check `[ -x "$hook_script" ]` |

```bash
# Validate post-execution hook formatter availability
if [ -f .claude/hooks/post-tool-use.js ]; then
  # Extract tool names from the hook
  grep -oP "execSync\(\`[^`]*\`" .claude/hooks/post-tool-use.js | \
    grep -oP '(prettier|eslint|black|isort|rustfmt|ruff)' | sort -u | while read tool; do
      command -v "$tool" >/dev/null 2>&1 && \
        npx "$tool" --version >/dev/null 2>&1 || \
        echo "HARNESS FAIL: Post-hook references '$tool' but it's not available"
    done
fi
```

### MCP Servers ↔ Environment

| Check | What Could Go Wrong | Validation |
|-------|-------------------|------------|
| Required env vars are set | GitHub MCP needs GITHUB_TOKEN but it's not in the environment | Parse mcpServers config, extract ${VAR} references, verify each |
| Server commands are installed | MCP server uses `npx` but npm isn't installed | Check command availability |
| Server isn't redundant with hooks | Both an MCP server and a hook try to manage the same concern | Flag if database MCP and a database hook both exist |

### .claudeignore ↔ permissions.deny

| Check | What Could Go Wrong | Validation |
|-------|-------------------|------------|
| No contradictory intent | Same path soft-filtered AND hard-blocked | Flag redundancy — keep hard-block, suggest removing soft-filter |

### Handover Brief ↔ CLAUDE.md

| Check | What Could Go Wrong | Validation |
|-------|-------------------|------------|
| First action is achievable | Handover says "run the test suite" but CLAUDE.md shows no test framework configured | Cross-reference first action against available verification commands |
| Stack references match | Handover says "Next.js" but CLAUDE.md says "Vite + React" | Compare stack descriptions (should be impossible if both generated from same intake, but catches manual edits) |

---

## Validation Execution

Run all applicable checks as a batch after the last active phase completes.

### Execution Script

```bash
#!/usr/bin/env bash
# === Harness Validation ===

FAILURES=0
WARNINGS=0

echo "Running harness validation..."

# --- CLAUDE.md ↔ Packages ---
if [ -f CLAUDE.md ]; then
  # Check verification commands
  grep -A 20 "## Verification" CLAUDE.md 2>/dev/null | \
    grep -oP '`[a-zA-Z][a-zA-Z0-9_-]*' | tr -d '`' | sort -u | while read cmd; do
      if ! command -v "$cmd" >/dev/null 2>&1; then
        echo "❌ CLAUDE.md verification references '$cmd' — not installed"
        ((FAILURES++))
      fi
    done

  # Check @docs/ references against .claudeignore
  if [ -f .claudeignore ]; then
    grep -oP '@[a-zA-Z0-9_./-]+' CLAUDE.md 2>/dev/null | sed 's/@//' | while read ref; do
      dir=$(echo "$ref" | cut -d'/' -f1)
      if grep -q "^${dir}" .claudeignore 2>/dev/null; then
        echo "❌ CLAUDE.md references '$ref' but '$dir/' is in .claudeignore"
        ((FAILURES++))
      fi
    done
  fi
fi

# --- Hooks ↔ Stack ---
for hook in .claude/hooks/*; do
  [ -f "$hook" ] || continue
  if [ ! -x "$hook" ] && head -1 "$hook" | grep -q "^#!"; then
    echo "⚠️  Hook '$hook' is not executable (missing chmod +x)"
    ((WARNINGS++))
  fi
  # Check shebang runtime
  runtime=$(head -1 "$hook" | grep -oP '(node|python3?|bash)')
  if [ -n "$runtime" ] && ! command -v "$runtime" >/dev/null 2>&1; then
    echo "❌ Hook '$hook' requires '$runtime' — not installed"
    ((FAILURES++))
  fi
done

# --- MCP Servers ↔ Environment ---
if [ -f .claude/settings.json ]; then
  grep -oP '\$\{(\w+)\}' .claude/settings.json 2>/dev/null | tr -d '${}' | sort -u | while read var; do
    if [ -z "${!var}" ]; then
      echo "❌ MCP server requires \$$var — not set in environment"
      ((FAILURES++))
    fi
  done
fi

# --- Summary ---
echo ""
if [ $FAILURES -eq 0 ] && [ $WARNINGS -eq 0 ]; then
  echo "✅ Harness validation passed — all artifacts are connected"
else
  echo "Harness validation complete: $FAILURES failure(s), $WARNINGS warning(s)"
fi
```

---

## Harness Report Format

Present results before handover. Every failure must be resolved or explicitly
accepted before the protocol transitions to execution.

```
## Harness Validation

### ❌ Failures (must resolve before coding)

1. **CLAUDE.md → Package**: Verification command `pnpm test` references
   pnpm, but pnpm is not installed.
   Fix: `npm install -g pnpm` or change verification command to `npm test`

2. **Hook → Formatter**: Post-execution hook runs `prettier` but prettier
   is not in devDependencies.
   Fix: `npm install -D prettier` or remove prettier from hook

### ⚠️ Warnings

1. **.claudeignore ↔ permissions.deny**: Path `.env` appears in both.
   Suggestion: Remove from .claudeignore — permissions.deny is sufficient.

### ✅ Passed
- CLAUDE.md references are accessible
- Hook runtimes are installed
- MCP server environment variables are set
- No contradictory artifact instructions detected
```

### Proficiency-Calibrated Presentation

- **L3:** Show the report. They'll fix it.
- **L2:** Show the report with brief explanations. Offer to fix each item.
- **L1:** Walk through each failure conversationally: "I found a small
  wiring issue — the configuration file tells me to run a testing tool
  called pnpm, but pnpm isn't installed on your system yet. I can either
  install it for you or adjust the configuration to use npm instead, which
  you already have. Which would you prefer?"

---

## When to Skip

- **Lightweight projects:** Only a CLAUDE.md was generated (or nothing at all).
  If there's only one artifact, there are no connections to validate. Skip.
- **Advisory environments** (Claude Desktop, web chat): Artifacts were presented
  as copy-paste text, not written to disk. Cannot validate programmatically.
  Instead, include a "Before you start" checklist in the handover summary noting
  what the user should verify manually.
- **User explicitly opts out:** Respect it, but note in the handover brief that
  harness validation was skipped.
