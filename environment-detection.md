# Environment Detection — Scripts and Capability Matrix

Detect the execution environment at the start of intake to determine which
protocol features are available. Load when Phase 0 detection runs.

---

## Detection Script

Run these checks in sequence. The first match wins.

```bash
# === Phase 0: Environment Detection ===

ENV_TYPE="unknown"
ENV_CAPABILITIES=""

# Check 1: Are we in Claude Code CLI or VS Code integrated terminal?
if [ -n "$CLAUDE_CODE" ] || command -v claude >/dev/null 2>&1; then
  # We're in Claude Code — now determine if VS Code is the host
  if [ -n "$VSCODE_PID" ] || [ -n "$TERM_PROGRAM" ] && [ "$TERM_PROGRAM" = "vscode" ]; then
    ENV_TYPE="claude-code-vscode"
  elif [ -n "$VSCODE_IPC_HOOK" ] || [ -n "$VSCODE_IPC_HOOK_CLI" ]; then
    ENV_TYPE="claude-code-vscode"
  else
    ENV_TYPE="claude-code-terminal"
  fi
fi

# Check 2: Can we access the local filesystem at a real project path?
if [ "$ENV_TYPE" = "unknown" ]; then
  if [ -d "/home/claude" ] && [ -d "/mnt/user-data" ]; then
    ENV_TYPE="claude-ai-container"
  fi
fi

# Check 3: Can we run bash at all?
if [ "$ENV_TYPE" = "unknown" ]; then
  if command -v bash >/dev/null 2>&1; then
    ENV_TYPE="claude-desktop"
  else
    ENV_TYPE="claude-web-chat"
  fi
fi

echo "DETECTED: $ENV_TYPE"
```

### Fallback: Contextual Inference

If the bash detection is inconclusive, infer from contextual signals available
in the system prompt or conversation:

- **Available tools include `bash_tool`, `create_file`, `str_replace`:** → Claude.ai with computer use (container environment)
- **Available tools include `bash_tool` AND user has a real project path (not /home/claude):** → Claude Code (CLI or VS Code)
- **No `bash_tool` available, but MCP tools present:** → Claude Desktop
- **No `bash_tool`, no MCP tools, only conversation:** → Claude web/mobile chat (protocol cannot execute, only advise)

---

## Capability Matrix

| Capability | Claude Code CLI | Claude Code VS Code | Claude.ai Container | Claude Desktop | Claude Web Chat |
|-----------|:-:|:-:|:-:|:-:|:-:|
| Read/write local filesystem | ✅ | ✅ | ⚠️ sandbox only | ⚠️ limited | ❌ |
| Run bash commands | ✅ | ✅ | ✅ container | ⚠️ via MCP | ❌ |
| Execute hooks | ✅ | ✅ | ❌ | ❌ | ❌ |
| Install npm/pip packages | ✅ | ✅ | ✅ container | ❌ | ❌ |
| Access git | ✅ | ✅ | ✅ container | ❌ | ❌ |
| Write CLAUDE.md to project | ✅ | ✅ | ❌ no project | ❌ | ❌ |
| Write handover brief | ✅ | ✅ | ⚠️ to container | ❌ | ❌ |
| Configure .claudeignore | ✅ | ✅ | ❌ | ❌ | ❌ |
| Configure permissions.deny | ✅ | ✅ | ❌ | ❌ | ❌ |
| Use /compact, /rewind, /clear | ✅ | ✅ | ❌ | ❌ | ❌ |
| Use Plan Mode | ✅ | ✅ | ❌ | ❌ | ❌ |
| MCP server connections | ✅ | ✅ | ❌ | ✅ | ❌ |
| Ecosystem scan (GitHub API) | ✅ | ✅ | ✅ | ⚠️ limited | ❌ |
| Image/screenshot input | ⚠️ complex | ✅ drag-drop | ✅ | ✅ | ✅ |
| Agent Teams | ✅ | ✅ | ❌ | ❌ | ❌ |

---

## Protocol Adaptation by Environment

### Claude Code CLI / Claude Code VS Code (Full Power)
All 6 phases available. No adaptations needed. This is the canonical environment.

### Claude.ai with Computer Use (Container)
The protocol can run but with significant constraints:
- **Phase 3 (Configuration):** CLAUDE.md, .claudeignore, and hooks are generated but
  live in the container — NOT in the user's actual project repo. The user must manually
  copy them to their local project. Flag this explicitly: "I've generated these files
  in my container, but they're not in your local project yet. You'll need to copy them."
- **Phase 4 (Session Strategy):** /compact, /rewind, /clear are not available. Session
  hygiene must be managed differently — shorter sessions, explicit summaries.
- **Phase 4.4 (Ecosystem Scan):** GitHub API search works. Installation commands won't
  affect the user's local environment — provide commands for them to run locally.
- **Phase 5 (Plan Mode):** Not available. Plans are written as markdown files in the
  container instead.
- **Phase 6 (Handover):** Handover brief is written to container. Present it as
  downloadable content or copy-paste text the user transfers to their local `.claude/` dir.

### Claude Desktop (Mac/Windows)
Severely limited for this protocol's purpose:
- **Phase 1-2 (Discovery):** Work fully — these are conversational.
- **Phase 3 (Configuration):** Cannot write files to the user's project. Generate all
  configuration content as copy-paste blocks with clear instructions on where to put them.
- **Phase 4-6:** Most features unavailable. The protocol functions as a guided advisor
  producing artifacts the user must manually create.
- **MCP:** Can configure MCP servers through Claude Desktop's settings, which is valuable.

### Claude Web/Mobile Chat (Advisory Only)
The protocol can only function as a conversational advisor:
- Run Phases 1-2 as normal conversation
- For Phase 3+, generate all artifacts as formatted text blocks the user copies into
  their project manually
- Cannot execute any commands, install any packages, or verify any configurations
- Explicitly state: "I can help you design your environment setup, but you'll need to
  create these files yourself. Here's exactly what to create and where to put it."

---

## Detection Output Format

After detection, present the environment context briefly before proceeding:

**Claude Code (CLI or VS Code) — state nothing.** This is the expected environment.
Proceed directly to Phase 1.

**Any other environment — state the constraints up front:**
> "I've detected that we're running in [environment]. This means [key limitation].
> I can still help you plan and design your project setup, but [specific adaptation].
> Want to proceed?"

This prevents the user from expecting features that aren't available and prevents
the protocol from attempting operations that will fail.
