# my-pi-pkg

Personal pi package — a collection of extensions, skills, prompt templates, and themes for the [pi coding agent](https://pi.dev).

## Install

```bash
# From a local path
pi install /path/to/my-pi-pkg

# Or from relative path (in project settings)
pi install -l ./my-pi-pkg
```

## Contents

| Resource | Directory | Description |
|----------|-----------|-------------|
| Extensions | `extensions/` | Custom extensions |
| Skills | `skills/` | Reusable skills |
| Prompts | `prompts/` | Prompt templates |
| Themes | `themes/` | Theme definitions |

## Development

```bash
# Load a single extension temporarily
pi -e ./extensions/my-extension.ts

# List installed packages
pi list
```

## License

MIT