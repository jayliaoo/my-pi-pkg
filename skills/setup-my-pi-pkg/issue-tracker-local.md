# Local-Markdown Issue Tracker

Issues are tracked as local markdown files under `.scratch/`.

## Convention

- One feature per directory: `.scratch/<feature-slug>/`
- The spec is `.scratch/<feature-slug>/spec.md`
- Implementation issues are one file per ticket at `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered from `01`, never a single combined tickets file
- Triage state is recorded as a `Status:` line near the top of each issue file, using the default triage labels (`needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `done`)
- Comments and conversation history append to the bottom of the file under a `## Comments` heading

This is the only tracker. There are no remote variants.

## When a skill says "publish to the issue tracker"

Create a new file under `.scratch/<feature-slug>/` (creating the directory if needed).

## When a skill says "fetch the relevant ticket"

Read the file at the referenced path. The user will normally pass the path or the issue number directly. Always read the ticket's `Parent` spec alongside it.

## Wayfinding operations

- **Find issues by status:** search for files under `.scratch/` containing `Status: <role>` (e.g. `rg -l "^Status: ready-for-agent" .scratch/`).
- **Find issues by feature:** list files under `.scratch/<feature-slug>/issues/`.
- **Read an issue:** read the `.scratch/<feature-slug>/issues/<NN>-<slug>.md` file.
