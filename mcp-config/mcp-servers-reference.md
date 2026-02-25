# MCP Servers Reference

## Metadata
- Last updated: 2026-02-25
- Version: 1.0

## What MCP Servers Are

MCP (Model Context Protocol) servers give Claude Code additional capabilities — like
reading Figma files, querying Notion, searching the web, or accessing documentation.
They run alongside Claude Code and provide tools that can be called during conversations.

## Available MCP Servers

| Server | What It Does | When to Use |
|--------|-------------|------------|
| **Figma** | Read designs from Figma, extract design context | When Scott provides a Figma URL or mentions Figma wireframes |
| **Notion** | Read/write Notion databases and pages | When working with Background Ops, ideas, or Notion-based tracking |
| **Playwright** | Browser automation, screenshots, testing | When testing web apps or taking screenshots for design review |
| **Context7** | Up-to-date documentation for any library | When you need current docs (SurrealDB, Nuxt, Tailwind, etc.) |
| **n8n MCP** | Search/validate n8n workflows and nodes | When building or debugging n8n workflows |
| **Exa** | Advanced web search and research | When you need deep web research beyond basic search |
| **Memory** | Persistent knowledge graph | When storing/retrieving structured information across sessions |
| **Sequential Thinking** | Structured reasoning | When working through complex multi-step problems |
| **Gamma** | AI presentations and documents | When Scott needs to create presentations or docs |
| **Canva** | Design creation and editing | When Scott needs marketing materials or designs |
| **Workspace (Google)** | Gmail, Drive, Docs, Sheets, Calendar | When working with Google Workspace tools |
| **Firebase** | Firebase project management | When working with Firebase projects |

## Context7 Library IDs (for Quick Lookups)

These IDs let you look up docs instantly without searching:

| Library | Context7 ID |
|---------|------------|
| SurrealDB | `/surrealdb/docs.surrealdb.com` |
| Nuxt UI v4 | `/llmstxt/ui4_nuxt_llms_txt` |
| Tailwind CSS v4 | `/websites/tailwindcss` |

### Usage example:
```
1. Call resolve-library-id with the library name
2. Call query-docs with the resolved ID and your question
```

## When to Use Each Server

### Figma MCP
- Scott shares a Figma URL → use `get_design_context` to extract code and screenshots
- Design Proof phase → read wireframes and extract layout patterns
- Always adapt Figma output to Nuxt UI components (don't copy raw HTML)

### Context7
- Need current API docs → resolve the library ID, then query
- Checking if a SurrealQL function exists in v3 → query SurrealDB docs
- Verifying Nuxt UI component props → query Nuxt UI docs

### Notion
- Background Ops workflow → read/write to the Notion system
- Idea capture → add to the Notion inbox
- Weekly review data → query Notion databases

### n8n MCP
- Building a workflow → search for nodes and templates
- Validating workflow JSON → use the validate tool
- Finding the right node for a task → search nodes

### Playwright
- Testing a web app → navigate and take screenshots
- Visual regression → compare screenshots
- Form testing → fill forms and verify behavior

## Servers NOT to Use Without Asking

Some servers can take actions that are visible to others:
- **Workspace (Google):** Sends emails, creates calendar events, modifies documents
- **Notion:** Can create/modify pages and databases
- **Canva:** Creates designs

Always confirm with Scott before taking actions through these servers.
