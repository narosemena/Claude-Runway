# Hook Implementation Templates

Reference templates for Claude Code pre-execution and post-execution hooks.
These are scaffolds — adapt to your project's specific tooling.

---

## Pre-Execution Safety Hook (Node.js)

Place at: `.claude/hooks/pre-tool-use.js`

This hook intercepts shell commands before execution and blocks destructive operations.

```javascript
#!/usr/bin/env node

const fs = require('fs');

// Read the agent's intended action from stdin
let input = '';
process.stdin.on('data', chunk => input += chunk);
process.stdin.on('end', () => {
  const payload = JSON.parse(input);
  const command = payload?.tool_input?.command || '';

  const blocked = [
    // Destructive filesystem operations
    { pattern: /rm\s+-rf\s+[\/~]/, reason: 'Blocked: rm -rf targeting root or home directory' },
    { pattern: /rm\s+-rf\s+\.(?!\/)/, reason: 'Blocked: rm -rf targeting current directory' },
    // Force-pushing to protected branches
    { pattern: /git\s+push\s+.*--force.*\s+(main|master|production)/, reason: 'Blocked: force push to protected branch' },
    { pattern: /git\s+push\s+-f\s+.*\s+(main|master|production)/, reason: 'Blocked: force push to protected branch' },
    // Destructive database operations without WHERE clause
    { pattern: /DROP\s+(TABLE|DATABASE)/i, reason: 'Blocked: DROP TABLE/DATABASE command' },
    { pattern: /DELETE\s+FROM\s+\w+\s*;/i, reason: 'Blocked: DELETE without WHERE clause' },
    // Fork bombs and resource exhaustion
    { pattern: /:\(\)\s*\{/, reason: 'Blocked: potential fork bomb' },
  ];

  for (const rule of blocked) {
    if (rule.pattern.test(command)) {
      // Exit code 2 = block the tool execution
      console.error(JSON.stringify({
        error: rule.reason,
        suggestion: 'Use a safer alternative or ask the user for explicit permission.'
      }));
      process.exit(2);
    }
  }

  // Exit code 0 = allow the tool execution
  process.exit(0);
});
```

---

## Post-Execution Formatting Hook (Node.js)

Place at: `.claude/hooks/post-tool-use.js`

This hook auto-formats files after the agent writes or edits them.

```javascript
#!/usr/bin/env node

const { execSync } = require('child_process');
const path = require('path');

let input = '';
process.stdin.on('data', chunk => input += chunk);
process.stdin.on('end', () => {
  const payload = JSON.parse(input);

  // Extract the modified file path from the agent's action
  const filePath = payload?.tool_input?.file_path
    || payload?.tool_input?.path
    || null;

  if (!filePath) {
    process.exit(0); // No file path = nothing to format
  }

  const ext = path.extname(filePath).toLowerCase();

  try {
    // JavaScript/TypeScript: Prettier + ESLint
    if (['.js', '.jsx', '.ts', '.tsx', '.css', '.json', '.md'].includes(ext)) {
      execSync(`npx prettier --write "${filePath}"`, { stdio: 'pipe' });
      if (['.js', '.jsx', '.ts', '.tsx'].includes(ext)) {
        execSync(`npx eslint --fix "${filePath}"`, { stdio: 'pipe' });
      }
    }

    // Python: Black + isort
    if (ext === '.py') {
      execSync(`python -m black "${filePath}"`, { stdio: 'pipe' });
      execSync(`python -m isort "${filePath}"`, { stdio: 'pipe' });
    }

    // Rust: rustfmt
    if (ext === '.rs') {
      execSync(`rustfmt "${filePath}"`, { stdio: 'pipe' });
    }

  } catch (err) {
    // Feed formatting errors back to the agent
    console.error(JSON.stringify({
      warning: `Formatting issue in ${filePath}`,
      details: err.stderr?.toString() || err.message
    }));
    // Exit 0 — don't block execution, just report
    process.exit(0);
  }

  process.exit(0);
});
```

---

## Registering Hooks in .claude/settings.json

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "command": "node .claude/hooks/pre-tool-use.js",
        "description": "Block destructive shell commands"
      }
    ],
    "PostToolUse": [
      {
        "command": "node .claude/hooks/post-tool-use.js",
        "description": "Auto-format modified files"
      }
    ]
  }
}
```

---

## Design Decisions

**Why Node.js over Python for hooks?**
Node.js has faster cold-start than Python for rapid intercept events. Since hooks fire on
every tool use, startup latency matters. Use Python if your project is Python-only and
you don't want to add a Node dependency.

**Why post-format on every write instead of at commit time?**
Formatting at commit time means the agent sees unformatted code in its context window
during the session, which can cause it to learn and repeat bad formatting patterns.
Formatting immediately keeps the agent's view of the codebase consistent with standards.

**When to block vs. warn:**
Pre-execution hooks should hard-block (exit 2) for genuinely destructive operations.
Post-execution hooks should warn (exit 0 with stderr) for formatting issues — let the
agent self-correct rather than halting its reasoning chain.
