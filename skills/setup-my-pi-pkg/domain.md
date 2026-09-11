# Domain Documentation Rules

Where domain documentation lives in this repo, and how skills should read it.

## Files to read

- **`CONTEXT.md`** at the repo root (or context root): the domain glossary and ubiquitous language.
- **`CONTEXT-MAP.md`** at the repo root if it exists: it points at one `CONTEXT.md` per context. Read each one relevant to the topic.
- **`docs/adr/`**: read ADRs that touch the area you're about to work in. In multi-context repos, also check `src/<context>/docs/adr/` for context-scoped decisions.

If any of these files don't exist, **proceed silently**. Don't flag their absence; don't suggest creating them upfront. The `/grill-with-docs` skill creates them lazily when terms or decisions actually get resolved.

## File structure

Single-context repo (the default):

```
/
├── CONTEXT.md
└── docs/adr/
    ├── 0001-record-decisions.md
    └── 0002-use-postgres.md
```

Multi-context repo (indicated by the presence of `CONTEXT-MAP.md` at the repo root):

```
/
├── CONTEXT-MAP.md
├── docs/adr/                     ← system-wide ADRs
├── src/
│   ├── ordering/
│   │   ├── CONTEXT.md
│   │   └── docs/adr/             ← context-scoped ADRs
│   └── billing/
│       ├── CONTEXT.md
│       └── docs/adr/
```

## Respect the glossary

When your output names a domain concept (in an issue title, a refactor proposal, a hypothesis, a test name), use the term as defined in `CONTEXT.md`. Don't drift to synonyms the glossary explicitly avoids.

If the concept you need isn't in the glossary yet, that's a signal: either you're inventing language the project doesn't use (reconsider) or there's a real gap (note it for `/grill-with-docs`).

## Flag ADR conflicts

If what you are asked to build conflicts with an accepted ADR, flag it before implementing. Don't silently violate the decision, and don't silently rewrite the ADR.
