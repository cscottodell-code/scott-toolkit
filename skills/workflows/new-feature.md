# New Feature

## Metadata
- Last updated: 2026-02-25
- Version: 1.0
- Changelog:
  - v1.0: Initial workflow

## Purpose
Add a new feature to an existing project. This is a lighter version of the new-project
workflow — instead of building a full PRD, you create a mini-PRD for just the new feature.

Use this when Scott says "I want to add [X] to [project]" or "let's build a new feature."

## Prerequisites
- The project already exists with a CLAUDE.md and PRD.md
- The project is in a working state (not mid-milestone)

## Instructions for Claude Code
Read the project's CLAUDE.md and PRD.md first to understand the current state.
Then walk Scott through each phase. Keep it lightweight — this shouldn't feel
as heavy as starting a new project.

## Phase 1: Feature Description

### What this phase does
Understand what the new feature is and why it's needed.

### Questions to ask Scott
1. What's the feature? Describe it in one or two sentences.
2. Who needs this? (Same users as the main app, or different?)
3. Why is this needed now? What triggered this request?
4. How urgent is it? (Blocking other work, or a nice enhancement?)

### Output
A clear, one-paragraph feature description.

### Done when
Scott has clearly described what the feature is and why it matters.

## Phase 2: Impact Assessment

### What this phase does
Figure out what existing parts of the app this feature touches. New features
rarely exist in isolation — they usually affect data models, existing pages,
navigation, etc.

### Questions to ask Scott
1. Does this feature need new data (new tables or fields in SurrealDB)?
2. Does it change any existing data structures?
3. Which existing pages will be affected?
4. Does it need a new page or just additions to existing ones?
5. Any new dependencies or external services needed?
6. Could this break anything that currently works?

### Checks to perform
- Review the SurrealDB schema section of CLAUDE.md
- Review the current file structure
- Check for components that might need modification

### Output
A list of files and systems that will be affected, with the nature of each change.

### Done when
The full scope of impact is understood.

## Phase 3: Mini-PRD

### What this phase does
Create a lightweight PRD for just this feature. Not the full 11-section template —
just the essentials.

### Sections to include
1. **Feature name and description** (from Phase 1)
2. **User flow** — step-by-step walkthrough of how the user interacts with this feature
3. **Data changes** — new tables, new fields, modified relationships
4. **Acceptance criteria** — how do you know this feature is "done"?
5. **Edge cases** — what happens when things go wrong or data is missing?

### Output
A mini-PRD document (can be added to the project's PRD.md as an appendix or kept separate).

### Done when
Scott approves the mini-PRD.

## Phase 4: Build

### What this phase does
Implement the feature.

### Steps
1. Enter plan mode — break the feature into tasks
2. Write tasks to tasks/todo.md
3. If data model changes are needed, do those first:
   - Update schema.surql
   - Update CLAUDE.md's SurrealDB Schema section
4. Build the feature, following the project's CLAUDE.md behavior rules
5. Verify each task works before marking it complete
6. Test the complete feature end-to-end

### Output
Working feature, verified and tested.

### Done when
All acceptance criteria from the mini-PRD are met.

## Phase 5: Update CLAUDE.md

### What this phase does
Keep the project's living documentation current.

### Updates to make
1. **SurrealDB Schema** section: Add any new tables or field changes
2. **Key Decisions Log**: Record any architectural decisions made
3. **Current Status**: Update what was last completed and what's next
4. **Constraints & Rules**: Add any new rules that emerged from this feature

### Output
Updated CLAUDE.md reflecting the new feature.

### Done when
CLAUDE.md accurately reflects the current state of the project.

## Completion Checklist
- [ ] Feature clearly described
- [ ] Impact on existing code assessed
- [ ] Mini-PRD written and approved
- [ ] Feature built and verified
- [ ] CLAUDE.md updated
- [ ] tasks/todo.md updated
