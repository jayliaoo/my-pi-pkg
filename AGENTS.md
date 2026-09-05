# my-pi-pkg — Agent Guide

Personal pi package for extensions, skills, prompt templates, and themes.

## Context

- **Pi package** — installed via `pi install ./relative/path` or `pi install /absolute/path`.
- Resources are declared in `package.json` under the `pi` key, or auto-discovered from conventional directories (`extensions/`, `skills/`, `prompts/`, `themes/`).

## Structure

| Directory      | Contents                          |
|----------------|-----------------------------------|
| `extensions/`  | `.ts` / `.js` extension files     |
| `skills/`      | `SKILL.md` skill folders or `.md` |
| `prompts/`     | `.md` prompt templates            |
| `themes/`      | `.json` theme files               |

## Adding a Resource

1. Place the file in the correct conventional directory.
2. If the file is outside the conventional directory, add its path to the corresponding array in `package.json` → `pi`.
3. Run `pi list` locally to verify the resource is discovered.

## Development

- Test extensions with `pi -e <path>` to load temporarily.
- Run `pi update --self` to update the pi CLI itself.
- Run `pi update --extensions` to reconcile pinned git refs.