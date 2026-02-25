# Scott's Toolkit

A shared repository of reusable templates, skills, references, and retrospectives for building apps with Claude Code. Lives at `~/scott-toolkit/` on all of Scott's machines. Every project references this toolkit.

## Who It's For

- **Scott O'Dell** - AI orchestrator building prototypes with Claude Code
- **Brett** - Co-developer on Bresco (may use this in the future)

## Quick Start

1. Clone the repo:
   ```bash
   git clone git@github.com:cscottodell-code/scott-toolkit.git ~/scott-toolkit
   ```

2. Verify the structure:
   ```bash
   ls ~/scott-toolkit/
   # Should see: templates/ skills/ references/ mcp-config/ retros/ README.md SETUP.md CHANGELOG.md
   ```

3. Point your project's `CLAUDE.md` to the toolkit:
   ```markdown
   ## External References
   Before starting work, read the relevant skills from ~/scott-toolkit/skills/
   Specific skills for this project:
   - ~/scott-toolkit/skills/reference/surrealdb-v3.md
   - ~/scott-toolkit/skills/reference/tauri-nuxt.md
   ```

That's it. The toolkit is NOT a submodule. It's a standalone repo at a fixed path. Update it once, every project benefits immediately.

## How to Use It

### Starting a New Project
Tell Claude Code: "Read ~/scott-toolkit/skills/workflows/new-project.md and walk me through it."

This will guide you through:
- Brain dump (capture what you want to build)
- Clarify & challenge (poke holes, find gaps)
- Draft a PRD (using the PRD template)
- Create the repo with proper structure
- Design proof (nail the look before building features)
- Build milestone 1

### Adding a Feature to an Existing Project
Tell Claude Code: "Read ~/scott-toolkit/skills/workflows/new-feature.md and walk me through it."

### Running a Retrospective
After finishing a project or milestone, tell Claude Code: "Read ~/scott-toolkit/skills/workflows/retro.md and walk me through it."

This captures lessons and feeds them back into the toolkit so the next project is better.

### Debugging
When stuck, tell Claude Code: "Read ~/scott-toolkit/skills/workflows/debug.md and walk me through it."

### Updating the Toolkit
After a retro or when you discover better patterns: "Read ~/scott-toolkit/skills/workflows/toolkit-update.md and walk me through it."

### Handing Off to Gary
When a prototype is ready for production: "Read ~/scott-toolkit/skills/workflows/handoff-to-gary.md and walk me through it."

### Resuming a Project After Time Away
Tell Claude Code: "Read ~/scott-toolkit/skills/workflows/resume-project.md and walk me through it."

## Repo Structure

```
scott-toolkit/
├── README.md              # This file
├── SETUP.md               # Environment setup guide
├── CHANGELOG.md           # Running log of toolkit updates
│
├── templates/             # Project starter templates
│   ├── PRD-TEMPLATE.md
│   ├── CLAUDE-MD-TEMPLATE.md
│   ├── FILE-STRUCTURE-TEMPLATE.md
│   ├── DESIGN-INTENT-TEMPLATE.md
│   └── RETRO-TEMPLATE.md
│
├── skills/
│   ├── workflows/         # Interactive step-by-step process skills
│   │   ├── _WORKFLOW-TEMPLATE.md
│   │   ├── new-project.md
│   │   ├── new-feature.md
│   │   ├── retro.md
│   │   ├── debug.md
│   │   ├── handoff-to-gary.md
│   │   ├── toolkit-update.md
│   │   └── resume-project.md
│   │
│   └── reference/         # Passive knowledge for Claude Code
│       ├── _SKILL-TEMPLATE.md
│       ├── surrealdb-v3.md
│       ├── surrealql.md
│       ├── tauri-nuxt.md
│       ├── nuxt-ui-v4.md
│       ├── rust-tauri-commands.md
│       ├── n8n-integration.md
│       ├── error-handling.md
│       └── frontend-design.md
│
├── references/            # Business and stack context
│   ├── stack-overview.md
│   ├── advosy-context.md
│   └── bresco-context.md
│
├── mcp-config/            # MCP server documentation
│   ├── mcp-servers-reference.md
│   └── mcp-setup-guide.md
│
└── retros/                # Post-project retrospectives
    └── _retro-index.md
```

## How It Improves Over Time

```
Start project (use toolkit)
  -> Build with Claude Code
  -> Things go well or wrong
  -> Run retro workflow
  -> Capture lessons in retro file
  -> Identify toolkit updates
  -> Apply updates to skills/templates
  -> Push to GitHub
  -> Next project starts smarter
```
