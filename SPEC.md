# `herdr-core-skills` 技術規格書（SPEC）

> **歸屬與獨立聲明**：Herdr（包含 `herdr-core`、`hc` 與 `herdr-app`）為 [herdrdev](https://github.com/herdrdev) 所有。本專案為獨立開源專案，僅基於 `herdr-core`／`herdr-app` 之公開指令與本機已安裝 CLI 行為建立 Agent 技能、規格書與操作規則，非 Herdr 官方專案，亦不代表 `herdrdev` 立場。

本規格書定義 `herdr-core-skills` 的範圍、資料結構、事實優先級、CLI 運作契約、安全不變量、跨平台一致規範與版本漂移驗收標準。

---

## 1. 範圍與非目標（Scope & Non-goals）

### 納入範圍
- **`herdr-core`（官方 CLI 名稱 `hc`）**：持續終端管理、背景程式與慢指令卸載、互動式 coding agent 啟動與協作、訊息傳遞與等待、遠端主機（machines）執行與檔案讀寫。
- **`herdr-app`**：新版 Herdr 桌面圖形介面之 spaces、tabs、panes、overlays、版面與 JavaScript mods 控制。

### 非目標與版本區隔
以下系統與介面**不在本技能範圍內**，不得將其指令或設定套用於新版 `herdr-core`／`herdr-app`：
- **舊版 `herdr` TUI / multiplexer**（如 `herdr workspace`、`herdr pane`）。
- **Sheprd**（如 `sheprd browser` 與 Luau mod API）。
- 不得僅憑安裝資料夾名稱或 `HERDR_ENV=1` 推定為新版介面，必須查驗實際可執行檔與其 `--help`／`--version`。
- 舊版 `herdr` 或 Sheprd 的設定不自動遷移至新版 `herdr-app`。

---

## 2. 技能封裝與漸進式揭露規格（Skill Package & Progressive Disclosure）

```text
herdr-core-skills/
├── README.md                              # 專案說明、安裝方式與歸屬聲明
├── SPEC.md                                # 本規格書
├── AGENTS.md                              # Agent 操作規則與本 repo 維護規範
├── LICENSE                                # MIT 授權
├── .gitignore                             # 忽略暫存與截圖檔
├── rules/
│   └── herdr-core.md                      # 獨立可複製的 Agent Rule（不含 repo 維護規則）
└── herdr-core/                            # Agent Skill 目錄
    ├── SKILL.md                           # 精簡入口（建議 40–50 行，YAML frontmatter + 路由表 + 核心原則）
    └── references/
        ├── core-workflows.md              # hc / herdr-core 詳細操作與邊界條件
        ├── app-workflows.md               # herdr-app do / mod / check / shot 詳細規範
        └── official-sources.md            # 官方來源索引、查核日期與已驗證 binary 版本
```

- **漸進式揭露原則**：`herdr-core/SKILL.md` 僅保留環境確認步驟、任務路由表、不可違反的執行原則與驗證回報要求；完整命令範例、錯誤碼、設定路徑與平台差異下放至 `herdr-core/references/*.md`，由 Agent 依任務按需讀取。
- **語言規範**：說明文字使用繁體中文；所有 CLI 名稱、子命令、旗標、環境變數、設定鍵、JSON 欄位與路徑樣板保留精確英文原文。

---

## 3. 事實優先級（Truth Hierarchy）

當不同來源的說明出現差異時，依以下順序裁決：

1. **已安裝本機 Binary 的即時輸出**：`hc --version`、`hc --help`、`hc --skill`、`hc schema`、`herdr-app --version`、`herdr-app --help`、`herdr-app --skill`、`herdr-app commands`。
2. **本機 App Home 自動生成的 API 文件**：`AGENTS.md`、`herdr.d.ts`、`gpui-kit.d.ts`（位於 `HERDR_APP_HOME`，預設 `~/.config/herdr-app`；唯讀參考，app 啟動時會重寫）。
3. **上游官方文件（`herdrdev` `master` 分支）**：用於補充架構脈絡、安裝說明與追蹤版本差異，不得覆蓋本機已安裝版本的實際參數 schema。

---

## 4. Core 工作流程契約（`hc` / `herdr-core`）

基準驗證版本：`hc 0.1.21`。

### 4.1 探索、定址與輸出協定
- 執行檔解析：優先使用 `hc`；若 PATH 上僅有 `herdr-core`（例如 Windows 常見僅安裝 `herdr-core.exe`），先確認其 `--help` 對應相同 CLI 後再以該執行檔操作，不硬編安裝路徑。
- 位址格式：`t7`、自訂名稱（1–32 字元，僅允許 `a-z 0-9 - _`）或遠端完整位址 `machine/t7`（不可截斷前綴）。
- 目錄語意：`ls --json` 的 `cwd` 為啟動目錄；實際前景工作目錄以 `get --json` 的 `foreground.cwd` 為準。
- 輸出格式：除 `run`、`exec`、`read`、`file read` 外，輸出被管線導向（piped）時預設為 JSON。

### 4.2 啟動、卸載與父子關係
- 背景 worker：`hc run --bg`，不附加 GUI labels（`herdr.space`／`herdr.tab`）。
- 慢指令卸載：會阻塞 shell 工具的長耗時指令使用 `hc run --bg --rm --no-notify`；若需讀取輸出則不得加 `--rm`。
- 父子生命週期：子終端預設隨父程式結束；僅在必要時使用 `--outlive-parent`。必要時以 `--parent <addr>` 指定父端或 `--no-parent` 解除關聯。無 `$HC_TERMINAL_ID` 的外部呼叫方不應依賴 `wait --children`，須追蹤 launch 回傳之明確位址。
- 首個任務與回報：啟動互動式代理時先查 `hc templates`（Codex 須含 `--no-daemon`），將首個任務放在 launch 並檢查 `first_message.sent`。父端收到之子代理訊息以 `<hc from="...">` 標記；長輸出應寫入檔案並僅回報路徑。不允許子代理再建立子代理。

### 4.3 傳訊、等待與讀取
- 傳訊與檔案傳遞：`hc send` 會貼上文字並送出 Enter（非唯讀），支援 `--wait` 與 `--if <idle|working>`；`hc put <TERMINAL> <FILE>` 將本機檔案送到目標終端所在主機的同處並印出落地路徑、`hc keys` 送出按鍵、`hc label` 管理標籤（`key=` 移除）。
- 等待：使用 `hc wait`（預設逾時 180 秒），禁止以 `sleep` 或高頻 `ls`/`read` 輪詢。`timeout`（或前景 `run` 退出碼 124）代表等待逾時而非任務失敗。`wait --output` 僅比對開始等待後的新輸出。
- 狀態與讀取：狀態由 OSC 7501 回報，不可從畫面文字臆測代理狀態。`read --lines 0` 讀取可見畫面；`working`／`blocked`／`error` 時歷史讀取可能標記 `incomplete`。設定 `restore_screen = true` 時，恢復畫面會出現在 `hc read` 輸出。
- 觀測與清理：`hc events` 串流事件；`hc schema` 查詢輸出結構。取得交付物後僅對本次建立的終端執行 `kill` 與 `rm`。

### 4.4 遠端操作與設定
- 遠端命令：`hc machine ls --json`、`hc run --on <machine>`、`hc exec [--on <machine>] [-i]`（直接傳 argv，不經 shell）、`hc file read/edit/write --on <machine>`（`file edit` 須唯一匹配且全有或全無）。
- 設定檔路徑：Unix 為 `~/.config/herdr-core/config.toml`；Windows 為 `%APPDATA%\herdr-core\config.toml`。重要鍵值包含 `draft_wait`（預設 60 秒）與 `restore_screen`。
- 錯誤與退出碼：錯誤結構於 stderr 以 `{error: {kind, message, hint}}` 呈現。常見退出碼：`3` 找不到目標、`7` 需要目標、`9` blocked、`10` status 不符、`11` 已結束。

---

## 5. App 與 Mod 工作流程契約（`herdr-app`）

基準驗證版本：`herdr-app 0.1.16`。

### 5.1 探索與 `do` 呼叫契約
- 查詢命令：`herdr-app ls`、`herdr-app status`（需已開啟視窗）、`herdr-app commands`（不需視窗，為參數 schema 權威來源）。
- 嚴禁以無參數 `herdr-app` 查詢狀態（會啟動或拉起視窗）。
- 命名空間區隔：terminal ID（如 `t7`）與 pane ID 屬不同命名空間，必須透過 `herdr-app ls` 對應，不得混用。
- Agent 呼叫 `herdr-app do` 契約：
  - 絕不開啟缺參數對話框，呼叫方必須傳入所有必要參數。
  - 成功回傳建立的 ID JSON；失敗輸出 `{"error": "..."}` 並以 exit 1 結束。
  - 啟動指令若於 2 秒內非零退出，GUI 會顯示錯誤 toast。
  - 參數支援 `--key value`（陣列／物件傳 JSON）或單一完整 JSON 物件；目標 ID 必須為字串，`null` 省略可能退回目前焦點。

### 5.2 GUI 操作與副作用區分
- 終端相關：`terminal.open`（`--folder`／`--place`）、`terminal.keep`（將背景終端顯示於 GUI，與 `hc keep` 遺失恢復不同）、`terminal.park`（移除 pane 並停用 core keep，程式續跑）、`terminal.pin`、`terminal.peek`／`show`、`terminal.kill`（會連帶結束子代理）、`terminal.select*`／`scroll`／`search`。
- 版面與覆蓋層：`space.new`（遠端指定 `machine` 與遠端 `folder`；明確指定本機傳 `--machine ""`）、`tab.new`／`pane.split`（可帶 `--mod`、`--props`）、`tab.settle`／`unsettle`、`pane.swap`／`zoom`／`focus`／`go`、`overlay.close`。
- 高風險命令：`space.close`、`tab.close`、`pane.close` 會結束所含終端（與 `window.close` 不同）；`app.reset`／`layout.reset` 為破壞性重置。
- 設定與回報：使用者設定位於 app home 的 `settings.json`（如 `terminal_line_height`、`cell_width`、`scroll_speed`、`worktrees.directory`）；`herdr-app feedback <text>` 僅草擬回報，不代表已送出。

### 5.3 Mod 開發與驗證契約
- 結構：`mods/<name>/main.js` 與可選 `mod.json`（需鍵盤焦點時設 `"focusable": true`）。內建 mod／layout 須先 `mod.clone`／`layout.clone`，僅修改個人副本。
- 生命週期與安全：
  - Mods 為具備程式、檔案與網路能力的受信任 JavaScript（非 sandbox），第三方程式須先審查再載入。
  - `herdr-app check` 期間 `exec`、`hc`、`writeTextFile` 仍會執行，`init()` 必須以 `checking()` 守衛副作用。
  - `watchFolder(dir)` 須於適當時機呼叫 `.cancel()`；`View.close(cx)` 於退出時僅有 250ms 預算；不得跨 callback 保存 `cx`；動畫使用 native animation。
- 驗證流程：修改後依序執行 `herdr-app check <mod-path>` → `herdr-app status` → `herdr-app shot [output.png]`（預設輸出 `herdr-shot.png`，擷取最後一個已繪製 frame，即使視窗被遮擋或最小化亦可擷取），並實際檢視截圖。

---

## 6. 安全不變量（Safety Invariants）

所有操作 `herdr-core` 與 `herdr-app` 的 Agent 必須無條件遵守以下不變量：

1. **禁止未授權啟停與變更**：不執行無參數 `herdr-app`；未經使用者要求不停止／重啟使用者的 core 或 app（含 `hc server restart --fresh`、`app.reset`、`layout.reset`）；不因診斷自行安裝、更新或修改 `config.toml`／`settings.json`。
2. **禁止代為批准權限**：遇 `blocked` 或 trust／permission 提示時，不得使用 `hc send --force` 或自動送出按鍵替使用者核准；不得以 `confirmed` 參數假定已獲授權。
3. **禁止跨範圍讀取與清理**：不讀取與任務無關的使用者終端內容；只清理本次任務自行建立的 worker。
4. **禁止真實螢幕輸入自動化**：絕不在使用者的真實顯示器上執行滑鼠／鍵盤自動化腳本。
5. **測試環境隔離**：測試 core 啟停、恢復或破壞性流程時，必須使用獨立 `HC_SESSION` 與隔離設定／資料目錄，不得影響使用者既有 core 或 app 視窗。

---

## 7. 跨平台一致性（Cross-Platform Parity）

| 面向 | Unix / macOS / WSL | Windows PowerShell |
| --- | --- | --- |
| Core CLI 名稱 | 通常為 `hc`（或 `herdr-core`） | 常見僅安裝 `herdr-core.exe`（`hc` 未必在 PATH） |
| 呼叫變數內執行檔 | `"$coreCli" ...` | `& $coreCli ...`，並立即檢查 `$LASTEXITCODE` |
| 環境變數讀取 | `$HC_TERMINAL_ID` | `$env:HC_TERMINAL_ID` |
| `config.toml` 路徑 | `~/.config/herdr-core/config.toml` | `%APPDATA%\herdr-core\config.toml` |
| `hc server status` | 依版本行為檢查狀態 | 絕不會自動啟動 core |
| 遠端 `--via` 切分 | POSIX shell／argv 規則 | 採 `CommandLineToArgvW` 切分（如經 WSL 連線須留意引號） |
| OSC 7501 狀態清除 | 依終端與 shell 整合清除 | 返回 PowerShell 時目前不會自動清除狀態 |
| `herdr-app do` JSON | 單引號包覆 JSON 字串 | 多欄位用 `ConvertTo-Json -Compress`，複雜流程寫 `.ps1` |

---

## 8. 版本漂移與驗收標準（Version Drift & Acceptance Criteria）

當本機或上游 `herdr-core`／`herdr-app` 升級時，更新本專案須通過以下驗收條件：

1. **唯讀盤點**：執行 `<core-cli> --version`、`--help`、`--skill`、`schema` 與 `herdr-app --version`、`--help`、`--skill`、`commands`，比對現有檔案差異。
2. **無臆造原則**：文件內每個指令、旗標、設定鍵與退出碼皆須對應已驗證 binary 輸出或官方文件，不得憑記憶補齊未驗證參數。
3. **精簡度控制**：`herdr-core/SKILL.md` 維持約 40–50 行，詳細規格留在 `references/` 與 `SPEC.md`。
4. **版本戳記更新**：同步更新 `herdr-core/references/official-sources.md` 與 `README.md` 的查核日期與已驗證 binary 版本。
5. **連結與語法檢查**：確認所有相對 Markdown 連結皆指向有效檔案，繁體中文與英文 CLI 原文無混亂或斷裂。
