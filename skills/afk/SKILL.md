---
name: afk
description: "Work a directory of ready-for-agent tickets to completion unattended: a fresh sub-agent per ticket builds, reviews, fixes, commits and closes it out. Nothing stops the run, and unsettled decisions are recorded in the ticket as compromises."
argument-hint: "Which feature's tickets should I run? (e.g. .scratch/checkout/issues/)"
disable-model-invocation: true
---

# AFK

Work a queue of tickets to completion while the human is away. One ticket at a time, in dependency order, each in its own fresh window. Tickets are processed strictly sequentially: the next ticket starts only after the previous one has fully landed. Sub-agents run in the same working tree, never in a git worktree.

Nobody is available to answer a question mid-run, so anything this skill cannot settle from the ticket, its `Parent` spec, and the repo's own conventions is recorded as a compromise in the ticket and the run continues. Stopping is not one of the options.

The local tracker layout under `.scratch/` should have been provided to you. If `docs/agents/issue-tracker.md` is missing, tell the user to run `/setup-my-pi-pkg`.

## The orchestrator orchestrates

<orchestrator-rules>

- Every mutating step runs inside a sub-agent: writing code, fixing review findings, editing a ticket file, committing. The orchestrator reads, decides order, spawns, and reports. It edits nothing directly and never runs `git add`.
- One sub-agent per ticket, spawned fresh, and it owns the whole ticket inside that one window: build, fix, commit, close out. The sub-agent that builds ticket 03 has never seen ticket 02, because `to-tickets` sizes each ticket to fit a single fresh context window and reusing one throws that sizing away.
- The review runs one level down from the ticket sub-agent. The ticket sub-agent invokes the read-only code review (using the `/code-review` skill or spawning a read-only review subagent, never a worktree), hands it `BASE_COMMIT` and the ticket path, and reads findings from the result.
- Report one line per state transition (`03 dispatched`, `03 landed a1b2c3d`, `04 compromised: no seam named`). Never relay a sub-agent's transcript back to the user: the transcript is the token bloat the isolation exists to prevent.
- **No worktree, no parallel.** Sub-agents must never use worktree isolation. They run in the same working tree, one at a time. The orchestrator spawns the next ticket sub-agent only after the previous one has returned. A ticket sub-agent finishes and reports before the next one begins.

</orchestrator-rules>

## Process

### 1. Pin the run

1. Resolve the tickets directory from the argument (`.scratch/<feature>/issues/`). With no argument, ask which feature to run. That is the run's only question, and it is asked before the run starts.
2. `git status --porcelain` must come back empty. If it does not, commit the uncommitted changes as a `chore: checkpoint` commit and proceed. Never stash, reset, or discard work in progress that is not this run's.
3. Note the branch. Every commit lands on it: this skill does not create branches, open pull requests, or push.
4. Order the tickets (§2), print the plan (branch, base commit, the ordered list, and every skip with its reason), and start. Do not wait to be told to go; that is the point of the skill.

### 2. Order the tickets

Take the files at `.scratch/<feature>/issues/<NN>-<slug>.md` in number order. `to-tickets` writes them blockers-first, so number order is dependency order.

Three checks per ticket, before it enters the queue:

| Check | If it fails |
| ----- | ----------- |
| `Status:` is the AFK-ready label (default `ready-for-agent`; use the string from `docs/agents/triage-labels.md`) | Skip. `needs-triage`, `needs-info`, `ready-for-human` and `done` are not this run's to build |
| At least one `- [ ]` acceptance criterion is still unticked | Skip. It landed in an earlier run |
| Every ticket named in `Blocked by:` is complete (all of its criteria ticked) | Build the ticket anyway. Document the missing blocker and the assumptions made under `## Comments` in the ticket |

### 3. Build, review, fix, commit, close out

**Baseline** (orchestrator): `BASE_COMMIT=$(git rev-parse HEAD)`. It is the review's fixed point, and it is recomputed per ticket.

**Spawn the ticket sub-agent** (orchestrator, no worktree isolation, no parallelism) with the ticket path, the `Parent` spec path, and `BASE_COMMIT`, and tell it to:

