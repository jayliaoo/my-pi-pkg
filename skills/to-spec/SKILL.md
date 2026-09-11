---
name: to-spec
description: "Turn the current conversation into a spec saved as a local file: no interview, just synthesis of what you've already discussed. Ends by pointing at the next step (/to-tickets or /implement)."
disable-model-invocation: true
---

This skill takes the current conversation context and codebase understanding and produces a spec. Do NOT interview the user; just synthesize what you already know.

The local tracker layout under `.scratch/` should have been provided to you. If not, tell the user to run `/setup-my-pi-pkg`.

## Process

1. Explore the repo to understand the current state of the codebase, if you haven't already. Use the project's domain glossary vocabulary throughout the spec, and respect any ADRs in the area you're touching.

2. Sketch out the seams at which you're going to test the feature. Existing seams should be preferred to new ones. Use the highest seam possible. If new seams are needed, propose them at the highest point you can. The fewer seams across the codebase, the better: the ideal number is one.

Check with the user that these seams match their expectations.

3. Write the spec using the template below, then publish it as `.scratch/<feature-slug>/spec.md` (creating the directory if needed). Record `Status: ready-for-agent` near the top of the file.

<spec-template>

**Status:** ready-for-agent

## Problem Statement

The problem that the user is facing, from the user's perspective.

## Out of Scope

What this feature is NOT going to do.

## User Stories

A LONG, numbered list of user stories, numbered from 1. Never reorder or renumber stories after publishing, because tickets reference these numbers in their `Covers` field.

Write the whole spec, including user stories, in the language of the current conversation. The story shape follows the conversation language: in English, `As an <actor>, I want a <feature>, so that <benefit>`; in Chinese, `作为<actor>，我想要<feature>，以便<benefit>`.

<user-story-example>
1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
2. As an admin, I want to lock a customer's account, so that they can't make any further transactions
</user-story-example>

## Implementation Decisions

A list of decisions that were made during the planning phase. Include:

- What components are going to be built
- What third-party libraries will be used
- Key assumptions about the codebase or architecture
- Any architectural trade-offs that were made

Do NOT include specific file paths or code snippets. They may end up being outdated very quickly.

## Testing Decisions

A list of testing decisions that were made. Include:

- The seams at which the feature will be tested
- What tests will be written (unit, integration, end-to-end)
- What test fixtures/factories will be needed
- How existing tests will be affected

## Further Notes

Any further notes about the feature.

</spec-template>

## Next step

After publishing, point the user onward: `/to-tickets` to cut the spec into tickets, or `/implement` directly with the spec path if it needs no tickets.
