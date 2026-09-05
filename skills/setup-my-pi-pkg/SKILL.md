---
name: setup-my-pi-pkg
description: "Configure this pi package repo: set up local issue tracker, triage labels, and domain doc layout. Run once before first use of the other skills."
---

# Setup My Pi Package

Scaffold the per-repo configuration that the skills in this package assume.
This is a deterministic skill — it writes the standard configuration without asking questions.

## What it does

### Issue tracker

Sets up a **local markdown** tracker. Issues live as markdown files under `.scratch/`.

See `docs/agents/issue-tracker.md`.

### Triage labels

Writes the five canonical triage roles with default label strings (each equal to its name):

| Role | Label string |
|------|-------------|
| `needs-triage` | `needs-triage` |
| `needs-info` | `needs-info` |
| `ready-for-agent` | `ready-for-agent` |
| `ready-for-human` | `ready-for-human` |
| `wontfix` | `wontfix` |

See `docs/agents/triage-labels.md`.

### Domain docs

**Single-context** layout: one `CONTEXT.md` + `docs/adr/` at the repo root.

See `docs/agents/domain.md`.

### AGENTS.md configuration

Checks for `AGENTS.md` at the repository root. **If `AGENTS.md` does not exist, create it first** (with a top-level heading and basic project context). Then add or update the `## Agent skills` section to point to the issue tracker and skill conventions.

## Files written

- `docs/agents/issue-tracker.md` — local markdown issue tracker conventions
- `docs/agents/triage-labels.md` — label mapping
- `docs/agents/domain.md` — domain doc consumer rules + layout
- `AGENTS.md` at the repo root — **created first if it does not exist**, then updated with an `## Agent skills` section

## Usage

Run this skill once after cloning the repo. Re-run to reset the configuration to defaults.