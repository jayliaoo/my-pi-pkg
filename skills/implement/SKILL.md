---
name: implement
description: "Implement a piece of work from a ticket (and its parent spec), directly from a spec, or from user input / conversation history."
disable-model-invocation: true
---

Implement the work described by a ticket, a spec, or directly from user input or conversation history.

## Mode 1: From a ticket (default)

1. Read the ticket file.
2. Read its `Parent` spec: the Implementation Decisions, Testing Decisions, agreed seams, Out of Scope, and the stories named in `Covers`.

## Mode 2: Directly from a spec

If the user provides a spec path directly (no ticket), skip the ticket step and go straight to the spec.

1. Read the spec file.
2. The spec's Implementation Decisions, Testing Decisions, agreed seams, and Out of Scope sections define the scope.

## Mode 3: From user input or conversation history

If the user provides neither a ticket nor a spec path, derive the implementation scope from the current conversation.

1. Look at the user's latest message and the surrounding conversation history to understand what needs to be built.
2. Identify the specific files, features, or changes the user is asking for.

## Build (all modes)

Build at the spec's agreed seams, or at the seams you derive from the conversation. Do not invent a new seam without justification.

Record the base commit before you start: `BASE_COMMIT=$(git rev-parse HEAD)`. The review after building diffs against it.

Use /tdd where possible, at those pre-agreed or derived seams.

Run single test files regularly, and the full test suite once at the end.

Once done, run code review (via `/code-review` or spawning a read-only review agent) with `BASE_COMMIT` as the fixed point and a spec source. In modes 1 and 2, the spec source is the ticket or spec path. In mode 3 there is no file to point at, so write the scope you derived from the conversation out as an inline spec in the prompt (e.g. "Spec: <the agreed scope, in words>").

Fix every issue the code review raises. After each fix, run only the related tests. Once all issues are fixed, re-run the full test suite to confirm nothing is broken.

Commit your work to the current branch.
