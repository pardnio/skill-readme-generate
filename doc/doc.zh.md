# readme-generate - 技術文件

> 返回 [README](./README.zh.md)

## 前置需求

- Python 3.8 或更高版本（使用 `dict[str, Any]`、`list[...]` 等原生泛型語法）
- [Claude Code](https://claude.ai/claude-code) CLI 已安裝並設定完成
- Git（用於讀取 `git remote`、首次提交年份等資訊）

## 安裝

### 從 GitHub 複製

```bash
git clone https://github.com/pardnchiu/skill-readme-generate.git \
    ~/.claude/skills/readme-generate
```

### 手動安裝

將下列檔案放置於 `~/.claude/skills/readme-generate/`：

```
readme-generate/
├── scripts/
│   ├── analyze_project.py    # 原始碼分析腳本
│   ├── setup_config.py       # 作者設定腳本
│   └── examples/             # README / doc 範本
├── SKILL.md                  # Skill 定義與流程協議
├── LICENSE
├── README.md
└── doc/
    ├── README.zh.md
    ├── doc.md
    ├── doc.zh.md
    ├── architecture.md
    └── architecture.zh.md
```

安裝完成後，於 Claude Code 中以 `/readme-generate` 呼叫即可。

## 設定

### 作者設定檔

首次執行 `/readme-generate` 時會檢查 `~/.skill-readme-generate.json`，若不存在則互動式建立。

| 欄位 | 必填 | 說明 |
|------|------|------|
| `author_name` | 是 | 作者姓名，顯示於 README Author 區段與版權頁尾 |
| `author_email` | 是 | 聯絡 Email |
| `author_url` | 是 | 個人連結（LinkedIn / GitHub / 個人網站） |
| `github_owner` | 是 | GitHub 使用者名稱，用於預設 `{owner}` 與頭像 |

範例 `~/.skill-readme-generate.json`：

```json
{
  "author_name": "邱敬幃 Pardn Chiu",
  "author_email": "hi@pardn.io",
  "author_url": "https://linkedin.com/in/pardnchiu",
  "github_owner": "pardnchiu"
}
```

### 手動初始化

於終端機直接執行腳本可在 Claude Code 外建立或檢視設定：

```bash
python3 ~/.claude/skills/readme-generate/scripts/setup_config.py
```

若檔案已存在，腳本會直接印出現有設定；若不存在則以 `input()` 逐欄詢問。

### 非互動寫入

提供四個參數可於腳本環境直接寫入：

```bash
python3 ~/.claude/skills/readme-generate/scripts/setup_config.py write \
    "邱敬幃 Pardn Chiu" \
    "hi@pardn.io" \
    "https://linkedin.com/in/pardnchiu" \
    "pardnchiu"
```

### 覆蓋機制

指令列傳入 `REPO_PATH`（含 `github.com/{owner}/{repo}`）時，`{owner}` 取自該路徑，其餘作者欄位仍由設定檔提供。亦可直接編輯或刪除 `~/.skill-readme-generate.json` 觸發重新設定。

## 使用方式

### 基本用法

```bash
/readme-generate
```

於當前 Claude Code 工作目錄執行，依序：

1. 載入或建立作者設定
2. 執行 `analyze_project.py` 分析專案
3. 依特色生成六個文件檔案
4. 若無 LICENSE 則預設產生 MIT

### 指定授權類型

```bash
/readme-generate Apache-2.0
```

從 MIT / Apache-2.0 / GPL-3.0 / BSD-3-Clause / ISC / Unlicense / Proprietary 七種之一生成對應 LICENSE。

### 私有模式

```bash
/readme-generate private
```

隱藏徽章與星標歷史，僅保留標語與內容，適合公司內部專案。

### 覆蓋儲存庫路徑

```bash
/readme-generate github.com/foo/bar
```

將 README 內所有 GitHub URL 的 `{owner}/{repo}` 替換為 `foo/bar`。

### 組合使用

```bash
/readme-generate private MIT github.com/foo/bar
```

三個參數順序無關，可自由組合。

### 手動執行原始碼分析

```bash
python3 ~/.claude/skills/readme-generate/scripts/analyze_project.py /path/to/project
```

輸出包含語言、名稱、版本、匯出型別、函式與相依性的 JSON，可用於除錯或整合至其他工具。

## 命令列參考

### Slash Command 參數

| 參數 | 格式 | 說明 |
|------|------|------|
| `private` | 關鍵字（不區分大小寫） | 隱藏徽章與星標歷史 |
| `LICENSE_TYPE` | 授權識別碼 | 生成對應的 LICENSE 檔案 |
| `REPO_PATH` | `github.com/{owner}/{repo}` | 覆蓋自動偵測的擁有者與儲存庫 |

三個參數皆為選填且順序無關。

### 參數偵測規則

| 模式 | 偵測為 |
|------|--------|
| `private`（不區分大小寫） | `PRIVATE_MODE` 旗標 |
| 包含 `github.com/` | `REPO_PATH` |
| 符合已知授權別名 | `LICENSE_TYPE` |

### 支援的授權類型

| 類型 | 別名（不區分大小寫） |
|------|----------------------|
| MIT | `mit` |
| Apache-2.0 | `apache`、`apache2`、`apache-2.0` |
| GPL-3.0 | `gpl`、`gpl3`、`gpl-3.0` |
| BSD-3-Clause | `bsd`、`bsd3`、`bsd-3-clause` |
| ISC | `isc` |
| Unlicense | `unlicense`、`public-domain` |
| Proprietary | `proprietary`（自動啟用 `private` 模式） |

### 輸出檔案

| 檔案 | 說明 |
|------|------|
| `README.md` | 英文主要文件，置於專案根目錄 |
| `doc/README.zh.md` | 繁體中文版本 |
| `doc/doc.md` | 英文詳細技術文件 |
| `doc/doc.zh.md` | 繁體中文詳細技術文件 |
| `doc/architecture.md` | 英文詳細架構圖 |
| `doc/architecture.zh.md` | 繁體中文詳細架構圖 |
| `LICENSE` | 依指定類型生成；未指定時預設 MIT |

### setup_config.py 子指令

| 指令 | 行為 |
|------|------|
| `setup_config.py` | 互動模式；有設定則印出，無則以 `input()` 詢問 |
| `setup_config.py check` | 檢查設定；存在且完整 exit 0 並印出 JSON，否則 exit 1 |
| `setup_config.py write NAME EMAIL URL OWNER` | 非互動寫入，四參數全必填 |

### analyze_project.py 參數

| 參數 | 說明 |
|------|------|
| `<project_path>` | 要分析的專案根目錄絕對路徑 |

完整解析（型別、函式、相依）：Python（AST）、Go、JavaScript、TypeScript。僅檔案層級偵測：PHP、Swift。輸出 JSON 包含 `language`、`name`、`version`、`files`、`types`、`functions`、`dependencies`。

### 參數優先順序

`{owner}` 與 `{repo}` 解析順序：

1. 指令列 `REPO_PATH`（最高優先）
2. `~/.skill-readme-generate.json` 的 `github_owner`
3. 本地 `git remote get-url origin`
4. 當前資料夾名稱（最低優先）
