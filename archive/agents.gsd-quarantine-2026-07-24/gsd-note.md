# GSD agent quarantine, 2026-07-24

31 `gsd-*` agent definitions moved here from `~/.claude/agents/`.
Nothing was deleted. This directory is a sibling of `agents/`, so Claude
Code does not scan it.

## Why

- GSD was abandoned 2026-04-28 (recorded in
  `~/Scott/claude-os/rules/claude-behavior.md`). These files are dated
  Apr 17, ten days before that.
- Their entry points were already gone: no `gsd-*` commands in
  `~/.claude/commands/`, no active `gsd` skills, no `.planning/`
  directories in `~/Scott/claude-projects/`. Every one of these agents
  is documented as "Spawned by /gsd-* orchestrator", and those
  orchestrators do not exist.
- The GSD skills were quarantined the same way on 2026-06-09 (see
  `~/.claude/skills.gsd-quarantine-2026-06-09/`, 88 directories). The
  agents were left behind in that pass. This finishes it.
- `~/Scott/claude-os/work/2026-05-07-prompt-engineering-research.md:390`
  had already reached this conclusion: "GSD agent count confirmed: 31.
  Plus 5 non-GSD agents that should stay."

## Effect

Agent descriptions load into every session. Removing these cut roughly
7,360 characters, about 1,840 tokens, per session. The larger benefit is
that 31 dead agent names no longer sit in the routing table where they
can be dispatched to orchestrators that no longer exist.

## Kept in `agents/`

`api-connector`, `automation-tester`, `business-consultant`,
`code-explainer`, `error-translator`. All five map to live work and are
handed to Brett in `~/.claude/brett-setup-instructions.md`.

## To reverse

    mv ~/.claude/agents.gsd-quarantine-2026-07-24/gsd-*.md ~/.claude/agents/

## Known dead references left alone, deliberately

Not agents, and out of scope for this pass:

- `~/.claude/settings.json:254` runs `hooks/gsd-statusline.js`.
- `~/.claude/settings.local.json` holds stale `Skill(gsd:*)` permission
  entries and a `gsd-tools.js` Bash permission.
- `~/.claude/gsd-file-manifest.json`.