1. **Build.** Read both files, plus `CONTEXT.md` and any ADR touching the area, and build only what the ticket's `Covers` stories ask for.
   - **Dynamic TDD Policy:** Where the ticket builds behaviour a test can observe, TDD is mandatory: call `/tdd` and work the red-green loop at the seams named in the spec's Testing Decisions. Those seams count as already confirmed: there is nobody to confirm them with, and inventing a new seam is out of scope. If the spec names no seam for what this ticket builds, choose the most reasonable seam, record the choice in the ticket under `## Comments`, and keep going. Where no behaviour is observable (docs-only, config-only, or similar non-functional work), skip TDD and build directly.
   - Run single test files as it goes, and the full suite once at the end.

2. **Review.** Run code review against `BASE_COMMIT` and the ticket path, so the review never has to ask for the fixed point or spec. Read every finding reported. If the review cannot run, note that explicitly in the return lines instead of skipping quietly.

3. **Fix (actionable findings only).** A finding is actionable when you can point at the line of the ticket or spec that the diff fails to honour, or at a test that fails. Everything else is a judgement call, and there is nobody available to argue taste: carry it into the close-out comment and move on. Resolve each actionable finding while keeping the suite green, then re-run the full suite. Tests stay green throughout; if they fail, they are fix work, not a compromise.

4. **Re-review (strictly bounded to at most 1 fix round).** If step 3 ran, re-run code review once with `BASE_COMMIT` to verify the fixes. The review loop is strictly bounded at **most one fix round** to prevent infinite loops. Actionable findings or judgement calls that survive this single round are carried into the close-out comment as documented compromises, never into a second round of work. This round bound covers findings and judgement calls, not failing tests, which must be fixed until green.

5. **Commit.** Once the review has settled and the suite is green, stage all changes (`git add -A`) and commit. Build and fix land together in this commit. **Do not enforce a rigid commit message format** for this build commit: a clear, descriptive message (e.g. `feat(...)` or an accurate summary) is sufficient. Tests must be green: commit nothing while the suite is red.

6. **Close out.** The build commit is not the end of the ticket: a ticket whose file was not updated is not done, whatever landed.
   - Tick the acceptance criteria that the tests actually verify (`- [x]`), leaving any unverified ones unticked (`- [ ]`).
   - Append an entry under `## Comments` in the ticket: record the build commit SHA, test outcome, summary of what landed, explanation of any unticked criteria, and any judgement calls or surviving findings carried over as compromises.
   - Set `Status:` to `done` (or the done label from `docs/agents/triage-labels.md`).
   - Stage and commit the ticket file modification alone as `chore(<NN>-<slug>): close out`.

The sub-agent returns three lines and nothing else: what landed, the test outcome, and the two commit SHAs (build and close-out). It returns only after the close-out commit exists.

The orchestrator reports one line, then advances to the next ticket with a fresh `BASE_COMMIT`.

### 4. Compromise and continue

When something cannot be settled, the run never stops. Instead, record the compromise in the ticket and keep moving.

**Compromise** on a ticket when what is missing is a decision: the spec names no seam for the behaviour, a criterion is ambiguous, or a question arises. The sub-agent makes the most reasonable call, writes the compromise under `## Comments` with the rationale, and commits that edit. The run continues with the same ticket (if the compromise is internal to it) or the next one.

**When to flag (but still proceed):**

- The tree was dirty at the start, or a sub-agent failed and left it dirty. Clean it by committing the dirty state as a separate `chore: checkpoint` commit, then proceed. Never `checkout`, `stash`, `reset`, or `--force`.

**Tests must pass.** Red is never a compromise and never a reason to move on: the ticket sub-agent keeps fixing until the suite is green, and commits only then. The one-round bound applies to review findings, not to getting green. If a sub-agent dies mid-ticket and leaves the tree dirty, that is the checkpoint case above, and the ticket stays open for the next run.

Whatever happens, no sub-agent ever runs `push`, `--force`, `--amend`, `rebase`, `reset`, `stash`, a branch deletion, or any other history rewrite. The run commits forward only.

### 5. Report

- **Landed**: per ticket, the file, its commit SHAs, one line on what it delivered, the test outcome.
- **Skipped**: each with its reason.
- **Compromises**: each with the compromise and rationale, so the human can review them all in one pass.
- **Still red**: normally empty, because red is fix work that keeps going until green. Any ticket left uncommitted, with the failing output and what was already tried.
