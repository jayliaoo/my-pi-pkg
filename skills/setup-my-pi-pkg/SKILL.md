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

## Files written

- `docs/agents/issue-tracker.md` — local markdown issue tracker conventions
- `docs/agents/triage-labels.md` — label mapping
- `docs/agents/domain.md` — domain doc consumer rules + layout
- An `## Agent skills` section in `AGENTS.md` (if it exists)

## Usage

Run this skill once after cloning the repo. Re-run to reset the configuration to defaults.