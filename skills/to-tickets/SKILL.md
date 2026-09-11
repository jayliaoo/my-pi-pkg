---
name: to-tickets
description: Break a spec into a set of tracer-bullet tickets, each declaring its blocking edges, written as local files. Ends by pointing at the next step (/implement or /afk).
disable-model-invocation: true
---

# To Tickets

Break a spec into a set of **tickets**: tracer-bullet vertical slices, each declaring the tickets that **block** it and the spec stories it covers.

The local tracker layout under `.scratch/` should have been provided to you. If not, tell the user to run `/setup-my-pi-pkg`.

## Process

### 1. Gather context

This skill only accepts a spec as input. Ask the user for the spec file path (`.scratch/<feature-slug>/spec.md`). Read the full spec. If there is no spec yet, stop and tell the user to run `/to-spec` first; do not reconstruct one from conversation.

### 2. Explore the codebase (optional)

If you haven't already, explore the codebase to understand the current architecture and patterns.

### 3. Draft the tickets

Decompose the spec into a sequence of tickets:

- Each ticket is a **tracer bullet**: a complete vertical slice through the stack that delivers an end-to-end behavior the user can see or exercise. It is not an architectural layer (no "build the database schema", "build the API", "build the UI" tickets).
- Each ticket is sized to be **doable in a single, focused session** (roughly 1–2 hours of work).
- Slices should build on each other naturally: the first ticket creates the thinnest possible walking skeleton, and each subsequent ticket fattens it.
- Tickets must be strictly ordered so the codebase stays green and working after every ticket.

Give each ticket its **blocking edges**: the other tickets that must complete before it can start. A ticket with no blockers can start immediately.

**Wide refactors are the exception to vertical slicing.** A **wide refactor** is one mechanical change (rename a column, retype a shared symbol) whose **blast radius** fans across the whole codebase, so a single edit breaks thousands of call sites at once and no vertical slice can land green. Don't force it into a tracer bullet; sequence it as **expand and contract**. First expand: add the new form beside the old so nothing breaks. Then migrate the call sites over in batches sized by blast radius (per package, per directory), each batch its own ticket blocked by the expand, keeping CI green batch to batch because the old form still exists. Finally contract: delete the old form once no caller remains, in a ticket blocked by every migrate batch. When even the batches can't stay green alone, keep the sequence but let them share an integration branch that all block a final integrate-and-verify ticket; green is promised only there.

### 4. Quiz the user

Present the draft list of tickets to the user. For each ticket, state:

- **Title**: short imperative description
- **Covers**: the spec user stories it delivers (e.g. "Story 1, Story 3")
- **Blocked by**: which other tickets (if any) must complete first
- **What it delivers**: the end-to-end behaviour this ticket makes work

Then put the three questions to the user in prose:

- Does the granularity feel right? (too coarse / too fine)
- Are the blocking edges correct: does each ticket only depend on tickets that genuinely gate it?
- Does each ticket deliver a visible slice of behavior?

Iterate until the user approves the breakdown.

### 5. Publish the tickets

Write one file per ticket under `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered from `01` in dependency order (blockers first), never a single combined file. Record the spec path in each file's `Parent` field and the story numbers the ticket delivers in its `Covers` field, exactly as approved.

Work the **frontier**: any ticket whose blockers are all done. For a purely linear chain that means top to bottom.

Do NOT modify the parent spec.

<local-ticket-template>

# <NN>: <Ticket title>

**Parent:** the spec file path this ticket was cut from (for example, `../spec.md`). Always filled in.

**Covers:** the spec user story numbers this ticket delivers a slice of (for example, `Story 3, Story 5`).

**What to build:** the end-to-end behaviour this ticket makes work, from the user's perspective, not a layer-by-layer implementation list.

**Blocked by:** the numbers/titles of the tickets that gate this one, or "None (can start immediately)".

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2

</local-ticket-template>

Avoid specific file paths or code snippets: they go stale fast.

## Next step

After publishing, point the user onward: `/implement` on the tickets one at a time, or `/afk` to run all the tickets unattended.
