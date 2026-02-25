# Resume Project

## Metadata
- Last updated: 2026-02-25
- Version: 1.0
- Changelog:
  - v1.0: Initial workflow

## Purpose
Pick up a project after time away. Gets Claude Code back up to speed on where
things stand and confirms direction with Scott before continuing work.

Use this when Scott says "let's pick up [project]" or "where were we on [project]"
or returns to a project after days/weeks away.

## Prerequisites
- An existing project with CLAUDE.md
- Time has passed since the last work session

## Instructions for Claude Code
Read everything first, then summarize. Don't start working until Scott confirms
the direction. Priorities may have changed since the last session.

## Phase 1: Read Context

### What this phase does
Get fully up to speed on the project's current state by reading all relevant files.

### Files to read (in this order)
1. **CLAUDE.md** — especially the Current Status section
2. **PRD.md** — to remember the project's goals
3. **tasks/todo.md** — to see what tasks remain
4. **tasks/lessons.md** — to review past mistakes
5. **Recent git log** — to see what was last committed
6. **Any open issues or pending work**

### Output
A mental model of where the project stands.

### Done when
All context files have been reviewed.

## Phase 2: Summarize State

### What this phase does
Tell Scott exactly where things stand in plain English.

### Format
Present a summary like this:

"Here's where we left off on [Project Name]:

**Last completed:** [what was finished in the last session]
**Current milestone:** [which milestone we're on] — [X of Y tasks done]
**Next up:** [what the next task or feature is]
**Known issues:** [any bugs or problems noted in CLAUDE.md or lessons.md]
**Time since last work:** [based on git log dates]"

### Done when
Scott confirms the summary is accurate.

## Phase 3: Confirm Direction

### What this phase does
Check whether priorities have changed since the last session.

### Questions to ask Scott
1. Does this summary match what you remember?
2. Have your priorities changed since we last worked on this?
3. Is there anything new you want to add or change before we continue?
4. Do you want to continue with the planned next task, or pivot to something else?
5. Any feedback from Brett, Gary, or others since the last session?

### Output
Confirmed direction for this session.

### Done when
Scott confirms what to work on.

## Phase 4: Resume Work

### What this phase does
Continue from where the project left off.

### Steps
1. Review tasks/lessons.md to avoid repeating past mistakes
2. Pick up the next task from tasks/todo.md
3. If Scott redirected to something else, update tasks/todo.md accordingly
4. Follow the project's CLAUDE.md behavior rules
5. Work through tasks, checking in as appropriate

### Output
Progress on the project.

### Done when
The session's work is complete (either the task is done or Scott ends the session).

## Completion Checklist
- [ ] Project context fully read
- [ ] State summarized for Scott
- [ ] Direction confirmed
- [ ] Work resumed (or redirected based on Scott's input)
- [ ] CLAUDE.md Current Status updated at end of session
