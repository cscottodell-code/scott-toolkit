# MCP Server Setup Guide

## Metadata
- Last updated: 2026-02-25
- Version: 1.0

## Overview

This guide covers how to set up MCP servers on a new machine. MCP servers are
configured per-machine (not shared via the toolkit repo) because they often
require local API keys and paths.

## Where MCP Config Lives

MCP server configurations are stored in Claude Code's settings:
- Global: `~/.claude/` directory
- Project-specific: `.claude/` in the project root

The actual server configurations may be in:
- `~/.claude/claude_desktop_config.json` (for Claude Desktop)
- Claude Code's built-in settings (for CLI)

## Setup Checklist for a New Machine

### 1. Core MCP Servers (Set up first)

These are used in almost every project:

- [ ] **Context7** — Documentation lookup
  - No API key needed
  - Used for looking up current docs for SurrealDB, Nuxt, Tailwind, etc.

- [ ] **Sequential Thinking** — Structured reasoning
  - No API key needed
  - Helps with complex problem solving

### 2. Design & Development Servers

- [ ] **Figma** — Design file reading
  - Requires: Figma account connected via Claude.ai
  - Used during Design Proof phase

- [ ] **Playwright** — Browser automation
  - Install: `npm install -g @anthropic-ai/mcp-playwright`
  - Used for testing web apps

### 3. Productivity Servers

- [ ] **Notion** — Notion integration
  - Requires: Notion API key
  - Used for Background Ops, idea tracking

- [ ] **Workspace (Google)** — Google services
  - Requires: Google OAuth setup
  - Used for email, calendar, docs

### 4. Specialized Servers

- [ ] **n8n MCP** — n8n workflow tools
  - May require n8n instance credentials
  - Used when building/debugging workflows

- [ ] **Exa** — Advanced web search
  - May require API key
  - Used for deep research

- [ ] **Memory** — Knowledge graph
  - No API key needed
  - Persistent memory across sessions

## Verifying Setup

After setting up servers, verify they're working:

1. Start a new Claude Code session
2. Ask Claude Code to list available MCP tools
3. Test each server with a simple operation:
   - Context7: "Look up the SurrealDB CREATE syntax"
   - Figma: "Can you read Figma files?" (should confirm access)
   - Notion: "Search my Notion workspace" (should return results)

## Troubleshooting

### Server not appearing in tools list
- Check that the server configuration is in the right config file
- Restart Claude Code after adding/changing server configs
- Check server logs for startup errors

### Authentication errors
- API keys may have expired — regenerate and update config
- OAuth tokens may need refreshing — re-authenticate

### Server crashing on startup
- Check that all dependencies are installed (Node.js, npm packages)
- Check that the server version is compatible with your Claude Code version
- Look at the server's stderr output for error messages

## Machine Sync Note

MCP configs are NOT synced between machines because:
1. API keys are machine-specific (security)
2. File paths may differ between machines
3. Some servers need local dependencies installed

When setting up a new machine, use this guide to configure each server from scratch.
Don't copy config files between machines — set up fresh to avoid stale credentials.
