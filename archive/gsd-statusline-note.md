# Statusline patch backup, 2026-07-24

Two files, both copies of `~/.claude/hooks/gsd-statusline.js`, which is
not version controlled anywhere else.

| File | What it is |
|---|---|
| `gsd-statusline.js` | The patched version, live in `~/.claude/hooks/` as of 2026-07-24 |
| `gsd-statusline.js.pre-nag-patch-2026-07-24.bak` | The original, kept so the patch is reversible |

These are ARCHIVE COPIES. Nothing loads them from here. The live file is
`~/.claude/hooks/gsd-statusline.js`, referenced by `settings.json`.

## Do not mistake this for a GSD component

Despite the name, this is the standard Claude Code statusline with GSD
extras bolted on. It also:

- renders the context-usage meter and the current in-progress task,
- writes `claude-ctx-<session>.json` to the temp directory, which the
  separate `gsd-context-monitor.js` PostToolUse hook reads to warn about
  low context.

Deleting the statusline outright would blind that monitor hook. It was
considered and rejected on 2026-07-24 for exactly this reason.

## What the patch removed

1. The `/gsd-update` nag. Its cache (`~/.cache/gsd/gsd-update-check.json`)
   went stale 2026-06-09 with `update_available: true` for GSD 1.37.1 to
   1.42.3, and nothing regenerates it, so the prompt displayed in every
   session in every project forever, pointing at a command that no longer
   exists.
2. The `.planning/STATE.md` lookup. No such directory exists anywhere in
   `~/Scott/claude-projects/` since GSD was abandoned 2026-04-28, so it
   walked up 10 directory levels calling `existsSync` on every statusline
   render and always returned empty.

Kept deliberately: `readGsdState`, `parseStateMd`, `formatGsdState` and
their `module.exports`, because `~/.claude/hooks/gsd-check-update-worker.js`
imports them.

## Verified after patching

`node --check` passes. Plain render, context-meter render, and malformed
stdin all exit 0. The three helpers still import. The context bridge file
is still written with correct values.

Before: `⬆ /gsd-update │ Opus │ advosy-sales-eos`
After:  `Opus │ advosy-sales-eos │ ████░░░░░░ 48%`

## To reverse

    cp ~/Scott/claude-os/archive/gsd-statusline.js.pre-nag-patch-2026-07-24.bak \
       ~/.claude/hooks/gsd-statusline.js

Related: [agents.gsd-quarantine-2026-07-24](agents.gsd-quarantine-2026-07-24/gsd-note.md)
