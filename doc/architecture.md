# readme-generate - Architecture

> Back to [README](../README.md)

## Overview

```mermaid
graph TB
    User[User] -->|/readme-generate| SKILL[SKILL.md<br/>Orchestration]
    SKILL --> Parser[Argument Parser<br/>private / LICENSE_TYPE / REPO_PATH]
    SKILL --> Config[setup_config.py<br/>Author Config]
    SKILL --> Analyze[analyze_project.py<br/>Source Analysis]
    Config --> JSON[~/.skill-readme-generate.json]
    Analyze --> Data[Language / Types / Functions<br/>Deps / File List]
    Parser --> Generator[Generator<br/>ZH first → EN translation]
    Data --> Generator
    JSON --> Generator
    Generator --> Output[README.md<br/>doc/README.zh.md<br/>doc/doc.md / doc.zh.md<br/>doc/architecture.md / architecture.zh.md<br/>LICENSE]
```

## Module: SKILL.md (Orchestration)

Defines the workflow, section ordering, and validation checklist that Claude must strictly follow when executing the skill. Contains no executable code — it constrains LLM behavior through prompt instructions.

```mermaid
graph TB
    subgraph SKILL["SKILL.md"]
        Step0[Step 0<br/>Author Config Check] --> Step1[Step 1<br/>Argument Parsing]
        Step1 --> Step2[Step 2<br/>Project Analysis]
        Step2 --> Step3[Step 3<br/>Feature Extraction<br/>3–5 items]
        Step3 --> Step4[Step 4<br/>ZH Section Generation]
        Step4 --> Step5[Step 5<br/>EN Translation]
        Step5 --> Step6[Step 6<br/>LICENSE Generation]
        Step6 --> Step7[Step 7<br/>Validation Checklist]
    end
    SlashCmd[/readme-generate/] --> SKILL
    SKILL --> Files[Six Output Files + LICENSE]
```

## Module: setup_config.py (Author Config)

Provides three modes: interactive creation, non-interactive write, and existence check. The config is stored as JSON at `~/.skill-readme-generate.json` with all four fields required.

```mermaid
graph TB
    subgraph Config["setup_config.py"]
        Main[main<br/>Subcommand Dispatch] --> Check[cmd_check<br/>Load + Validate]
        Main --> Write[cmd_write<br/>Four-Arg Write]
        Main --> Default[cmd_default<br/>Interactive or Print]
        Check --> Load[load_config<br/>Read + Field Validation]
        Default --> Load
        Default --> Prompt[prompt_interactive<br/>TTY input]
        Prompt --> Save[write_config<br/>UTF-8 JSON]
        Write --> Save
    end
    JSON[~/.skill-readme-generate.json] <--> Load
    JSON <--> Save
    SKILL[SKILL.md Step 0] -->|check / write| Main
```

**Inputs / Outputs**:

| Subcommand | stdin | stdout | exit |
|------------|-------|--------|------|
| `check` | - | JSON or MISSING | 0 / 1 |
| `write` | - | JSON | 0 / 2 |
| default | TTY | JSON | 0 / 2 |

## Module: analyze_project.py (Source Analysis)

After auto-detecting the primary language, dispatches to the corresponding extractor to pull structural information. Output is a unified `ProjectAnalysis` dataclass serialized as JSON.

```mermaid
graph TB
    subgraph Analyzer["analyze_project.py"]
        Entry[analyze_project<br/>Entry Point] --> Detect[detect_language<br/>Extensions + Indicator Files]
        Detect -->|go| Go[extract_go_info<br/>Parse go.mod<br/>type / func]
        Detect -->|python| Py[extract_python_info<br/>Parse pyproject.toml<br/>class / def]
        Detect -->|js/ts| JS[extract_js_ts_info<br/>Parse package.json<br/>export function / class]
        Detect -->|other| Fallback[List Files Only]
        Go --> Result[ProjectAnalysis]
        Py --> Result
        JS --> Result
        Fallback --> Result
        Result --> Serialize[asdict + json.dumps]
    end
    SKILL[SKILL.md Step 2] -->|project_path| Entry
    Serialize -->|stdout JSON| SKILL
```

**Dataclasses**:

```mermaid
classDiagram
    class ProjectAnalysis {
        +str language
        +str name
        +str description
        +str version
        +list~TypeInfo~ types
        +list~FunctionInfo~ functions
        +list~str~ files
        +list~str~ dependencies
        +list~str~ entry_points
    }
    class TypeInfo {
        +str name
        +str kind
        +list~dict~ fields
        +str doc
        +str file
    }
    class FunctionInfo {
        +str name
        +str signature
        +str doc
        +bool exported
        +str file
        +int line
    }
    ProjectAnalysis --> TypeInfo
    ProjectAnalysis --> FunctionInfo
```

## Data Flow

Complete flow of a single `/readme-generate` invocation:

```mermaid
sequenceDiagram
    participant User
    participant Claude as Claude Code
    participant Skill as SKILL.md
    participant Config as setup_config.py
    participant Analyze as analyze_project.py
    participant FS as Filesystem

    User->>Claude: /readme-generate [args]
    Claude->>Skill: Load skill definition
    Skill->>Config: setup_config.py check
    alt Config missing
        Config-->>Skill: exit 1
        Skill->>User: AskUserQuestion (4 fields)
        User-->>Skill: Author info
        Skill->>Config: setup_config.py write ...
        Config->>FS: Write ~/.skill-readme-generate.json
        Config-->>Skill: JSON
    else Config complete
        Config-->>Skill: JSON
    end
    Skill->>Skill: Parse PRIVATE_MODE / LICENSE_TYPE / REPO_PATH
    Skill->>Analyze: analyze_project.py <path>
    Analyze->>FS: Recursively scan source files
    Analyze-->>Skill: ProjectAnalysis JSON
    Skill->>Skill: Extract 3–5 features
    Skill->>FS: Write doc/README.zh.md
    Skill->>FS: Write README.md
    Skill->>FS: Write doc/doc.zh.md / doc.md
    Skill->>FS: Write doc/architecture.zh.md / architecture.md
    alt Missing LICENSE or explicit type
        Skill->>FS: Write LICENSE
    end
    Skill-->>User: Completion notice
```

## Argument Parsing State Machine

Detection and classification of the three optional arguments:

```mermaid
stateDiagram-v2
    [*] --> Token: Read next token
    Token --> PrivateCheck: Token present
    Token --> Done: No token
    PrivateCheck --> SetPrivate: Matches private (case-insensitive)
    PrivateCheck --> RepoCheck: No match
    RepoCheck --> SetRepo: Contains github.com/
    RepoCheck --> LicenseCheck: No match
    LicenseCheck --> SetLicense: Matches known license alias
    LicenseCheck --> Ignore: No match
    SetPrivate --> Token
    SetRepo --> Token
    SetLicense --> Token
    Ignore --> Token
    Done --> Proprietary: LICENSE_TYPE == proprietary
    Done --> Finalize: otherwise
    Proprietary --> SetPrivate
    SetPrivate --> Finalize
    Finalize --> [*]
```
