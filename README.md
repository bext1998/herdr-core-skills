# herdr-core-skills

> **歸屬與獨立聲明**：Herdr（包含 `herdr-core`、`hc` 與 `herdr-app`）為 **[herdrdev](https://github.com/herdrdev)** 所有。本專案為獨立開源的第三方專案，僅基於 `herdr-core`／`herdr-app` 之公開指令與本機已安裝 CLI 行為建立 Agent 技能（Skill）、技術規格書（SPEC）與操作規則（Rules），非 Herdr 官方專案。

一套給 Coding Agents（Antigravity、Claude Code、Codex、OpenCode 等）使用的 **`herdr-core`（`hc`）與 `herdr-app`** 技能與規則集，聚焦安全邊界、漸進式揭露文件、跨平台細節（Windows PowerShell／Unix／WSL）與可驗證的工作流程。

## 已驗證基準版本

- **查核日期**：`2026-11-22`
- **`herdr-core`（`hc`）**：`0.1.21`
- **`herdr-app`**：`0.1.16`

> 實際執行時一律以本機已安裝版本的 `--help`、`--skill`、`hc schema` 與 `herdr-app commands` 為最高依據。

## 儲存庫結構

```text
herdr-core-skills/
├── README.md                              # 專案說明與快速安裝
├── SPEC.md                                # 技術規格書（範圍、契約、安全不變量、跨平台規範）
├── AGENTS.md                              # Agent 執行規則（Part A）與本 repo 維護規範（Part B）
├── LICENSE                                # MIT License
├── .gitignore
├── rules/
│   └── herdr-core.md                      # 獨立可貼入的 Agent Rule（不含 repo 維護規則）
└── herdr-core/                            # Agent Skill 套件目錄
    ├── SKILL.md                           # 精簡入口技能檔
    └── references/
        ├── core-workflows.md              # hc / herdr-core 進階指令、設定、錯誤碼與遠端操作
        ├── app-workflows.md               # herdr-app do 契約、版面控制、mod 開發與 shot 驗證
        └── official-sources.md            # 官方來源索引與版本辨識
```

## 快速安裝與使用

本專案提供兩種使用方式，可單獨使用或搭配啟用：

### 方式一：安裝 Agent Skill（推薦）

將整個 [`herdr-core/`](herdr-core/) 目錄複製或符號連結至對應工具的 skills 目錄：

| 工具 | 全域 Skills 路徑 | 專案層級 Skills 路徑 |
| --- | --- | --- |
| **Antigravity** | `~/.gemini/antigravity/skills/herdr-core` | `<repo>/.agents/skills/herdr-core` |
| **Claude Code** | `~/.claude/skills/herdr-core` | `<repo>/.claude/skills/herdr-core` |
| **Codex** | `~/.codex/skills/herdr-core` | `<repo>/.agents/skills/herdr-core` |
| **OpenCode** | `~/.config/opencode/skills/herdr-core` | `<repo>/.opencode/skills/herdr-core` |

安裝後，當你要求 Agent 操作 `herdr-core`、`hc`、`herdr-app` 或開發 Herdr mods 時，Agent 會自動載入 [`herdr-core/SKILL.md`](herdr-core/SKILL.md)，並依任務按需讀取 `references/` 內的詳細工作流程。

### 方式二：直接引入獨立 Agent Rule（Drop-in Rule）

若只需要輕量化的常駐規則（不含本儲存庫維護規範），可直接將 [`rules/herdr-core.md`](rules/herdr-core.md) 的內容複製到你的全域或專案規則檔中：

- **Antigravity / Gemini**：`~/.gemini/GEMINI.md` 或 `<repo>/GEMINI.md`
- **Claude Code**：`~/.claude/CLAUDE.md` 或 `<repo>/CLAUDE.md`
- **Codex / OpenCode / 通用**：`<repo>/AGENTS.md`
- **Cursor**：`<repo>/.cursor/rules/herdr-core.mdc`（依需求加上 frontmatter）

## 專案文件導覽

- [herdr-core/SKILL.md](herdr-core/SKILL.md)：Agent Skill 精簡入口
- [rules/herdr-core.md](rules/herdr-core.md)：獨立可貼入的 Agent Rule
- [SPEC.md](SPEC.md)：完整技術規格書與版本漂移驗收標準
- [AGENTS.md](AGENTS.md)：操作規則與本儲存庫維護／貢獻指引
- [herdr-core/references/official-sources.md](herdr-core/references/official-sources.md)：官方來源與版本辨識

## 上游官方專案（herdrdev）

- Herdr 官方組織：[https://github.com/herdrdev](https://github.com/herdrdev)
- `herdr-core`：[https://github.com/herdrdev/herdr-core](https://github.com/herdrdev/herdr-core)
- `herdr-app`：[https://github.com/herdrdev/herdr-app](https://github.com/herdrdev/herdr-app)
- Herdr 官網：[https://herdr.dev/](https://herdr.dev/)

## 授權條款

本專案採用 [MIT License](LICENSE) 釋出。Herdr 相關名稱與上游軟體版權歸 [herdrdev](https://github.com/herdrdev) 所有。
