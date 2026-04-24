# readme-generate - 架構

> 返回 [README](./README.zh.md)

## 概覽

```mermaid
graph TB
    User[使用者] -->|/readme-generate| SKILL[SKILL.md<br/>流程協調]
    SKILL --> Parser[參數解析<br/>private / LICENSE_TYPE / REPO_PATH]
    SKILL --> Config[setup_config.py<br/>作者設定]
    SKILL --> Analyze[analyze_project.py<br/>原始碼分析]
    Config --> JSON[~/.skill-readme-generate.json]
    Analyze --> Data[語言 / 型別 / 函式<br/>相依 / 檔案清單]
    Parser --> Generator[生成器<br/>中文先行 → 英文翻譯]
    Data --> Generator
    JSON --> Generator
    Generator --> Output[README.md<br/>doc/README.zh.md<br/>doc/doc.md / doc.zh.md<br/>doc/architecture.md / architecture.zh.md<br/>LICENSE]
```

## Module: SKILL.md（流程協調）

定義 Claude 執行此 skill 時需嚴格遵守的工作流程、區段順序與驗證清單。本身不含執行碼，僅以提示詞約束 LLM 行為。

```mermaid
graph TB
    subgraph SKILL["SKILL.md"]
        Step0[Step 0<br/>作者設定檢查] --> Step1[Step 1<br/>參數解析]
        Step1 --> Step2[Step 2<br/>專案分析]
        Step2 --> Step3[Step 3<br/>特色提煉<br/>3–5 個]
        Step3 --> Step4[Step 4<br/>ZH 區段生成]
        Step4 --> Step5[Step 5<br/>EN 翻譯]
        Step5 --> Step6[Step 6<br/>LICENSE 生成]
        Step6 --> Step7[Step 7<br/>驗證清單]
    end
    SlashCmd[/readme-generate/] --> SKILL
    SKILL --> Files[六檔輸出 + LICENSE]
```

## Module: setup_config.py（作者設定）

提供三種模式：互動建立、非互動寫入、存在性檢查。設定以 JSON 格式存放於 `~/.skill-readme-generate.json`，四個欄位全部必填。

```mermaid
graph TB
    subgraph Config["setup_config.py"]
        Main[main<br/>子指令分派] --> Check[cmd_check<br/>載入並驗證]
        Main --> Write[cmd_write<br/>四參數寫入]
        Main --> Default[cmd_default<br/>互動或印出]
        Check --> Load[load_config<br/>讀取 + 欄位驗證]
        Default --> Load
        Default --> Prompt[prompt_interactive<br/>TTY input]
        Prompt --> Save[write_config<br/>UTF-8 JSON]
        Write --> Save
    end
    JSON[~/.skill-readme-generate.json] <--> Load
    JSON <--> Save
    SKILL[SKILL.md Step 0] -->|check / write| Main
```

**輸入／輸出**：

| 子指令 | stdin | stdout | exit |
|--------|-------|--------|------|
| `check` | - | JSON 或 MISSING | 0 / 1 |
| `write` | - | JSON | 0 / 2 |
| 預設 | TTY | JSON | 0 / 2 |

## Module: analyze_project.py（原始碼分析）

自動偵測主要語言後，依語言別調用對應 extractor 提取結構資訊。輸出統一的 `ProjectAnalysis` 資料類別序列化為 JSON。

```mermaid
graph TB
    subgraph Analyzer["analyze_project.py"]
        Entry[analyze_project<br/>入口] --> Detect[detect_language<br/>副檔名 + 指標檔]
        Detect -->|go| Go[extract_go_info<br/>解析 go.mod<br/>type / func]
        Detect -->|python| Py[extract_python_info<br/>解析 pyproject.toml<br/>class / def]
        Detect -->|js/ts| JS[extract_js_ts_info<br/>解析 package.json<br/>export function / class]
        Detect -->|其他| Fallback[列出檔案清單]
        Go --> Result[ProjectAnalysis]
        Py --> Result
        JS --> Result
        Fallback --> Result
        Result --> Serialize[asdict + json.dumps]
    end
    SKILL[SKILL.md Step 2] -->|project_path| Entry
    Serialize -->|stdout JSON| SKILL
```

**資料類別**：

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

## 資料流

單次 `/readme-generate` 呼叫的完整流程：

```mermaid
sequenceDiagram
    participant User as 使用者
    participant Claude as Claude Code
    participant Skill as SKILL.md
    participant Config as setup_config.py
    participant Analyze as analyze_project.py
    participant FS as 檔案系統

    User->>Claude: /readme-generate [args]
    Claude->>Skill: 載入 skill 定義
    Skill->>Config: setup_config.py check
    alt 設定缺失
        Config-->>Skill: exit 1
        Skill->>User: AskUserQuestion 四欄位
        User-->>Skill: 作者資訊
        Skill->>Config: setup_config.py write ...
        Config->>FS: 寫入 ~/.skill-readme-generate.json
        Config-->>Skill: JSON
    else 設定完整
        Config-->>Skill: JSON
    end
    Skill->>Skill: 解析 PRIVATE_MODE / LICENSE_TYPE / REPO_PATH
    Skill->>Analyze: analyze_project.py <path>
    Analyze->>FS: 遞迴掃描原始檔
    Analyze-->>Skill: ProjectAnalysis JSON
    Skill->>Skill: 提煉 3–5 特色
    Skill->>FS: 寫入 doc/README.zh.md
    Skill->>FS: 寫入 README.md
    Skill->>FS: 寫入 doc/doc.zh.md / doc.md
    Skill->>FS: 寫入 doc/architecture.zh.md / architecture.md
    alt 無 LICENSE 或指定類型
        Skill->>FS: 寫入 LICENSE
    end
    Skill-->>User: 完成通知
```

## 參數解析狀態機

三個選填參數的偵測與分類：

```mermaid
stateDiagram-v2
    [*] --> Token: 讀取下一個 token
    Token --> PrivateCheck: token 存在
    Token --> Done: 無 token
    PrivateCheck --> SetPrivate: 符合 private（不區分大小寫）
    PrivateCheck --> RepoCheck: 不符
    RepoCheck --> SetRepo: 含 github.com/
    RepoCheck --> LicenseCheck: 不符
    LicenseCheck --> SetLicense: 符合已知授權別名
    LicenseCheck --> Ignore: 全部不符
    SetPrivate --> Token
    SetRepo --> Token
    SetLicense --> Token
    Ignore --> Token
    Done --> Proprietary: LICENSE_TYPE == proprietary
    Done --> Finalize: 其他
    Proprietary --> SetPrivate
    SetPrivate --> Finalize
    Finalize --> [*]
```
