> [!NOTE]
> 此 README 由 [SKILL](https://github.com/pardnchiu/skill-readme-generate) 生成，英文版請參閱 [這裡](./README.md)。

# readme-generate

[![license](https://img.shields.io/github/license/pardnchiu/skill-readme-generate)](LICENSE)

> 從原始碼分析自動生成雙語 README 的 Claude Code Skill，支援私有模式與多種授權類型<br>
> 此專案主要由 [Claude Code](https://claude.ai/claude-code) 生成，作者僅做部分調整。

## 目錄

- [功能特點](#功能特點)
- [安裝](#安裝)
- [使用方法](#使用方法)
- [命令列參考](#命令列參考)
- [授權](#授權)

## 功能特點

### 原始碼驅動的內容提取

執行內建的 `analyze_project.py` 腳本掃描實際專案檔案（package.json、go.mod、pyproject.toml 等），自動提取套件名稱、版本與相依資訊。生成的 README 內容來自程式碼本身而非人工填寫，確保文件與實際原始碼保持同步。

### 彈性的參數化控制

支援三個選填參數且順序無關：`private` 旗標隱藏公開徽章與星標歷史，適合內部專案；`LICENSE_TYPE` 從七種授權類型（MIT、Apache-2.0、GPL-3.0、BSD-3-Clause、ISC、Unlicense、Proprietary）中生成對應的 LICENSE 檔案；`REPO_PATH` 以 `github.com/owner/repo` 格式覆蓋自動偵測的擁有者與儲存庫名稱。三個參數可自由組合，任意順序出現。

### 四檔案雙語文件策略

每次執行產出四個檔案：精簡的 README.md（英文）與 doc/README.zh.md（繁體中文）以三個核心特色吸引讀者；完整的 doc/doc.md 與 doc/doc.zh.md 則提供安裝、設定、使用方式與 CLI 參考等技術細節。中文優先撰寫以確保術語一致性，再翻譯為英文。

## 安裝

將此技能放置於 Claude Code 的技能目錄：

```bash
~/.claude/skills/readme-generate/
```

目錄結構：

```
readme-generate/
├── scripts/
│   └── analyze_project.py    # 原始碼分析腳本
├── SKILL.md                  # 技能定義檔
├── LICENSE
├── README.md
└── README.zh.md
```

## 使用方法

```bash
/readme-generate [private] [LICENSE_TYPE] [REPO_PATH]
```

### 使用範例

```bash
# 僅生成 README，公開模式
/readme-generate

# README + MIT LICENSE
/readme-generate MIT

# 私有模式（不含徽章與星標歷史）
/readme-generate private

# 私有 README + MIT LICENSE
/readme-generate private MIT

# 使用自訂儲存庫路徑
/readme-generate github.com/foo/bar

# 私有 README + 自訂路徑 + 授權
/readme-generate private MIT github.com/foo/bar
```

## 命令列參考

### 參數

| 參數 | 格式 | 說明 |
|------|------|------|
| `private` | 關鍵字（不區分大小寫） | 隱藏徽章與星標歷史 |
| `LICENSE_TYPE` | 授權識別碼 | 生成對應的 LICENSE 檔案 |
| `REPO_PATH` | `github.com/{owner}/{repo}` | 覆蓋自動偵測的擁有者與儲存庫 |

### 支援的授權類型

| 類型 | 別名（不區分大小寫） |
|------|----------------------|
| MIT | `mit` |
| Apache-2.0 | `apache`、`apache2`、`apache-2.0` |
| GPL-3.0 | `gpl`、`gpl3`、`gpl-3.0` |
| BSD-3-Clause | `bsd`、`bsd3`、`bsd-3-clause` |
| ISC | `isc` |
| Unlicense | `unlicense`、`public-domain` |
| Proprietary | `proprietary`（隱含 `private` 模式） |

### 參數偵測規則

| 模式 | 偵測為 |
|------|--------|
| `private`（不區分大小寫） | `PRIVATE_MODE` 旗標 |
| 包含 `github.com/` | `REPO_PATH` |
| 符合已知授權類型 | `LICENSE_TYPE` |

### 輸出檔案

| 檔案 | 說明 |
|------|------|
| `README.md` | 英文主要文件（精簡，3 特色驅動） |
| `doc/README.zh.md` | 繁體中文版本（精簡，3 特色驅動） |
| `doc/doc.md` | 英文詳細技術文件 |
| `doc/doc.zh.md` | 繁體中文詳細技術文件 |
| `LICENSE` | 依指定類型生成；未指定時預設 MIT |

## 授權

本專案採用 [MIT LICENSE](LICENSE)。
