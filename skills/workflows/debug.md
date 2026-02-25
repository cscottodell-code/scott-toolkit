# Debug

## Metadata
- Last updated: 2026-02-25
- Version: 1.0
- Changelog:
  - v1.0: Initial workflow

## Purpose
Structured troubleshooting when something isn't working. Instead of randomly
trying fixes, this workflow follows a systematic approach: describe the problem,
gather evidence, hypothesize, test, and capture the lesson.

Use this when Scott says "this isn't working", "I'm getting an error", or "something broke."

## Prerequisites
- A project with a running (or broken) app
- A specific problem to investigate

## Instructions for Claude Code
Follow the scientific method. Do NOT jump to a fix without understanding the problem first.
The most common debugging mistake is fixing the wrong thing because you didn't gather
enough evidence. Be methodical.

## Phase 1: Describe the Problem

### What this phase does
Get a clear picture of what's happening vs what should be happening.

### Questions to ask Scott
1. What were you trying to do? (The action)
2. What did you expect to happen? (Expected behavior)
3. What actually happened? (Actual behavior)
4. What page/screen were you on?
5. Did this work before? If so, what changed recently?
6. Can you reproduce it consistently, or is it intermittent?
7. Did you see any error messages? (Exact text if possible)

### Output
A clear problem statement: "When [action], expected [X] but got [Y] on [page]."

### Done when
The problem is clearly and specifically described.

## Phase 2: Gather Evidence

### What this phase does
Collect all the clues before forming theories.

### Evidence to check (in this order)
1. **Browser console errors** — any red text in the developer console?
2. **Network requests** — any failed API calls or unexpected responses?
3. **SurrealDB logs** — any query errors or connection issues?
4. **Tauri logs** (desktop only) — any Rust panics or IPC errors?
5. **Recent changes** — what files were modified recently? (`git diff` or `git log`)
6. **The relevant code** — read the files involved in the broken flow

### Output
A list of evidence findings:
- "Console shows: [error]"
- "Network tab shows: [request] returned [status]"
- "The code at [file:line] does [X], which might be the issue because [Y]"

### Done when
You have enough evidence to form hypotheses.

## Phase 3: Hypothesize

### What this phase does
List the most likely causes, ranked by probability.

### Steps
1. Based on the evidence, list 2-3 possible causes
2. For each, explain:
   - What the cause would be
   - Which evidence supports this theory
   - How likely it is (most likely / possible / unlikely)
3. Present to Scott: "Based on what I found, here are the most likely causes..."

### Output
A ranked list of hypotheses.

### Done when
Hypotheses are listed and the most likely one is identified.

## Phase 4: Test & Fix

### What this phase does
Work through hypotheses starting with the most likely.

### Steps
1. Start with the most likely cause
2. Implement the fix
3. Test it — does the original problem go away?
   - **Yes:** Move to Phase 5
   - **No:** Revert the fix, move to the next hypothesis
4. If all hypotheses fail:
   - Gather more evidence (back to Phase 2)
   - Ask Scott for more context
   - Check tasks/lessons.md for similar past issues

### Output
A working fix with an explanation of what was wrong and why the fix works.

### Done when
The original problem is resolved and Scott confirms it works.

## Phase 5: Capture Lesson

### What this phase does
If this was a non-obvious bug, document it so it doesn't happen again.

### Questions to consider
1. Was this bug caused by a common pattern? (Could it happen again?)
2. Should this be added to tasks/lessons.md?
3. Is this a pattern worth adding to a toolkit reference skill?
4. Could better error handling have caught this earlier?

### Steps
1. If it's a project-specific lesson: Add to tasks/lessons.md
2. If it's a pattern that applies across projects: Note it for the next retro
3. Format: "Bug: [description]. Cause: [root cause]. Fix: [what fixed it].
   Prevention: [how to avoid this in the future]."

### Output
Lesson captured in the appropriate location.

### Done when
The lesson is documented (or determined not worth documenting).

## Completion Checklist
- [ ] Problem clearly described
- [ ] Evidence gathered
- [ ] Hypotheses ranked
- [ ] Fix implemented and verified
- [ ] Lesson captured (if applicable)
