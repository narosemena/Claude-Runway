# MCP Server Catalog & Configuration Guide

Reference for selecting and configuring Model Context Protocol servers
based on project needs. Only connect servers required for the current session.

---

## Selection Principle

Every active MCP server injects tool definitions into your context window.
More servers = more token consumption = less room for your actual work.

**Rule of thumb:** If you won't use it in the next 30 minutes, don't connect it.

Audit current token usage after connecting servers: `/context`

---

## Server Categories

### Version Control & Issue Tracking

| Server | Connect When | What It Provides |
|--------|-------------|-----------------|
| GitHub MCP | Working with GitHub repos, PRs, issues | Read repo history, PR discussions, CI logs, issue context |
| Linear | Project uses Linear for issue tracking | Fetch issue details, project status, sprint context |
| Sentry | Debugging production errors | Stack traces, error frequency, affected users |

### Database Architecture

| Server | Connect When | What It Provides |
|--------|-------------|-----------------|
| PostgreSQL MCP | Project uses Postgres | Live schema introspection, relationship mapping, query validation |
| Supabase MCP | Project uses Supabase | Auth config, table schemas, RLS policies, realtime setup |
| MongoDB MCP | Project uses MongoDB | Collection schemas, index definitions, aggregation validation |

### Documentation & Search

| Server | Connect When | What It Provides |
|--------|-------------|-----------------|
| Context7 | Using unfamiliar libraries/APIs | Live, version-specific documentation retrieval |
| Exa | Need semantic search across web sources | High-quality search results for technical questions |
| Firecrawl | Need to ingest web documentation | Markdown extraction from web pages for context injection |

### Design & Testing

| Server | Connect When | What It Provides |
|--------|-------------|-----------------|
| Figma MCP | Translating designs to code | Design tokens, component specs, styling variables |
| Playwright MCP | End-to-end browser testing | Automated browser validation, screenshot comparison |

---

## Configuration in .claude/settings.json

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    }
  }
}
```

---

## Context Window Impact Warning

Typical token consumption per active MCP server:

- Lightweight servers (1-3 tools): ~500-1,000 tokens
- Medium servers (5-10 tools): ~2,000-4,000 tokens
- Heavy servers (15+ tools): ~5,000-10,000+ tokens

With 5 heavy MCP servers connected, you can lose 25,000-50,000 tokens of
context window before the session even starts. This directly degrades
the agent's ability to follow your CLAUDE.md instructions and maintain
coherent reasoning across file edits.

**Mitigation strategies:**
1. Connect only what you need for the current task
2. Disconnect servers when you shift to a different task type
3. Audit regularly with `/context`
4. Prefer lightweight, focused servers over monolithic ones
