---
name: afk
description: "Sequentially execute and complete all tickets in a directory without human intervention. All implementation, review fixing, and commits run strictly inside the general-purpose subagent. For each ticket: record the base commit, implement via general-purpose with /tdd, review via /code-review, fix and commit via general-purpose, and advance."
disable-model-invocation: true
---

Automate unattended, end-to-end implementation of a sequence of tickets from a directory. Each ticket runs through an isolated implementation, review, fix, and commit cycle.

> **CRITICAL RULE — ALWAYS USE THE `general-purpose` SUBAGENT:**
> Never execute ticket implementation, code review fixes, or git commits directly in the parent agent session. All mutating work must occur inside a `general-purpose` subagent (e.g., via `Agent` with `subagent_type: "general-purpose"`). If Step 4 is executed, reuse that same subagent instance to commit in Step 5; if Step 4 is skipped, spawn a fresh `general-purpose` subagent for Step 5. This strictly protects the main orchestrator's context window from token bloat across multi-ticket sequences and prevents context contamination between tickets.

## Workflow

### 1. Discover and order tickets

1. Resolve the tickets directory from the user's input (e.g., `tickets/`, `docs/tickets/`, or an explicit path provided by the user). If not provided, ask the user.
2. List all ticket files (e.g., `*.md`) and sort them in natural sequential order (e.g., `01-*.md`, `02-*.md`, or alphanumeric).
3. Confirm the git working tree is clean before starting (`git status --porcelain`). If dirty, inform the user and halt.

### 2. Process each ticket in sequence

For each ticket in the ordered list, execute the following five-step loop:

#### Step 1: Record baseline commit

Record the current HEAD commit hash before making any changes for this ticket:

```bash
BASE_COMMIT=$(git rev-parse HEAD)
```

Keep this SHA accessible throughout the ticket's cycle; it serves as the fixed point for code review.

#### Step 2: Implement via `general-purpose` subagent using `/tdd`

Read the ticket content and **always spawn the `general-purpose` subagent** (e.g., using `Agent` with `subagent_type: "general-purpose"`) to carry out the implementation:
- Provide the `general-purpose` subagent with the full ticket content, target files, and clear instructions.
- Instruct the `general-purpose` subagent to adapt `/tdd` for fully unattended execution:
  - **Autonomously identify public seams**: Determine the public interfaces and testing boundaries directly from the ticket spec and existing codebase. **Do NOT prompt or wait for user confirmation**—test exclusively at public module seams without coupling to private internals.
  - **Vertical slices**: Write a failing test for a seam first, write only enough code to pass, then proceed to the next slice.
  - **Zero interaction**: The subagent must operate 100% autonomously without stopping to ask questions.
  - Run project tests and typechecking frequently.
- Sub-agent completion criterion: All newly written and existing tests pass, typechecks succeed, and implementation satisfies the ticket requirements.

#### Step 3: Review with `/code-review`

Invoke the `/code-review` skill using `$BASE_COMMIT` as the fixed point:
- **Pass the ticket file path as the explicit spec source** so the review runs completely unattended without asking where the spec is.
- The review examines `git diff --merge-base $BASE_COMMIT`.
- Evaluates both axes in parallel:
  - **Standards**: repo coding conventions and code smell baseline.
  - **Spec**: fidelity to the ticket requirements and scope creep detection.
- Collect and inspect the final review report.

#### Step 4: Address review findings via `general-purpose` subagent

Evaluate the findings from Step 3:
- If there are actionable issues (unimplemented requirements, standards violations, code smells, regressions):
  - **Spawn the `general-purpose` subagent** (e.g., using `Agent` with `subagent_type: "general-purpose"`) with the review report, the ticket spec, and current git state. Do not fix findings directly in the main orchestrator session.
  - Instruct the subagent to:
    1. Resolve each valid finding while keeping the test suite green.
    2. Run the full test suite and typechecks to verify the fixes.
    3. **Execute Step 5 inside this same subagent instance:** Once fixes are verified, immediately stage all changes (`git add -A`), commit with `feat(<ticket-identifier>): <concise summary>`, and return the commit SHA to the parent orchestrator.
- If the review found no actionable issues, skip directly to Step 5.

#### Step 5: Commit changes via `general-purpose` subagent

Commits must always be executed inside a `general-purpose` subagent, never directly by the parent orchestrator:
- **If Step 4 ran (not skipped):** The commit was already performed by the same `general-purpose` subagent instance from Step 4. The parent orchestrator simply receives and records the returned commit SHA.
- **If Step 4 was skipped (no issues found):** Spawn a fresh `general-purpose` subagent whose sole task is to:
  1. Stage all changes: `git add -A`
  2. Commit with: `git commit -m "feat(<ticket-identifier>): <concise summary>"`
  3. Return the resulting commit SHA (`git rev-parse HEAD`).
- Record the new commit SHA and report progress before proceeding to the next ticket.

---

### 3. Final Summary

When all tickets are completed, present a concise summary report:

- Total tickets completed.
- Per-ticket breakdown:
  - Ticket file / identifier
  - Commit SHA
  - Summary of changes and test outcomes
