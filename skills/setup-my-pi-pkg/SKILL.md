---
name: setup-my-pi-pkg
description: "Configure this repo for the engineering skills: set up its local issue tracker, default triage labels, and domain doc layout. Run once before first use of the other engineering skills."
disable-model-invocation: true
---

# Setup Engineering Skills

Scaffold the per-repo configuration that the engineering skills assume:

- **Issue tracker**: local markdown files under `.scratch/`
- **Triage labels**: the default five canonical triage roles
- **Domain docs**: where `CONTEXT.md` and ADRs live, and the consumer rules for reading them

This is a prompt-driven skill, not a deterministic script. Explore, then write. Every decision below has one right answer for the repo in front of you, so nothing is asked.

## Process

### 1. Explore

Look at the current repo to understand its starting state. Read whatever exists; don't assume:

- `AGENTS.md` and `CLAUDE.md` at the repo root: does either exist? Is there already an `## Agent skills` section in either?
- `CONTEXT.md` at the repo root
- `docs/adr/` and any `src/*/docs/adr/` directories
- `docs/agents/`: does this skill's prior output already exist?
- `.scratch/`: a sign that a local-markdown issue tracker convention is already in use

### 2. Write

Write everything immediately. Do not ask the user to confirm, do not present a draft for review, do not wait for a go-ahead. Just write.

**Determine the target file:**

- If `AGENTS.md` exists, edit it.
- Else if `CLAUDE.md` exists, edit it.
- Else (neither exists), create `AGENTS.md`: pi and other harnesses discover it alongside CLAUDE.md, so one file serves all of them. Do not ask the user which to create; the answer is always `AGENTS.md`.

If an `## Agent skills` block already exists in the chosen file, update its contents in-place rather than appending a duplicate. Don't overwrite user edits to the surrounding sections.

The block:

```markdown
## Agent skills

### Issue tracker

[one-line summary of where issues are tracked]. See `docs/agents/issue-tracker.md`.

### Triage labels

[one-line summary of the label vocabulary]. See `docs/agents/triage-labels.md`.

### Domain docs

[one-line summary of layout: "single-context"]. See `docs/agents/domain.md`.
```

Always include the `### Triage labels` sub-block and always write `docs/agents/triage-labels.md` with the defaults.

Then write the docs files using the seed templates in this skill folder as a starting point:

- [issue-tracker-local.md](./issue-tracker-local.md): local-markdown issue tracker
- [triage-labels.md](./triage-labels.md): default label mapping
- [domain.md](./domain.md): domain doc consumer rules + layout

### 3. Done

Tell the user the setup is complete, list every file written, name the decisions taken (the tracker, the labels, the single-context domain docs, and which file the `## Agent skills` block went into), and say which engineering skills will now read from them. Mention they can edit `docs/agents/*.md` directly later; re-running this skill is only necessary if they want to restart from scratch.
