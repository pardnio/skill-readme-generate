# readme-generate - Documentation

> Back to [README](../README.md)

## Prerequisites

- Python 3.8 or higher (uses native generic syntax like `dict[str, Any]`, `list[...]`)
- [Claude Code](https://claude.ai/claude-code) CLI installed and configured
- Git (used to read `git remote`, first-commit year, etc.)

## Installation

### Clone from GitHub

```bash
git clone https://github.com/pardnchiu/skill-readme-generate.git \
    ~/.claude/skills/readme-generate
```

### Manual Installation

Place the following files under `~/.claude/skills/readme-generate/`:

```
readme-generate/
├── scripts/
│   ├── analyze_project.py    # Source analysis script
│   ├── setup_config.py       # Author config script
│   └── examples/             # README / doc templates
├── SKILL.md                  # Skill definition and protocol
├── LICENSE
├── README.md
└── doc/
    ├── README.zh.md
    ├── doc.md
    ├── doc.zh.md
    ├── architecture.md
    └── architecture.zh.md
```

Once installed, invoke `/readme-generate` from Claude Code.

## Configuration

### Author Config File

On the first invocation of `/readme-generate`, the skill checks `~/.skill-readme-generate.json` and prompts interactively if the file is missing.

| Field | Required | Description |
|-------|----------|-------------|
| `author_name` | Yes | Author name, shown in README Author section and copyright footer |
| `author_email` | Yes | Contact email |
| `author_url` | Yes | Personal URL (LinkedIn / GitHub / homepage) |
| `github_owner` | Yes | GitHub username, used as the default `{owner}` and avatar source |

Example `~/.skill-readme-generate.json`:

```json
{
  "author_name": "邱敬幃 Pardn Chiu",
  "author_email": "hi@pardn.io",
  "author_url": "https://linkedin.com/in/pardnchiu",
  "github_owner": "pardnchiu"
}
```

### Manual Initialization

Run the script directly in a terminal to inspect or create the config outside Claude Code:

```bash
python3 ~/.claude/skills/readme-generate/scripts/setup_config.py
```

If the file exists, the script prints the current config; otherwise it prompts for each field via `input()`.

### Non-Interactive Write

Supply four arguments to write the config from a shell or automation:

```bash
python3 ~/.claude/skills/readme-generate/scripts/setup_config.py write \
    "邱敬幃 Pardn Chiu" \
    "hi@pardn.io" \
    "https://linkedin.com/in/pardnchiu" \
    "pardnchiu"
```

### Override Precedence

When `REPO_PATH` (`github.com/{owner}/{repo}`) is passed on the command line, `{owner}` is taken from that path while other author fields still come from the config file. Edit or delete `~/.skill-readme-generate.json` at any time to trigger a reset.

## Usage

### Basic

```bash
/readme-generate
```

Run from the current Claude Code working directory to:

1. Load or create the author config
2. Execute `analyze_project.py` against the project
3. Generate six documentation files from the extracted features
4. Emit an MIT LICENSE if no LICENSE file exists

### Specify License Type

```bash
/readme-generate Apache-2.0
```

Generates the corresponding LICENSE from one of seven templates: MIT, Apache-2.0, GPL-3.0, BSD-3-Clause, ISC, Unlicense, Proprietary.

### Private Mode

```bash
/readme-generate private
```

Hides badges and star history, keeping only the tagline and body copy — appropriate for internal projects.

### Override Repository Path

```bash
/readme-generate github.com/foo/bar
```

Replaces every GitHub `{owner}/{repo}` URL in the README with `foo/bar`.

### Combined Arguments

```bash
/readme-generate private MIT github.com/foo/bar
```

All three parameters are optional and order-independent.

### Run Source Analysis Manually

```bash
python3 ~/.claude/skills/readme-generate/scripts/analyze_project.py /path/to/project
```

Outputs JSON containing language, name, version, exported types, functions, and dependencies — useful for debugging or downstream tooling.

## CLI Reference

### Slash Command Parameters

| Parameter | Format | Description |
|-----------|--------|-------------|
| `private` | Keyword (case-insensitive) | Hide badges and star history |
| `LICENSE_TYPE` | License identifier | Generate the corresponding LICENSE file |
| `REPO_PATH` | `github.com/{owner}/{repo}` | Override auto-detected owner and repository |

All three parameters are optional and order-independent.

### Parameter Detection Rules

| Pattern | Detected As |
|---------|-------------|
| `private` (case-insensitive) | `PRIVATE_MODE` flag |
| Contains `github.com/` | `REPO_PATH` |
| Matches a known license alias | `LICENSE_TYPE` |

### Supported License Types

| Type | Aliases (case-insensitive) |
|------|----------------------------|
| MIT | `mit` |
| Apache-2.0 | `apache`, `apache2`, `apache-2.0` |
| GPL-3.0 | `gpl`, `gpl3`, `gpl-3.0` |
| BSD-3-Clause | `bsd`, `bsd3`, `bsd-3-clause` |
| ISC | `isc` |
| Unlicense | `unlicense`, `public-domain` |
| Proprietary | `proprietary` (auto-enables `private` mode) |

### Output Files

| File | Description |
|------|-------------|
| `README.md` | English primary documentation, placed at project root |
| `doc/README.zh.md` | Traditional Chinese version |
| `doc/doc.md` | English detailed technical documentation |
| `doc/doc.zh.md` | Traditional Chinese detailed technical documentation |
| `doc/architecture.md` | English detailed architecture diagrams |
| `doc/architecture.zh.md` | Traditional Chinese detailed architecture diagrams |
| `LICENSE` | Generated by specified type; defaults to MIT when unspecified |

### setup_config.py Subcommands

| Command | Behavior |
|---------|----------|
| `setup_config.py` | Interactive mode; prints existing config or prompts via `input()` |
| `setup_config.py check` | Exit 0 with JSON on stdout if config is complete; exit 1 otherwise |
| `setup_config.py write NAME EMAIL URL OWNER` | Non-interactive write, all four arguments required |

### analyze_project.py Arguments

| Argument | Description |
|----------|-------------|
| `<project_path>` | Absolute path to the project root to analyze |

Full extraction (types, functions, dependencies): Python (AST), Go, JavaScript, TypeScript. File-level detection only: PHP, Swift. The JSON output includes `language`, `name`, `version`, `files`, `types`, `functions`, and `dependencies`.

### Resolution Priority

`{owner}` and `{repo}` are resolved in this order:

1. Command-line `REPO_PATH` (highest)
2. `github_owner` from `~/.skill-readme-generate.json`
3. Local `git remote get-url origin`
4. Current folder name (lowest)
