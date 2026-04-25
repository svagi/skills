---
name: to-issues
description: Break a plan, spec, or PRD into independently-grabbable tasks saved as Markdown files in ./plans/, using tracer-bullet vertical slices. Use when user wants to convert a plan into tasks, create implementation tickets, or break down work into slices.
---

# To Issues

Break a plan into independently-grabbable tasks using vertical slices (tracer bullets). Each task is saved as a Markdown file in `./plans/`.

## Process

### 1. Gather context

Work from whatever is already in the conversation context. If the user passes a path to a PRD/plan file (e.g. `./plans/foo-prd.md`), read it. If the user passes a GitHub issue number or URL as an argument, fetch it with `gh issue view <number>` (with comments).

### 2. Explore the codebase (optional)

If you have not already explored the codebase, do so to understand the current state of the code.

### 3. Draft vertical slices

Break the plan into **tracer bullet** tasks. Each task is a thin vertical slice that cuts through ALL integration layers end-to-end, NOT a horizontal slice of one layer.

Slices may be 'HITL' or 'AFK'. HITL slices require human interaction, such as an architectural decision or a design review. AFK slices can be implemented and merged without human interaction. Prefer AFK over HITL where possible.

<vertical-slice-rules>
- Each slice delivers a narrow but COMPLETE path through every layer (schema, API, UI, tests)
- A completed slice is demoable or verifiable on its own
- Prefer many thin slices over few thick ones
</vertical-slice-rules>

### 4. Quiz the user

Present the proposed breakdown as a numbered list. For each slice, show:

- **Title**: short descriptive name
- **Type**: HITL / AFK
- **Blocked by**: which other slices (if any) must complete first
- **User stories covered**: which user stories this addresses (if the source material has them)

Ask the user:

- Does the granularity feel right? (too coarse / too fine)
- Are the dependency relationships correct?
- Should any slices be merged or split further?
- Are the correct slices marked as HITL and AFK?

Iterate until the user approves the breakdown.

### 5. Write the task files

Create `./plans/` if it doesn't exist. For each approved slice, write a Markdown file named `NN-<slug>.md` where `NN` is a zero-padded ordinal reflecting dependency order (e.g. `01-add-schema.md`, `02-wire-api.md`). Use the template below.

Write files in dependency order (blockers first) so you can reference real file names in the "Blocked by" field.

<task-template>
# <Slice Title>

**Type**: HITL | AFK

## Parent

<path to source PRD/plan, e.g. `./plans/foo-prd.md`, or `#<gh-issue-number>` if the source was a GitHub issue — otherwise omit this section>

## What to build

A concise description of this vertical slice. Describe the end-to-end behavior, not layer-by-layer implementation.

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Blocked by

- `NN-<slug>.md` (if any)

Or "None - can start immediately" if no blockers.

</task-template>

Do NOT modify the source PRD/plan file. Do NOT create or modify any GitHub issues.
